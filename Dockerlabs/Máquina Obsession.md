Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh obsession.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2.</ins>

  ![image](https://github.com/user-attachments/assets/fd17958f-5233-4e8b-bfef-073509d8eb50)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/3eba5a68-7145-4324-9456-442c7b1014ad)

Nos encontramos con una serie de puertos abiertos, como los puertos ***21***, ***22*** y ***80***, correspondientes a los protocolos ***FTP***, ***SSH*** y ***HTTP***, respectivamente.

Nos dirigimos al navegador y en el buscador ponemos http://172.17.0.2:80.

  ![image](https://github.com/user-attachments/assets/b7487d17-c2e9-4086-ac1e-b1708c7629f1)

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información.
Usamos el comando ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*js* y .*txt*.

  ![image](https://github.com/user-attachments/assets/a972fe5d-5e0c-42f8-9388-af3891930e30)

Encontramos dos subdirectorios web bastante interesantes, llamados **backup** e **important**.

Visualizamos el contenido del subdirectorio backup y nos encontramos con un archivo de texto denominado **backup.txt**, con este contenido:

  ![image](https://github.com/user-attachments/assets/e66eecf9-423f-4f8f-8c6f-037d53de4b89)

Hemos obtenido un posible nombre de usuario de la máquina víctima, por lo que podemos realizar un ataque Hydra para obtener la contraseña de este usuario.

Realizamos un ataque de fuerza bruta para obtener la contraseña del servicio SSH del usuario **russoski**:

  ![image](https://github.com/user-attachments/assets/16dc1f21-b02d-4f5d-92c3-9dcaced68833)

La contraseña del usuario russoski es "**iloveme**". Con esta clave podremos acceder a la máquina mediante SSH.

Nos conectamos por SSH:

  ![image](https://github.com/user-attachments/assets/f8e81678-753d-4fd4-b06e-40b5858513bf)

Hemos accedido a la máquina.

Ahora, visualizamos con el comando **sudo -l** los archivos binarios que puede ejecutar el usuario. En este caso, el usuario "russoski" puede ejecutar el binario /usr/bin/vim como usuario root, sin necesidad de ingresar ninguna contraseña en la CLI.

  ![image](https://github.com/user-attachments/assets/1f2f9aad-9ea9-447a-b249-2180e270e034)

Miramos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/usr/bin/vim***.

  ![image](https://github.com/user-attachments/assets/e3109543-0a32-4483-a012-fc50d964ec31)

**sudo vim -c ':!/bin/sh'**

  ![image](https://github.com/user-attachments/assets/6e4a22d1-b3dc-40f7-97ff-aa6f616a2d0c)

Hemos realizado la escalada de privilegios de manera exitosa, ya que ahora somos el usuario root y contamos con todos los privilegios existentes en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/33c8409d-e82a-4c4d-9a96-0fde3120a3e7)
