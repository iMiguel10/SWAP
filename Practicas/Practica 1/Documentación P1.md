# **PRACTICA 1:** Presentación de las prácticas y preparación de las herramientas

---

## Cuestiones a resolver

En esta práctica el objetivo es configurar las máquinas virtuales para trabajar en prácticas posteriores, asegurando la conectividad entre dichas máquinas.  
 
Como resultado de la práctica 1 se mostrarán dos máquinas funcionando al profesor en clase (accesos con curl para solicitar páginas web sencillas, así como el acceso por SSH entre ambas máquinas).  
 
Específicamente, hay que llevar a cabo las siguientes tareas: 

1.    Acceder por ssh de una máquina a otra.
2.    Acceder mediante la herramienta curl desde una máquina a la otra. 
 
El resultado de ejecutar estas tareas se debe documentar usando un archivo de texto y/o capturas de pantalla que se subirán a la cuenta de GitHub.

---

## Resolución

---

Para hacer que las máquinas estén en la misma red y tengan conectividad es necesario añadirles un nuevo adaptador de red, en este caso uno de red interna, e ir al archivo de configuración de las interfaces de red del servidor y añadir la nueva interfaz.  

`vi /etc/network/interfaces`  

### Máquina 1  
~~~
auto enp0s8  
iface enp0s8 inet static  
address 192.168.1.100  
gateway 192.168.1.1  
netmask 255.255.255.0  
network 192.168.1.0  
broadcast 192.168.1.255
~~~

### Máquina 2
~~~
auto enp0s8  
iface enp0s8 inet static  
address 192.168.1.101  
gateway 192.168.1.1  
netmask 255.255.255.0  
network 192.168.1.0  
broadcast 192.168.1.255
~~~

![imagen](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%201/Captura%20Demo%202%20maquinas.JPG)


**NOTA** : El sistema operativo de las máquinas es Ubuntu 16.04.3 Server AMD64.

---

### SSH

Para conectar las máquinas a través de ssh es necesario tener ssh instalado y corriendo en las 2 máquinas, ya que la que va a conectar necesita el cliente y a la que se va a conectar necesita el servidor de ssh.  
Una vez comprobado que todo funciona procedemos a conectarnos con la siguiente orden:

`shh usuario@ip_destino`

En mi caso voy a conectar la máquina 1 a la máquina 2: `ssh miguel@192.168.1.101`  
Una vez conectadas las 2 maquinas creo una carpeta y un archivo con la máquina 1 remotamente, y con la máquina 2 lo visualizo, como se puede ver en la siguiente imagen.

`mkdir nombreCarpeta` : *Crear la carpeta*

`touch archivo.extension`: *Crear archivo*

`ls -l  directorioCreacion`: *Visualizar los cambios realizados remotamente*

![imagen](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%201/Captura%20SSH.JPG)

---

### CURL

Para utilizar curl es necesario descargarlo con antelación `apt-get install curl`.  
Una vez descargado vamos a crear en las dos máquinas un archivo HTML para poder descargarlo con curl desde la otra máquina, el archivo se creará en /var/www/html y se llamará hola.html y contendrá lo siguiente:  

~~~
<HTML>  
  <BODY>  
    Esto funciona en la máquina ?  
  </BODY>  
</HTML>  
~~~

![imagen](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%201/Captura%20CURL%20completa.JPG)

**IMPORTANTE** : El servicio de apache debe estar en funcionamiento.  
Para comprobar si está funcionando `systemctl status apache2`.  
Para iniciarlo ( en caso de que no lo esté ) `systemctl start apache2`.   
Para comprobar si está instalado `apache2 -v`.  
Para instalar ( en caso de no tenerlo ) `apt-get install apache2`.  



