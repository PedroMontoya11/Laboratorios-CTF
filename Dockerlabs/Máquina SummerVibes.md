Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh summervibes.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/4cef2145-7574-4f41-9c00-233ceeef4e5c)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/7964cb07-993b-4735-86d6-c9f215cde720)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP*** respectivamente.

Vamos a intentar atacar mediante HTTP. Nos dirigimos al navegador y en el buscador ponemos http://172.17.0.2:80.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/dfaf6e60-4e0f-4594-8b73-f51cf94fa32d)

Revisando el código fuente de la página nos damos cuenta que aparece la siguiente línea comentada:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0a426f3f-d4e9-4620-91b0-30a3fa43e4f4)

Esto quiere decir que hay una página de inicio correspondiente al servicio web **cmsms**. Revisamos escribiendo http://172.17.0.2/cmsms o realizando fuzzing con un diccionario que contenga cmsms entre las opciones.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/efd914ef-d353-48d0-918e-bda07056e7cd)

Este servicio se encuentra en la versión 2.2.19, la cual es vulnerable al **ataque SSTI** (*Server-Side Template Injection* o inyección de plantillas del lado del servidor).
Buscamos vulnerabilidades que se puedan explotar a esta versión del CMS Made Simple: [https://github.com/capture0x/CMSMadeSimple2](https://github.com/capture0x/CMSMadeSimple2)

El ataque consta de los siguientes pasos:
  1. Iniciar como administrador y navegar a **Layout > Design Manager > Breadcrumbs**
  2. Hacemos clic en editar e insertamos los siguientes payloads: {7*7}, {$smarty.version}, {{7*7}}.
  3. Clic en Apply y después en Submit.
  4. Visitamos la página http://172.17.0.2/cmsms/index.php?page=templates-and-stylesheets para comprobar el resultado.

Primero, tendremos que verificar que existe una página de administración disponible mediante fuzzing.

**ffuf -w /usr/share/wordlists/directory-list-lowercase-2.3-medium.txt:FUZZ -u http://172.17.0.2:80/cmsms/FUZZ**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0f55eb80-4257-4b6d-8f7b-c5fa79be9d2e)

Esto es lo que vemos en la dirección que encontramos:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/52929133-0168-4d78-a456-c2080175a981)

Ahora, vamos a interceptar el tráfico con el uso del proxy de Burp Suite, y con la información obtenida de dicho tráfico de red realizaremos el ataque de fuerza bruta para obtener las claves del administrador.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ae4665df-34f3-4d62-af75-74a819f97436)

**hydra -l admin -P passwords.txt 172.17.0.2 http-post-form "/cmsms/admin/login.php:username=^USER^&password=^PASS^&loginsubmit=Submit:User name or password incorrect" -f**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/7005529e-250f-424d-b3f2-d0a26e1779ae)

Probamos con estas claves en el login:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0d92f48a-2eee-4ee4-abc9-c168734e714f)

Estamos dentro de la Administración del servicio CMS Made Simple.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/ef3bd0e8-63bf-4393-8b91-653a6598f98c)

Ahora podemos realizar el ataque SSTI siguiendo los pasos de arriba.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/8817354f-42ba-4941-9924-3b1a4dd05156)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/d5f8e25f-c935-466d-866c-df4588d66087)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/236d31c7-0bd6-46a1-9c04-9c094a28ae7a)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/3fadc980-f554-41af-acdd-027338d5d53d)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0208d035-268f-4811-94cd-9f7a8c93a9fe)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2c2f6ed6-d12b-4d84-bd87-628a004f401c)

Otra vulnerabilidad presente en esta versión de CMS Made Simple es el **ataque RCE** (*Remote Code Execution*). Los pasos son los siguientes:

- Dentro de la Administración de CMS nos dirigimos a **Extensions>User Defined Tags>custom_copyright**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/66e82e71-ba06-438d-aa16-229cfac58519)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/e1a855f5-3857-4aca-a46c-e75d4bb66f85)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/c06057e5-8561-4f37-b67e-55ef9a42fadc)

- Escribimos esto: ***<?php echo system('id'); ?>***

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/d697f289-601b-4a1a-80ee-c1990d7e6bcb)

- Cuando hacemos clic en **Run** se nos muestra una ventana emergente que nos da información sobre los ID del sistema. Como podemos ver, somos el usuario www-data con ID 33.
  
  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/0d7be775-5a8e-4ce5-9a48-49104204df13)

Sabiendo que podemos ejecutar código desde esa ventana, buscamos en Internet la página web [https://www.revshells.com/](https://www.revshells.com/) , que utilizaremos para realizar una reverse shell y ejecutarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/f6eda275-c143-4f5a-a17b-9896e9bfdd47)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/24492737-d3c3-4dd5-875a-2f56ba0bbbbc)

Estamos dentro del sistema con el usuario *www-data*.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/4620019f-9584-4a93-ae17-0be3a6bc7e25)

Ahora, haremos unos ajustes en la terminal y posteriormente trataremos de acceder como el usuario root mediante un script de bash con el que obtendremos las claves del usuario.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/4d43b231-a4ae-434c-aa42-1448f8ae5f6a)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/14c9bb71-c971-4278-8166-a0e5068427ac)

  Vamos a ejecutar el .sh que se encuentra en [esta dirección web](https://github.com/Maalfer/Sudo_BruteForce) para obtener la contraseña del usuario root y así implementar la escalada de privilegios de manera correcta.

  **bash script.sh root passwords.txt**

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/1733ec1c-39d5-4917-8f8d-87fd165b1252)

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/189a06da-482d-45b2-a510-413f595d6858)

<ins>**La escalada de privilegios ha funcionado correctamente. Somos el usuario "root"**</ins>.
    
  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/45411f84-cf05-4204-8a02-4f0536732d67)

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/c43d6cde-2952-4cab-9610-9eef27e16684)
