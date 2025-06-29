1. Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh tproot.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
  <ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2.</ins>

    ![image](https://github.com/user-attachments/assets/f2a31f7a-c23c-4830-8283-5069f3f40335)

2. Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

    ![image](https://github.com/user-attachments/assets/c745aaeb-49d2-4b44-896d-580994717be4)

    Podemos ver que se encuentran abiertos los puertos ***21*** y ***80***, correspondientes a los servicios ***FTP*** y ***HTTP*** respectivamente.

3. A continuación, revisamos si la versión del servicio FTP es vulnerable a un ataque de Metasploit.<br>
Abrimos la consola con el comando **msfconsole –q** y buscamos la versión 2.3.4 del ftp con **search vsftpd 2.3.4**.

    ![image](https://github.com/user-attachments/assets/ff6eb47b-7216-4ef7-9239-b373614b574f)

4. Establecemos el exploit encontrado con **use 0** y accedemos a la configuración de dicho exploit con el comando **options**.

    ![image](https://github.com/user-attachments/assets/dca969ae-95d0-49f7-aa66-9d5fe8c4f1f5)
   
    ![image](https://github.com/user-attachments/assets/20c54c60-7c9f-4715-89da-97c33e2790e4)

    En RHOSTS se pone los hosts de destino para el ataque; que en este caso, es la dirección IP <ins>172.17.0.2</ins>.

5. Finalmente, ejecutamos el exploit poniendo **exploit** en la CLI.

    ![image](https://github.com/user-attachments/assets/bb953a55-1a66-4a79-9157-9c461a8e433d)

	  Hemos logrado acceder al sistema como el usuario root, por lo que tenemos todos los privilegios existentes.

    ![image](https://github.com/user-attachments/assets/93e2a4f7-9042-4fcb-88ec-d2c40cc1f295)

6. Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

    ![image](https://github.com/user-attachments/assets/ce7405b9-800f-4ea4-8928-698d2ab2ba50)
