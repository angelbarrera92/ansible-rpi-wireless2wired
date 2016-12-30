Ansible - Raspberry PI Wireless to Wired
===================

## Introducción 
Este repositorio contiene un *"role"* creado a modo de prueba de concepto para aprender **Ansible**. 

Este *"role"* permite crear un *pseudo router* que enlaza una conexión a internet via Wifi hacia la conexión cableada de la Raspberry Pi.

Un caso de uso en el que este *"role"* puede ser util es posibilitar la conexión a internet a dispositivos que no dispongan de conexión Wifi y no se tenga una toma de red cableada cerca. Un ejemplo podría ser un Televisor SmartTV *(sin wifi)* o una Playstation 2 :). 

La idea original y en la que está basado este *"role"* proviene de la web, [glennklockwood.com](http://www.glennklockwood.com/sysadmin-howtos/rpi-wifi-island.html), donde explican su caso de uso concreto: 

![Diagrama](http://www.glennklockwood.com/sysadmin-howtos/rpi-network-diagram.png)


## Requisitos
Para hacer funcionar este role necesitaremos: 

1. Raspberry Pi. *(Probado con Raspberry Pi 1 B+).* *(Alimentación, HDMI...)*
2. [Raspbian](https://downloads.raspberrypi.org/raspbian_lite_latest) recien instalado y con acceso [SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/). 
3. Adaptador Wifi USB *(Que sea compatible con Raspberry Pi).* [Ejemplo en Amazon](https://www.amazon.es/Edimax-EW-7811UN-Adaptador-Interfaz-negro/dp/B003MTTJOY).
4. PC con [Ansible](http://docs.ansible.com/ansible/intro_installation.html) y [GIT](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) conectados a la misma Red que la Raspberry Pi. 

## Tutorial
### **Instalación de Raspbian** en tarjeta SD. 
Existen muchos manuales por internet y no nos vamos a parar a explicarlo de nuevo. Simplemente enumero los pasos que se deberían de dar:

1. Insertar SD en un PC. 
2. Desmontar las particiones de la SD que pudiera tener.
3. Volcado de la imagen de Rasbian, previamente descargada, a la tarjeta de memoria. En un PC con linux sería algo así:  ```$ sudo dd bs=1M if=RUTA_DE_RASPBIAN.img of=RUTA_DE_TARJETA_SD```
4. Creación de fichero ```ssh``` en partición boot de la tarjeta SD. (*) Solo a partir de la versión Jessie. 

### **Arranque de Raspbian**. 
Inserción de la tarjeta SD en la Raspberry Pi y encendido de la misma. Una vez arranque deberemos averiguar con que IP se ha conectado. *(Conectada por cable de Red al router)*. 

Existen diversas formas de averiguar la IP de la Raspberry Pi: 

1. Si está conectada a un monitor, se ve en la propia pantalla. 
2. Si disponemos de un pc con linux podemos ejecutar un ```nmap``` para ver que dispositivos tienen el puerto 22 abierto en nuestra red. 
3. Acceder a la configuración del Router y ver la tabla DHCP.

Una vez obtenida la dirección IP la apuntamos para el siguiente apartado. 

### **Clonado del repositorio**. 
Debemos clonarnos este repositorio Git. Si habéis llegado hasta aquí, no hace falta que profundicemos mucho. 

``` git clone https://github.com/angelbarrera92/ansible-rpi-wireless2wired.git ```

``` cd ansible-rpi-wireless2wired ```

Una vez clonado y estando dentro del directorio descargado, editaremos el fichero ```hosts``` del repositorio sustituyendo la IP que está escrita por la IP de nuestra Raspberry Pi. 
Si la IP de la Raspberry Pi es: ```192.168.1.34```, el fichero ```hosts``` deberá quedar algo así
```
[rpis]
192.168.1.34
```
Falta editar los parámetros, variables, del playbook. Editamos el *playbook.yaml* y nos fijamos en el apartado vars: 
```
vars:
    LOCAL_IFACE: eth0
    LOCAL_ADDRESS: 192.168.2.1
    LOCAL_NETMASK: 255.255.255.0
    LOCAL_NETWORK: 192.168.2.0
    LOCAL_GATEWAY: 192.168.2.1
    LOCAL_BROADCAST: 192.168.2.255

    INET_IFACE: wlan0
    INET_ADDRESS: 192.168.1.101
    INET_NETMASK: 255.255.255.0
    INET_NETWORK: 192.168.1.0
    INET_GATEWAY: 192.168.1.1
    INET_BROADCAST: 192.168.1.255
    WIFI_SSID: 
    WIFI_PASSWORD: 
```
Tendremos que rellenar los datos de *INET_IFACE* con los correspondientes a como tengamos configurada la Red por la que sale a Internet así como el *SSID* y la *contraseña* de la red Wifi. 

Con esto editado, ya tenemos todo listo

### **Ejecución del playbook**. 
Ya tenemos todo listo para enviar el *"role"* a la Raspberry Pi. Para ello:

```ansible-playbook playbook.yaml -i hosts --ask-pass --become -c paramiko```

Nos pedirá la contraseña de conexión ssh: ***raspberry*** 
Pulsamos intro y esperamos a que se apague la Raspberry Pi. 
La próxima vez que la encendamos, ya podemos usar como punto de acceso por cable de Red. 
Solo tendremos que configurar el dispositivo con la configuración correcta conectandolo con el cable de Red desde la Raspberry Pi hasta nuestro dispositivo *(como un router)*.  
```
IP: 192.168.2.XXX (XXX diferente de 1)
Gateway: 192.168.2.1
Mask: 255.255.255.0
DNS: 8.8.8.8
```

## Disclaimer
Este tutorial se ha realizado a modo de auto aprendizaje. No se ha probado en profundidad, solo se ha probado que es funcional. 
Tampoco se ha probado con otras distribuciones de Raspberry Pi ni con los modelos superiores de la misma. 
Se ha realizado la prueba partiendo de una instalación limpia de Raspbian Jessie. 
Seguro que hay alguien que ha eliminado el usuario ***pi*** *(por seguridad)* o ha cambiado la contraseña por defecto o bien ha deshabilitado *ssh*, deberá adaptar este manual a sus necesidades...
