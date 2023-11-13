# 1. 
Primero creamos un contenedor utilizando visual code con una imagen de ubuntu con el siguiente comando:  
docker run -it --name examen ubuntu  
Para abrir una consola en un contenedor que se está ejecutando, podemos hacer uso del siguiente comando:  
docker exec -it examen /bin/bash  
o de la siguiente forma:  
docker exec -it examen sh

# 2. 
Para interactuar con las entradas y salidas del contenedor, tenemos que lanzar el contenedor con opciones lo permitan:  
-i  : Permite mantener abierta la entrada estandar aunque no esté conectada a un terminal. Se utilizar para interactuar con la consola del contenedor.  
-t  : Asigna un terminal a un contenedor.  
-rm  : Elimina automaticamente el contenedor después de salir de la consola. Evita que se acumulen contenedores activos.  
-name  : Permite asignar un nombre a un contenedor.  
-p  : Permite mapear puertos entre el host y el contenedor.
-network  : Permite especificar la red a la que se conectará el contenedor.  

# 3.  
Para que dos contenedor se comuniquen entre si en una red solo de ellos, el docker-compose debería ser este, por ejemplo: 

services:
  contenedor1:  
    image: httpd:2.4  
    networks:  
      - red_contenedor  
  
  contenedor2:  
    image: httpd:2.4  
    networks:  
      - red_contenedor  

networks:  
  red_contenedor:  
    driver: bridge  

Donde red_contenedor es el nombre de la red que se creará para la comunicación entre estos dos contenedores.  
Para ejecutarlo, usaremos el comando:  
docker-compose up  
Esto iniciará ambos contenedores y creará la red red_contenedor y podrán comunicarse entre si.  

# 4.  
Para que un contenedor tenga la ip fija, deberemos añadir el siguiente código al archivo docker-compose.yml:  

services:  
  contenedor1:    
    image: httpd:2.4    
    networks:  
       red_contenedor:  
          ipv4_adress: 172.18.0.1    IP fija del contenedor 1  
  
  contenedor2:  
    image: httpd:2.4  
    networks:  
       red_contenedor: 
          ipv4_adress: 172.18.0.2   IP fija del contenedor 2  

networks:  
  red_contenedor:  
    driver: bridge  
    ipam:  
      config:  
        -subnet: 172.18.0.0/16      Subred definida  

# 5. 
Para conocer las ips de los contenedores anteriores podemos utilizar el comando:  
docker inspect f '{{.NetworkSettings.Networks.red_contenedor.IPAdress}}' contenedor1  
docker inspect f '{{.NetworkSettings.Networks.red_contenedor.IPAdress}}' contenedor2  

# 6. 
El apartado ports en docker compose se utiliza para la asignación de puertos entre el host y los contenedores. Permite usar puertos específicos del contenedor al host o asignar puertos dinamicamente. 

# 7.  
El registro CNAME se utiliza en el sistema de nombres de dominios para establecer una asociación de alias entre dos nombres de dominio. Un registro CNAME apunta a un nombre de dominio, lo que permite que un nombre de host se refiera a otro de forma "oficial".  
Ejemplo:  servidor.examen.com. IN CNAME www.examen.com.  
De esta forma, servidor.examen.com se convierte en un alias de www.examen.com, y cuando alguien intente acceder a servidor.examen.com el DNS redigirá automaticxamente la solicitud a www.examen.com.  

# 8. 
Para configurar un servidor DNS específico o realizar cambios en la configuración de DNS dentro de un contenedor, podemos utilizar un contenedor específico con el archivo de configuración deseado. Para ello crearemos un contenedor con el archivo de configuración DNS. Para esto debemos modificar el archivo '/etc/resolv.conf'. Podemos copiarlo y añadirlo al directorio de trabajo de nuestro contenedor, y así modificar en el la configuración DNS.  

# 9.
