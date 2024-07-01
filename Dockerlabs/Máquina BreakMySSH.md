Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh breakmyssh.tar** en la carpeta donde se encuentre el contenido extraído del zip.
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/c023f6a2-7280-4e47-8173-fa4efad23eab)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/f472a9c9-9521-45a6-8219-24ec7ef4bf83)

Podemos ver que se encuentra abierto el puerto ***22***, correspondiente al servicio ***SSH***.

Realizamos un ataque Hydra para obtener las claves del usuario "root" mediante fuerza bruta. Probamos con el nombre de usuario root y utilizamos un diccionario personalizado para obtener su contraseña (*passwords.txt*).

**hydra -l root -P passwords.txt ssh://172.17.0.2**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/04dd7c69-53a4-45a5-9551-f573736f8895)

Hemos obtenido que la contraseña para conectarnos por SSH a la máquina vulnerable con el usuario root es "estrella".

Realizamos la conexión por SSH para comprobar que las credenciales son correctas.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/68e6c3cf-9a47-4f5f-ac57-62706e13f396)

Estamos dentro de la máquina vulnerable como el usuario root, por lo que podemos realizar cualquier acción en el sistema, al ser root un usuario privilegiado.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/f6160c57-9995-4f53-8666-24e149fcc222)

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/b98b2359-18ee-4343-916f-dc76882195e7)
