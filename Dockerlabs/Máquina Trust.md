Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh trust.tar** en la carpeta donde se encuentre el contenido extraído del zip.
   <ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/9e701fbe-6af4-487c-8165-e49d2d310852)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/581f6795-fa22-43b4-9193-1abde8c36e79)

Podemos ver que se encuentran abiertos los puertos ***22*** y ***80***, correspondientes a los servicios ***SSH*** y ***HTTP*** respectivamente.
Vamos a intentar atacar mediante HTTP. Nos dirigimos al navegador y en el buscador ponemos http://172.17.0.2:80.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/fb487f2f-5e05-42e3-823b-759329138901)

Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .html, .php, .sh y .py.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/79bfb2df-5f5d-4ab5-b466-ad16acbd7007)
  
Hemos hallado  una página llamada *secret.php*, por lo que vamos a buscarla en el navegador para ver su contenido.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/831e1cfa-eb51-49e5-962f-45b906479380)

En base al contenido de esta página podemos pensar que el usuario del ssh sea el usuario mario. Por lo tanto, vamos a intentar descifrar la contraseña de dicho usuario para con el usuario y la contraseña poder acceder a la máquina vulnerable mediante SSH. En mi caso, yo tengo un script que comprueba que contraseña es válida para dicho usuario (para esto puedes hacer uso de un diccionario como el rockyou o uno personalizado, para tener más éxito a la hora de obtener la contraseña). Yo usaré un diccionario personalizado de las 200 contraseñas más utilizadas. Realizamos un ataque de fuerza bruta Hydra usando lo siguiente: **hydra -l mario -P passwords.txt ssh://172.17.0.2**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/d9a68494-c3b2-4bb0-afdd-2ea3a41c67fa)
Al parecer la contraseña del usuario mario es **chocolate**, vamos a comprobarlo conectándonos por SSH a la máquina con las claves obtenidas.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2f1cebe4-c630-44ae-8d32-b088c046fe30)

Luego, una vez tenemos acceso a la máquina realizamos una escalada de privilegios para tener acceso como el usuario root al dispositivo, pudiendo ejecutar comandos y acceder a directorios que requieren de permisos de *sudo user*:

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/242c9c2f-939e-43d8-9139-a948f23112c1)
  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/b32041b4-5320-40e8-b6cf-05278f5614c6)
Hacemos clic en **Enter** para aplicarlo y salimos del vim.

Hemos accedido como el usuario root.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/185dcc4b-94c0-412f-a4d9-6d6885dc3444)
  
Otra manera de realizar esto mismo es escribiendo **sudo -u root /usr/bin/vim  -c ':!/bin/bash'**.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/4821353d-4279-4357-863c-8f3a51c8070f)

<ins>**Igualmente logramos acceder como root**</ins>.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/PedroMontoya11/Laboratorios-CTF/assets/145665312/2fbf1255-9593-4207-8cb4-93c8a1ffe3ba)
