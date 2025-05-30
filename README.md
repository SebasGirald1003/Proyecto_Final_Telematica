# Proyecto Final – Diseño e Implementación de un Servidor DNS con BIND en AWS

## Integrantes:

  - Santiago Álvarez Peña.
  - Sebastián Giraldo Álvarez.

## Capturas

https://github.com/SebasGirald1003/Proyecto_Final_Telematica/wiki/Evidencias-Fotos

## Video

link: https://youtu.be/Zh_8MFPQPZU

## Implementación de un Servidor DNS con BIND9 y Cliente en Ubuntu

### Introducción

En el marco del curso de redes, se nos planteó el diseño e implementación de un servidor DNS funcional utilizando el servicio BIND9 en un entorno virtualizado. Para este proyecto seleccionamos Amazon Web Services (AWS) como plataforma de despliegue, lo que nos permitió simular un entorno realista de nube y prácticas empresariales.

El objetivo principal fue comprender el funcionamiento interno de la resolución de nombres mediante DNS, configurando el servidor para resolver tanto dominios internos personalizados como externos, mediante el uso de reenviadores (forwarders). Asimismo, se configuró un cliente Ubuntu que consume el servicio DNS desde la misma VPC.

### Desarrollo

La solución se desarrolló en una instancia EC2 de AWS con Ubuntu Server 22.04. El proceso incluyó los siguientes pasos:

#### Entorno de trabajo

* Se creó una instancia EC2 con Ubuntu Server.
* Se configuró una VPC con subred pública.
* Se habilitaron los puertos 22 (SSH) y 53 (TCP/UDP) en el grupo de seguridad.
* Se asignó una IP pública para acceso remoto y una IP privada para pruebas internas entre cliente y servidor.

#### Instalación de BIND9 en el servidor

```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc dnsutils -y
```

#### Configuración del archivo `/etc/bind/named.conf.options`

Se modificó para permitir recursión y redirección de consultas a DNS externos:

```conf
options {
    directory "/var/cache/bind";

    recursion yes;
    allow-recursion { 127.0.0.1; 172.31.0.0/16; };
    allow-query { any; };

    forwarders {
        8.8.8.8;
        8.8.4.4;
        1.1.1.1;
    };

    dnssec-validation auto;
    listen-on-v6 { any; };
};
```

Se verificó la sintaxis y se reinició el servicio:

```bash
sudo named-checkconf
sudo systemctl restart bind9
```

#### Configuración del cliente

En la máquina cliente Ubuntu se modificó el archivo `/etc/resolv.conf` para que usara exclusivamente al servidor DNS configurado:

```bash
sudo rm /etc/resolv.conf
sudo nano /etc/resolv.conf
```

Contenido:

```
nameserver 172.31.88.182
options edns0 trust-ad
search .
```

Para evitar que el sistema lo sobreescriba:

```bash
sudo chattr +i /etc/resolv.conf
```

Adicionalmente se configuró correctamente el archivo `/etc/hosts`:

```
127.0.0.1 localhost
127.0.1.1 ip-172-31-83-68
```

#### Pruebas de funcionamiento

Desde el cliente se realizaron consultas exitosas:

```bash
dig google.com
dig host1.grupox.local
dig @172.31.88.182 google.com
dig @172.31.88.182 host1.grupox.local
```

### ¿Cómo el servidor DNS direcciona a cualquier dirección de Internet?

Cuando el cliente necesita acceder a un dominio como google.com, consulta al servidor DNS configurado. Este proceso ocurre de la siguiente manera:

1. El cliente envía la consulta al servidor DNS.
2. BIND9 revisa si el dominio está en sus zonas internas.
3. Si no lo encuentra, utiliza los forwarders configurados:

```conf
forwarders {
    8.8.8.8;
    8.8.4.4;
    1.1.1.1;
};
```

4. El servidor forwarder responde con la IP correspondiente.
5. El servidor DNS guarda la respuesta en caché y se la devuelve al cliente.

Este mecanismo permite que el servidor resuelva nombres tanto internos como externos.

### Aspectos logrados y no logrados

| Aspecto                               | Estado     | Detalles                                          |
| ------------------------------------- | ---------- | ------------------------------------------------- |
| Resolución de nombres internos        | Logrado    | Se configuró correctamente una zona personalizada |
| Redirección a internet vía forwarders | Logrado    | El cliente resolvió dominios como google.com      |
| Seguridad adicional (chroot)          | No logrado | No se configuró por falta de tiempo               |
| Alta disponibilidad                   | No logrado | Solo se configuró un servidor primario            |

### Conclusiones

Este proyecto permitió comprender cómo funciona un servidor DNS autoritativo con capacidad recursiva. Se logró configurar satisfactoriamente tanto la resolución de dominios internos como la redirección a servidores externos mediante forwarders. Además, se desarrolló experiencia en el manejo de infraestructura en la nube usando AWS.

El conocimiento adquirido permite implementar soluciones DNS corporativas básicas, y resalta la importancia de la correcta configuración y validación en servicios críticos de red como DNS.



