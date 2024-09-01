Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh pntopntobarra.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/986c5199-b067-4091-9ecd-d4040a873c0f)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/7bf34cf9-ec9d-43cf-b96c-1c3aca6a5d3f)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP***, respectivamente.

Nos dirigimos al navegador y en el buscador ponemos http://172.17.0.2:80.

  ![image](https://github.com/user-attachments/assets/848b1f81-08f7-4179-9410-dc53c171f8a9)

Nos sale un mensaje advirtiendo de que el sistema de la máquina víctima se encuentra en peligro por un virus llamado "LeFvIrus".<br>
Viendo las letras que aparecen en mayúsculas, podemos comprender que el tipo de ataque que realizaremos será un **ataque LFI** (*Local File Inclusion*).

Hacemos clic en el botón que pone "**Ejemplos de computadoras infectadas**".

  ![image](https://github.com/user-attachments/assets/8a44fad3-d433-4988-82ed-c99121ac1061)

Vemos que la dirección URL de la página web muestra una variable llamada "*images*" y seguido de esta, hay una ruta local del servidor (**./ejemplo1.png**).<br>
Sabiendo esto, podemos modificarla para realizar el ataque LFI, obteniendo acceso al contenido de archivos locales desde el servidor web.<br>
Escribimos /etc/passwd y vemos que se nos muestra:

  ![image](https://github.com/user-attachments/assets/7b10d969-83fe-4bdf-b005-24c8473fc372)

Hay un usuario llamado "**nico**", así que lo que haremos será acceder al contenido del archivo que cuenta con el id_rsa de este posible usuario SSH, ya que este puerto se encuentra abierto.

Vamos a la página http://172.17.0.2/ejemplos.php?images=/home/nico/.ssh/id_rsa y miramos su código fuente:

  ![image](https://github.com/user-attachments/assets/625a5dd3-0db9-41e3-a448-90770d29c518)

Copiamos el id_rsa que nos aparece.

Ahora, creamos un archivo que contenga el ID cifrado del usuario SSH.

  ![image](https://github.com/user-attachments/assets/04a5bdc7-2dd7-4647-8a1f-16395d691e30)

  ![image](https://github.com/user-attachments/assets/205eda70-bd0f-450e-ad63-6787f31e7053)

Para conectarnos al servicio SSH de manera exitosa, escribimos **ssh -i id_rsa** [***Nombre del archivo creado con el ID***] **nico@172.17.0.2**.

  ![image](https://github.com/user-attachments/assets/7473b920-20d9-4dd3-95fa-4d7738f5bf07)

Miramos con el comando **sudo -l** los archivos binarios que puede ejecutar el usuario "nico" de manera privilegiada.

  ![image](https://github.com/user-attachments/assets/61d85585-89cb-44cf-b30c-4f1b4e4b6afe)

Podemos ejecutar el binario /bin/env como usuario root, sin necesidad de ingresar ninguna contraseña en la CLI.

Buscamos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/bin/env***.

  ![image](https://github.com/user-attachments/assets/2e62111a-bd3c-4254-9ffc-f5caf04a03ad)
  
**sudo env /bin/sh**

  ![image](https://github.com/user-attachments/assets/7e4d43c0-1d78-4662-ac6c-770586448f89)

Finalmente, hemos logrado acceder al sistema como usuario root, por lo que ahora contamos con todos los privilegios existentes en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/dfa09592-015c-4497-be49-3903b67b7773)
