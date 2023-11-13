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