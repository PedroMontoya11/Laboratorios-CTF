Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh move.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/43a31721-b0fa-4214-aa9f-3ecd59df53d2)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/bc1afc2c-d156-482a-bf66-f49e63a720e1)

Nos encontramos con una serie de puertos abiertos, como los puertos ***21***, ***22***, ***80*** y ***3000***, correspondientes a los protocolos ***FTP***, ***SSH***, ***HTTP*** y el ***puerto de React***, respectivamente.

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información.
Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*py* y .*txt*.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/d28ba1f2-53e5-485d-b3dc-6cbb89075888)

Hemos obtenido que existe una página llamada *maintenance.html*, por lo que vamos a visualizarla en el navegador.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/3b373f32-cf42-4406-b63a-6c9ba4602638)

Se nos muestra un texto que nos dice que el sitio web se encuentra en mantenimiento, y que las claves para tener acceso a este se encuentran en el directorio */tmp/pass.txt* de la máquina a vulnerar.

Luego, nos vamos a encargar de investigar con el servicio FTP para encontrar más información que nos pueda servir a la hora de vulnerar a la máquina objetivo:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/8326a04b-20d8-4e27-bc22-cc431145d96d)

En el **Name** se escribe el usuario "***anonymous***" y cuando nos pida la contraseña hacemos clic en **Enter** sin ingresar texto para acceder mediante FTP.
Al hacer un listado de los archivos vemos que existe una carpeta llamada mantenimiento, y que dentro de ella, hay un fichero de Keepass con extensión .kdbx, lo que significa que es una base de datos de contraseñas guardadas en ese programa, las cuales se almacenan luego en un archivo de este tipo.

Nos encargamos de enviar ese archivo a nuestra máquina atacante para poder interactuar con él:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/74420da5-6aca-4246-97f4-7e3d63a6ecc7)

Abrimos el archivo con Keepass con el comando **keepass2 database.kdbx**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/4af03103-a476-46f0-bc22-dccc21c56ddb)

Se nos pide una contraseña para poder acceder al listado de contraseñas almacenadas, por lo que seguramente esa contraseña se encuentre dentro del archivo pass.txt del directorio /tmp de la otra máquina.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/d55b5b97-4a12-49d7-b902-1419ff3e0958)

Respecto al puerto que se encuentra abierto de React, al dirigirnos a la dirección URL http://172.17.0.2:3000, nos aparece una pantalla de Login de Grafana, que es un software libre que permite la monitorización de servidores en la nube, pudiendo obtener información del consumo de recursos y se asegura de que los servidores se encuentren funcionando.
En este caso, Grafana viene con las credenciales por defecto (usuario: admin, contraseña: admin), así que accedemos al panel de control de manera sencilla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/26d5e0db-44b6-4275-a5c2-6f83e76c0ddb)

Una vez hecho esto, se nos pedirá cambiar la contraseña por una nueva. Ponemos cualquiera, por ejemplo, admin123. Accedemos haciendo clic en **Submit**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/fb4ce5d8-f126-4544-b1ff-53d8106748ef)

En la barra de ayuda se nos muestra que la versión utilizada de Grafana es la 8.3.0, la cual se encuentra desactualizada (la última versión es la 11.0.0).
Haciendo una búsqueda de Grafana en la web de Searchsploit vemos que esta versión presenta una vulnerabilidad de tipo Directory Traversal and Arbitrary File Read.
En GitHub nos encontramos con un [script de Python](https://gist.github.com/bryanmcnulty/0f013fb75e94140bae70de2b0e986e45) que nos permitirá explotar esta vulnerabilidad:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0ad1c821-050e-45d4-8567-80623a3c09f8)

Ejecutamos el exploit:

**python3 exploit.py -u http://172.17.0.2:3000 -f /tmp/pass.txt -o pass.txt**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/7d6a0da9-2404-4056-9d26-6ec0a911b180)

Lo que hace este script es obtener el contenido del fichero pass.txt y almacenarlo en un archivo en nuestra máquina para poder visualizarlo con el comando **cat**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/1923d509-2113-43d3-8392-845f70ab818d)

Ahora que tenemos la clave, vamos a probar a insertarla como clave maestra de la base de datos del Keepass del paso anterior.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/76e3e463-9487-4dd2-96f3-effceeac9a21)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/c0abd609-87ec-44ba-a673-1869db4e6628)

Hemos conseguido las claves del usuario "freddy", por lo que nos conectaremos mediante SSH a la máquina víctima con sus credenciales, y en caso de no ser este usuario un *sudo user*, realizaremos una escalada de privilegios mediante la explotación de vulnerabilidades de archivos binarios del sistema.

Nos conectamos por SSH:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/5cf91769-78d2-4c10-adec-34c74b71697a)

Estamos dentro de la máquina, vamos a identificar ahora que archivos binarios puede ejecutar "freddy". Para ello, usamos el comando **sudo –l**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/1e423e36-1328-4367-b05f-66a3562ef2c9)

Modificamos el contenido del archivo *maintenance.py* e ingresamos la siguiente línea de código para realizar una reverse shell como usuario root desde nuestra máquina atacante:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/3d3ad495-67ba-43c1-8f16-4e01ce10805a)

Nos ponemos con NetCat en escucha por el puerto 443:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/b09fc8be-d4fd-459a-b98b-e8c6f16cae5c)

Por último, ejecutamos el binario *python3* escribiendo en la terminal **sudo /usr/bin/python3 /opt/maintenance.py**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/cb944116-9831-4b15-bcde-13373e4b6047)

Y esto es lo que vemos desde nuestra terminal de consola al ponernos en escucha por el puerto 443:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/d982f925-8529-42c1-b543-c530c7207ceb)

Hemos logrado acceder como usuario root a la máquina, por lo que ahora podremos realizar cualquier acción en ella a través de la CLI.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/57da3b0a-3f13-43d9-b11f-563c8074e6fb)
