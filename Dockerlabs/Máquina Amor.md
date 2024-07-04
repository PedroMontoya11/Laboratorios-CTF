Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh amor.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/fb492d61-9331-4bfd-b13b-e020bd3c37ca)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2d6e7b19-8dac-44c8-8de5-9c267cf09473)

Podemos ver que se encuentra abierto el puerto ***22*** y el puerto ***80***, correspondientes al servicio ***SSH*** y ***HTTP***.

Vamos a intentar realizar el ataque a la máquina mediante el servicio HTTP. Para ello, escribimos http://172.17.0.2:80 en el navegador y visualizamos la página.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/cd699a4a-1f70-45bc-a912-758ea4632477)

Se nos muestra en la página web una serie de avisos de la empresa SecurSEC S.L en la que anuncian que el empleado "Juan" ha sido despedido por enviar su contraseña a otro usuario, y que dentro de la empresa se ha detectado una contraseña débil. Nosotros nos centraremos en usar el nombre de "Carlota" como un usuario con el que poder conectarnos por SSH a la máquina víctima, una vez hayamos obtenido su contraseña.

Realizamos un ataque de fuerza bruta Hydra para obtener la contraseña del usuario "carlota":

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/92c82264-3c95-4cad-9baf-ff42b6772c51)

La contraseña de ***carlota*** es ***babygirl***.

Nos conectamos por SSH a la máquina vulnerable:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/7349ce1f-2946-4acd-80c0-42c42040bbce)

Estamos dentro de la máquina.

Escribimos el comando **sudo –l** para visualizar los archivos binarios con los que podríamos realizar la escalada de privilegios, siendo el usuario "carlota":

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/09be3852-8c7a-467a-ad62-aa4cc8271072)

Al parecer el usuario "carlota" no cuenta con ningún permiso de *sudo user* por lo que vamos a tener que buscar más información para lograr realizar la escala de privilegios.

Al revisar el directorio del usuario "carlota" nos encontramos con esto

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/66c189c3-6325-4e06-83eb-2242359329b7)

Vamos a descargar esa imagen en nuestra máquina atacante para analizarla. Para ello, usamos en nuestra terminal atacante usamos la línea de código **scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg** [*Ruta absoluta de la imagen encontrada*] **/home/kali** [*Ruta absoluta de nuestro directorio de trabajo*].

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/57fdc607-3b9c-41e3-b1a0-7ab99590c695)

Esta es la imagen:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/a1c5b6c3-5e9a-4811-bc77-00dfc1c19c77)

Muy bonita la imagen Carlota, pero creo que hay algo más aparte de esta foto...

En Ciberseguridad, existe un concepto que consiste en ocultar mensajes o información confidencial en un archivo (portador), por lo que esta información puede encontrarse contenida en una imagen, un documento de texto o un Excel sin ser visible por el usuario. Esto se conoce como **esteganografía**.<br><br>
Por suerte para nosotros, hay un comando que nos permite obtener los datos contenidos en un archivo, y ese comando es **steghide**.<br><br>
Escribimos en la terminal **steghide --extract -sf** [***Nombre del archivo***].

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ba7be85c-fe5b-489a-9684-13bcdd11631f)

Al darle a **Enter** cuando se nos pide la contraseña se crea un archivo de texto llamado *secret.txt*, el cual contiene una clave cifrada.

Cabe aclarar que el método de encriptación utilizado es Base64, por lo que para descodificarla usamos **echo** "[***Clave cifrada***]" **| base64 --decode**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/9ba3141f-9694-4637-9d16-a892aa46b889)

Hemos obtenido como clave "**eslacasadepinypon**". Podría tratarse de la contraseña de otro usuario presente en el sistema, pero desconocemos el usuario específico asociado a esta posible contraseña.

Si verificamos las variables de entorno del usuario "carlota" con el comando **env**, obtenemos lo siguiente:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/28b3f5d5-d82f-4317-aff0-3b3e499c6a62)

Hay una línea llamada "***SECRET***" en la que carlota se refiere a un usuario llamado "oscar". Por lo que podemos probar a acceder con este usuario con **su oscar**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/85142b73-4c31-4c8d-90e3-8b63acd4f8b6)

Usando el comando sudo -l nos percatamos que el usuario "oscar" puede ejecutar el binario /usr/bin/ruby, por lo que podemos usar este archivo binario para la escalada de privilegios.

Buscamos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/usr/bin/ruby***.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2b4a393b-c50d-47dc-b193-e8191a07d4c3)

**sudo ruby -e 'exec "/bin/sh"'**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/51cb5d16-ba9e-45ab-935e-20575f96b30f)

Hemos logrado acceder como el usuario "root", por lo que tenemos todos los permisos en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/5d1b3906-0bf3-4637-8aa7-70cb9e8a89b6)
