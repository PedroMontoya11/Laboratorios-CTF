Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh borazuwarahctf.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
  <ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/99e21b3d-2051-4d7a-a992-75d87c975d7f)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/933e0be8-5643-47f9-ba21-275a918720b7)

Podemos ver que se encuentra abierto el puerto ***22*** y el puerto ***80***, correspondientes al servicio ***SSH*** y ***HTTP***.

Vamos a intentar realizar el ataque a la máquina mediante el servicio HTTP. Para ello, escribimos http://172.17.0.2:80 en el navegador y visualizamos la página.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/8e0f3aca-eb0c-405e-b7cf-7ee5aabae25f)

¡Vaya! Parece ser que no hay nada que nos interese en esta página…¿aunque quizás podemos obtener información mediante la imagen?

Descargamos la imagen y le realizamos un análisis de metadatos escribiendo **exiftool** [***Nombre del archivo***], en mi caso, el archivo se llama penguin.jpg.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/a6c1ac1a-7953-4dba-9dd3-1082b090137b)

Nos devuelve de resultado estos datos. Podemos ver la resolución que tiene, los megapíxeles y el peso del archivo. Estos datos no son relevantes, por lo que tendremos que emplear otro comando que nos permita obtener información empleando esta imagen.

Entre la información obtenida, nos encontramos con dos (Description y Title), que cuentan con un parámetro de nombre de usuario y uno de contraseña. Vemos que el usuario es borazuwarah, por lo que vamos a obtener su contraseña con un ataque de fuerza bruta, para así poder conectarnos a la máquina mediante SSH.

Realizamos un ataque Hydra de fuerza bruta para poder conectarnos mediante SSH a la máquina con las credenciales del usuario borazuwarah:
**hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/1a2963d0-cea7-428b-9cdb-1ce7bc44337d)

Realizamos la conexión con SSH:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/519388e8-dda2-4cae-b5ef-3d59ef08e366)

Hemos logrado acceder a la máquina vulnerable como el usuario "borazuwarah".

Hacemos una escala de privilegios para conseguir acceder como el usuario "root".

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/1184f9c1-1e07-4246-b466-c4094be8fe24)

Ejecutando el comando **sudo -l** para ver los archivos binarios que puede ejecutar el usuario. En este caso, el usuario "borazuwarah" puede ejecutar el archivo */bin/bash* sin requerir de contraseña.
Buscamos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/bin/bash***.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/67221273-e9b1-4e7f-9513-4bfdb273d35e)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/44b78029-1861-433e-a417-b47da54800cb)

Ahora, estamos como el usuario "root", por lo que tenemos todos los permisos disponibles en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/1d2f571b-5527-47b1-a60a-4416d0b34d02)
