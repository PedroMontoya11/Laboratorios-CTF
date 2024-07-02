Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh hackpenguin.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/40dedfa4-a9ea-4d1d-a437-6332d89f53ff)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/4920cdf2-d6ae-42d7-a8ac-48026e2b008c)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP*** respectivamente.

Vamos a usar el puerto 80 para seguir obteniendo información; para posteriormente, acceder a la máquina mediante SSH y realizar desde dentro una escalada de privilegios.
Ponemos en el buscador http://172.17.0.2:80 y vemos que nos sale.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/417b01d4-6b8a-480e-8ce0-0b60e8f51b9e)

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información.
Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*sh* y .*py*.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/8d5d071e-2fb6-422d-9c1f-06c1c2306359)

Hallamos una página nueva llamada *penguin.html* y al parecer, hay una imagen subida a la web que es *penguin.jpg* (seguramente sea la foto existente en el HTML encontrado).

Nos dirigimos a http://172.17.0.2/penguin.html.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/36ecb18b-5ec0-4114-856a-375ec3c54d2c)

¡Vaya! Parece ser que no hay nada que nos interese en esta página…¿aunque quizás podemos obtener información mediante la imagen?

Descargamos la imagen y le realizamos un análisis de metadatos escribiendo **exiftool** [***Nombre del archivo***], en mi caso, el archivo se llama penguin.jpg.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/b0004eb8-72ed-4164-9202-77bb5a9ae959)

Nos devuelve de resultado estos datos. Podemos ver la resolución que tiene, los megapíxeles y el peso del archivo. Estos datos no son relevantes, por lo que tendremos que emplear otro comando que nos permita obtener información empleando esta imagen.

En Ciberseguridad, existe un concepto que consiste en ocultar mensajes o información confidencial en un archivo (portador), por lo que esta información puede encontrarse contenida en una imagen, un documento de texto o un Excel sin ser visible por el usuario. Esto se conoce como **esteganografía**.

Por suerte para nosotros, hay un comando que nos permite obtener los datos contenidos en un archivo, y ese comando es **steghide**.

Escribimos en la terminal **steghide --extract -sf** [***Nombre del archivo***].

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0627a235-2c4f-4ec9-bd73-56eaf8c33372)

Se nos pide una contraseña para poder ejecutar el comando. Para obtener dicha contraseña, realizamos este script de bash:

	#!/bin/bash
	
	dicc_path="/home/kali/passwords.txt"
	file="/home/kali/Downloads/penguin.jpg"
	
	# Verificar si existe el diccionario
	if ! test -f $file; then
	    echo "El archivo a examinar no existe."
	    exit 1
	fi
	
	# Verificar si existe el diccionario
	if ! test -f $dicc_path; then
	    echo "El diccionario no existe."
	    exit 1
	fi
	
	# Itera cada posible password del diccionario
	for password in $(cat "$dicc_path"); do
	
	    #Probamos el password y guardamos el código de estado.
	    echo "Probando con la contraseña: $password"
	    steghide --extract -sf $file -p "$password" 2>/dev/null
	    exit_code=$?
	
	    # Si el código de estado es exitoso, indica que se encontró la contraseña
	    if [[ $exit_code -eq 0 ]]; then
	        clear
	        echo "Contraseña encontrada: $password"
	        exit 0
	    fi
done

Una vez hecho esto, ejecutamos el script.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/1002bafd-fbd3-46b8-894c-99211ecaf89f)

Vemos que nos sale que la contraseña ha sido encontrada y es "chocolate". Vamos a comprobarla al ejecutar el comando steghide.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/58b5497f-309c-403e-9db8-7468a2460409)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/a40e2ac6-8fd1-4612-89df-8edc09450ef4)

Hemos extraído de la imagen un archivo .*kdbx*, correspondiente a KeePass, que es donde se encuentran almacenadas las claves ingresadas en la aplicación y que funciona como una base de datos de credenciales del usuario. Dentro del Explorador de archivos de la máquina Kali, al hacer clic en el archivo se nos muestra una ventana en la que se nos pide la contraseña maestra para acceder al contenido del KeePass.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/7cbf977f-a9ad-40ef-8e29-2c329ee7e55d)

Desencriptamos el contenido del fichero penguin.kdbx para dejarlo hasheado, ya que posteriormente mediante **John The Ripper**, obtendremos la clave sin hashear.

**keepass2 john penguin.kdbx >** [***Nombre del fichero donde guardaremos el hash***]

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/819e31cb-24e4-4e83-9c46-1378d0b59da5)

Y ahora con el hash obtenemos la clave del KeePass:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/8d23d945-2874-4429-b241-925e8021fd5b)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/a377fe42-79e5-4ef9-9f0d-022fc8725f17)

La clave maestra del KeePass es "password1".

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/9ee214fe-0a6f-423c-947f-f5ea579f581a)

Ya podemos visualizar las claves almacenadas y nos encontramos con una que nos puede interesar:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/dd43bf2b-7fd5-4358-91cd-17bf670c7ada)

Usaremos el nombre de usuario y contraseña para conectarnos a la máquina vulnerable mediante SSH.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/5dc5f302-78c5-4ac7-aa65-40ac27c72351)

Con el nombre de usuario "pinguino" no conseguimos conectarnos a la máquina, por lo que viendo el KeePass, me percató de que probablemente el usuario sea "penguin" y no "pinguino" (las claves que hemos encontrado se encuentran dentro de una carpeta llamada *penguin*). Probamos con este nuevo nombre de usuario:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/8ce9a3d2-4f47-4473-b5eb-7b0a51bcdbf4)

Ahora sí que ha funcionado y nos encontramos dentro de la máquina vulnerable Ubuntu.

Realizamos una escalada de privilegios. Al listar los elementos del directorio actual nos encontramos con un archivo .*sh*, que se trata de un script de bash (script.sh), el cual escribe en un documento de texto (archivo.txt) el texto "pinguino no hackeable".

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/061d0ed2-607b-4d49-9453-f9f4b42419bf)

Simplemente, escribimos **echo chmod u+s /bin/bash > script.sh** y después ejecutamos el comando **bash -p** para convertirnos en el usuario root mediante una escalada de privilegios en Linux.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/f30c7204-bc3a-4d73-af52-fd9e7658d26f)

<ins>**Ahora somos el usuario root**</ins>. Al parecer, el pingüino era más hackeable de lo que se pensaba...

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/08be9c53-8f66-4099-89ba-af6d33bacfb9)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/59f09939-4bbe-42a7-975a-31bd4c489d27)

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/3c85e85f-1b47-4b00-a35d-41480878268c)
