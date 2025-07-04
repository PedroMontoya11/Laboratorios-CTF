1. Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh apibase.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
   <ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

    ![image](https://github.com/user-attachments/assets/7edd639b-1dda-41b4-9e79-ad16353c978c)

2. Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

    ![image](https://github.com/user-attachments/assets/b47f9e6c-440a-4184-ab15-ea90554b4421)

    Podemos ver que se encuentra abierto el puerto ***22*** y el puerto ***5000***, correspondientes al servicio ***SSH*** y ***UPnP***.

3. Vamos al navegador y visualizamos el contenido que aparece si escribimos en el buscador http://172.17.0.2:5000:

    ![image](https://github.com/user-attachments/assets/0f44981b-170d-4cb7-9fbe-799f163e66dd)

    Al parecer se trata de una página web que usa el protocolo GET como método de solicitud HTTP, por lo que ésta seguramente espera recibir un parámetro específico junto con una clave.

4. Para saber de qué parámetro se trata, usamos el comando **wfuzz** de la siguiente manera:

    ![image](https://github.com/user-attachments/assets/382ab74d-87b2-4f02-b729-13523d8d5ba5)

    Aparentemente, la dirección http://172.17.0.2:5000/users puede recibir el parámetro "*users*".

5. Volvemos a realizar fuzzing con una lista de nombres de usuarios para conocer el usuario concreto:

    ![image](https://github.com/user-attachments/assets/87d4d16a-8287-49bf-9d4b-379dfa59d64f)

    Encontramos una coincidencia con "*d'anne*", por lo que revisamos que nos aparece en la web al añadir esta nueva información en el buscador:

    ![image](https://github.com/user-attachments/assets/4fdfa156-5d23-41df-8105-d223d76deadd)

    Parece ser que hemos descubierto una biblioteca de software llamada SQLite3; relacionada con el servicio SQL, por lo que podríamos comprobar si el servicio es resistente o no a **ataques de inyección SQL** (*SQL Injection*).

6. Le pasamos la siguiente línea al buscador, detrás de http://172.17.0.2:5000/users?username=:

    **admin' OR 1=1 -- -**

    ![image](https://github.com/user-attachments/assets/5d7796c2-46d0-42fd-8ce6-9d0a3a5e81a3)

    Efectivamente, es vulnerable a la inyección de código SQL, por lo que hemos sido capaces de obtener los usuarios con sus contraseñas.

7. Nos conectamos por SSH a la máquina víctima:

   ![image](https://github.com/user-attachments/assets/0965e0be-c4db-4542-8864-333b035d6d40)

   Hemos accedido al sistema como el usuario "**pingu**".<br>
    Haciendo uso de comandos como ***sudo -l*** y ***find / -perm –4000 2>/dev/null*** nos percatamos de que este usuario no cuenta con privilegios sobre archivos binarios que podamos explotar, así que nos tocará revisar directorios en busca de información privilegiada que nos ayude a escalar privilegios.

8. En el directorio *home* existe un fichero llamado "**network.pcap**" cuyo contenido es este:

    ![image](https://github.com/user-attachments/assets/5fa3a398-d7d2-4635-9c4a-1d13fb6fc5de)

    Parece ser que la contraseña del usuario "**root**" es "**balulero**", por lo que ya simplemente tendríamos que probar esta clave y habríamos finalizado el laboratorio.

   ![image](https://github.com/user-attachments/assets/e42d1c1c-4e85-4d4e-9726-28f146f58dd1)

   <ins>**Finalmente, hemos conseguido acceder al sistema como el usuario "root", por lo que tenemos todos los privilegios existentes en el sistema**</ins>.

9. Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

    ![image](https://github.com/user-attachments/assets/6ff4d35f-c8c9-4fbf-8ca2-3ca8cbac0931)
