Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh vacaciones.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
  <ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/38aa5427-a6ab-4d35-a6fd-35af7c4f60e9)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/76b0ed4e-fdb1-4b2a-b0d6-72e032a3a821)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP*** respectivamente.

Vamos a intentar atacar mediante HTTP. Nos dirigimos al navegador y en el buscador ponemos http://172.17.0.2:80.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/3c5aa7a8-6f43-45b3-8358-eb48c020f596)

Nos aparece una página de inicio en blanco completamente vacía, por ello, miramos el código fuente de la página y encontramos esto:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0987e31a-443f-4b86-82c0-1b2b430b4339)

<ins>Vale, no entiendo nada</ins>… Podemos probar con los nombres conectarnos por SSH a la máquina vulnerable.

Probamos de primeras con el usuario "juan":

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/92a3dee4-4fa0-4377-a0f0-364bf69d300c)

No hemos obtenido nada, probamos con el usuario "root" y tampoco hemos podido obtener las claves del usuario root para conectarnos por SSH, por lo que la última posibilidad que tenemos es intentar conectarnos con el usuario "camilo" y desde dentro del sistema realizar una escalada de privilegios:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/8709a19d-7025-4331-a31e-0b91da9dd528)

¡Vaya! Hemos obtenido las claves de SSH del usuario "camilo", así que podemos conectarnos a la máquina vulnerable por SSH con ellas.

Nos conectamos usando el comando **ssh camilo@172.17.0.2** y escribiendo la contraseña del usuario:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0e949274-0095-4028-82ac-b60a47137a72)

Estamos dentro de la máquina.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/f9c4ea8a-c6fd-43cb-a203-6b0e5617c58f)

Como no somos el usuario "root" tendremos que realizar una escalada de privilegios; pero antes de todo, vamos a ver el correo que le ha enviado el usuario "juan" al usuario "camilo".

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/c782bfcb-de29-45b0-8c6f-b25b49f84fc2)

Gracias Juan por darme tu contraseña, eres un crack…Seguramente el usuario "juan" sea un usuario del grupo sudo, permitiéndonos realizar la escalada de privilegios de manera más sencilla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/86fbabd8-cb71-4e34-a9a9-1c73714fcb51)

Finalmente ahora sí, realizamos la escalada de privilegios para acceder como el usuario "root":

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/03df04cc-e8a2-4333-8cd4-717b64c954e1)

Miramos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/usr/bin/ruby***.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/a570afa8-0483-4c31-8b97-4e5e7a506b3e)

**sudo ruby -e 'exec "/bin/sh"'**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/472a307e-4ae4-476c-9a30-d79cfff2427b)

Hemos logrado acceder como el usuario "root", por lo que tenemos todos los permisos en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/d7b8f230-9ff3-4867-9563-2d5fa17ad09e)
