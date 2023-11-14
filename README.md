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
También podemos utilizar volúmenes para almacenar la configuración del contenedor DNS. Para ello, cuando creamos un nuevo contenedor, tenemos que montar el volumen en la misma ubicación donde el conetendor DNS espera encontrar su configuración.  

# 9.
Dejo los archivos de configuración de zonas y el docker.compose.yml en el repositorio. Aqui explico las partes del docker-compose:  

services:  
  asir_bind9:  
    container_name: asir_bind9     #nombre del contenedor  
    image: ubuntu/bind9     #plataforma de la imagen  
    platform: linux/amd64  
    ports:  #mapeamos los puertos 53 tanto tcp como udp  
      - 53:53  
    networks:  
      bind9_subnet:  
        ipv4_address: 172.28.5.1  
    volumes:  
      - ./conf:/etc/bind   #Aquí mapeamos los archivos de configuración de bind  
      - ./zonas:/var/lib/bind   #Aqui mapeamos el directorio de zonas 
  
  cliente:  
    container_name: cliente  
    image: ubuntu  
    platform: linux/amd64  
    tty: true  
    stdin_open: true  
    dns:    #configuramos para que el cliente use el DNS  
      - 172.28.5.1  
    networks:  
      bind9_subnet:  #nombre de la red que usaremos para las pruebas  
        ipv4_address: 172.28.5.2  

networks:  
  bind9_subnet:  
    external: true  



Una vez tenemos configurado el docker-compose.yml y añadidos los archivos de configuracion de bind y de zona, creamos la subred de tal forma para poder probar nuestro servidor:  

$ docker network create \  
  --driver=bridge \  
  --subnet=172.28.0.0/16 \  
  --ip-range=172.28.5.0/24 \  
  --gateway=172.28.5.254 \  
  bind9_subnet  
  
Deespués levantamos el servicio con el comando:  
docker-compose up  
Para comprobar que funcionase usariamos el comando dig desde el contenedor cliente de tal forma:  
docker exec -it cliente dig www.tiendadeelectronica.int  
Debería devolver la IP 172.16.0.1  
Para ver los logs usariamos el comando:  
docker logs asir_bind9  
Este es el resultado del comando anterior:  

