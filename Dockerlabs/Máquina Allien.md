1. Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos bash auto_deploy.sh allien.tar en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

    ![image](https://github.com/user-attachments/assets/c306b7dd-4f95-4119-987d-57a05dba9534)

2. Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

    ![image](https://github.com/user-attachments/assets/c008f617-92b1-4cf4-9b99-5a88fc5a9098)

    Los puertos abiertos encontrados son el ***22***, ***80***, ***139*** y ***445***, correspondientes a los servicios ***SSH***, ***HTTP***, ***Netbios-ssn*** y ***SMB***, respectivamente.
   
3. Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información.<br>
    Usamos el comando **gobuster**, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*js* y .*txt*.

     ![image](https://github.com/user-attachments/assets/d932718b-c720-4645-8725-ed0b150a6ce3)

     Al revisar la página web, nos percatamos de que hay un formulario de login en el index.php y una página de venta de productos (productos.php).

4. Como estas páginas que hemos encontrado no cuentan con información suficiente como para acceder al sistema, intentaremos obtener contenido del servidor SMB.
     Para ello, utilizamos el comando **enum4linux** de la siguiente manera:

    ![image](https://github.com/user-attachments/assets/bbf7a9f8-9e3d-4008-b6c1-9b0a5b70f916)

    ![image](https://github.com/user-attachments/assets/f949a9e7-0b6f-40ac-af1a-075c932e5e28)

   Encontramos 5 usuarios en la máquina víctima; de los cuales, el más importante es el usuario "***administrador***".

5. Empleando el comando **crackmapexec** vamos a intentar obtener la contraseña del SMB de los usuarios:<br><br>
   **sudo crackmapexec smb 172.17.0.2 -u '[USER]' -p rockyou.txt --no-bruteforce**

    Cuando se prueba esta línea de código con el usuario "***satriani7***", se muestra esto por pantalla:

    ![image](https://github.com/user-attachments/assets/d4c0a7b8-229a-4b96-b1f5-69f87df6d1c8)

    La contraseña del SMB del usuario es "***50cent***".

6. Accedemos a la máquina mediante el servidor SMB y revisando directorios, encontramos los siguientes ficheros de texto:

    ![image](https://github.com/user-attachments/assets/bd23d32b-8a18-40fd-82c2-a64e4b329c94)

    Miramos el contenido del fichero credentials.txt:

    ![image](https://github.com/user-attachments/assets/eb7a78ff-657a-445b-a118-7b4c44178f2a)

    Ahora, ya tenemos la clave del usuario "administrador", que seguramente se trate de la contraseña del servicio SSH de dicho usuario.

7. Nos conectamos por SSH:

    ![image](https://github.com/user-attachments/assets/5e9f25b6-de6d-41f4-ac67-bd78b599845b)

8. Luego, podemos intentar hacer un **ataque de tipo RCE** (*Remote Code Execution*) creando un script que contenga una reverse shell.

    ![image](https://github.com/user-attachments/assets/7546259e-d5d2-4f07-815e-eea39e8f53d0)

    ![image](https://github.com/user-attachments/assets/d7080180-ab9c-418d-8c6d-7e548e122f68)

    Metemos estos archivos en la ruta */var/www/html* para poder ejecutarlos desde el navegador.<br>
    Escribimos http://172.17.0.2/shell.php y nos ponemos en escucha con NetCat por el puerto 443:

    ![image](https://github.com/user-attachments/assets/cb4b526c-b1e9-4688-a945-4aec44d881c7)

    ![image](https://github.com/user-attachments/assets/59ca50d1-5772-4105-a286-5e8da1197824)

    Hemos logrado entrar en el sistema como el usuario "www-data", el cual tiene capacidad de ejecutar el binario /usr/sbin/service como usuario "root".

9. Buscamos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/usr/sbin/service***.

   ![image](https://github.com/user-attachments/assets/5ac25fbf-8d96-4bea-bcd1-ec89ae790fd3)

    **sudo service ../../bin/sh**
   
    ![image](https://github.com/user-attachments/assets/693b6fe1-d9c8-47d1-b088-9fcf0ba358b6)

    Finalmente, hemos conseguido acceder al sistema como el usuario "root", por lo que tenemos todos los privilegios existentes en el sistema.

10. Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

    ![image](https://github.com/user-attachments/assets/38c82078-5c27-4fe7-b5d3-671d1cc50029)
