Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh library.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/8996d751-aed9-4d74-bc9e-18ba0e0cace2)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/34b19174-0c2c-4d83-89ac-2750e3d6721b)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP*** respectivamente.

Buscamos en el navegador http://172.17.0.2:80 y vemos que nos sale.

  ![image](https://github.com/user-attachments/assets/0fb60323-1e08-414d-af56-3a3d024c8627)

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información. Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*js*, .*txt*.

  ![image](https://github.com/user-attachments/assets/6f27b07e-c7bc-4498-9d1d-a47fc378ec71)

Nos encontramos con una página web llamada **index.php**. Vamos a visualizarla:

  ![image](https://github.com/user-attachments/assets/06174f99-abc9-42f7-b0cc-3ac90bbde21e)

El contenido que aparece en esta página parece que se trata de una contraseña en texto plano. Realizaremos un ataque Hydra para conocer al usuario existente en la máquina víctima cuya contraseña sea la obtenida:

  ![image](https://github.com/user-attachments/assets/16d45050-f0bc-44ea-b7d1-17eda5a13608)

Encontramos que al usuario "**carlos**" le corresponde la contraseña de la página index.php.

Nos conectamos por SSH:

  ![image](https://github.com/user-attachments/assets/369c6b45-7cfa-4d1e-bd73-22caff65f6b3)

Ahora estamos dentro de la máquina.

Revisamos con el comando **sudo -l** los archivos binarios que puede ejecutar el usuario de manera privilegiada, pudiendo realizar con ellos una escalada de privilegios.

  ![image](https://github.com/user-attachments/assets/5dde2b97-79a1-4443-b523-a0c94e77b97d)

El usuario "carlos" puede ejecutar los binarios ***/usr/bin/python3*** y el archivo ***/opt/script.py***, contando con privilegios de usuario root.

Editamos el contenido del script de Python de la siguiente manera:

  ![image](https://github.com/user-attachments/assets/6578eb08-4c83-49ea-9a03-5b03e849b198)

  ![image](https://github.com/user-attachments/assets/2b4366ae-1d31-49b2-9c2a-5a1b51e84f4f)

**sudo /usr/bin/python3 /opt/script.py**

  ![image](https://github.com/user-attachments/assets/203ea09f-3b83-4fe6-8c2b-4d5873dd60ad)

Finalmente, hemos accedido como el usuario root, por lo que actualmente tenemos todos los privilegios existentes en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/f5e89a32-ca7b-4240-8e9a-b67303ff45c0)
