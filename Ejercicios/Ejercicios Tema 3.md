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

##### WINDOWS

**Para configurar la seguridad TCP/IP:**  
<ul>
    <li>Haga clic en Inicio, seleccione Panel de control, Conexiones de red y haga clic en la conexión de área local que desee configurar.</li>
    <li>En el cuadro de diálogo Estado de conexión, haga clic en Propiedades.</li>
    <li>Haga clic en Protocolo Internet (TCP/IP)y, después, en Propiedades.</li>
    <li>En el cuadro de diálogo Propiedades de Protocolo Internet (TCP/IP), haga clic en Opciones avanzadas</li>
    <li>y, a continuación, en Opciones.</li>
    <li>En Configuración opcional, haga clic en Filtrado TCP/IP y en Propiedades.</li>
    <li>Active la casilla de verificación Habilitar filtrado TCP/IP (todos los adaptadores).</li>


*NOTA*  
Al activar esta casilla de verificación, se habilita el filtrado para todos los adaptadores, pero los filtros se configuran de forma individual para cada adaptador. Los mismos filtros no sirven para todos los adaptadores.
</ul>

<ul>
    <li>En el cuadro de diálogo Filtrado TCP/IPhay tres secciones en las que puede configurar el filtrado para puertos TCP, puertos UDP (protocolo de datagramas de usuarios) y protocolos Internet. Para cada sección, establezca la configuración de seguridad adecuada para el equipo que utiliza.</li>
</ul>

*NOTA*  
Cuando la opción Permitir todos está activada, se permiten todos los paquetes para el tráfico TCP o UDP. Permitir sólo permite únicamente el tráfico TCP o UDP seleccionado al agregar los puertos permitidos. Para especificar los puertos, se utiliza el botón Agregar. Para bloquear todo el tráfico UDP o TCP, haga clic en Permitir sólo y no agregue ningún número de puerto en las columnas Puertos UDP y Puertos TCP. No se puede bloquear el tráfico UDP o TCP seleccionando Permitir sólo para Protocolos IP y excluyendo los protocolos IP 6 y 17. 

[Información Windows](https://support.microsoft.com/es-es/help/816792/how-to-configure-tcp-ip-filtering-in-windows-server-2003)


##### LINUX

**Los principales comandos de IPtables son los siguientes (argumentos de una orden):**  
<ul>
    <li>-A –append → agrega una regla a una cadena.</li>
    <li>-D –delete → borra una regla de una cadena especificada.</li>
    <li>-R –replace → reemplaza una regla.</li>
    <li>-I –insert → inserta una regla en lugar de una cadena.</li>
    <li>-L –list → muestra las reglas que le pasamos como argumento.</li>
    <li>-F –flush → borra todas las reglas de una cadena.</li>
    <li>-Z –zero → pone a cero todos los contadores de una cadena.</li>
    <li>-N –new-chain → permite al usuario crear su propia cadena.</li>
    <li>-X –delete-chain → borra la cadena especificada.</li>
    <li>-P –policy → explica al kernel qué hacer con los paquetes que no coincidan con ninguna regla.</li>
    <li>-E –rename-chain → cambia el orden de una cadena.</li>
</ul>

**Condiciones principales para Iptables:**
<ul>
    <li>-p –protocol → la regla se aplica a un protocolo.</li>
    <li>-s –src –source → la regla se aplica a una IP de origen.</li>
    <li>-d –dst –destination → la regla se aplica a una Ip de destino.</li>
    <li>-i –in-interface → la regla de aplica a una interfaz de origen, como eth0.</li>
    <li>-o –out-interface → la regla se aplica a una interfaz de destino.</li>
</ul>

**Condiciones TCP/UDP**
<ul>
    <li>-sport –source-port → selecciona o excluye puertos de un determinado puerto de origen.</li>
    <li>-dport –destination-port → selecciona o excluye puertos de un determinado puerto de destino.</li>
</ul>  
Existen muchas mas condiciones para una configuración avanzada del firewall, pero las elementales ya las tenemos listadas.

**Configurar reglas por defecto**  
La configuración por defecto de un firewall debería ser, traducido al español, “bloquear todo excepto [reglas]”. Para configurar el firewall para que bloquee todas las conexiones debemos teclear:
<ul>
    <li>iptables -P INPUT DROP</li>
    <li>iptables -P FORWARD DROP</li>
    <li>iptables -P OUTPUT DROP</li>
</ul>
Con esto nos quedaremos sin internet, por lo que a continuación debemos empezar a crear reglas permisivas.


[Información Linux](https://www.redeszone.net/gnu-linux/iptables-configuracion-del-firewall-en-linux-con-iptables/)

---

