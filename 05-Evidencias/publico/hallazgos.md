# Hallazgos verificados — LAB-018

- Nmap detectó 80/tcp nginx 1.16.1, 6498/tcp OpenSSH 7.6p1 y 65524/tcp Apache 2.4.43.
- robots.txt y Gobuster permitieron localizar rutas ocultas en ambos sitios HTTP.
- /hidden/whatever/ contenía una cadena Base64 dentro de un elemento HTML oculto.
- El robots.txt de Apache reveló una pista de User-Agent y la referencia a easypeasy.txt.
- La página Apache contenía una cadena Base62 que conducía a /n0th1ng3ls3m4tt3r.
- Steghide extrajo secrettext.txt de binarycodepixabay.jpg.
- El usuario boring accedió por SSH en el puerto 6498.
- user.txt estaba codificado con ROT13.
- Cron ejecutaba como root un script modificable por boring.
- El Bash SUID permitió obtener euid=0 y leer /root/.root.txt.
- Las cinco flags se recuperaron; sus valores y las credenciales no se publican.
- No se confirmó una CVE.
