# Informe técnico — LAB-018 Easy Peasy

## Ficha del laboratorio

- Plataforma: TryHackMe
- Máquina: Easy Peasy
- IP de sesión: 10.66.129.175
- Sistema atacante: Parrot Security OS
- Dificultad: Fácil
- Fecha de trabajo: 26/11/2026
- Resultado: user flag y root flag recuperadas
- Estado de privilegios: uid=1000(boring), euid=0(root)
- CVE: no confirmada

## Objetivo y alcance

Enumerar los servicios TCP de la máquina asignada, revisar los servidores web, recuperar las pistas publicadas, conseguir acceso de usuario por SSH y comprobar si la configuración local permitía una escalada de privilegios. Las acciones se limitaron al objetivo autorizado de TryHackMe.

## Resumen ejecutivo

Nmap encontró HTTP en 80/tcp, SSH en 6498/tcp y HTTP en 65524/tcp. El nginx del puerto 80 mostraba la página por defecto y un robots.txt que señalaba la raíz como no permitida. Gobuster localizó /hidden/ y, dentro, /whatever/. La página de esa ruta tenía un párrafo oculto con una cadena Base64 que correspondía a la primera flag.

El segundo servidor era Apache 2.4.43. Su robots.txt contenía una regla para un User-Agent concreto y una pista para usar la lista easypeasy.txt. La página también revelaba un texto codificado en Base62 que conducía a /n0th1ng3ls3m4tt3r. Allí se encontró una imagen y un digest de 64 caracteres. La imagen contenía secrettext.txt; el texto extraído identificaba al usuario boring y una contraseña representada como bytes binarios.

La conexión SSH se realizó en el puerto 6498. user.txt estaba transformado con ROT13. Para la escalada, /etc/crontab mostraba una tarea ejecutada cada minuto como root: /var/www/.mysecretcronjob.sh. El archivo era modificable por boring. Al añadir dos órdenes, cron creó /tmp/rootbash con SUID de root. La ejecución de /tmp/rootbash -p dio euid=0; se leyó la flag desde /root/.root.txt.

Los valores de flags y credenciales se mantienen en el archivo privado local, ignorado por Git.

## Servicios observados

| Puerto | Servicio | Versión / observación |
|---|---|---|
| 80/tcp | HTTP | nginx 1.16.1; página por defecto |
| 6498/tcp | SSH | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 |
| 65524/tcp | HTTP | Apache 2.4.43 (Ubuntu); página por defecto |

El escaneo inicial de los 100 puertos más comunes encontró el puerto 80. El escaneo TCP completo confirmó los tres puertos abiertos. Los resultados están resumidos en 02-Reconocimiento.

## Cronología y análisis

### 1. Nginx en el puerto 80

La respuesta de / mostró la página estándar “Welcome to nginx!”. robots.txt incluía:

- User-Agent: *
- Disallow: /
- El texto “Robots Not Allowed”

Gobuster encontró /hidden/, /index.html y /robots.txt. En /hidden/ se obtuvo otra página de bienvenida a CTF. Una segunda enumeración halló /hidden/whatever/. La respuesta redirigía a la ruta con barra final y mostraba un párrafo HTML oculto con una cadena Base64. La decodificación reveló la primera flag.

### 2. Apache en el puerto 65524

Gobuster encontró /index.html y /robots.txt; .htaccess, .htpasswd, .hta y /server-status devolvieron 403. El robots.txt tenía una regla para un User-Agent específico, junto con una frase que indicaba que solo esa identidad debía acceder. HashID mostró varias posibilidades para el identificador de 32 caracteres, incluida MD5.

La enumeración de la página y sus pistas condujo al directorio /n0th1ng3ls3m4tt3r. El contenido de la página incluía la tercera flag y la ruta de una imagen, además de un digest hexadecimal de 64 caracteres. No se conserva en el historial una salida exitosa de John para las listas: easypeasy.txt no estaba en ~/Downloads y John devolvió “No such file or directory”. La extracción de la imagen sí se verificó en el sistema.

### 3. Esteganografía y credenciales de boring

Se descargó binarycodepixabay.jpg desde el directorio oculto. Steghide extrajo secrettext.txt con la frase de extracción utilizada durante la sesión. El archivo contenía el nombre boring y una secuencia de bytes de 8 bits. Python 3 la convirtió a texto legible y proporcionó la contraseña de SSH.

### 4. Acceso SSH y user flag

La conexión al servicio se hizo con ssh -p 6498 boring@10.66.129.175. Tras un primer intento fallido, el acceso tuvo éxito. id confirmó uid=1000(boring). El archivo ~/user.txt contenía un texto con ROT13. tr lo transformó a la flag de usuario.

### 5. Escalada mediante cron

/etc/crontab contenía la tarea:

    * * * * * root cd /var/www/ && sudo bash .mysecretcronjob.sh

/var/www/.mysecretcronjob.sh pertenecía a boring, tenía permisos -rwxr-xr-x y contenía únicamente el shebang y un comentario. boring podía añadir órdenes al archivo. Se añadieron:

    cp /bin/bash /tmp/rootbash
    chmod u+s /tmp/rootbash

En el siguiente ciclo de cron, /tmp/rootbash apareció como propiedad de root con permisos SUID. La ejecución /tmp/rootbash -p conservó privilegios efectivos. id informó uid=1000(boring) y euid=0(root): esto demuestra privilegio efectivo root, aunque el uid real siguió siendo 1000.

### 6. Lectura de la flag root

/root/root.txt no existía. La flag estaba en el archivo oculto /root/.root.txt. El contenido se leyó con la Bash SUID.

## Hallazgos

- **H-01 — Información de rutas en robots.txt:** el archivo revelaba restricciones y pistas útiles para el descubrimiento.
- **H-02 — Directorios ocultos accesibles:** las rutas web expuestas proporcionaban contenido CTF, banderas y una imagen con datos embebidos.
- **H-03 — Script ejecutado como root modificable por un usuario:** el cron ejecutaba como root un archivo propiedad de boring, permitiendo crear un Bash SUID.
- **H-04 — Servicio SSH en puerto alternativo:** OpenSSH estaba publicado en 6498/tcp.
- **CVE:** no se confirmó una CVE; la ruta de escalada fue una configuración insegura de cron y permisos.

## Estado final

- Tres flags web identificadas.
- User flag recuperada después de ROT13.
- Root flag leída desde /root/.root.txt.
- Bash obtuvo euid=0 mediante el archivo SUID.
- Máquina completada.

## Recomendaciones

1. No utilizar robots.txt como mecanismo de control de acceso.
2. Retirar páginas por defecto y restringir los directorios que no deban ser públicos.
3. Mantener los scripts invocados por cron bajo propiedad de root y sin permisos de escritura para usuarios ordinarios.
4. Evitar usar sudo dentro de tareas root salvo que sea necesario; revisar el entorno y la ruta de ejecución.
5. Limitar SSH a cuentas y redes autorizadas; emplear autenticación robusta.
6. Supervisar cambios en archivos ejecutados automáticamente y auditar permisos SUID.

## Evidencias relacionadas

- [Reconocimiento y puertos](../02-Reconocimiento/RECONOCIMIENTO.md)
- [Explotación y escalada](../03-Explotacion/EXPLOTACION-Y-ESCALADA.md)
- [Comandos](../06-Comandos/COMANDOS.md)
- [Hallazgos públicos](../05-Evidencias/publico/hallazgos.md)
