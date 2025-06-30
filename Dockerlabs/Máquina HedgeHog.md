1. Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh hedgehog.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

    ![image](https://github.com/user-attachments/assets/d45a5580-a151-4e09-b19c-910bf71212f6)

2. Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

    ![image](https://github.com/user-attachments/assets/cef46eb8-4170-4a87-8be6-fc145b38c7c6)

    Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP*** respectivamente.

3. Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información.<br>
    Usamos el comando **gobuster**, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*js* y .*txt*.

    ![image](https://github.com/user-attachments/assets/ef1bb237-4867-48c8-be5c-a1ce0a849810)

    Revisando en el navegador la dirección IP del servidor http://172.17.0.2:80 nos encontramos con un posible nombre de usuario del servicio SSH, llamado "**tails**".

    ![image](https://github.com/user-attachments/assets/416f0d4a-4cae-45d3-b5ce-8bda2cff48be)

4. Luego, intentamos obtener la contraseña del SSH de este usuario mediante un ataque Hydra de fuerza bruta:

    ![image](https://github.com/user-attachments/assets/2ebd6273-07ff-40d9-8866-54156d841807)

    La contraseña del usuario "***tails***" es "***3117548331***".

5. Nos conectamos al servidor SSH:

   ![image](https://github.com/user-attachments/assets/df38b8e3-d940-4b44-b68f-06e0cbd4f252)

   Hemos logrado acceder al sistema vulnerable como el usuario "tails", el cual tiene la posibilidad de escalar al usuario "sonic".<br>
   Por lo tanto, lo que realizaremos a partir de ahora es ir escalando de usuarios hasta llegar al usuario root.

   ![image](https://github.com/user-attachments/assets/a1caea6e-c32a-4a33-94ff-c81bd98621e7)

   ![image](https://github.com/user-attachments/assets/a5be9d6f-3a80-4222-89d3-98feebbbcd16)

    <ins>**Finalmente, somos el usuario root**</ins>.

6. Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

    ![image](https://github.com/user-attachments/assets/55961ca1-76ca-43c7-8333-aa5912a47c07)
