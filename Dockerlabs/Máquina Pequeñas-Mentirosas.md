Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh pequenas-mentirosas.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/1b86dc50-189a-447f-b579-a8854c7d4d6f)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/9c41a369-0b34-4904-abb1-d62591bb4de0)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP***.

Buscamos en el navegador [http://172.17.0.2](http://172.17.0.2) para visualizar el contenido del servidor web:

  ![image](https://github.com/user-attachments/assets/f0ac0871-437f-4713-9f65-452eebf8d9e4)

Parece ser que según la pista, existe un usuario llamado "**a**" en la máquina víctima.

Probamos a hacer un ataque de fuerza bruta Hydra para obtener la contraseña de este posible usuario:

  ![image](https://github.com/user-attachments/assets/c63829a9-2563-4b9e-8692-1687e5260482)

La contraseña del usuario "a" es "***secret***".

Nos conectamos a la máquina por el puerto SSH:

  ![image](https://github.com/user-attachments/assets/b7b35125-21dd-4738-81a2-ffe3a0042e80)

Hemos logrado acceder al sistema.

Ahora, revisamos si hay más usuarios existentes en la máquina vulnerable.

  ![image](https://github.com/user-attachments/assets/08a189ef-a475-45ea-9d4e-e20056fb8eb6)

Existe otro usuario llamado "**spencer**".

Luego, vamos a realizar de nuevo un ataque de fuerza bruta para obtener la contraseña del usuario.

  ![image](https://github.com/user-attachments/assets/c7162758-7bb7-428e-91d6-38bb9479c086)

La contraseña del usuario "spencer" es "***password1***".

Accedemos con el comando **su spencer**:

  ![image](https://github.com/user-attachments/assets/851756a3-f127-48a9-88b5-adfe96dcb7df)

Revisamos con el comando **sudo -l** los archivos binarios que puede ejecutar el usuario de manera privilegiada.
Podemos ejecutar el binario */usr/bin/python3* como usuario root.

Lo ejecutamos con este comando --> **sudo python3 -c 'import os; os.system("/bin/sh")'**

  ![image](https://github.com/user-attachments/assets/9e463ba7-cf9f-4323-ba08-fe82f1f789d9)

Finalmente, hemos conseguido acceder al sistema como el usuario root, por lo que ahora contamos con todos los privilegios existentes.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/c46a0e93-af90-4741-a50e-47fd1af6a9f9)

