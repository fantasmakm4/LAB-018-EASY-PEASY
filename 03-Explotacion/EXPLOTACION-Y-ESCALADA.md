# Explotación y escalada — LAB-018 Easy Peasy

## Alcance

Procedimiento realizado únicamente contra la máquina asignada por TryHackMe en 10.66.129.175.

## 1. Primera flag web

La enumeración de /hidden/whatever/ mostró en el HTML un párrafo marcado como hidden. Su contenido estaba en Base64. La decodificación entregó la primera flag.

    printf '%s\n' 'CADENA_BASE64' | base64 -d

## 2. Pistas del segundo servidor web

Apache en 65524 publicó un robots.txt con una identidad User-Agent específica y una referencia a easypeasy.txt. HashID propuso varios formatos de 128 bits, incluido MD5. El texto del sitio también contenía una cadena Base62 que condujo a /n0th1ng3ls3m4tt3r.

Las pruebas de John no llegaron a cargar easypeasy.txt: la salida mostrada fue “No such file or directory” para ~/Downloads/easypeasy.txt. En el repositorio no se declara un crack de John como exitoso.

## 3. Imagen y secreto

Se descargó binarycodepixabay.jpg desde el directorio indicado. La extracción de steghide produjo secrettext.txt. La secuencia binaria del archivo se convirtió con Python 3 a texto legible. El resultado proporcionó el usuario boring y la contraseña SSH.

## 4. Acceso SSH y user flag

    ssh -p 6498 boring@10.66.129.175

La sesión permitió leer ~/user.txt. El texto usaba ROT13; la transformación recuperó la user flag.

## 5. Escalada con cron

/etc/crontab mostró una tarea root cada minuto que ejecutaba /var/www/.mysecretcronjob.sh. El archivo era propiedad de boring y el usuario podía modificarlo. Se añadieron estas órdenes:

    cp /bin/bash /tmp/rootbash
    chmod u+s /tmp/rootbash

Cuando cron ejecutó el script, /tmp/rootbash apareció con propietario root y bit SUID. Se inició así:

    /tmp/rootbash -p

id confirmó uid=1000(boring), euid=0(root). La shell conservó privilegios efectivos root.

## 6. Root flag

El archivo /root/root.txt no existía. El archivo correcto era /root/.root.txt; se leyó desde la Bash SUID.

## Resultado

- Acceso SSH como boring: conseguido.
- User flag: recuperada mediante ROT13.
- Bash con euid=0: conseguido.
- Root flag en /root/.root.txt: leída.
- Máquina: completada.
