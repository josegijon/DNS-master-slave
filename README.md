# Práctica de Configuración de Servidores DNS Maestro y Esclavo

## Introducción

Esta práctica consiste en la configuración de un servidor DNS maestro y un servidor DNS esclavo utilizando BIND (Berkeley Internet Name Domain). Se establecen zonas de dominio y se verifica la correcta resolución de nombres y la transferencia de zonas entre ambos servidores.

## Tabla de contenidos:
---

1. [Requisitos Previos](#requisitos-previos)
2. [Configuración del Servidor DNS Maestro](#configuración-del-servidor-dns-maestro)
   - [Archivo db.sistema.test](#1-archivo-dbsistematest)
   - [Archivo db.57.168.192](#2-archivo-db57168192)
   - [Configuración de named.conf.localmaster](#3-configuración-de-namedconflocalmaster)
   - [Configuración de named.conf.options](#4-configuración-de-namedconfoptions)
3. [Configuración del Servidor DNS Esclavo](#configuración-del-servidor-dns-esclavo)
4. [Verificación y Pruebas](#verificación-y-pruebas)
   - [Comprobación de la Resolución de Nombres](#1-comprobación-de-la-resolucion-de-nombres)
   - [Comprobación de la Transferencia de Zonas](#2-comprobación-de-la-transferencia-de-zonas)
   - [Verificación con nslookup](#3-verificación-con-nslookup)
   - [Logs del Servidor DNS](#4-logs-del-servidor-dns)

## Requisitos Previos

- Dos máquinas virtuales en una red privada:
  - **Servidor Maestro**: IP 192.168.57.103
  - **Servidor Esclavo**: IP 192.168.57.102

- Instalación de BIND en ambas máquinas:
  ```bash
  sudo apt update
  sudo apt install bind9

## Configuración del Servidor DNS Maestro
### 1. Archivo db.sistema.test
El archivo de zona directa (db.sistema.test) en el servidor maestro define los registros DNS para el dominio sistema.test:
  ```bash
  $TTL	86400
  @	IN	SOA	tierra.sistema.test. admin.sistema.test. (
              4		; Serial
        604800		; Refresh
          86400		; Retry
        2419200		; Expire
          7200 )	; Negative Cache TTL
  ;
  @			IN	NS	tierra.sistema.test.
  @			IN 	NS 	venus.sistema.test.
  @			IN	MX	10	marte.sistema.test.

  tierra		IN	A	192.168.57.103
  venus		IN	A	192.168.57.102
  mercurio	IN	A	192.168.57.101
  marte		IN	A	192.168.57.104

  ns1			IN 	CNAME	tierra.sistema.test.
  ns2			IN 	CNAME	venus.sistema.test.
  mail		IN 	CNAME  	marte.sistema.test.
  ```

### 2. Archivo db.57.168.192
El archivo de zona inversa define la resolución inversa de las IPs:
```bash
  $TTL	604800
  @	IN	SOA	tierra.sistema.test. admin.sistema.test. (
              2		; Serial
        604800		; Refresh
          86400		; Retry
        2419200		; Expire
        7200 )	; Negative Cache TTL
  ;
  @		IN			NS		tierra.sistema.test.
  103		IN			PTR		tierra.sistema.test.
  102		IN			PTR		venus.sistema.test.
  101		IN			PTR		mercurio.sistema.test.
  104		IN			PTR		marte.sistema.test.
  ```

### 3. Configuración de named.conf.localmaster
En el archivo de configuración local del maestro, se definen las zonas de dominio y los permisos de transferencia de zona:
```bash
zone "sistema.test" {
    type master;
    file "/etc/bind/db.sistema.test";
    allow-transfer { 192.168.57.102; };
};

zone "57.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.57.168.192";
    allow-transfer { 192.168.57.102; };
};
```

### 4. Configuración de named.conf.options
```bash
acl "red-local" {
    127.0.0.0/8;
    192.168.57.0/24;
};

options {
	directory "/var/cache/bind";
	
	forwarders {
        208.67.222.222;
    };

	allow-recursion { red-local; };
	allow-query { red-local; };
	allow-transfer { red-local; };

	dnssec-validation yes;

	listen-on-v6 { any; };
	listen-on { any; };
};
```

## Configuración del Servidor DNS Esclavo
### Archivo named.conf.localslave
En el servidor esclavo, el archivo de configuración define que este es un servidor de zona esclavo:
```bash
zone "sistema.test" {
    type slave;
    file "/etc/bind/db.sistema.test";
    masters { 192.168.57.103; };
};

zone "57.168.192.in-addr.arpa" {
    type slave;
    file "/etc/bind/db.57.168.192";
    masters { 192.168.57.103; };
};
```

## Verificación y pruebas
### 1. Comprobación de la Resolución de Nombres
Para verificar que los registros DNS están correctamente configurados, usa el comando dig:
```bash
dig @localhost sistema.test A
dig @localhost sistema.test NS
```

Debes obtener los registros de los servidores:
```bash
sistema.test.   86400   IN  NS  tierra.sistema.test.
sistema.test.   86400   IN  NS  venus.sistema.test.
```

### 2. Comprobación de la Transferencia de Zonas
Para comprobar que la transferencia de zonas se realiza correctamente, ejecuta el siguiente comando desde el servidor esclavo:
```bash
dig @192.168.57.103 sistema.test AXFR
```

El resultado debe mostrar todos los registros de la zona sistema.test, confirmando que se ha realizado correctamente la transferencia.

![AXFR-result](IMG/AXFR-result.png)

### 3. Verificación con nslookup
Utiliza el comando nslookup para verificar que los alias y las resoluciones inversas funcionan correctamente:
```bash
nslookup ns1.sistema.test
nslookup 192.168.57.103
```

### 4. Logs del Servidor DNS
Revisa los logs de BIND para verificar si ha habido problemas con la transferencia de zona:
```bash
sudo journalctl -xe | grep named
```

### 5. Verificación con test.sh
![AXFR-result](IMG//test-result.png)