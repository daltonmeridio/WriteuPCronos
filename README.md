# WriteUp Cronos

CronOS se enfoca principalmente en diferentes vectores para la enumeración y también enfatiza los riesgos asociados con agregar archivos de escritura mundial al crontab raíz. Esta máquina también incluye una vulnerabilidad de inyección SQL de nivel introductorio.

| Información | Cronos |
| --- | --- |
| Categorías | Web, Red, Evaluación de vulnerabilidad |
| Área de interés | Inyección de comandos del sistema operativo, deserialización,
Transferencia de zona |
| Lenguajes | PHP, Python |

## Matriz de máquina

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled.png)

## Reconocimiento

- Enumeración con NMAP:
    
    Inicialmente usamos nmap para hacer un análisis de puertos a la maquina, esto nos servirá para saber que puertos y servicios tiene corriendo la maquina.
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%201.png)
    
    **-sS**: Para el análisis de puertos TCP SYN
    
    **—min-rate**: para que envié 5000 paquetes por segundo (En esta ocasión usamos esta opción           dado que estamos en un entorno controlado)
    
    **-p-** : Nos analiza todo el rango de puerto, los 65536 puertos.
    
    **-n** : Para que nos quite la resolución DNS
    
    **-Pn**: Deshabilita la detección de host
    
    Todo esto lo vamos a pasar al fichero allPorts en formato Grepeable, pues se van a extraer los puertos para su análisis en versiones de puertos y servicios, esto lo haremos con la utilidad de S4vitar extractPorts.
    
    La extracción de puertos nos dice que se ejecuta el puerto 80 esta abierto con lo que podemos adelantar un poco de trabajar y lanzar la herramienta **Whatweb** para revisar las tecnologías de la pagina web.
    
    Extracción de Puertos:
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%202.png)
    
    Ahora podemos hacer la enumeración de los puertos y servicios, seguimos usando nmap, ya con los puertos en mano usamos la opción **-sCV** que de las opciones combinadas **-sC** y **-sV** nos intenta determinar los servicios que se ejecutan y nos lanza scripts predeterminados.
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%203.png)
    
    Podemos observar que hay un puerto que llama la atención, este puerto es el 53, no es muy usual que se encuentre expuesto puesto que nos puede dar pie a la enumeración de DNS. 
    
    Revisión de las tecnologías, con la herramienta ya dicha, podemos ver que tecnologías corren en la pagina web, haciendo búsqueda de vulnerabilidades para ellas, que en este caso no aplica.
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%204.png)
    

## ****DNS****

- Enumeración TCP/UDP Puerto 53
    
    Domain Name System (DNS) es una parte integral de Internet. Por ejemplo, a través de nombres de dominio, como [academy.hackthebox.com](https://academy.hackthebox.com/) o [www.hackthebox.com](https://www.hackthebox.eu/) , podemos llegar a los servidores web que el proveedor de alojamiento ha asignado una o varias direcciones IP específicas.
    
    Antes de enumerar el DNS debemos resolver la ip, esto lo podemos hacer con **nslookup** apuntando hacia la maquina.
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%205.png)
    
    Esto exitosamente nos devuelve un nombre de dominio y confirma el dominio **coronos.thb** esto podemos incluirlo en nuestro archivo **/etc/hosts**
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%206.png)
    
    Ahora si podemos hacer enumeración de DNS, ya con la información obtenida, lo primero que se me viene a la mente es hacer una transferencia de zona 
    
    ### Transferencia de zona DNS
    
    Una transferencia de zona es un tipo de transacción DNS. Es uno de los muchos mecanismos disponibles para los administradores repliquen la base de datos DNS entre un conjunto de servidores DNS.
    
    Sucede que a través de ellos se llega a recolectar información de una **red corporativa**, exponiendo en ocasiones sus **direcciones IP** internas, servidores y equipos. Para recolectar esta información debe usarse el parámetro “**axfr**” (a este tipo de ataque también se lo denomina [AXFR](https://tools.ietf.org/html/rfc5936)) donde el comando queda de la siguiente manera:
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%207.png)
    
    Lo cual nos arroja información valiosa, pues podemos ver el subdominio **admin** y la pagina **www.cronos.htb**
    
    Esto también lo guardamos en el archivo de **hosts**
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%208.png)
    
    Teniendo esto ya recopilado, podemos ver el recurso mas llamativo que es el portal del administrador.
    
    ## Inyección SQL
    
    Al entrar lo primero que vemos es un loggin, lo primero que pensé fue usar credenciales predeterminadas, como esto no funciono, pensé en un clásico ataque de inyección SQL
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%209.png)
    
    Lo cual fue exitoso.
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2010.png)
    
    Al entrar podemos ver que es una herramienta que genera un ping hacia una red, estuve jugando con esta y solo pude generar “ping” hacia la ip predeterminada. 
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2011.png)
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2012.png)
    
    Con esto en el tablero, decidí ver que pasaba por detrás en el banckend, me dispuse a usar **Burp** 
    
    ![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2013.png)
    
    Al interceptar la petición puedo ver que se ejecutan unos comandos, los cuales me dan el indicio de que puede tratarse de una inyección de comandos 
    

