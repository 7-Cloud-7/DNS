# Crear y configurar un servidor DNS

## Instalar la imagen de bind9
~~~
docker pull internetsystemsconsortium/bind9:9.16
~~~
## Volumen por separado de la configuración:

~~~
version: '3.3'
services:
 asir_bind9_2:
 container_name: asir_bind9_2
 image: internetsystemsconsortium/bind9:9.16
 ports:
 - 5300:53/udp
 - 5300:53/tcp
 volumes:
 - /home/asir2a/Escritorio/SRI/DNS_2(Alpine-bind9)/DNSbind9/conf:/etc/bind
 - /home/asir2a/Escritorio/SRI/DNS_2(Alpine-bind9)/DNSbind9/zonas:/var/lib/bind
~~~


## Red propia:
~~~
docker network create --driver=bridge --subnet=10.0.2.0/24 asir_bind9_subnet2
~~~
~~~
{
"Name": "asir_bind9_subnet2",
"Id": "055246d6181d0a0ab65d30654c9283cadd9664004081d27efb3a6a7cb2da71e8",
"Created": "2022-10-13T17:38:17.499420073+02:00",
"Scope": "local",
"Driver": "bridge",
"EnableIPv6": false,
"IPAM": {
"Driver": "default",
"Options": {},
"Config": [
 {
"Subnet": "11.1.2.0/24"
 }
 ]
 },
"Internal": false,
"Attachable": false,
"Ingress": false,
"ConfigFrom": {
"Network": ""
 },
"ConfigOnly": false,
"Containers": {},
"Options": {},
"Labels": {},
"CreatedTime": 1665675497499
}
~~~

## IP fija:

~~~
version: '3.3'
services:
asir_bind9_2:
container_name: asir_bind9_2
image: internetsystemsconsortium/bind9:9.16
networks:
asir_bind9_subnet2:
ipv4_address: 11.1.2.10
ports:
 - 5300:53/udp
 - 5300:53/tcp
volumes:
 - /home/asir2a/Escritorio/SRI/DNS_2(Alpine-bind9)/DNSbind9/conf:/etc/bind
 - /home/asir2a/Escritorio/SRI/DNS_2(Alpine-bind9)/DNSbind9/zonas:/var/lib/bind
networks:
asir_bind9_subnet2:
external: true
~~~

## Configurar forwarders:
~~~
options {

        forwarders {
            8.8.8.8;
            8.8.4.4;
        };
};
~~~

## Crear una zona propia (Registros):

### Archivo /conf/named.conf.local
~~~
zone "clouddomain.com." {
        type master;
        file "/var/lib/bind/db.clouddomain.com";
        notify explicit;
};
~~~
 

### Archivo zonas/db.clouddomain.com

~~~
$TTL    3600
@       IN      SOA     ns.clouddomain.com. igonzalezvila.danielcastelao.org. (
                   2007010401           ; Serial
                         3600           ; Refresh [1h]
                          600           ; Retry   [10m]
                        86400           ; Expire  [1d]
                          600 )         ; Negative Cache TTL [1h]
;
@       IN      NS      ns.clouddomain.com.
@       IN      MX      10 serveremail.clouddomain.org.

ns     IN       A       11.1.2.10
etch    IN      A       11.1.2.2

pop     IN      CNAME   ns
www     IN      CNAME   etch
mail    IN      CNAME   etch
~~~

## Cliente con herramientas de red:

Añadir al docker-compose.yml el siguiente codigo
~~~
asir_cliente_2:
container_name: asir_cliente_2
image: alpine
networks:
 - asir_bind9_subnet2

stdin_open: true
tty: true#docker run -t
dns:
 - 11.1.2.10#ip del contenedor DNS
~~~
Cliente con herramientas de red:
~~~
docker exec -it asir_bind9_2 sh
apt install net-tools
~~~

## Procedimiento de creación de servicios (contenedor)
Usar el comando "docker-compose up" con el siguiente archivo docker-compose.yml:
~~~
version: '3.3'
services:
asir_bind9_2:
container_name: asir_bind9_2
image: internetsystemsconsortium/bind9:9.16
~~~

## Modificación de la configuración, arranque y parada de servicio bind9

Para arrancar los contenedores usaremos el comando "docker-compose up" y para detenerlos "docker-compose down"

### Archivos de configuración:

named.conf :
~~~
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
~~~

named.conf.local :
~~~
zone "clouddomain.com." {
        type master;
        file "/var/lib/bind/db.clouddomain.com";
        notify explicit;
};
~~~

named.conf.options :
~~~
options {
        directory "/var/cache/bind";
        forwarders {
            8.8.8.8;
            8.8.4.4;
        };
};
~~~
