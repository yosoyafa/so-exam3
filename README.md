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

Para desplegar el sistema propuesto, se utilizaron varias máquinas, una de ellas fue utilizada como **balanceador de carga** y otra como un **descubridor de servicios**; el resto de máquinas brindad los **microservicios**. El sistema funciona de tal forma que un cliente cualquiera se conecta a él, y el balanceador de cargas que pide servicios a un descubridor de servicios, el cual ya ha *descubierto* servidores que brindan ciertos servicios, y hace que el cliente utilice alguno de ellos, si es el caso.  
   
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
    
    Ya para el montaje del descubridor de servicios, se debe usar consul en la máquina designada pero en modo servidor, para poder recibir a los clientes.
    
    Primero se instalan las dependencias:
    
```
# yum install -y wget unzip
# wget https://releases.hashicorp.com/consul/1.0.0/consul_1.0.0_linux_amd64.zip -P /tmp
# unzip /tmp/consul_1.0.0_linux_amd64.zip -d /tmp
# mv /tmp/consul /usr/bin
# mkdir /etc/consul.d
# mkdir -p /etc/consul/data
```

Se crea el usuario consul, donde se implementarán estos servicios:

```
# adduser consul
# passwd consul
# chown -R consul:consul /etc/consul
# chown -R consul:consul /etc/consul.d
```

Se abren los puertos necesarios para desplegar el servicio:
```
# firewall-cmd --zone=public --add-port=8301/tcp --permanent
# firewall-cmd --zone=public --add-port=8300/tcp --permanent
# firewall-cmd --zone=public --add-port=8500/tcp --permanent
# firewall-cmd --reload
```

Ahora se inicia el agente en modo servidor:
    
 ![][4]  
      
Con el servidor de descubrimeiento montado, ya se pueden visualizar los agentes de consul que han sido descubiertos:
+```
+$ consul members
+```  
        
![][5]
        
Funcionamiento de descubrimiento de servicios:
      
![][6]

Ahora se procede a la configuracion de los microservicios en diferentes servidores que trabajaran como *consul-agents*.  
Primero se deben instalar las dependencias necesarias:
```
# yum install -y wget unzip
# wget https://bootstrap.pypa.io/get-pip.py -P /tmp
# python /tmp/get-pip.py
# wget https://releases.hashicorp.com/consul/1.0.0/consul_1.0.0_linux_amd64.zip -P /tmp
# unzip /tmp/consul_1.0.0_linux_amd64.zip -d /tmp
# mv /tmp/consul /usr/bin
# mkdir /etc/consul.d
# mkdir -p /etc/consul/data
```
Se crea el usuario consul, donde se implementarán estos servicios:
```
# adduser consul
# passwd consul
# chown -R consul:consul /etc/consul
# chown -R consul:consul /etc/consul.d
```

Se abren los puertos necesarios para desplegar el servicio:
```
# firewall-cmd --zone=public --add-port=8301/tcp --permanent
# firewall-cmd --reload
# firewall-cmd --zone=public --add-port=8080/tcp --permanent
# firewall-cmd --reload
```
  
En un usuario designado para el despliegue del microservicio se instala la librería de *flask* y se procede a editar en python el microservicio:
  
```
$ pip install flask
$ vi afaservicio.py
```
![][7]
  

Se inicia el microservicio y se verifica que funcione correctamente:
```
$ python afaservicio.py
```
  
![][8]

Servicio funcionando:  
  
![][9]

Se crea un archivo de configuración para el microservicio con un healthcheck en el usuario *consul*:
  
![][10]

Se inicia el *consul-agent*:

![][11]

Por útlimo, se une el microservicio al descubridor, y se verifica que se encuentre el microservcio en la lista de miembros:

![][12]


### Punto 4  

Con varios microservicios disponibles, el balanceador puede referir el cliente a alguno de ellos, y para esto utiliza el metodo *round-robin* que va direccionando de manera secuencial cada cliente, al siguiente servidor:

Funcionamiento del balanceador: 
![][13]

Evidencias del balanceo:

![][14]

![][15]

![][16]


### Punto 5

Para incluir un microservicio diferente al ya desplegado, se requiere un API Gateway, el cual será el que reciba las solicitudes de los clientes (anteriromente este era el trabajo del balanceador) y se conecta a los balanceadores de cada servicio. El API Gateway se encargará de reunir los servicios y así poder hacer un uso más eficiente de la infraestructura desplegada, codificando los servicios prestados. Ahora bien, todo esto es posible gracias al paradigma reactivo, el cual hace un envio asíncrono de la información solicitado, sin tener que lidiar con los resultados obtenidos o los errores presentados, los cuales se revisan en momentos posteriores.

        
[1]: images/configuracionBalanceador.png
[2]: images/BalanceadorCorriendo.png
[3]: images/configuracionConsulTemplates.png
[4]: images/consul_agent_server.PNG
[5]: images/consul_members.PNG
[6]: images/consul_logs.PNG	
[7]: images/py.png
[8]: images/gets.png
[9]: images/afaservicio.png
[10]: images/json.png
[11]: images/consulagent.png
[12]: images/afaConsulMembers.png
[13]: images/DemostracionFuncionBalanceador.png
[14]: images/b1.png
[15]: images/b2.png
[16]: images/b3.png 
  
