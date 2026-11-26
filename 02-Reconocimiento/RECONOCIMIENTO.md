# Reconocimiento — LAB-018 Easy Peasy

## Objetivo

Identificar los servicios abiertos y enumerar los dos servidores web visibles en el objetivo autorizado.

## Escaneo inicial

Comando empleado:

    nmap --top-ports 100 -sV -sC -oN nmap100.txt 10.66.129.175

Resultado relevante: host activo; 80/tcp abierto con nginx 1.16.1; título “Welcome to nginx!”; robots.txt encontrado. Los otros 99 puertos consultados aparecieron cerrados.

## Escaneo TCP completo

Comando empleado:

    nmap -p- --open -sV -oN nmap-completo.txt 10.66.129.175

El escaneo encontró tres puertos TCP abiertos:

| Puerto | Servicio | Resultado |
|---|---|---|
| 80/tcp | HTTP | nginx 1.16.1 |
| 6498/tcp | SSH | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 |
| 65524/tcp | HTTP | Apache httpd 2.4.43 (Ubuntu) |

Un segundo escaneo con -sC -sV -T5 confirmó los mismos puertos. Nmap avisó que algunos puertos podían quedar filtrados por el límite de retransmisiones; el inventario no mostró otros puertos abiertos.

## Web 80 — nginx

La raíz devolvió HTTP 200 y la página por defecto. robots.txt respondía HTTP 200 y contenía una política de disallow para toda la raíz. Gobuster con common.txt encontró:

- /hidden/ — 301 a /hidden/
- /index.html — 200
- /robots.txt — 200

La enumeración de /hidden/ encontró /whatever/ y /index.html. /hidden/whatever/ contenía una cadena Base64 dentro de un párrafo oculto en el HTML.

## Web 65524 — Apache

Gobuster con common.txt encontró:

- /index.html — 200
- /robots.txt — 200
- /.htaccess, /.htpasswd, /.hta y /server-status — 403

El robots.txt tenía una regla para un User-Agent concreto y una pista sobre la lista easypeasy.txt. La raíz contenía una cadena Base62; su decodificación reveló el directorio:

    /n0th1ng3ls3m4tt3r/

Ese directorio exponía binarycodepixabay.jpg y un digest hexadecimal de 64 caracteres.

## Evidencias de escaneo

- [Escaneo inicial](nmap-inicial.txt)
- [Escaneo completo](nmap-completo.txt)
- [Gobuster puerto 80](gobuster-80.txt)
- [Gobuster dentro de /hidden](gobuster-hidden.txt)
- [Gobuster puerto 65524](gobuster-65524.txt)
