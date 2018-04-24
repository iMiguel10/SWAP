# **PRACTICA 4:**  Asegurar la Granja Web 

---

## Cuestiones a resolver

El objetivo de esta práctica es configurar todos los aspectos relativos a la seguridad de la granja web ya creada.  
 
Hay que llevar a cabo las siguientes tareas obligatorias: 
 
1. Crear e instalar en la máquina 1 un certificado SSL autofirmado para configurar el acceso HTTPS a los servidores. Una vez configurada la máquina 1, se debe copiar al resto de máquinas servidoras y al balanceador de carga. Se debe configurar nginx adecuadamente para aceptar y balancear correctamente tanto el tráfico HTTP como el HTTPS.  
 
2. Configurar las reglas del cortafuegos con IPTABLES para asegurar el acceso a uno de los servidores web, permitiendo el acceso por los puertos de HTTP y HTTPS a dicho servidor. Esta configuración se hará en una de las máquinas servidoras finales (p.ej. en la máquina 1), y se debe poner en un script con las reglas del cortafuegos que se ejecute en el arranque del sistema (según la versión de Linux, se llevará a cabo de una forma u otra). 
 
Adicionalmente, y como primera tarea opcional para conseguir una mayor nota en esta práctica, se propone realizar la instalación de un certificado del proyecto Certbot en lugar de uno autofirmado. Es importante tener en cuenta que para obtener este tipo de certificado, es necesario disponer de un dominio real con IP pública (no se puede hacer en máquinas virtuales). 
 
Como segunda tarea opcional para conseguir una mayor nota en esta práctica, se propone realizar la configuración del cortafuegos en una cuarta máquina (M4) que se situará delante del balanceador. Esa M4 sólo tendrá configuradas las iptables, para hacer el filtrado y posterior reencaminamiento del tráfico hacia el balanceador. En esta configuración más compleja sólo a esa M4-cortafuegos se le hará la configuración de iptables (el resto de máquinas de la granja tendrá la configuración por defecto, aceptando todo el tráfico como política por defecto). 
 
Como resultado de la práctica 4 se mostrará al profesor el funcionamiento del acceso por HTTPS a las páginas web almacenadas en los servidores finales, haciendo peticiones con la herramienta curl por HTTP/HTTPS tanto a la máquina 1 como al balanceador de carga. También se mostrará el funcionamiento del filtrado del tráfico HTTP/HTTPS con las reglas de iptables configuradas. En el documento de texto a entregar se describirá cómo se han realizado las diferentes configuraciones (tanto configuraciones y comandos de terminal a ejecutar en cada momento). 

---

## Resolución

---

### Certificado SSL Autofirmado ( Falta pasar los archivos a las otras maquinas y al balanceador al igual que configurarlo )

Para generar un certificado SSL autofirmado en Ubuntu Server solo debemos activar el módulo SSL de Apache, generar los certificados y especificarle la ruta a los certificados en la configuración. Así pues, como root ejecutaremos:  
 
`a2enmod ssl `  
`systemctl restart apache2`    
`mkdir /etc/apache2/ssl `  
`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout     /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt` 

Nos pedirá una serie de datos para configurar el dominio.  Ponemos lo siguiente en orden ( por ejemplo ):  

*  ES
*  Granada  
*  Granada  
*  SWAP  
*  SWAP  
*  SWAP  
*  swap@info.es


Editamos el archivo de configuración del sitio *default-ssl*:   
`vi /etc/apache2/sites-available/default-ssl.conf`  

 Y agregamos estas lineas debajo de donde pone *SSLEngine on*:  

~~~

SSLCertificateFile /etc/apache2/ssl/apache.crt  
SSLCertificateKeyFile /etc/apache2/ssl/apache.key 

~~~

![img]()  
[CAPTURA FICHERO DEFAULT SSL]

Activamos el sitio default-ssl y reiniciamos apache:  
`a2ensite default-ssl`  
`systemctl restart apache2`    


Una vez reiniciado Apache, accedemos al servidor web mediante el protocolo HTTPS utilizando la herramienta curl, ejecutamos lo siguiente:  
`curl –k https://ipmaquina1/index.html`  
`curl –k https://192.168.1.100/index.html` --> En mi caso  
 
Por otra parte para copiar los archivos .crt y .key es necesario usar scp de la siguiente manera:  
`scp usuario@ip-maquina :"ruta remota" "ruta del pc local"`  
`scp miguel@192.168.1.100 :/etc/apache2/ssl/apache.crt "ruta del pc local"`  
`scp miguel@192.168.1.100 :/etc/apache2/ssl/apache.key "ruta del pc local"`  

Además para NginX es necesario cambiar el archivo de configuración y añadir lo siguiente:  
~~~

