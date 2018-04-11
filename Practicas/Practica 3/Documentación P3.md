# **PRACTICA 3:**  Balanceo de carga

---

## Cuestiones a resolver

En esta práctica el objetivo es configurar las máquinas virtuales de forma que dos hagan de servidores web finales mientras que la tercera haga de balanceador de carga por software.  
 
En esta práctica se llevarán a cabo, como mínimo, las siguientes tareas: 

1. Configurar una máquina e instalar el NginX como balanceador de carga.
2. Configurar una máquina e instalar el HAProxy como balanceador de carga.
3. Someter a la granja web a una alta carga, generada con la herramienta Apache Benchmark, teniendo primero nginx y después haproxy.  
 
En las tareas 1 y 2 debemos hacer peticiones a la dirección IP del balanceador y comprobar que realmente se reparte la carga. Para ello, el index.html en las máquinas finales deben ser diferentes para ver cómo las respuestas que recibimos al hacer varias peticiones son diferentes (eso indicará que el balanceador deriva tráfico a las máquinas servidoras finales). 
 
Además, se comprobará el funcionamiento de los algoritmos de balanceo round-robin y con ponderación (en este caso supondremos que la máquina 1 tiene el doble de capacidad que la máquina 2).   
En la tarea 3 debemos usar la herramienta ab para someter a una carga muy alta (gran número de peticiones y con alta concurrencia) a la granja web, primero estando NginX como balanceador, y a continuación estando HAProxy como balanceador. Como resultado, se debe realizar una comparación de los tiempos medios de servicio entre ambos balanceadores, para poder determinar cuál funciona mejor. 
 
Adicionalmente, y como tarea opcional para conseguir una mayor nota en esta práctica, se propone el uso de algún otro software de balanceo diferente a los dos explicados en este guión (por ejemplo Pound). 
 
Como resultado de la práctica 3 se mostrará al profesor el funcionamiento del balanceo de carga con los diferentes balanceadores. En el documento a entregar se describirá cómo se ha realizado la configuración de ambas máquinas y del software, así como los resultados de la comparación de balanceadores al someterlos a la misma alta carga de peticiones HTTP. 

---

## Resolución

---

### Configuración NginX
Para hacer la instalación, se recomienda ejecutar las siguientes órdenes:  
`sudo apt-get update && sudo apt-get dist-upgrade && sudo apt-get autoremove`   
`sudo apt-get install nginx`  
`sudo systemctl start nginx`  

 Una vez instalado, podemos proceder a su configuración como balanceador de carga. 

La configuración básica de nginx no nos vale tal cual está, ya que corresponde a una funcionalidad de servidor web, así es que tenemos que modificar el fichero de configuración /etc/nginx/conf.d/default.conf.

Debemos eliminar el contenido que tuviese antes, para crear la configuración que necesitamos. Si ese fichero no existiese, se crea nuevo con el contenido que describiremos a continuación. 

~~~

upstream apaches {  
    
    server 192.168.1.100;     
    server 192.168.1.101;  
} 
 
server{   
    
    listen 80;   
    server_name balanceador; 
    
    access_log /var/log/nginx/balanceador.access.log;  
    error_log /var/log/nginx/balanceador.error.log;   
    root /var/www/; 
    
   location /   {     
            proxy_pass http://apaches;     
            proxy_set_header Host $host;     
            proxy_set_header X-Real-IP $remote_addr;     
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;     
            proxy_http_version 1.1;     
            proxy_set_header Connection "";  
   }   
} 

~~~

![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%203/Nginx%20Conf.PNG)

Una vez que lo tenemos configurado, volvemos a lanzar el servicio : `sudo systemctl restart nginx`

Podemos probar la configuración haciendo peticiones a la IP de esta máquina balanceador (.200). Por ejemplo, podemos usar el comando cURL de la siguiente forma:   
`curl 192.168.1.200`  
`curl 192.168.1.200`


![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%203/Captura%20CURL%201%20al%20balanceador.PNG)
![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%203/Captura%20CURL%202%20al%20balanceador.PNG)

Por defecto tiene el valor 1, por lo que si no modificamos ninguna máquina del grupo, todas recibirán la misma cantidad de carga y distribuirá con un algoritmo Round-Robin, pero si queremos un algoritmo con ponderaciones necesario añadir lo siguiente: 

~~~

upstream apaches {  
    
    server 192.168.1.100 weight=2;    
    server 192.168.1.101 weight=1;

} 
 
