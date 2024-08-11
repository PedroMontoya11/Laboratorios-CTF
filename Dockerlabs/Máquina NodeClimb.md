Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh nodeclimb.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/76ba850e-470d-4ac7-a2f8-3ff2632383a7)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/d7cf9ae5-0234-4340-b3c2-84aa66bacb2b)

Podemos ver que se encuentran abiertos los puertos ***21*** y ***22***, correspondientes a los servicios ***FTP*** y ***SSH*** respectivamente.

Luego, sabiendo que el puerto HTTP se encuentra cerrado, intentaremos obtener información mediante FTP:

  ![image](https://github.com/user-attachments/assets/a30f42ab-6cb1-4842-9de2-4141564bb96c)

Nos encontramos con un archivo .*zip* bastante interesante llamado "**secretitopicaron.zip**", por lo que vamos a usar el comando **get** para obtenerlo y así poder analizarlo en nuestro sistema, en busca de obtener información de algún usuario de la máquina víctima.

  ![image](https://github.com/user-attachments/assets/2d78b1d3-a062-4214-b833-09f1b97b2225)

Intentamos extraer el contenido del zip:

  ![image](https://github.com/user-attachments/assets/74c434f1-6d00-49d4-a869-34d2ccc3d7c4)

El archivo se encuentra cifrado y necesitamos una clave para descifrarla. Usamos la herramienta de Kali Linux **zip2john** para obtener el hash del archivo; para después, este hash desencriptarlo con **John the Ripper**.

  ![image](https://github.com/user-attachments/assets/3d5e3996-e7f8-40ad-a82a-d7a5fec3d959)

  ![image](https://github.com/user-attachments/assets/05fac0c6-b63f-4018-abfe-d217ab92bfcc)

Hemos obtenido la contraseña necesaria para extraer el contenido del zip, la cual es "**password1**".

Ahora si ejecutamos el comando **unzip** en la CLI:

  ![image](https://github.com/user-attachments/assets/a699c9fa-d41e-41cc-a9eb-e9fb5650189e)

Parece que hemos obtenido las credenciales de un posible usuario de la máquina vulnerable. Vamos a emplearlas en el SSH para conectarnos a ella.

Nos conectamos por SSH:

  ![image](https://github.com/user-attachments/assets/5fb04f8a-447a-4a3c-a00f-1a1e54ff9466)

Al ejecutar el comando **sudo -l** para visualizar los archivos binarios que puede ejecutar el usuario mario, vemos que este usuario puede ejecutar el binario /usr/bin/node y un script de JavaScript situado en la ruta /home/mario/script.js como usuario privilegiado, pudiendo así realizar una escalada de privilegios a usuario root.

  ![image](https://github.com/user-attachments/assets/4df078fd-6b07-4947-8af5-a1e3239e19d2)

El script se encuentra vacío, por lo que vamos a editarlo, añadiéndole la siguiente línea de código:<br>
**require('child_process').spawn('/bin/sh', { stdio: [0, 1, 2] });**

  ![image](https://github.com/user-attachments/assets/0dd5ab6a-ae5d-49af-b44f-4e89affc8ac4)

Finalmente, escribimos en la terminal **sudo /usr/bin/node /home/mario/script.js** para realizar la escalada de privilegios.

  ![image](https://github.com/user-attachments/assets/02270b3c-9766-4621-ae18-133de29437da)

Hemos logrado acceder como root, por lo que contamos con todos los permisos disponibles en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/e99ac85d-0a10-498a-974b-95af1845a008)
