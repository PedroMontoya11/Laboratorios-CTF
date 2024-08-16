Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh mirame.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/765c7060-f608-4402-833d-7e3416a2a61f)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/e6889a2e-548b-4dd9-b59b-0945373d0829)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP*** respectivamente.

Buscamos en el navegador http://172.17.0.2:80 y vemos que nos sale.

  ![image](https://github.com/user-attachments/assets/ec1c00d3-bef6-44ce-9391-a2e362d4fe4e)

En este formulario de inicio de sesión que nos aparece vamos a probar con credenciales predeterminadas (admin:admin o root:root) para ver si conseguimos acceder.

  ![image](https://github.com/user-attachments/assets/da297f60-b35c-4519-ba16-6c0dadf4ce4e)

Al usar estas credenciales nos da error, por lo que podemos probar la herramienta **SQLMap**, que se encarga de detectar vulnerabilidades en el servicio SQL y explotarlas, pudiendo así obtener información sobre los datos existentes en las bases de datos.

Escribimos la siguiente línea de código: **sqlmap -u "http://172.17.0.2/index.php" --forms --batch --dbs**

  ![image](https://github.com/user-attachments/assets/c90082a8-32c3-432e-80c8-af03321f830a)

  ![image](https://github.com/user-attachments/assets/565e5ef6-9b51-4f0d-a145-b4b8a2860a72)

Empleando esta función, hemos obtenido las bases de datos existentes, que son "**information_schema**" y "**users**". Vamos a intentar obtener el contenido de la base de datos de usuarios, para así conseguir credenciales que nos serán útiles en el formulario de la página web.

**sqlmap -u "http://172.17.0.2/index.php" --forms --batch -D users --tables**

  ![image](https://github.com/user-attachments/assets/6549f2d4-6586-4554-ab55-f8fb4be5e7cb)

  ![image](https://github.com/user-attachments/assets/e8881139-8131-4a2f-9cd8-f1183ff18c05)

La base de datos "users" cuenta con una única tabla, llamada "**usuarios**".

**sqlmap -u "http://172.17.0.2/index.php" --forms --batch -D users -T usuarios --dump**

  ![image](https://github.com/user-attachments/assets/184ff7b6-3be6-464e-8668-f5e0528a39b2)

  ![image](https://github.com/user-attachments/assets/404da4bf-fb53-4f8b-9399-f81ac09f4585)

Iniciamos sesión con las credenciales encontradas:

  ![image](https://github.com/user-attachments/assets/a1cc5ce1-2ab0-49a0-8fde-66bacc4cfca2)

La ventana que nos aparece a continuación, se trata de una página que te dice la temperatura en Celsius de la ubicación que el usuario ingrese.<br>
Esto no nos sirve para nada, por lo que podemos probar a ingresar los nombres de usuario en el buscador para detectar posibles directorios personales, los cuales no podrían ser obtenidos con las herramientas ***gobuster*** o ***ffuf***.

Al probar con la contraseña del usuario "directorio"; la cual es "directoriotravieso", nos encontramos con esto:

  ![image](https://github.com/user-attachments/assets/8f615dbd-b74d-4712-bb86-b49741269e6c)

Descargamos la imagen para analizar que se trate de un caso de esteganografía. En Ciberseguridad, existe un concepto que consiste en ocultar mensajes o información confidencial en un archivo (portador), por lo que esta información puede encontrarse contenida en una imagen, un documento de texto o un Excel sin ser visible por el usuario.
Esto se conoce como **esteganografía**.

Por suerte para nosotros, hay un comando que nos permite obtener los datos contenidos en un archivo, y ese comando es **steghide**.

Escribimos en la terminal **steghide --extract -sf** [***Nombre del archivo***].

  ![image](https://github.com/user-attachments/assets/54eef2aa-ced3-4199-b23a-0dd632e7b4fa)

Se nos pide una contraseña y como no la tenemos, no podemos extraer nada. Mientras tanto, vamos a tratar de hacer un ataque de diccionario al archivo con la herramienta **stegseek** usando el comando:

**stegseek extract -sf miramebien.jpg -wl rockyou.txt**

  ![image](https://github.com/user-attachments/assets/f83ab8f6-280d-40c4-8efa-d778ec49568c)

Obtenemos que la clave para extraer los datos contenidos en la imagen es "**chocolate**". Probamos de nuevo a ejecutar el comando steghide:

  ![image](https://github.com/user-attachments/assets/d2d1e3d6-625b-4af3-807a-a3cf9912cf04)

Ahora tenemos un archivo .*zip* que también debemos descifrar, ya que viene con contraseña.

  ![image](https://github.com/user-attachments/assets/34035b60-29b8-4b72-a650-0c9aa45ab044)

  ![image](https://github.com/user-attachments/assets/c2552c43-fb50-4544-8eb6-de324ef171ca)

Luego, sabemos ahora que la clave del zip es "**stupid1**".

Desciframos el contenido con el comando **unzip** y se extrae un archivo de texto denominado "secret.txt", que cuenta con las claves de un usuario llamado "carlos".

  ![image](https://github.com/user-attachments/assets/1496686f-591b-4597-a211-9da71a596581)

Con estas claves vamos a intentar acceder a la máquina víctima mediante el servicio SSH.

Nos conectamos por SSH:

  ![image](https://github.com/user-attachments/assets/f3628787-33ec-4ee9-8919-031b35bb2ba8)

Estamos dentro del sistema.

Verificamos si hay algún binario que podamos aprovechar en el SUID con el comando **find / -perm -4000 2>/dev/null**.

  ![image](https://github.com/user-attachments/assets/a85cb7bc-3fe6-4453-9453-96cba8ae2cb3)

Buscamos en GTFOBins acerca del binario ***/usr/bin/find***.

  ![image](https://github.com/user-attachments/assets/cfa052be-916b-4435-adc6-57c157a643df)

**find . -exec /bin/sh -p \; -quit**

  ![image](https://github.com/user-attachments/assets/d1b219cf-861f-47ea-b38b-b1acef847768)

Finalmente, hemos conseguido hacer una escalada de privilegios, pudiendo acceder al sistema como el usuario root.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.
