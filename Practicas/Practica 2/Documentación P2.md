# **PRACTICA 2:**  Clonar la información de un sitio web

---

## Cuestiones a resolver

En esta práctica el objetivo es configurar las máquinas virtuales para trabajar en modo espejo, consiguiendo que una máquina secundaria mantenga siempre actualizada la información que hay en la máquina servidora principal.  
 
Hay que llevar a cabo las siguientes tareas: 

1. Probar el funcionamiento de la copia de archivos por ssh 
2. Clonado de una carpeta entre las dos máquinas 
3. Configuración de ssh para acceder sin que solicite contraseña 
4. Establecer una tarea en cron que se ejecute cada hora para mantener actualizado el contenido del directorio /var/www entre las dos máquinas 
 
Como resultado de la práctica 2 se mostrará al profesor el funcionamiento del proceso automático de clonado de la información. En el documento a entregar se describirá cómo se ha realizado la configuración de ambas máquinas y del software.  


---

## Resolución

---

### Copia Archivos Por SSH

Para probar el funcionamiento de la copia de archivos por ssh hemos utilizado el siguiente comando que nos permite crear un tar.gz de un equipo y dejarlo en otro, si no dispusiésemos de espacio en disco local. Pudiendo usar ssh para crearlo directamente en el equipo destino.  
`tar czf - directorio | ssh equipodestino 'cat > ~/tar.tgz'`  
`tar czf - maq1/ | ssh 102.168.1.101 'cat > ~/tar.tgz'` ---> En mi caso

Se puede ver en la siguiente imagen:

[CAPTURA TAR]

---

### Clonado de Carpetas

Para hacer el clonado de carpetas necesitaremos la herramienta rsync y ssh, para ello primero debemos descargarla (rsync, porque ssh ya la tenemos de la sesión anterior).
`apt-get install rsync` --> root  
`sudo apt-get install rsync`  --> usuario

Una vez instalado ya podemos hacer el clonado de carpetas entre las 2 máquinas con la siguiente instrucción:  
`rsync -avz -e ssh ipmaquina1:/var/www/ /var/www/`  
`rsync -avz -e ssh 192.168.1.100:/var/www/ /home/miguel/`

Se puede ver en la siguiente imagen:

[CAPTURA RSYNC]

**NOTA**: En este caso vamos a trabajar en modo usuario, y como detalle previo, tenemos que hacer que el usuario sea el dueño de la carpeta donde residen los archivos que hay en el espacio web (en ambas máquinas):  
`sudo chown miguel:miguel –R /var/www`

**NOTA**: No se hizo a /var/www/ porque fue una prueba.


---

### Acceso SSH sin Contraseña

Para la configuración de las máquinas para el acceso por ssh sin que solicite la contraseña es necesario, generar un par de claves pública y privada, y esto se hará de la siguiente forma:  
`ssh-keygen -b 4096 -t rsa`  
`chmod 600 ~/.ssh/authorized_keys` (Por defecto esto no es necesario)  
`ssh-copy-id  maquina1`  
`ssh-copy-id  192.168.1.100` --> En mi caso

Para ver que funciona dejo la siguiente captura:

[CAPTURA SSH SIN CONTRASEÑA]
  

---

### Tarea Cron

Para establecer una tarea en cron que se ejecute cada hora para mantener actualizado el contenido del directorio /var/www entre las dos máquinas es necesario irse al fichero /etc/crontab y añadir una línea, de manera que el fichero quede de la siguiente forma.

### Línea añadida al fichero /etc/crontab  

~~~

`0 * * * * miguel rsync -avz -e ssh 192.168.1.100:/var/www/ /home/miguel/`

~~~

[CAPTURA CRONTAB]

---