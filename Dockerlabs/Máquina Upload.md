Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh upload.tar** en la carpeta donde se encuentre el contenido extraído del zip.
  <ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ff6ddac4-0189-4278-8a88-e514eb4b9d3e)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/c981b54c-164d-44eb-a561-9eb395f963dd)

Podemos ver que se encuentra abierto únicamente el puerto ***80***, correspondiente al servicio ***HTTP***.

Al dirigirnos a la dirección URL http://172.17.0.2:80 nos aparece una página web de subida de archivos, lo cual significa que la vulnerabilidad a explotar corresponde a un **Local File Inclusion** (*LFI*), que es una vulnerabilidad de seguridad que permite a un atacante leer o incluir archivos locales en un servidor a través de una aplicación web. Lo que haremos en este caso será ejecutar una *reverse shell* mediante la subida de un archivo PHP.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0d0623b6-51aa-40d1-8aa9-2556616bf2bf)

Antes de la reverse shell, vamos a realizar fuzzing en la página web en busca de directorios y archivos presentes.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/bfc1befa-b0d0-4d43-9c81-461d8528ab6f)

Vemos que existe un subdirectorio llamado "uploads", que será donde se almacenen los archivos subidos.

Creamos un archivo llamado index.php que tendrá este código:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/3d1af567-0427-48ae-a869-f7cdfbe5a228)

Este archivo lo subiremos a la página, y de lo que se encargará este código es de permitirnos la ejecución de comandos de manera remota, mostrando la salida en la propia página web.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/85b72da3-949e-4de2-91cb-ac04f4fe6e3d)

Si nos dirigimos a la ruta http://172.17.0.2/uploads/index.php --> [Nombre del archivo], escribimos el parámetro GET **cmd** y añadimos un comando a ejecutar (precedido del signo = ), podemos ver el resultado que muestra la ejecución del comando. En este caso, arrancamos el comando "id" y vemos que somos el usuario *www-data* con ID 33.

Buscamos en GitHub [el siguiente enlace](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) para descargar el archivo llamado **php-reverse-shell.php**. Puedes descargarlo o copiar el contenido del fichero.

Modificamos el contenido del archivo cambiando la variable ip por la dirección IP de nuestra interfaz de red (172.17.0.1) y el puerto que usaremos para la reverse shell (443).

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/c7a2345a-9ab7-488b-97d5-054a82793739)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/28f48570-9970-4a1b-9513-51b71cedb6f2)

Nos ponemos en escucha por el puerto 443 con el comando **nc -nlvp 443**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ee15780f-50c5-4ded-ba7d-ee0a43c40cb6)

Ejecutamos el contenido del fichero escribiendo en el navegador http://172.17.0.2/uploads/file.php.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/1c5b949e-9daa-40de-94a6-35b374b961eb)

Verificamos en nuestra terminal que hemos logrado acceder a la máquina vulnerable.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/884b2d59-2080-4778-bdb3-8c0806c4ea2d)

Realizamos una escalada de privilegios para acceder a la consola como usuario root, pudiendo así ejecutar cualquier comando y contando con todos los permisos disponibles.
Al hacer un **sudo -l** vemos que el usuario root no requiere de contraseña para conectarse mediante el binario /usr/bin/env y buscamos en https://gtfobins.github.io/ acerca del binario ***/usr/bin/env***.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2d2c6ed5-524f-4989-96c1-ed2b7f510b07)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/03be6dfe-02a0-48fe-b525-33a2f9a92419)

  Hemos accedido como el usuario "root" de manera exitosa.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/9b26fe67-8cdd-4807-81c9-1877514cd0e6)