Starting named...  
exec /usr/sbin/named -u "bind" "-g" "-g -c /etc/bind/named.conf -u bind -f"
13-Nov-2023 16:27:56.860 starting BIND 9.18.18-0ubuntu2-Ubuntu (Extended Support Version) <id:>
13-Nov-2023 16:27:56.860 running on Linux x86_64 6.2.0-36-generic #37~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Mon Oct  9 15:34:04 UTC 2
13-Nov-2023 16:27:56.860 built with  '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--runstatedir=/run' '--disable-maintainer-mode' '--disable-dependency-tracking' '--libdir=/usr/lib/x86_64-linux-gnu' '--sysconfdir=/etc/bind' '--with-python=python3' '--localstatedir=/' '--enable-threads' '--enable-largefile' '--with-libtool' '--enable-shared' '--disable-static' '--with-gost=no' '--with-openssl=/usr' '--with-gssapi=yes' '--with-libidn2' '--with-json-c' '--with-lmdb=/usr' '--with-gnu-ld' '--with-maxminddb' '--with-atf=no' '--enable-ipv6' '--enable-rrl' '--enable-filter-aaaa' '--disable-native-pkcs11' 'build_alias=x86_64-linux-gnu' 'CFLAGS=-g -O2 -ffile-prefix-map=/build/bind9-UHPUkp/bind9-9.18.18=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/build/bind9-UHPUkp/bind9-9.18.18=/usr/src/bind9-1:9.18.18-0ubuntu2 -fno-strict-aliasing -fno-delete-null-pointer-checks -DNO_VERSION_DATE -DDIG_SIGCHASE' 'LDFLAGS=-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now' 'CPPFLAGS=-Wdate-time -D_FORTIFY_SOURCE=2'
13-Nov-2023 16:27:56.860 running as: named -u bind -g -g -c /etc/bind/named.conf -u bind -f
13-Nov-2023 16:27:56.860 compiled by GCC 13.2.0
13-Nov-2023 16:27:56.860 compiled with OpenSSL version: OpenSSL 3.0.10 1 Aug 2023
13-Nov-2023 16:27:56.860 linked to OpenSSL version: OpenSSL 3.0.10 1 Aug 2023
13-Nov-2023 16:27:56.860 compiled with libuv version: 1.44.2
13-Nov-2023 16:27:56.860 linked to libuv version: 1.44.2
13-Nov-2023 16:27:56.860 compiled with libxml2 version: 2.9.14
13-Nov-2023 16:27:56.860 linked to libxml2 version: 20914
13-Nov-2023 16:27:56.860 compiled with json-c version: 0.17
13-Nov-2023 16:27:56.860 linked to json-c version: 0.17
13-Nov-2023 16:27:56.860 compiled with zlib version: 1.2.13
13-Nov-2023 16:27:56.860 linked to zlib version: 1.2.13
13-Nov-2023 16:27:56.860 ----------------------------------------------------
13-Nov-2023 16:27:56.860 BIND 9 is maintained by Internet Systems Consortium,
13-Nov-2023 16:27:56.860 Inc. (ISC), a non-profit 501(c)(3) public-benefit 
13-Nov-2023 16:27:56.860 corporation.  Support and training for BIND 9 are 
13-Nov-2023 16:27:56.860 available at https://www.isc.org/support
13-Nov-2023 16:27:56.860 ----------------------------------------------------
13-Nov-2023 16:27:56.860 found 4 CPUs, using 4 worker threads
13-Nov-2023 16:27:56.860 using 4 UDP listeners per interface
13-Nov-2023 16:27:56.860 DNSSEC algorithms: RSASHA1 NSEC3RSASHA1 RSASHA256 RSASHA512 ECDSAP256SHA256 ECDSAP384SHA384 ED25519 ED448
13-Nov-2023 16:27:56.860 DS algorithms: SHA-1 SHA-256 SHA-384
13-Nov-2023 16:27:56.860 HMAC algorithms: HMAC-MD5 HMAC-SHA1 HMAC-SHA224 HMAC-SHA256 HMAC-SHA384 HMAC-SHA512
13-Nov-2023 16:27:56.860 TKEY mode 2 support (Diffie-Hellman): yes
13-Nov-2023 16:27:56.860 TKEY mode 3 support (GSS-API): yes
13-Nov-2023 16:27:56.860 config.c: option 'trust-anchor-telemetry' is experimental and subject to change in the future
13-Nov-2023 16:27:56.860 loading configuration from '/etc/bind/named.conf'
13-Nov-2023 16:27:56.860 open: /etc/bind/named.conf: file not found
13-Nov-2023 16:27:56.860 loading configuration: file not found
13-Nov-2023 16:27:56.860 exiting (due to fatal error)
Starting named...
exec /usr/sbin/named -u "bind" "-g" "-g -c /etc/bind/named.conf -u bind -f"
14-Nov-2023 16:07:39.690 starting BIND 9.18.18-0ubuntu2-Ubuntu (Extended Support Version) <id:>
14-Nov-2023 16:07:39.690 running on Linux x86_64 6.2.0-36-generic #37~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Mon Oct  9 15:34:04 UTC 2
14-Nov-2023 16:07:39.690 built with  '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--runstatedir=/run' '--disable-maintainer-mode' '--disable-dependency-tracking' '--libdir=/usr/lib/x86_64-linux-gnu' '--sysconfdir=/etc/bind' '--with-python=python3' '--localstatedir=/' '--enable-threads' '--enable-largefile' '--with-libtool' '--enable-shared' '--disable-static' '--with-gost=no' '--with-openssl=/usr' '--with-gssapi=yes' '--with-libidn2' '--with-json-c' '--with-lmdb=/usr' '--with-gnu-ld' '--with-maxminddb' '--with-atf=no' '--enable-ipv6' '--enable-rrl' '--enable-filter-aaaa' '--disable-native-pkcs11' 'build_alias=x86_64-linux-gnu' 'CFLAGS=-g -O2 -ffile-prefix-map=/build/bind9-UHPUkp/bind9-9.18.18=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/build/bind9-UHPUkp/bind9-9.18.18=/usr/src/bind9-1:9.18.18-0ubuntu2 -fno-strict-aliasing -fno-delete-null-pointer-checks -DNO_VERSION_DATE -DDIG_SIGCHASE' 'LDFLAGS=-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now' 'CPPFLAGS=-Wdate-time -D_FORTIFY_SOURCE=2'
14-Nov-2023 16:07:39.690 running as: named -u bind -g -g -c /etc/bind/named.conf -u bind -f
14-Nov-2023 16:07:39.690 compiled by GCC 13.2.0
14-Nov-2023 16:07:39.690 compiled with OpenSSL version: OpenSSL 3.0.10 1 Aug 2023
14-Nov-2023 16:07:39.690 linked to OpenSSL version: OpenSSL 3.0.10 1 Aug 2023
14-Nov-2023 16:07:39.690 compiled with libuv version: 1.44.2
14-Nov-2023 16:07:39.690 linked to libuv version: 1.44.2
14-Nov-2023 16:07:39.690 compiled with libxml2 version: 2.9.14
14-Nov-2023 16:07:39.690 linked to libxml2 version: 20914
14-Nov-2023 16:07:39.690 compiled with json-c version: 0.17
14-Nov-2023 16:07:39.690 linked to json-c version: 0.17
14-Nov-2023 16:07:39.690 compiled with zlib version: 1.2.13
14-Nov-2023 16:07:39.690 linked to zlib version: 1.2.13
14-Nov-2023 16:07:39.690 ----------------------------------------------------
14-Nov-2023 16:07:39.690 BIND 9 is maintained by Internet Systems Consortium,
14-Nov-2023 16:07:39.690 Inc. (ISC), a non-profit 501(c)(3) public-benefit 
14-Nov-2023 16:07:39.690 corporation.  Support and training for BIND 9 are 
14-Nov-2023 16:07:39.690 available at https://www.isc.org/support
14-Nov-2023 16:07:39.690 ----------------------------------------------------
14-Nov-2023 16:07:39.690 found 4 CPUs, using 4 worker threads
14-Nov-2023 16:07:39.690 using 4 UDP listeners per interface
14-Nov-2023 16:07:39.690 DNSSEC algorithms: RSASHA1 NSEC3RSASHA1 RSASHA256 RSASHA512 ECDSAP256SHA256 ECDSAP384SHA384 ED25519 ED448
14-Nov-2023 16:07:39.690 DS algorithms: SHA-1 SHA-256 SHA-384
14-Nov-2023 16:07:39.690 HMAC algorithms: HMAC-MD5 HMAC-SHA1 HMAC-SHA224 HMAC-SHA256 HMAC-SHA384 HMAC-SHA512
14-Nov-2023 16:07:39.690 TKEY mode 2 support (Diffie-Hellman): yes
14-Nov-2023 16:07:39.690 TKEY mode 3 support (GSS-API): yes
14-Nov-2023 16:07:39.694 config.c: option 'trust-anchor-telemetry' is experimental and subject to change in the future
14-Nov-2023 16:07:39.694 loading configuration from '/etc/bind/named.conf'
14-Nov-2023 16:07:39.694 unable to open '/etc/bind/bind.keys'; using built-in keys instead
14-Nov-2023 16:07:39.694 looking for GeoIP2 databases in '/usr/share/GeoIP'
14-Nov-2023 16:07:39.694 using default UDP/IPv4 port range: [32768, 60999]
14-Nov-2023 16:07:39.694 using default UDP/IPv6 port range: [32768, 60999]
14-Nov-2023 16:07:39.698 listening on IPv4 interface lo, 127.0.0.1#53
14-Nov-2023 16:07:39.698 listening on IPv4 interface eth0, 172.28.5.1#53
14-Nov-2023 16:07:39.698 generating session key for dynamic DNS
14-Nov-2023 16:07:39.706 sizing zone task pool based on 1 zones
14-Nov-2023 16:07:39.706 none:99: 'max-cache-size 90%' - setting to 14386MB (out of 15985MB)
14-Nov-2023 16:07:39.706 using built-in root key for view _default
14-Nov-2023 16:07:39.706 set up managed keys zone for view _default, file 'managed-keys.bind'
14-Nov-2023 16:07:39.710 configuring command channel from '/etc/bind/rndc.key'
14-Nov-2023 16:07:39.710 command channel listening on 127.0.0.1#953
14-Nov-2023 16:07:39.710 configuring command channel from '/etc/bind/rndc.key'
14-Nov-2023 16:07:39.710 command channel listening on ::1#953
14-Nov-2023 16:07:39.710 not using config file logging statement for logging due to -g option
14-Nov-2023 16:07:39.714 managed-keys-zone: loaded serial 0
14-Nov-2023 16:07:39.714 zone tiendadeelectronica.int/IN: loading from master file /var/lib/bind/db.tiendadeelectronica.int failed: file not found
14-Nov-2023 16:07:39.714 zone tiendadeelectronica.int/IN: not loaded due to errors.
14-Nov-2023 16:07:39.714 all zones loaded
14-Nov-2023 16:07:39.722 running
14-Nov-2023 16:07:39.742 managed-keys-zone: Initializing automatic trust anchor management for zone '.'; DNSKEY ID 20326 is now trusted, waiving the normal 30-day waiting period.  

