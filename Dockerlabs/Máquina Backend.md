Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh backend.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/6b1a1ebc-d38c-4e45-9213-48bf0c141b0b)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/da2490b5-0f29-4576-a6e5-dd95e6202490)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP***.

Vamos a visualizar la página web. Para ello, escribimos http://172.17.0.2:80 en el navegador.

  ![image](https://github.com/user-attachments/assets/6d222bdf-61eb-4461-8a27-85c759c30a6f)

Parece que nos encontramos con una página que se encuentra en desarrollo, por lo que seguramente no sea funcional.<br>También nos encontramos con una página de inicio de sesión (*Login*), la cual no sirve para mucho, al encontrarse la web inacabada.

Ahora, probamos la herramienta **SQLMap**, que se encarga de detectar vulnerabilidades en el servicio SQL y explotarlas, pudiendo así obtener información sobre los datos existentes en las bases de datos.

Escribimos la siguiente línea de código: **sqlmap -u "http://172.17.0.2/login.html" --forms --batch --dbs**

  ![image](https://github.com/user-attachments/assets/ce373677-5a1c-46c9-a65a-59f7fc011b8f)

  ![image](https://github.com/user-attachments/assets/d30102e5-b6a7-4ec2-8f79-8292c166cd7e)

Obtenemos que la base de datos "**users**" cuenta con una única tabla, llamada "**usuarios**", que son "**paco**", "**pepe**" y "**juan**", cada uno con su respectiva contraseña.

Con las claves encontradas probamos a conectarnos por SSH. Al probar con el usuario "pepe", logramos acceder de manera exitosa a la máquina víctima.

  ![image](https://github.com/user-attachments/assets/f0b3e58f-d76d-4fcc-9117-fa17b96bc402)

Ahora somos el usuario "pepe". Vamos a intentar realizar una escalada de privilegios, para así lograr convertirnos en usuario *root*.

Para ello, revisamos los archivos binarios que puede ejecutar este usuario de manera privilegiada:

  ![image](https://github.com/user-attachments/assets/e9f008e1-2f36-416b-a0d0-86638a80bced)

Vemos que se encuentra /usr/bin/ls dentro del SUID, por lo que accedemos al contenido del directorio "**/root**".
Escribimos en la terminal **/usr/bin/ls /root**.

  ![image](https://github.com/user-attachments/assets/d636ac3d-52b0-4a23-88e3-c74032d5156b)

Parece que hemos encontrado el hash de la contraseña del usuario "root", por lo que para leer el contenido del fichero ejecutamos lo siguiente:<br>
**LFILE=/root/pass.hash<br>
/usr/bin/grep '' $LFILE**

  ![image](https://github.com/user-attachments/assets/881b7642-3a2d-4f45-833b-55ea295dbfdd)

Teniendo el hash, nos vamos a [Decode.fr](https://www.dcode.fr/funcion-hash-md5) y obtenemos la contraseña:

  ![image](https://github.com/user-attachments/assets/32bab29c-4d8f-464a-b30c-bbf2d9799aad)

La contraseña del usuario root es "**spongebob34**".

Finalmente, accedemos como root con el comando **su root**:

  ![image](https://github.com/user-attachments/assets/146eb561-42e4-4eba-968a-488d815657a0)

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/4eebae2b-45bb-4557-b5e7-c2dd7a890fd8)
