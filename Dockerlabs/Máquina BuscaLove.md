Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh buscalove.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>En este caso, la dirección IP de la máquina a vulnerar es la 172.18.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ffca8ce4-ea0f-45b8-b854-dc12470a3e98)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/c4a2e60f-03ee-41b4-9a91-576f72c5b1e2)

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información.
Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*sh* y .*py*.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/6388652f-8dab-4093-94d9-e51714cf4a0e)

Vemos que existe un subdirectorio web llamado *wordpress*, por lo que vamos a visualizar su contenido en la web.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/b08fbfea-95d2-4ca0-826d-f7e199e47bad)

Se nos muestra esta página de inicio en la que aparecen unos enlaces, que al hacer clic en ellos, nos redirige a esta misma página.

Revisamos el código fuente de la página en búsqueda de información:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/df411765-5e7c-4567-8ea8-1f2e89b66a24)

Esta página web cuenta con una vulnerabilidad de tipo **LFI** (*Local File Inclusion*), el cual consiste en que el usuario es capaz de acceder y visualizar desde la página web al contenido de archivos locales del servidor, por lo que si nosotros mediante **Path Traversal** escribimos en el buscador detrás de la ruta que tenemos la cadena de texto "***?love=../../../../../../etc/passwd***", podremos visualizar el contenido del fichero passwd, el cual se utiliza en Linux para guardar la información principal de cada cuenta (nombre de usuario, grupo, ID de usuario, ID de grupo, etc).

Esto es lo que se nos muestra del archivo localizado en /etc/passwd:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/86a22fe5-fb65-45a3-a94c-71ebaf2a0bef)

Probamos a usar el usuario "rosa" y realizar un ataque Hydra para obtener mediante fuerza bruta su contraseña del SSH.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/6749e1f3-e7a8-4944-a054-67325eb1fa9f)

La contraseña del usuario **rosa** es **lovebug**.

Nos conectamos por SSH:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/e6d720ff-96d1-4993-a808-f0d340ae8969)

Ahora estamos dentro de la máquina vulnerable como el usuario "rosa".

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/6f2fade3-87a8-4bef-897c-aed80b5003f5)

Usamos el comando **sudo -l** para ver los archivos binarios que puede ejecutar este usuario como *sudo user*:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0bf3f5f1-5983-4ece-a5f1-dca4b77f1cc9)

El usuario "rosa" puede ejecutar los binarios ***/usr/bin/ls*** y ***/usr/bin/cat*** como un usuario privilegiado, por lo que seremos capaces de listar cualquier directorio y visualizar el contenido de cualquier archivo del sistema; y además, sin necesidad de incluir la contraseña a la hora de ejecutar los comandos **ls** y **cat**.

Visualizamos el directorio /root con **sudo -u root ls -la /root**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/024bcb6f-1b9a-4a5d-9015-6222d34e460f)

Hay un archivo llamado *secret.txt*, vamos a visualizarlo:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/74142e16-3322-4f26-94ab-a2d8052f57b1)

El fichero cuenta con un texto encriptado en hexadecimal, por lo que tendremos que descodificarlo a base32 para conocer el texto real.

Para descifrarlo utilizamos la página web de [CyberChef](https://gchq.github.io/CyberChef/) y convertimos de hexadecimal a base32:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/5d5e8ad8-3d7d-4526-b7f9-cd8f7b5748b8)

El output nos devuelve el texto "**noacertarasosi**".

Probamos a usar ese texto como contraseña del otro usuario existente (pedro) para acceder como ese usuario:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ace8279c-a813-4f9d-b593-0f493092c7cf)

Hemos accedido como el usuario "pedro". Al ejecutar el comando sudo -l se puede ver que el usuario "pedro" tiene permiso de ejecución del binario /usr/bin/env como sudo user.

Buscamos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/usr/bin/env***.

**sudo env /bin/sh**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/293d27dc-86d3-471a-9906-70346eaecd3b)

Ejecutamos el comando para realizar la escalada de privilegios:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/dd753872-ebe8-4adf-a7cd-f4bebbe90783)

Hemos logrado acceder como usuario root, por lo que ahora podremos realizar cualquier acción en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/9df4d677-97ef-4635-9aa4-da4b9cc37a1b)
