# **PRACTICA 5:**  Replicación de bases de datos MySQL 
---

## Cuestiones a resolver

En esta práctica el objetivo es configurar las máquinas virtuales para trabajar de forma que se mantenga actualizada la información en una BD entre dos servidores (la máquina secundaria mantendrá siempre actualizada la información que hay en la máquina servidora principal).  
 
Hay que llevar a cabo las siguientes tareas obligatorias:  

1. Crear una BD con al menos una tabla y algunos datos. 
2. Realizar la copia de seguridad de la BD completa usando mysqldump en la máquina principal y copiar el archivo de copia de seguridad a la máquina secundaria. 
3. Restaurar dicha copia de seguridad en la segunda máquina (clonado manual de la BD), de forma que en ambas máquinas esté esa BD de forma idéntica. 
4. Realizar la configuración maestro-esclavo de los servidores MySQL para que la replicación de datos se realice automáticamente. 
 
Adicionalmente, y como tarea opcional para conseguir una mayor nota en esta práctica, se propone realizar la configuración maestro-maestro entre las dos máquinas de bases de datos. 
 
Como resultado de la práctica 5 se mostrará al profesor el funcionamiento del proceso de clonado automático de la información entre bases de datos MySQL en las máquinas principal y secundaria (configuración maestro-esclavo y/o maestro-maestro, en su caso). En el documento de texto a entregar se describirá en detalle cómo se ha realizado la configuración de ambos servidores (configuraciones y comandos de terminal ejecutados en cada momento).

---

## Resolución

---

### Crear BD