~~~

En el ejemplo anterior, se comprobará el funcionamiento del algoritmo de balanceo con ponderación (en este caso supondremos que la máquina 1 tiene el doble de capacidad que la máquina 2).   
 
Podemos hacer un balanceo por IP, de forma que todo el tráfico que venga de una IP se sirva durante toda la sesión por el mismo servidor final. Para ello, como hemos indicado antes, usaremos la directiva ip_hash al definir el upstream: 

~~~

upstream apaches {  
    
    ip_hash;
    server 192.168.1.100;    
    server 192.168.1.101;

} 

~~~


**NOTA** : Antes de configurar NginX es necesario conectarla a la red interna y configurar la interfaz enp0s8 ( La IP del balanceador es 192.168.1.200).  
**NOTA** : Tener en cuenta de que si no funciona como balanceador tenemos que ir al fichero */etc/nginx/nginx.conf* y comentar la siguiente línea:
   
~~~

# include /etc/nginx/sites-enabled/*;

~~~


---

### Configuración HAProxy

Una vez instalado el servidor es necesario descargar el balanceador con la siguiente orden:  
`sudo apt-get install haproxy`  

Una vez instalado, debemos modificar el archivo /etc/haproxy/haproxy.cfg ya que la configuración que trae por defecto no nos vale.  
`sudo vi /etc/haproxy/haproxy.cfg `   

Un balanceador sencillo debe escuchar tráfico en el puerto 80 y redirigirlo a alguna de las máquinas servidoras finales. Usemos como configuración inicial la siguiente: 

~~~

 global  
    daemon  
    maxconn 256 

 defaults  
    mode http  
    contimeout    4000  
    clitimeout    42000  
    srvtimeout    43000 

 frontend http-in  
    bind *:80  
    default_backend servers 

 backend servers  
    server m1 192.168.1.100:80 maxconn 32  
    server m2 192.168.1.101:80 maxconn 32 

~~~

![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%203/Haproxy%20Conf.PNG)

Una vez salvada la configuración en el fichero, lanzamos el servicio haproxy mediante el comando:   
`sudo /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg`

Podemos probar la configuración haciendo peticiones a la IP de esta máquina balanceador (.200). Por ejemplo, como antes podemos usar el comando cURL de la siguiente forma:   
`curl 192.168.1.200`  
`curl 192.168.1.200`

---

### Someter a alta carga la granja web

Apache Benchmark (ab) es una utilidad que se instala junto con el servidor Apache y permite comprobar el rendimiento de cualquier servidor web. Para utilizarlo debemos entrar en un terminal y ejecutar el comando "ab" como sigue:  
`ab -n 1000 -c 10 http://IP-MAQ/index.html`  
`ab -n 1000 -c 10 http://192.168.1.200/index.html` -->  En mi caso  
Los parámetros indicados en la orden anterior le indican al benchmark que solicite la página con dirección http://192.168.1.200/index.html 1000 veces (-n 1000 indica el número de peticiones) y hacer esas peticiones concurrentemente de 10 en 10 (-c 10 indica el nivel de concurrencia). 

#### NginX
![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%203/Captura%20ab%20nginx.PNG)
#### HAProxy
![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%203/Captura%20ab%20haproxy.PNG)

**IMPORTANTE:** Hay que hacer ab a la dirección IP del balanceador (192.168.1.200, en mi caso).  

Para ver la sobrecarga de trabajo de las maquinas de la granja (máquina 1, máquina 2 y balanceadores) se ha utilizado otra herramienta, HTop, a continuación veremos unas capturas del resultado, que se han hecho 10000 peticiones para que se pueda ver correctamente.  
`sudo apt-get install htop` --> Para descargarlo  
`htop` --> Para iniciarlo  

`ab -n 10000 -c 10 http://192.168.1.200/index.html` --> En el cliente  

#### HTOP NGINX  
![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%203/Captura%20htop%20nginx.PNG)  
#### HTOP HAPROXY  
![img](https://github.com/iMiguel10/SWAP/blob/master/Practicas/Practica%203/Captura%20htop%20haproxy.PNG)

**Recordatorio:** Las IP de cada una de las máquinas son:  

*  Maquina 1: 192.168.1.100
*  Maquina 2: 192.168.1.101
*  Balanceador Nginx: 192.168.1.200
*  Balanceador HAProxy: 192.168.1.200
*  Cliente: 192.168.1.201

---