listen 443;

ssl on;
ssl_certificate /etc/nginx/ssl/nginx.crt;
ssl_certificate_key /etc/nginx/ssl/nginx.key;

~~~

![img]()  
[CAPTURA FICHERO NGINX CONFIGURACION]

**NOTA** : Hay que tener en cuenta que el ssh debe estar corriendo, se puede ver y activar con lo siguiente:  
`systemctl status sshd`    
`systemctl restart sshd`    


---------------------------------------------------

### Configuración del Cortafuegos

Para abrir los puertos HTTP/HTTPS (80 y 443) en el servidor web hacemos lo siguiente:  
`iptables -A INPUT -m state --state NEW -p tcp --dport 80 -j ACCEPT`  
`iptables -A INPUT -m state --state NEW -p tcp --dport 443 -j ACCEPT` 

Por último, conviene comprobar el funcionamiento del cortafuegos recién configurado. Para ello, pediremos al sistema que nos muestre qué puertos hay abiertos y qué demonios o aplicaciones los tienen en uso. Para ello, utilizaremos la orden netstat. Por ejemplo, para asegurarnos del estado (abierto/cerrado) de los puertos 80 y 443 (HTTP Y HTTPS), ejecutamos:  
`netstat -tulpn | grep :80`  
`netstat -tulpn | grep :443`  

![img]()  
[CAPTURA NETSTAT]

Lo habitual es crear un script que se ejecute en el arranque del sistema. Por ello crearemos el siguiente script que nos permitirá una configuración básica de un servidor web ( iptables.sh ).

~~~

 # (1) Eliminar todas las reglas (configuración limpia) 
    iptables -F 
    iptables -X 
    iptables -Z
    iptables -t nat -F 

 # (2) Política por defecto: denegar todo el tráfico 
    iptables -P INPUT DROP 
    iptables -P OUTPUT DROP
    iptables -P FORWARD DROP 

 # (3) Permitir cualquier acceso desde localhost (interface lo) 
    iptables -A INPUT  -i lo -j ACCEPT 
    iptables -A OUTPUT -o lo -j ACCEPT 

 # (4) Abrir el puerto 22 para permitir el acceso por SSH 
    iptables -A INPUT  -p tcp --dport 22 -j ACCEPT 
    iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT 

 # (5) Abrir los puertos HTTP (80) de servidor web 
    iptables -A INPUT  -p tcp --dport 80 -j ACCEPT 
    iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT 

 # (6) Abrir los puertos HTTPS (443) de servidor web 
    iptables -A INPUT  -p tcp --dport 443 -j ACCEPT 
    iptables -A OUTPUT -p tcp --sport 443 -j ACCEPT 

 # (7) Permitir la salida del equipo (output) con conexiones nuevas que   
 # solicitemos, conexiones establecidas y relacionadas. Permitir la   
 # entrada (input) solo de conexiones establecidas y relacionadas:    
    iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT   
    iptables -A OUTPUT -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT 

~~~

![img]()  
[CAPTURA IPTABLES.SH]

En cualquier momento, si hubiéramos cometido algún error, podemos poner la configuración que tenía la máquina inicialmente (permitir todo el tráfico) con el siguiente script ( limpiaiptables.sh ): 

~~~

 # (1) Eliminar todas las reglas (configuración limpia) 
    iptables -F 
    iptables -X 
    iptables -Z
    iptables -t nat -F 

 # (2) Política por defecto: aceptar todo 
    iptables −P INPUT ACCEPT 
    iptables −P OUTPUT ACCEPT 
    iptables −P FORWARD ACCEPT 

# (3) Mostrar información
    iptables -L -n -v

~~~


![img]()  
[CAPTURA LIMPIAIPTABLES.SH]

**NOTA** : Acordaos de darle permisos a los .sh con *chmod +x nombre_sh.sh*  
**NOTA** : Para que se ejecute el script cuando se inicie la máquina hay que:  

* Añadir al script lo siguiente:  
~~~
#!/bin/bash  
### BEGIN INIT INFO  
# Provides:    blablá  
# Required-Start:    $syslog  
# Required-Stop:    $syslog  
# Default-Start:    2 3 4 5  
# Default-Stop:    0 1 6  
# Short-Description:    blablá  
# Description:  
#    
### END INIT INFO  
~~~
* Moverlo a la carpeta */etc/init.d*  
* Ejecutar el siguiente comando: `update-rc.d script defaults`  

**Recordatorio:** Las IP de cada una de las máquinas son:  

*  Maquina 1: 192.168.1.100
*  Maquina 2: 192.168.1.101
*  Balanceador Nginx: 192.168.1.200
*  Balanceador HAProxy: 192.168.1.200
*  Cliente: 192.168.1.201

---


