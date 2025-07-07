1. Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh influencerhate.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

    ![image](https://github.com/user-attachments/assets/e8034c69-bc60-444e-b1b2-3d403ceb1e91)

2. Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

    ![image](https://github.com/user-attachments/assets/d13feac0-5eb5-40b5-8e6e-e80c5337027a)

    Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP*** respectivamente.

3. Al abrir el navegador escribimos la dirección http://172.17.0.2 y nos aparece un formulario de login inicial para autentificar el acceso antes de mostrar el contenido de la web.

    ![image](https://github.com/user-attachments/assets/b6557f6e-bc5f-49e5-86a0-e5342cd584eb)

4. Para conocer las credenciales vamos a realizar un ataque Hydra de fuerza bruta:

    ![image](https://github.com/user-attachments/assets/260df42a-c93d-4776-8273-3f6383b40a68)

    Las claves son: usuario "***httpadmin***", contraseña: "***fhttpadmin***".

    Logramos entrar en la web pero se trata simplemente de un index del servidor Apache:

    ![image](https://github.com/user-attachments/assets/4465cb78-696b-473d-a6df-8e362433bd6a)

5. Aplicamos fuzzing web para hallar subdirectorios o contenido oculto existente, usando esta línea de código:

    **wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -z list,php,txt,html --hc 401,403,404 --basic httpadmin:fhttpadmin http://172.17.0.2/FUZZ.FUZ2Z**

    ![image](https://github.com/user-attachments/assets/fc8a27ed-af3d-41c2-9cae-a7393b8351a0)

	  Este resultado parece tratarse de otro login, vamos a revisarlo:

    ![image](https://github.com/user-attachments/assets/2b354aa7-6c65-43dc-be63-e842d0deb72e)

    ![image](https://github.com/user-attachments/assets/58165684-283e-4524-b1dd-ed8758c546e6)

6. Probamos el obtener la contraseña de un posible usuario "admin" con este script:

    ![image](https://github.com/user-attachments/assets/8e24c64e-ec22-489f-af29-89c498a4361c)

    ![image](https://github.com/user-attachments/assets/990ce963-fe2e-427b-9c56-645ef81ee516)

    Con el usuario "***admin***" encontramos la contraseña "***chocolate***".

    Al ingresar las claves encontradas en el formulario de inicio de sesión se nos menciona a un usuario llamado "balutin":

    ![image](https://github.com/user-attachments/assets/d1c99538-502a-4715-b939-e293db0593f9)

    Es posible que este se trate de un usuario del servicio SSH, por lo que con Hydra vamos a intentar obtener su contraseña:

    ![image](https://github.com/user-attachments/assets/57aec088-c148-4cee-a88d-3ca9ac3cf145)

    La contraseña es "***estrella***".

7. Nos conectamos por SSH:

   ![image](https://github.com/user-attachments/assets/b3155480-60bb-4789-b4bc-3238ded559c2)

   Hemos logrado acceder en el sistema con el usuario "balutin", por lo que ahora nos centraremos en escalar privilegios para conseguir ser el usuario "root".

8. Creamos otro script para obtener mediante un ataque de fuerza bruta la contraseña del usuario root:

   ![image](https://github.com/user-attachments/assets/a01fb8c9-7b24-4ffd-a905-b18cbdf55a34)

   ![image](https://github.com/user-attachments/assets/dc6aaf4e-b1e7-43e5-adcf-d1565a3224a3)

9. Finalmente, escribimos el comando **su** e ingresamos como el usuario root:

    ![image](https://github.com/user-attachments/assets/74bcd13f-c15c-442f-a435-145ff3664a1a)

10. Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

    ![image](https://github.com/user-attachments/assets/6320e4ea-f429-4725-8b6e-f59f61f61c79)
