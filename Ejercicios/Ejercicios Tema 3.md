# Ejercicios Tema 3
---

### T3.1 :Buscar con qué órdenes de terminal o herramientas gráficas podemos configurar bajo Windows y bajo Linux el enrutamiento del tráfico de un servidor para pasar el tráfico desde una subred a otra.

##### WINDOWS

Esta configuración se muestra de forma gráfica:  
**Primer paso:**   
Instalar la tarjeta de red y los protocolos que se van a necesitar (el protocolo TCP/IP).  
**Segundo paso:**  
Configurar el protocolo TCP/IP. Para ello debe abrirse la ventana de propiedades de entorno de red: En Windows 98 Inicio  > Configuración  > Entorno de red  
**Tercer paso:**  
Seleccionar TCP/IP y abrir sus propiedades:  

![imagen](https://github.com/iMiguel10/SWAP/blob/master/Ejercicios/tcpip1.jpg)
![imagen](https://github.com/iMiguel10/SWAP/blob/master/Ejercicios/tcpip2.jpg)


Se marca Especificar una dirección IP y se van poniendo las direcciones incrementándose de una unidad en una unidad el último octeto del rango en cada equipo perteneciente a esa subred.  

**Cuarto paso:**  
Poner nombre al grupo de trabajo bajo el cual va a identificarse esta subred

![imagen](https://github.com/iMiguel10/SWAP/blob/master/Ejercicios/tcpip4.jpg)


Para configurar la segunda subred, utilizaremos las direcciones IP del rango siguiente:
10.0.1.x y como máscara de subred 255.255.255.0.  

**Configuración del acceso al servidor de datos.**  
Pretendemos que todos los ordenadores de la Intranet puedan acceder a un ordenador servidor de datos y simultáneamente, la red de gestión del centro quede aislada del resto de las subredes.  
Para ello, la mejor solución es hacer que ese ordenador servidor tenga dos tarjetas de red, una con un rango IP del orden 10.0.0.x (con lo que a través de ese interfaz puede verse con la red de gestión del centro) y la otra tarjeta con la dirección IP 10.0.1.2, que permite que se pueda acceder desde los otros segmentos de la subred, pero quedando aislada de ella el resto de las subredes como se pretende.  

[Información Windows](http://recursostic.educacion.es/observatorio/web/ca/equipamiento-tecnologico/redes/74-enrutamiento-del-trafico-entre-subredes)


##### LINUX

**// Activar el enrutamiento en un sistema Linux**  
`sudo echo "1" > /proc/sys/net/ipv4/ip_forward`

**// Haciendo NAT en el servidor**  
`sudo iptables -A FORWARD -j ACCEPT`  
`sudo iptables -t nat -A POSTROUTING -s 10.0.0.0/8 -o eth0 -j MASQUERADE`  

**//Crear una ruta para una IP concreta**  
`sudo route add 80.58.12.27 eth1`  

**//Crear una ruta para una red concreta**  
`sudo route add -net 193.144.238.0/24 eth1`  

**//Eliminar una ruta**  
`sudo route del -net 193.144.238.0/24 `

**//Ver rutas**  
`sudo route`


[Información Linux](http://www.ite.educacion.es/formacion/materiales/85/cd/linux/m6/enrutamiento_en_linux.html)

---


### T3.2 : Buscar con qué órdenes de terminal o herramientas gráficas podemos configurar bajo Windows y bajo Linux el filtrado y bloqueo de paquetes. 

---

