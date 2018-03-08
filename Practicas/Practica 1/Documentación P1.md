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

##Resolución

Para hacer que las máquinas estén en la misma red es necesario añadirles un nuevo adaptador de red, en este caso uno de red interna, e ir al archivo de configuración de las interfaces de red del servidor y añadir la nueva interfaz.  

`vi /etc/network/interfaces`  

### Máquina 1  
> 
auto enp0s8  
iface enp0s8 inet static  
address 192.168.1.100  
gateway 192.168.1.1  
netmask 255.255.255.0  
network 192.168.1.0  
broadcast 192.168.1.255

Para mostrar que las máquinas tienen conectividad entre ellas e incluso a internet mostraré una captura haciendo un simple ping.