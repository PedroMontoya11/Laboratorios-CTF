Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh consolelog.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>En este caso, la dirección IP de la máquina a vulnerar es la 172.17.0.2.</ins>

  ![image](https://github.com/user-attachments/assets/f9e51544-4137-4530-9ffe-abef1d9ca288)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/96315beb-f191-4e64-9908-2189a63b329d)

Podemos ver que se encuentran abiertos los puertos ***80***, ***3000*** y ***5000*** correspondiente al servicio ***HTTP***, ***Node.js*** y ***SSH*** (en este caso, ya que por defecto es el puerto 22).

Nos dirigimos al navegador y escribimos la dirección URL http://172.17.0.2:80:

  ![image](https://github.com/user-attachments/assets/6d367e77-e196-476f-8c77-1099030d5377)

Miramos el código fuente de la página web en busca de información adicional:

  ![image](https://github.com/user-attachments/assets/a22d1918-aa1f-400f-a4ca-112d6928ccd6)

Vemos que en el apartado de **<script>** se encuentra un archivo llamado "**authentication.js**" y cuyo contenido es este:

  ![image](https://github.com/user-attachments/assets/66ce653e-5434-4609-839b-a7066d0b9103)

Aparentemente, existe un token válido para el subdirectorio web "/recurso/" que es "tokentraviesito".

Realizamos un escaneo de directorios mediante el comando **gobuster** y nos encontramos con uno que se llama **backend**.

  ![image](https://github.com/user-attachments/assets/cd4bfa16-be66-4c51-9691-451f2103599a)

  ![image](https://github.com/user-attachments/assets/bc4c19e7-f3be-4392-92ac-fc583b8bf14f)

Hacemos clic en "**server.js**" para visualizar el contenido de este fichero:

  ![image](https://github.com/user-attachments/assets/07fd4bc6-8805-4a8c-abcf-4ea49426d5f1)

Nos damos cuenta de que parece ser que este servidor escucha peticiones POST en la ruta /recurso/ y devuelve una contraseña si el token proporcionado es correcto.

Podemos probar esto usando curl, enviando una solicitud POST con el token encontrado, para visualizar que nos devuelve la salida.

  ![image](https://github.com/user-attachments/assets/8e296d5b-8a6d-46ef-a0ac-af4aa7f1495e)

Al enviar el token correcto obtenemos una contraseña. Veamos que ocurre en caso de que el token sea incorrecto:

  ![image](https://github.com/user-attachments/assets/5425c607-c02c-4516-b8a8-7292e7f6eb47)

Obtenemos el mensaje de "Unauthorized", ya que no hemos ingresado la clave correcta para obtener acceso al servidor Node.js.

Con la contraseña que tenemos intentaremos realizar un ataque Hydra para obtener el nombre de usuario del SSH.

  ![image](https://github.com/user-attachments/assets/2566f455-a7c8-483e-acbb-14a9f7b179d9)

El usuario cuya contraseña es "**lapassworddebackupmaschingonadetodas**" es "**lovely**".

Nos conectamos por SSH desde el puerto 5000:

  ![image](https://github.com/user-attachments/assets/16a2f7b8-12fc-4451-81db-df1874934d8d)

Hemos logrado acceder a la máquina.

Miramos en el archivo /etc/passwd si existe otro usuario con el que tengamos que realizar la escalada de privilegios.

  ![image](https://github.com/user-attachments/assets/c4814486-1db1-471b-8200-a25af6c495d7)

Comprobamos los archivos binarios que podemos ejecutar como el usuario lovely:

  ![image](https://github.com/user-attachments/assets/dca51a27-1add-429a-9621-9008974ef641)

Tenemos la capacidad de interactuar como usuario root con el comando **nano**, por lo que podemos editar el archivo passwd y quitarle la x al campo de texto del usuario root para poder acceder sin credenciales.

  ![image](https://github.com/user-attachments/assets/bb6b0df3-7480-456a-bf92-7c4b9c7491e2)

Finalmente, accedemos como el usuario root escribiendo en la terminal **su root**.

  ![image](https://github.com/user-attachments/assets/bc9606f4-bdd7-491b-b754-140fcab76a09)

Hemos accedido como el usuario root, por lo que ahora tenemos todos los privilegios existentes dentro del sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/3592457c-5759-4900-9260-2ecc2ffe0f91)
