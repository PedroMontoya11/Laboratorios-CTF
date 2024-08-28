Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh psycho.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/7d7deb0e-96ea-492c-9243-a2e76f6fb3d4)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/19de82ab-1383-4e51-9c98-1d16386afa1f)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP***, respectivamente.

Nos dirigimos al navegador y en el buscador ponemos http://172.17.0.2:80.

  ![image](https://github.com/user-attachments/assets/bf7f5e8c-7fcf-441b-84b1-bef83afb3b40)

  ![image](https://github.com/user-attachments/assets/d2fd0476-bd69-4cab-9a3b-a69fe0d5ebff)

Vemos que en el código fuente de la página de inicio se encuentra un mensaje de error y que el index es un archivo PHP. Por lo tanto, podemos suponer que se está realizando una llamada mediante una solicitud GET de manera incorrecta.

Por ello, realizaremos fuzzing para observar si identificamos un parámetro. Usamos wfuzz y podemos observar que el parámetro obtenido es "**secret**".

  ![image](https://github.com/user-attachments/assets/fffb29ec-b9ef-4472-8380-86b088cc0435)

Al dirigirnos a la ruta http://172.17.0.2/index.php?secret=../../../../../../etc/passwd vemos lo siguiente:

  ![image](https://github.com/user-attachments/assets/6bd55324-9929-41da-b01e-cdc60dbc0a69)

La página web es vulnerable a **Path Traversal**, que es un tipo de ciberataque en el que al escribir una ruta de directorios del servidor se muestra la salida en la web, pudiendo así obtener información al contenido de los archivos del servidor web.<br>En este caso, al escribir la ruta */etc/passwd* junto con la clave obtenida anteriormente, obtenemos los usuarios existentes en el sistema.

Tenemos dos usuarios con los que podemos probar a acceder a la máquina víctima, el usuario "**vaxei**" y el usuario "**luisillo**".

Intentamos obtener las credenciales mediante Hydra:

  ![image](https://github.com/user-attachments/assets/76283207-9169-4cc3-ba61-24bded239049)

  ![image](https://github.com/user-attachments/assets/c8df80c4-e2c8-4157-89da-7600607e9760)

Ya que no hemos tenido éxito, podemos aplicar la vulnerabilidad Path Traversal para obtener el id_rsa del SSH de los usuarios (estos no permite conectarnos al SSH sin necesidad de contraseña). El ID del servicio SSH se encuentra en la ruta ***/home/$USER/.ssh/id_rsa***.

  ![image](https://github.com/user-attachments/assets/ca48d8ac-230a-4b2d-aec2-04ff59096fcf)

El usuario "luisillo" no es un usuario SSH, ya que no tiene clave RSA. Probamos con el usuario "vaxei":

  ![image](https://github.com/user-attachments/assets/f77b679b-3ee3-4bae-8cdf-34f92958b661)

  ![image](https://github.com/user-attachments/assets/fe705ea6-36b5-4593-9c04-c35f002538ba)

Hemos obtenido la clave privada del SSH de "vaxei".

Para conectarnos de manera exitosa, creamos un archivo que cuente con la clave encontrada y escribimos **ssh -i id_rsa** [***Nombre del archivo creado con el ID***] **vaxei@172.17.0.2**.

  ![image](https://github.com/user-attachments/assets/8173bf3d-1302-49ce-b260-c85d9fd9d13e)

Hemos accedido al sistema como el usuario "vaxei".

Miramos con el comando **sudo -l** los archivos binarios que puede ejecutar el usuario actual de manera privilegiada.

  ![image](https://github.com/user-attachments/assets/c913e219-593c-4d1b-8fc5-85f2379da19c)

El usuario puede ejecutar el binario /usr/bin/perl para acceder como el usuario luisillo, sin necesidad de insertar su contraseña en la CLI.

Buscamos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/usr/bin/perl***.

  ![image](https://github.com/user-attachments/assets/c7214192-9466-4c48-a474-d3922ae138d4)

**sudo -u luisillo /usr/bin/perl -e 'exec "/bin/sh";'**

  ![image](https://github.com/user-attachments/assets/226a6b80-c6d7-4a0a-bd7a-889a80c26bb6)

Realizamos otra escalada de privilegios mediante archivos binarios que podamos ejecutar de manera privilegiada.

  ![image](https://github.com/user-attachments/assets/78cace71-2405-407d-8af5-015b08393121)

El usuario "luisillo" es capaz de ejecutar el binario /usr/bin/python3 y el script /opt/paw.py como usuario root, sin necesidad de insertar ninguna contraseña.

Modificamos el contenido del script de la siguiente manera:

  ![image](https://github.com/user-attachments/assets/d64bd26c-f1df-4dbc-ab33-9c6bcdf0a96f)

Luego, escribimos **sudo /usr/bin/python3 /opt/paw.py** en la terminal y lo ejecutamos:

  ![image](https://github.com/user-attachments/assets/4531cf0b-4da0-45d6-84f6-f9c9f6065b5e)

  ![image](https://github.com/user-attachments/assets/af059aef-8852-40e3-b7c2-63d114af2ee0)

Finalmente hemos logrado acceder como usuario root, por lo que contamos con todos los privilegios disponibles en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/6553eccf-01fa-4e91-bffb-1e2835bd86e1)
