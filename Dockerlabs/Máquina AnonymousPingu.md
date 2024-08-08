Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh anonymouspingu.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/e09e41dc-91b1-41b0-adff-843e00b6a618)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/698c9e57-1651-4d67-a48e-b2b797e495cc)

Podemos ver que se encuentran abiertos los puertos ***21*** y ***80***, correspondientes a los servicios ***FTP*** y ***HTTP***.

Vamos a intentar realizar el ataque a la máquina mediante el servicio HTTP. Para ello, escribimos http://172.17.0.2:80 en el navegador y visualizamos la página.

  ![image](https://github.com/user-attachments/assets/0212a008-9cec-4312-899b-e669344a4d3d)

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información. Debido a que revisando el código fuente de la página y mediante el uso del comando **ffuf** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado **gobuster**, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*js*, .*txt*.

  ![image](https://github.com/user-attachments/assets/5a8518df-141d-4aaf-80ea-735d205fd9f0)

Nos encontramos con varios directorios y archivos web, nosotros nos centraremos en "**/uploads**", ya que seguramente seamos capaces de realizar un **ataque LFI** (*Local File Inclusion*), mediante un archivo PHP, con el que poder realizar una reverse shell y al ejecutarlo, acceder de manera remota a la máquina víctima.

Para subir el fichero PHP a la carpeta "/uploads" utilizaremos el servicio FTP. Hacemos lo siguiente:

![image](https://github.com/user-attachments/assets/39dbeba0-d18c-484a-ba14-39ce0b356344)

![image](https://github.com/user-attachments/assets/37c0c387-504c-4e82-a3cd-1f6b1a840a45)

Nos ponemos en escucha con NetCat por el puerto 443:

  ![image](https://github.com/user-attachments/assets/6ff6bdbe-4992-4f12-a412-f62d173bdb05)

Revisamos con el comando **sudo -l** los archivos binarios que puede ejecutar el usuario *www-data*. En este caso, el usuario mencionado tiene permisos de ejecutar el binario /usr/bin/man como el usuario pingu, sin necesidad de ingresar ninguna contraseña en la terminal.

  ![image](https://github.com/user-attachments/assets/f73a1f0f-e38a-45ba-88ed-a530519687ca)

Buscamos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/usr/bin/man***.

  ![image](https://github.com/user-attachments/assets/393ba16d-a0d1-41d2-9861-24f9e0763d40)

**sudo -u pingu /usr/bin/man man**

Dentro del binario del comando man, escribimos **!/bin/sh** para realizar la escalada de privilegios.

  ![image](https://github.com/user-attachments/assets/21ed675b-30b2-4b79-9b36-eca603f340d2)

Ahora somos el usuario "pingu". Repetimos el paso anterior, buscando los binarios con los que poder realizar una escalada de privilegios, ya sea para acceder como el usuario root o en caso contrario, como otro usuario distinto de "pingu".

  ![image](https://github.com/user-attachments/assets/d1e2bf75-5807-4521-b675-cd8ed7fdab39)

En este caso, podemos ejecutar los archivos /usr/bin/nmap y /usr/bin/dpkg para realizar una escalada de privilegios, accediendo al sistema como el usuario "gladys" sin necesidad de ingresar su contraseña.

Buscamos en [GTFOBins](https://gtfobins.github.io/) acerca de los binarios ***/usr/bin/nmap*** y ***/usr/bin/dpkg***, teniendo la posibilidad de elegir cuál de las dos vulnerabilidades emplear.

  ![image](https://github.com/user-attachments/assets/ce49a295-5aca-4008-9847-6576a5fc60ff)

  ![image](https://github.com/user-attachments/assets/7ba0a455-fa65-467c-b261-685c3cab7762)

En este caso, vamos a usar la vulnerabilidad del binario /usr/bin/dpkg:

**sudo -u gladys /usr/bin/dpkg -l**

Dentro del binario del comando man, escribimos **!/bin/sh** para realizar la escalada de privilegios.

  ![image](https://github.com/user-attachments/assets/6c363b44-a265-44a6-bcb2-78d25bcd7d49)

Ahora, somos el usuario "gladys". Verificamos si somos usuario privilegiado o si debemos realizar otra escalada de privilegios más.

Visualizamos con sudo -l los archivos binarios que podemos vulnerar para realizar la escalada de privilegios:

  ![image](https://github.com/user-attachments/assets/8ea6c6a9-408f-4eec-a017-85b90689fabd)

El usuario "gladys" puede acceder al sistema como usuario root mediante el binario /usr/bin/chown, sin necesidad de ingresar ninguna clave de seguridad.

Miramos en [GTFOBins](https://gtfobins.github.io/) acerca de ***/usr/bin/chown***.

  ![image](https://github.com/user-attachments/assets/caa82a79-dd35-48d8-aa4e-bd8bfe027918)

**sudo chown /usr/bin/chown gladys:gladys /etc/passwd**

  ![image](https://github.com/user-attachments/assets/b7b1b018-bb1c-4071-9c7d-accf8a7db789)

Cambiamos el usuario y grupo propietario del archivo /etc/passwd, para luego poder crear un nuevo usuario root que no tenga contraseña:

**echo 'root2::0:0:root:/root:/bin/bash' >> /etc/passwd**

  ![image](https://github.com/user-attachments/assets/76f328a3-9b06-4ad9-b096-49f474ebacf3)

Finalmente somos el usuario root, por lo que podremos realizar cualquier acción en el sistema, ya que contamos con todos los permisos existentes.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/aff3e1fe-7df0-4ba1-b43e-7e6cbe311187)
