Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh firsthacking.tar** en la carpeta donde se encuentre el contenido extraído del zip.
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/acca802b-6939-4136-9382-5b58b66ca862)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/db9e26d7-626e-47ef-a493-9a74b441f424)

Podemos ver que se encuentra abierto el puerto ***21***, correspondiente al servicio ***FTP***. El servidor FTP empleado por la máquina a atacar es el VSFTPD versión 2.3.4.

Atacaremos a este servicio buscando vulnerabilidades en la versión de VSFTPD. Una herramienta muy buena para practicar pentesting  y aprender de hacking ético es Metasploit Framework.
Lo ejecutamos mediante el comando **msfconsole** (***-q*** si no queremos que se muestre el banner del programa en la terminal).

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/4e9687c1-a60d-4b8a-994c-695d843704ce)

Estando dentro de la consola de Metasploit, tendremos que realizar la búsqueda de exploits a ejecutar en la máquina. Para ello, usamos el comando **search** seguido de una cadena de texto descriptiva; como por ejemplo, search vsftpd 2.3.4 (servicio y versión del servicio FTP).

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/a57e292a-da70-49a3-84de-3c14174395d6)

Hemos encontrado un exploit que podemos utilizar, el cual se encarga de realizar un **backdoor** (puerta trasera).
<ins>Un backdoor es un tipo de virus diseñado para permitir el acceso remoto de usuarios NO autorizados a un dispositivo</ins>.

Usamos ese módulo con **use 0**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/d35edda4-937f-415a-a6df-155e5d970fc3)

Ahora tocará editar la configuración del exploit para poder ejecutarlo. La revisamos con **options** o **show options** y en este caso, solo editaremos lo siguiente:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/9c8de702-951e-4cbc-a0d9-03708a334495)

**set RHOSTS 172.17.0.2** [IP de la máquina a comprometer]

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/13828a2a-eef7-4821-b90e-c10b4d11bec7)

También podemos añadirle un payload al exploit, por lo que buscamos los payloads disponibles para este módulo con **show payloads**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2d550159-560a-4f03-ba80-6dd1dff02f97)

Tenemos únicamente el payload con nombre *payload/cmd/unix/interact*. Lo seleccionamos escribiendo ***set payload*** [***número del #***] --> **set payload 0**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/c5ff588d-9191-4548-8a7d-6ccddb880604)

Finalmente, arrancamos el backdoor con el comando **run** (también es válido usar el comando **exploit**).

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/5d3cc5ae-b14c-4e18-bc51-9fa3920bcc2f)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/a4b7b400-b58e-42d7-8b62-610fd77380ad)

Hemos accedido a la máquina vulnerable mediante un backdoor de manera correcta, y además como el usuario "root", por lo que tenemos todos los permisos en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/fc296b4f-296d-41db-8865-9abd7680f788)
