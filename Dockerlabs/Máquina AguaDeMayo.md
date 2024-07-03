Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh aguademayo.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/b14cf139-26f4-4185-82cb-4f9799c16548)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2fc8d287-ffc5-41f2-ab01-2abeaa34512a)

Podemos ver que se encuentra abierto el puerto ***22*** y el puerto ***80***, correspondientes al servicio ***SSH*** y ***HTTP***.

Vamos a intentar realizar el ataque a la máquina mediante el servicio HTTP. Para ello, escribimos http://172.17.0.2:80 en el navegador y visualizamos la página.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/68ca2c26-377c-4971-9709-9ca730df5303)

Revisando el código fuente de la página, encontramos lo siguiente:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/8dd31f0d-3a18-4baf-b060-56cbd2e0ab98)

Se trata de un texto cifrado en el lenguaje de programación **BrainFuck**.<br>
Para descifrarla utilizamos la página web de [Decode.fr](https://www.dcode.fr/brainfuck-language) e insertamos la línea de texto para traducirlo a texto plano legible.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/b0a8b900-4506-4d2a-9d26-c8c61b36c647)

Este el texto obtenido de decodear la línea encontrada en el código fuente de la página web, correspondiente a la máquina víctima.

Realizamos un ataque Hydra para obtener mediante fuerza bruta el nombre de usuario SSH al que le correspondería este texto como contraseña.<br>
En caso de obtener algún usuario cuya contraseña para conectarse por SSH sea "*bebeaguaqueessano*", podríamos acceder directamente mediante este protocolo a la máquina vulnerable.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2dc9cd47-f40a-456e-a099-368ba15a9109)

Hemos obtenido que el nombre del usuario; cuya contraseña es "***bebeaguaqueessano***", es el usuario "***agua***".

Nos conectamos por SSH:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ec33dd99-a9fc-44aa-a5d4-8a6dd2d3444f)

Estamos dentro de la máquina, por lo que hará vamos a realizar una escalada de privilegios para lograr acceder como el usuario root.

Al ejecutar el comando **sudo –l** para ver los archivos binarios que el usuario "agua" puede ejecutar, nos encontramos con un binario llamado */usr/bin/bettercap*.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/4f2968ff-d217-46e5-bed8-35db7080540e)

Lo iniciamos escribiendo **sudo /usr/bin/bettercap** en la terminal:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/49d7ea02-e8bb-48b4-a84d-06185e634d30)

Luego de ejecutar el binario, se nos muestra una especie de consola interactiva. Primero de todo, escribimos **help** para visualizar los comandos que podemos emplear.

Habiendo hecho esto, nos percatamos de que si escribimos **!** [***COMANDO***] se ejecuta un comando correspondiente a nuestra CLI y se muestra la salida en esta CLI de Bettercap.

Escribimos el comando *whoami* precedido del signo *!* para comprobar nuestro usuario actual:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/916fdd34-0e6d-474e-891e-dbd8e91d56e5)

Somos el usuario root, por lo que a través de esta consola podemos ejecutar cualquier comando, al contar con todos los privilegios en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/27e4d7a5-d7dc-4e25-8d6e-453ecf3b0267)
