Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh los40ladrones.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>La dirección IP de la máquina a vulnerar siempre es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/bdb9371c-4fc2-4570-8d96-7e4105e35dad)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/3d679c73-5cce-4b52-8ed4-39014c929241)

Podemos ver que se encuentra abierto únicamente el puerto ***80***, correspondiente al servicio ***HTTP***.

Buscamos en el navegador http://172.17.0.2:80 y vemos que nos sale.

  ![image](https://github.com/user-attachments/assets/847ba6fa-d1a7-4347-89ce-b05374b206d2)

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información. Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*js*, .*txt*.

  ![image](https://github.com/user-attachments/assets/6a81b7b3-0746-42c7-a08e-7a7b999e1b4e)

Nos encontramos con un fichero de texto llamado **qdefense.txt**. Procederemos a visualizar su contenido en la web.

  ![image](https://github.com/user-attachments/assets/4264b501-095f-40f6-850f-2328d417612a)

En este archivo encontramos información interesante, como un posible usuario del sistema llamado "**toctoc**" y una secuencia de números que se asemeja a una sucesión de puertos (**7000 8000 9000**). Este código parece tratarse de un ***ataque port knocking***, que consiste una técnica en la cual se establece en un firewall una secuencia de puertos, y si alguien se intenta conectar en orden a esos puertos preestablecidos, se abre un puerto que antes estaba cerrado.

Para realizar esto, usamos la herramienta *knock* (no se encuentra por defecto en Kali Linux, por lo que debemos instalarla primero) e insertamos en la terminal **knock 172.17.0.2 7000 8000 9000**:

  ![image](https://github.com/user-attachments/assets/41683657-5b86-4f7b-ba0b-2e4289deb7e2)

Volvemos a realizar un escaneo de puertos para comprobar si se ha abierto algún puerto adicional:

  ![image](https://github.com/user-attachments/assets/cd392cc0-3fc1-426a-80cc-d11099cab5dc)

Ahora también contamos con el puerto de ***SSH*** (puerto ***22***) abierto, por lo que podemos mediante un ataque Hydra obtener la contraseña del SSH del posible usuario "toctoc".

  ![image](https://github.com/user-attachments/assets/7ee1c16c-ed66-4dfb-8183-4b0356b95da1)

La contraseña del servicio SSH del usuario "toctoc" es "**kittycat**".

Nos conectamos por SSH:

  ![image](https://github.com/user-attachments/assets/71a90422-c885-419b-a995-5efa16f8c979)

Estamos dentro de la máquina víctima.

Miramos con el comando **sudo -l** los archivos binarios que puede ejecutar el usuario actual de manera privilegiada.

  ![image](https://github.com/user-attachments/assets/3e4699d6-456b-41c0-b760-07495440cb03)

Nos encontramos con 2 archivos que podemos ejecutar para realizar una escalada de privilegios, y así poder ser usuario root. De estos dos, nosotros haremos énfasis en ***/opt/bash***, el cual parece ser un fichero que simula al interpretador de código empleado por el sistema operativo Linux para entender el código insertado por el usuario en la CLI.

Escribimos en la terminal **sudo /opt/bash** para arrancarlo:

  ![image](https://github.com/user-attachments/assets/354380e1-fb14-4548-bf44-8eb41c4c7346)

Finalmente somos usuario root, por lo que contamos con todos los privilegios disponibles en el sistema.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/5da9ef9a-0bd0-464a-a179-43968d3d6f6c)
