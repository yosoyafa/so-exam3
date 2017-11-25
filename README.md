### Examen 3
**Universidad ICESI**  
**Curso:** Sistemas Operativos  
**Docente:** Daniel Barragán C.  
**Tema:** Descubrimiento de servicios, Microservicios  
**Correo:** daniel.barragan at correo.icesi.edu.co  

**Nombre:** Carlos Andrés Afanador Cabal  
**Código:** A00054652  
**Correo:** carlosafanador97 at gmail  
**URL Git:** https://github.com/yosoyafa/so-exam3  


### Punto 3  

Para desplegar el sistema propuesto, se utilizaron varias máquinas, una de ellas fue utilizada como **balanceador de carga** y otra como un **descubridor de servicios**; el resto de máquinas brindad los microservicios. El sistema funciona de tal forma que un cliente cualquiera se conecta a él, y el balanceador de cargas que pide servicios a un descubridor de servicios, el cual ya ha *descubierto* servidores que brindan ciertos servicios, y hace que el cliente utilice alguno de ellos, si es el caso.  
   
   Primeramente, se configuró el balanceador de cargas, instalando las dependencias de HAProxy:
   ```
$ sudo yum info haproxy
$ sudo yum install gcc pcre-static pcre-devel -y
$ wget https://www.haproxy.org/download/1.7/src/haproxy-1.7.8.tar.gz -O ~/haproxy.tar.gz
$ tar xzvf ~/haproxy.tar.gz -C ~/
$ cd ~/haproxy-1.7.8
$ make TARGET=linux2628
$ sudo make install
```

  Ahora se procede a configurar el HAProxy:  
  
  ```
  $ sudo mkdir -p /etc/haproxy
$ sudo mkdir -p /var/lib/haproxy 
$ sudo touch /var/lib/haproxy/stats
$ sudo cp ~/haproxy-1.7.8/examples/haproxy.init /etc/init.d/haproxy
$ sudo chmod 755 /etc/init.d/haproxy
$ sudo systemctl daemon-reload
$ sudo chkconfig haproxy on
$ sudo useradd -r haproxy
```

Se abren los puertos necesarios para habilitar el servicio:

```
$ sudo firewall-cmd --permanent --zone=public --add-service=http
$ sudo firewall-cmd --permanent --zone=public --add-port=8181/tcp
$ sudo firewall-cmd --reload
```

Por último se configura el archivo donde se registran los servicios que estan disponibles:

```
$ sudo vi /etc/haproxy/haproxy.cfg
$ sudo systemctl restart haproxy
``` 

Configuracion del balanceador:

  ![][1]

Ya se puede ver el funcionamiento del balanceador de cargas:

  ![][2]
  
  Ahora bien, para poder hacer actualizaciones automáticas de los servicios descubiertos, se usa consul-tamplate, agregando el nombre del micorservicio, junto su IP:
  
    ![][3]
    
    Ya para el montaje del descubridor de servicios, se debe usar consul en la máquina designada pero en modo servidor, para poder recibir a los clientes:
    
    Primero se instalan las dependencias:
    ```
 yum install -y wget unzip
 wget https://releases.hashicorp.com/consul/1.0.0/consul_1.0.0_linux_amd64.zip -P /tmp
 unzip /tmp/consul_1.0.0_linux_amd64.zip -d /tmp
 mv /tmp/consul /usr/bin
 mkdir /etc/consul.d
 mkdir -p /etc/consul/data
```
 ![][4]
      
      Con el servidor de descubrimeiento montado, ya se pueden visualizar los agentes de consul que han sido *descubiertos*:
      
        ![][5]
        
      Funcionamiento de descubrimiento de servicios:
      
        ![][6]
        
        
   
