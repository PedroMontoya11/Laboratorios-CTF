Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh injection.tar** en la carpeta donde se encuentre el contenido extraído del zip.
  <ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/00be1f41-43a5-4ad9-9590-791feea674a9)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/63c1d4f8-84d8-4b37-98c5-bd2f3999fcef)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP*** respectivamente.

Vamos a usar el puerto 80 para seguir obteniendo información; para posteriormente, acceder a la máquina mediante SSH y realizar desde dentro una escalada de privilegios. Ponemos en el buscador http://172.17.0.2:80 y vemos que nos sale.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/a16ac676-bec9-4e63-a2cf-c4d34aaef9c0)

Al escribir la IP de la máquina atacante y buscar en el navegador en lugar de mostrarse una página de inicio correspondiente al servicio HTTP instalado en la máquina vulnerable o alguna página web relacionada con ella, se nos muestra una página de inicio de sesión (*login*) con dos inputs de usuario y contraseña. Tendremos que obtener las claves válidas para obtener acceso al servicio HTTP.

Podemos interpretar que por el nombre de la máquina Dockerlabs, la manera en la que obtendremos acceso al servicio (en este caso, MariaDB Server) es mediante una inyección SQL. 
Para el servicio de MariaDB, la inyección SQL se realiza ingresando en los campos de usuario y contraseña la cadena **UNION' or 1=1; --** :

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/65944bfa-3356-44a5-a16b-2c21c7a50a06)

Al hacer clic en el botón azul de Login se nos redirige a una ventana que contiene este mensaje:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2b27aaf8-4b89-4708-9dbb-1c546263c6ae)

Usaremos el nombre del usuario (***Dylan***) y la contraseña (***KJSDFG789FGSDF78***) para acceder a la máquina vulnerable mediante SSH.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/744bdf5d-e604-4bb5-818e-17a4a45e93a7)

Estamos dentro de la máquina de Dylan. Ahora, podemos probar a realizar una escalada de privilegios, para obtener acceso como usuario "root" y así contar con todos los privilegios en el sistema.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/1fa86ca4-ed77-446c-bd89-ce2fc591432f)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/396f7a91-a5f7-49a2-aa5d-c79b7ac8e95a)

Podemos obtener acceso mediante el archivo del sistema */usr/bin/env*. Buscamos en Internet [GTFOBins](https://gtfobins.github.io/) La sección correspondiente a "***env***" e introducimos los comandos que nos pone en la terminal de nuestra máquina Kali.

**/usr/bin/env /bin/bash -p**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/71cc4f17-f92d-44ee-9222-c19856b83736)

<ins>**Hemos realizado la escalada de privilegios correctamente, por lo que hemos conseguido acceder al sistema como el usuario "root"**</ins>.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/71bc8ade-6d33-4d1a-ab47-6a2b97364c80)
