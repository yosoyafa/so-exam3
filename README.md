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
   
   Primeramente, se configuró el balanceador de cargas, instalando las dependencias de HAProxy de consul-tamplate:
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

   
   
