# Proyecto Final – Diseño e Implementación de un Servidor DNS con BIND en AWS

## Introducción

En el marco del curso de redes, se nos planteó el diseño e implementación de un servidor DNS funcional utilizando el servicio BIND en un entorno virtualizado. Para este proyecto seleccionamos Amazon Web Services (AWS) como plataforma de despliegue, lo que nos permitió simular un entorno realista de nube y prácticas empresariales.

El objetivo principal fue comprender el funcionamiento interno de la resolución de nombres mediante DNS, aplicando una configuración personalizada con archivos de zona directa e inversa, asegurando su correcto funcionamiento a través de pruebas de consulta DNS.

## Desarrollo

La solución se desarrolló en una instancia EC2 de AWS, corriendo Ubuntu Server 22.04. El proceso se dividió en los siguientes pasos:

### Entorno de trabajo

- Se creó una instancia EC2 con Ubuntu.
- Se configuró una VPC con subred pública.
- Se habilitaron los puertos 22 (SSH) y 53 (TCP/UDP) en el grupo de seguridad.
- Se asignó una IP pública para acceso remoto y una IP privada para pruebas internas.

### Instalación de BIND

Accedimos por SSH y ejecutamos los siguientes comandos:

sudo apt update
sudo apt install bind9 bind9utils bind9-doc dnsutils -y


Esto instaló el servicio DNS y sus herramientas asociadas.

### Configuración del servidor DNS

Editamos el archivo `named.conf.local` para registrar nuestras zonas:

zone "grupox.local" {
type master;
file "/etc/bind/zones/db.grupox.local";
};

zone "1.168.192.in-addr.arpa" {
type master;
file "/etc/bind/zones/db.192.168.1";
};


Luego creamos los archivos:

- Zona directa (`db.grupox.local`): Incluye registros SOA, NS y A.
- Zona inversa (`db.192.168.1`): Incluye registros PTR y SOA.

También se aplicaron los permisos adecuados y se verificó la sintaxis con los siguientes comandos:

sudo named-checkconf
sudo named-checkzone grupox.local /etc/bind/zones/db.grupox.local
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/zones/db.192.168.1


### Activación del servicio

Se habilitó y reinició el servicio con los siguientes comandos:

sudo systemctl enable named
sudo systemctl restart named


### Validación

Realizamos consultas directas e inversas con la herramienta `dig`:

- `dig @127.0.0.1 host1.grupox.local` → 192.168.1.11
- `dig @127.0.0.1 -x 192.168.1.11` → host1.grupox.local

Ambas respuestas fueron exitosas.

### Consideración de seguridad: Ventajas del modo chroot en BIND

El uso del **modo chroot (modo jaula)** en el servicio BIND presenta las siguientes ventajas:

- **Mayor seguridad**: Aísla el servicio DNS en un entorno limitado del sistema de archivos, reduciendo el riesgo de que un atacante acceda a otros archivos del sistema si compromete el servicio.

- **Aislamiento del proceso**: BIND solo puede interactuar con los archivos dentro de su jaula, minimizando el impacto de errores o configuraciones maliciosas.

- **Entorno controlado**: Solo se incluyen los archivos necesarios, lo que reduce la superficie de ataque.

- **Facilita la auditoría**: Permite una mejor trazabilidad del comportamiento del servicio DNS.


## Problemas encontrados y solucionados

Durante la verificación del archivo de zona directa, recibimos el siguiente error:

dns_rdata_frontext: /etc/bind/zones/db.grupox.local:2: near '20250552801': out of range


Este error se debía a que el número de serie (Serial) en el registro SOA superaba el máximo permitido por el estándar DNS (4294967295). Inicialmente habíamos usado `20250552801`, pero este número tiene 11 dígitos.

La solución fue corregir el serial a un valor de 10 dígitos dentro del rango permitido, como `2025052801`.

Este problema fue importante porque impedía que el servidor cargara correctamente la zona.

## Logros

- Instalación y configuración completa de BIND9.
- Definición y activación de zonas directa e inversa.
- Validación exitosa con herramientas como `dig`.
- Documentación clara del proceso.
- Simulación de un entorno DNS interno empresarial.

## Aspectos no logrados

- No se implementó chroot por falta de tiempo, pero se investigaron sus ventajas: mejora la seguridad al aislar el servicio BIND del resto del sistema de archivos.

## Conclusiones

Este proyecto permitió entender en la práctica cómo opera un servidor DNS autoritativo y cómo se configuran manualmente sus zonas. Además, nos dio experiencia en el manejo de infraestructura en la nube mediante AWS, fortaleciendo habilidades técnicas en redes, seguridad y despliegue de servicios.

La resolución del problema con el número de serie nos mostró la importancia de los estándares en la configuración de servicios DNS.

Gracias a las pruebas realizadas, confirmamos que el servidor funciona correctamente y cumple con los objetivos planteados.


