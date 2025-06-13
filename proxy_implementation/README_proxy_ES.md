
# Laboratorio de Proxy en Linux con Autenticación (Squid + ACL)

## Descripción

Este proyecto es una implementación práctica de segmentación de red, reglas de firewall y un proxy autenticado utilizando Squid en Ubuntu Server 22.04. Forma parte del curso "Protección de la Información en Redes de Computadoras" de la UrFU. El entorno está simulado con equipos Cisco y máquinas virtuales basadas en Linux.

## Vista General de la Topología

La red está dividida en tres zonas:
- DMZ (Zona Desmilitarizada)
- Servidores Internos
- Clientes Internos

![topology](/screenshots/topology.jpg)

## Tabla de Nodos

| Zona             | Nodo  | Interfaz | Interfaz DMZ | Interfaz Servidores Internos | Interfaz Clientes Internos | Dirección IP |
|------------------|--------|----------|---------------|-------------------------------|-----------------------------|---------------|
| -                | GW-FW  | fa1/0    | Gi0/3         | -                             | -                           | 172.16.1.1    |
| -                | GW-FW  | fa2/0    | -             | Gi0/3                         | -                           | 172.16.2.1    |
| -                | GW-FW  | fa3/0    | -             | -                             | Gi0/3                       | 172.16.2.3    |
| DMZ              | DMZ1   | e0       | Gi0/0         | -                             | -                           | 172.16.1.2    |
| DMZ              | DMZ2   | e0       | Gi0/1         | -                             | -                           | 172.16.1.3    |
| DMZ              | DMZ3   | e0       | Gi0/2         | -                             | -                           | 172.16.1.4    |
| Servidores       | SRV1   | e0       | -             | Gi0/0                         | -                           | 172.16.2.2    |
| Servidores       | SRV2   | e0       | -             | Gi0/1                         | -                           | 172.16.2.3    |
| Servidores       | SRV3   | e0       | -             | Gi0/2                         | -                           | 172.16.2.4    |
| Clientes         | PC1    | e0       | -             | -                             | Gi0/0                       | 172.16.3.2    |
| Clientes         | PC2    | e0       | -             | -                             | Gi0/1                       | 172.16.3.3    |
| Clientes         | PC3    | e0       | -             | -                             | Gi0/2                       | 172.16.3.4    |

## Configuración Principal

![config](/screenshots/ip_brief.jpg)

### Gateway-Firewall (Cisco)

![fire](/screenshots/firewall_setup.jpg)

#### NAT y Enrutamiento

![nat](/screenshots/nat_config.jpg)

```shell
ip routing
access-list 1 permit 172.16.0.0 0.0.255.255
ip nat inside source list 1 interface fa0/0 overload
interface fa1/0
 ip nat inside
interface fa2/0
 ip nat inside
interface fa3/0
 ip nat inside
interface fa0/0
 ip nat outside
ip route 0.0.0.0 0.0.0.0 192.168.36.2
```

#### Pools de DHCP

![dchp](/screenshots/dhcp.jpg)

```shell
ip dhcp pool INTERNAL_CLIENTS
 network 172.16.3.0 255.255.255.0
 default-router 172.16.3.1

ip dhcp pool INTERNAL_SERVERS
 network 172.16.2.0 255.255.255.0
 default-router 172.16.2.1

ip dhcp pool DMZ
 network 172.16.1.0 255.255.255.0
 default-router 172.16.1.1
```

#### Reglas ACL

```shell
access-list 100 deny tcp 172.16.3.0 0.0.0.255 any eq www
access-list 100 deny tcp 172.16.3.0 0.0.0.255 any eq 443
access-list 100 permit ip any any
```

### Servidor Proxy (Squid en Ubuntu SRV1)

![squid](/screenshots/srv1_squid.jpg)

```bash
sudo apt update
sudo apt install squid apache2-utils
sudo htpasswd -c /etc/squid/passwords bruno
```

#### squid.conf

![config](/screenshots/auth_config.jpg)

```bash
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwords
auth_param basic realm Proxy Authentication
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive off

acl authenticated proxy_auth REQUIRED
http_access allow authenticated
http_access deny all
```

#### Variables de Entorno del Cliente

![auth](/screenshots/allow_proxy_gui.jpg)

![auth_dos](/screenshots/allow_proxy_forever.jpg)

```bash
export http_proxy="http://172.16.2.2:3128"
export https_proxy="http://172.16.2.2:3128"
```

## Validación

- El proxy solicita credenciales al navegar
- curl devuelve "407 Proxy Authentication Required" si no se ingresa usuario
- Los logs de Squid confirman la recepción del tráfico
- Las ACL bloquean el acceso directo si no se usa el proxy

![curl](/screenshots/curl_request.jpg)

![request](/screenshots/request_firefox_proxy.jpg)

## Autor

Bruno Paolo Huaman Vela  (Lima, Peru)
УрФУ – Seguridad de la Información en Sistemas de Telecomunicaciones  
