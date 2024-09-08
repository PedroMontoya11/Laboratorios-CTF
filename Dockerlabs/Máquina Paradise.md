Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh paradise.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/fc1ed529-62b7-431b-905d-3a0f085e9446)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/9f7cc61f-1d63-41d6-93fc-f8da5ff2494d)

Podemos ver que se encuentran abiertos los puertos ***22***, ***80***, ***139*** y ***445*** correspondientes a los servicios ***SSH***, ***HTTP***, ***Netbios-ssn*** y ***Microsoft-ds*** respectivamente.

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información.
Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*py* y .*txt*.

  ![image](https://github.com/user-attachments/assets/3e6d9d5d-ea80-42b3-ae64-46f73cb242d5)

Nos dirigimos al navegador y en el buscador ponemos http://172.17.0.2:80.

  ![image](https://github.com/user-attachments/assets/37293b8e-076e-425b-9363-1689ecb48502)

Miramos en  http://172.17.0.2/galery.html para ver qué información encontramos.<br>
Al revisar el código fuente de la página web nos encontramos con esta última línea comentada:

  ![image](https://github.com/user-attachments/assets/05be193f-09aa-459b-84c4-5a912e6ca59b)

Parece ser un texto encriptado en ***base64***, por lo que vamos a descifrarlo:

  ![image](https://github.com/user-attachments/assets/b3e848bd-674e-4d6b-bf1c-98dac63bd075)

Luego, añadimos el texto obtenido como ruta de un posible subdirectorio web (http://172.17.0.2/estoesunsecreto):

  ![image](https://github.com/user-attachments/assets/e8716267-502e-4166-896b-5b286e4bf191)

Nos encontramos con un fichero de texto que va para un posible usuario de la máquina víctima llamado "lucas". Veamos el contenido del archivo *txt*:

  ![image](https://github.com/user-attachments/assets/880b3adb-d646-40a0-9d37-679bc0d46f09)

El mensaje dice que el usuario debe modificar su contraseña, ya que está es muy débil; y por ende, vulnerable a ataques de fuerza bruta.

Probamos a obtener la contraseña del usuario "lucas" con un ataque Hydra.

  ![image](https://github.com/user-attachments/assets/38092cc6-933e-4279-af5e-743b0a302ddd)

La clave del usuario "**lucas**" es "**chocolate**".

Nos conectamos por SSH:

  ![image](https://github.com/user-attachments/assets/a6f65191-e849-4de4-aec6-3ad53a9e79d6)

Hemos accedido al sistema como el usuario "lucas".

Revisamos los archivos binarios que puede ejecutar este usuario de manera privilegiada:

  ![image](https://github.com/user-attachments/assets/631fbb83-5f6c-4e20-a7df-9a9dca041c03)
  
Vemos que hay un archivo denominado "***/usr/local/bin/privileged_exec***" dentro del SUID, por lo que podemos arrancarlo sin necesidad de ser usuario root o el propietario del archivo.

Para ejecutarlo, ingresamos en la CLI **/usr/local/bin/privileged_exec**.

  ![image](https://github.com/user-attachments/assets/a8d66f1c-93c6-4fcb-9a29-3fa2ee567ee4)

Finalmente, hemos accedido como usuario root, por lo que contamos con todos los privilegios existentes en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/cfc9b71c-863c-462f-be17-6d0cab227506)