## Explotación

Empecé probando si omita un carácter especial, para así poder seguiendo con la enumeración  

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2014.png)

Lo cual nos regresa una salida exitosa, entonces comencé a jugar con esa entrada, en este sentido hay que tener cuidado ya que, al cometer errores en la entra la pagina se traba y hay que iniciar con la interceptación de nuevo. 

 

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2015.png)

Jugando con la entrada de la ip podemos ver que con el carácter especial y aplicando el comando **ls** podemos listar, lo cual nos confirma la ejecución de comandos.

Con esto podemos obtener una shell, aquí hay dos caminos, hacerlo directamente desde la web cargando una shell o con el mismo burp.

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2016.png)

En esta parte usare Burp ejecutando una shell de bash, que puedes obtener de [aquí](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md) y tendremos que poner un puerto a la escucha. 

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2017.png)

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2018.png)

Con el usuario obtenido, podemos elevar privilegios a ROOT

## Escalación de privilegios

Intente una serie de comandos para enumerar el sistema sin tener éxito, por lo que procedí a usar [LinPeas](https://github.com/rebootuser/LinEnum), esta misma la tuve transferir al sistema levantando un servidor con python, y copiándola al directorio **/tmp**

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2019.png)

De toda la información que nos arroja la herramienta y después de unas horas de búsqueda, podemos rescatar lo siguiente 

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2020.png)

Un trabajo cron que se ejecuta en  directorio atrás del cual nos arrojo la shell, esto es importante ya que vemos que es el usuario root el que lo esta ejecutando. Entonces podemos revisar este directorio, y vemos que somos tenemos permisos de escritura en el archivo. 

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2021.png)

Al leer el documento podemos ver que se trata de un archivo de PHP 

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2022.png)

Dado que somos propietarios de este escript podemos escribir en el, con lo cual podríamos reemplazarlo para generar una shell, para elevar a root.

Para esto, podemos obtener una shell de php [aquí](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). Vamos a repetir el proceso anterior, vamos a descargar la shell en el sistema a través de un servidor de python que levantare. 

En el uso de la revshell, debemos alterar estos campos con nuestra ip y puerto a donde se conectara.

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2023.png)

Realizamos la descarga 

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2024.png)

Y reemplazamos el archivo 

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2025.png)

Ya realizado esto y con el puerto a la escucha, solo queda esperar un minuto a que se ejecute el archivo para que nos de la shell de root 

![Untitled](WriteUp%20Cronos%2064a27ba995c544bbad9fc1aa4debf7e1/Untitled%2026.png)

Muy bien, espero que te gustara, si te fue de ayuda para aprender nuevas cosas, por favor comparte. 

Happy Hacking.