Para el resto de la práctica debemos crearnos una BD en MySQL e insertar algunos datos. Así tendremos datos con los cuales hacer las copias de seguridad. En todo momento usaremos la interfaz de línea de comandos del MySQL:   
~~~
mysql -uroot -p
mysql> create database contactos; 
mysql> use contactos;
mysql> show tables;
mysql> create table datos(nombre varchar(100),tlf int);
mysql> show tables; 
mysql> insert into datos(nombre,tlf) values ("pepe",95834987);
mysql> insert into datos(nombre,tlf) values ("juan",95834111);
mysql> insert into datos(nombre,tlf) values ("miguel",95834521);
mysql> select * from datos; `  
~~~

![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%205/Captura%20crear%20BD.PNG)  

---

###  Replicar una BD MySQL con mysqldump 

MySQL ofrece la una herramienta para clonar las BD que tenemos en nuestra maquina. Esta herramienta es mysqldump. EL volcado contiene comandos SQL para crear la BD, las tablas y rellenarlas.  
Esta herramienta soporta una cantidad considerable de opciones. Ejecuta como root el siguiente comando:  `mysqldump --help` para obtener la lista completa. En la siguiente URL se explican con detalle todas las opciones posibles: http://dev.mysql.com/doc/refman/5.0/es/mysqldump.html  

Así, en el servidor de BD principal (maquina1) hacemos:  
~~~
mysql -u root –p  
mysql> FLUSH TABLES WITH READ LOCK;  
mysql> quit 
~~~

Ahora ya sí podemos hacer el mysqldump para guardar los datos. En el servidor principal (maquina1) hacemos: 
`mysqldump ejemplodb -u root -p > /tmp/ejemplodb.sql`   
`mysqldump contactos -u root -p > /tmp/contactos.sql` --> En mi caso  
Como habíamos bloqueado las tablas, debemos desbloquearlas (quitar el “LOCK”):  
~~~
mysql -u root –p   
mysql> UNLOCK TABLES;   
mysql> quit   
~~~

![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%205/Captura%20clonar%20BD.PNG)  

Ya podemos ir a la máquina esclavo (maquina2, secundaria) para copiar el archivo .SQL con todos los datos salvados desde la máquina principal (maquina1):  
`scp maquina1:/tmp/ejemplodb.sql /tmp/`   
`scp miguel@192.168.1.100:/tmp/contactos.sql /tmp/`  --> En mi caso  
y habremos copiado desde la máquina principal (1) a la máquina secundaria (2) los datos que hay almacenados en la BD.


![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%205/Captura%20paso%20de%20BD.PNG)  

---

###  Restaurar la BD en la máquina secundaria  
Una vez que ya tenemos el archivo .sql en la segunda máquina pasamos a restaurarla, pero la orden mysqldump no incluye en ese archivo la sentencia para crear la BD (es necesario que nosotros la creemos en la máquina secundaria en un primer paso, antes de restaurar las tablas de esa BD y los datos contenidos en éstas). 
 
Para importar la BD completa en el MySQL, debemos en un primer paso crear la BD:  
~~~
mysql -u root –p   
mysql> CREATE DATABASE ‘ejemplodb’;  
mysql> CREATE DATABASE contactos;  
mysql> quit   
~~~
Y en un segundo paso restauramos los datos contenidos en la BD (se crearán las tablas en el proceso):  
`mysql -u root -p ejemplodb  < /tmp/ejemplodb.sql`  
`mysql -u root -p contactos  < /tmp/contactos.sql`  

Para comprobar su correcto funcionamiento hacemos lo siguiente:  
~~~
mysql -uroot -p  
mysql> use contactos;  
mysql> select * from datos;  
~~~

![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%205/Captura%20importacion%20de%20BD.PNG)  

---

###  Replicación de BD mediante una configuración maestro-esclavo 

La opción anterior funciona perfectamente, pero es algo que realiza un operador a mano. Sin embargo, MySQL tiene la opción de configurar el demonio para hacer replicación de las BD sobre un esclavo a partir de los datos que almacena el maestro.  
 
Se trata de un proceso automático que resulta muy adecuado en un entorno de producción real. Implica realizar algunas configuraciones, tanto en el servidor principal como en el secundario.  
 
A continuación se detalla el proceso a realizar en ambas máquinas, para lo cual, supondremos que partimos teniendo clonadas las base de datos en ambas máquinas:  
Lo primero que debemos hacer es la configuración de mysql del maestro. Para ello editamos, como root, el /etc/mysql/mysql.conf.d/mysqld.cnf y realizamos las siguientes modificaciones:

* Comentamos el parámetro bind-address que sirve para que escuche a un servidor:  
~~~ 
# bind-address 127.0.0.1   
~~~
* Le indicamos el archivo donde almacenar el log de errores. De esta forma, si por ejemplo al reiniciar el servicio cometemos algún error en el archivo de configuración, en el archivo de log nos mostrará con detalle lo sucedido:   
~~~
log_error = /var/log/mysql/error.log 
~~~
* Establecemos el identificador del servidor. 
~~~
server-id = 1 --> Para M1 (Maestro)
server-id = 2 --> Para M2 (Esclavo)
~~~
* El registro binario contiene toda la información que está disponible en el registro de actualizaciones, en un formato más eficiente y de una manera que es segura para las transacciones: 
~~~
log_bin = /var/log/mysql/bin.log 
~~~

![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%205/Captura%20conf%20mysql.PNG)   

Guardamos el documento y reiniciamos el servicio:  
`systemctl restart mysql` 

A continuación volvemos al maestro para crear un usuario y darle permisos de acceso para la replicación.  
Entramos en mysql y ejecutamos las siguientes sentencias: 
~~~
mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo'; 
mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo'; 
mysql> FLUSH PRIVILEGES; 
mysql> FLUSH TABLES; 
mysql> FLUSH TABLES WITH READ LOCK; 
~~~

Para finalizar con la configuración en el maestro, obtenemos los datos de la BD que vamos a replicar para posteriormente usarlos en la configuración del esclavo:
~~~
mysql> SHOW MASTER STATUS;
~~~

![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%205/Captura%20conf%20mysql%202.PNG)  

Volvemos a la máquina esclava, entramos en mysql y le damos los datos del maestro. 
(ojo con la IP, con el valor de "master_log_file" y del "master_log_pos" del maestro): 
~~~
mysql> CHANGE MASTER TO MASTER_HOST='192.168.31.100', ( IP de la M1 )
MASTER_USER='esclavo',
MASTER_PASSWORD='esclavo', 
MASTER_LOG_FILE='mysql-bin.000001', (Podria cambiar)
MASTER_LOG_POS=980, (Cambia)
MASTER_PORT=3306; 
~~~

Por último, arrancamos el esclavo y ya está todo listo para que los demonios de MySQL de las dos máquinas repliquen automáticamente los datos que se introduzcan/modifiquen/borren en el servidor maestro: 
~~~
mysql> START SLAVE;
~~~

Por último, volvemos al maestro y volvemos a activar las tablas para que puedan meterse nuevos datos en el maestro: 
~~~
mysql> UNLOCK TABLES; 
~~~
Ahora, si queremos asegurarnos de que todo funciona perfectamente y que el esclavo no tiene ningún problema para replicar la información, nos vamos al esclavo y con la siguiente orden: 
~~~
mysql> SHOW SLAVE STATUS\G 
~~~
![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%205/Captura%20behind%20master%200.PNG)  

Ahora, podemos hacer pruebas en el maestro y deberían replicarse en el esclavo automáticamente. 
 
![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%205/Captura%20funcionamiento%20maestro%20esclavo.PNG)  

**NOTA**: En caso de que se produzca el error: *Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs;* es necesario realizar lo siguiente:

* Parar MySQL --> `systemctl stop mysql`
* Eliminar el archivo *auto.cnf* --> `rm /var/lib/mysql/auto.cnf`
* Volvemos a iniciar MySQL --> `systemctl start mysql`


---

**Recordatorio:** Las IP de cada una de las máquinas son:  

*  Maquina 1: 192.168.1.100
*  Maquina 2: 192.168.1.101
*  Balanceador Nginx: 192.168.1.200
*  Balanceador HAProxy: 192.168.1.200
*  Cliente: 192.168.1.201

---


