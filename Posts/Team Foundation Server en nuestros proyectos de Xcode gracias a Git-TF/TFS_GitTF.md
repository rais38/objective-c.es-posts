Como sabréis, existe una [lista](http://en.wikipedia.org/wiki/Comparison_of_revision_control_software) extensa con distintos controles de versiones. En mi carrera profesional he podido utilizar varios como *CVS* o *SVN* (Control de versiones centralizado) pero desde que desarrollo con Cocoa los que más utilizo son los llamados "control de versiones distribuidos" como *Mercurial* o *Git*, sobretodo este último.

<img src="http://objective-c.es/wp-content/uploads/2012/12/logo_git-300x165.png" alt="Logo Git" title="Logo Git" width="300" height="165" class="aligncenter size-medium wp-image-717" />

No podemos olvidar eso de *"No sólo de Git vive el hombre"* y tenemos que ver más allá, ya que existen algunas opciones con unas características realmente interesantes. El objetivo de esta entrada no es vender *Team Foundation Server* (a partir de ahora *TFS*) pero por varios motivos, en mi empresa actual nos hemos decantado por el uso de este control de versiones. *TFS* está perfectamente integrado en *Visual Studio* (faltaría más ya que es un producto de *Microsoft*) y en *Eclipse* existe un [plugin](http://marketplace.eclipse.org/content/tfs-plug-eclipse#.UMinfJPm6zA) desde hace tiempo. ¿Y en *Xcode*? Eso mismo me preguntaba yo, ya que antes de verano era un suplício el poder integrarlo. Se utilizaba [SVN Brigde](http://svnbridge.codeplex.com/) y teníamos que hacer modificaciones en nuestro *IIS* y en nuestra máquina local. En definitiva, cuando me enteré de todo el follón que tenía que montar para poder hacer que *Xcode* y *TFS* trabajasen juntos, pensé muy seriamente en presentar mi carta de dimisión ya que no sé que sería peor ;).

<img src="http://objective-c.es/wp-content/uploads/2012/12/dolor_de_cabeza.jpeg" alt="Dolor de cabeza" title="Dolor de cabeza" width="449" height="178" class="aligncenter size-full wp-image-727" />

Por suerte, *Microsoft* fue misericordioso con los desarrolladores *Cocoa* y presentó (13 de Agosto 2012) Git-TF para Visual Studio Team Foundation Server 2012.

###Git-TF

Es un conjunto de herramientas para la terminal que nos facilita el uso de repositorios locales *Git* con *TFS*. Nos ayuda en poder clonar fuentes y buscar actualizaciones desde *TFS* como también nos permite enviar *commit* locales a *TFS*. En otras palabras, nosotros podremos seguir trabajando a nivel local con *Git* pero *Git-TF* nos hará de "traductor" y se enviarán los cambios de manera casi transparente a *TFS*.

##Instalación

Primero de todo, tenemos que descargarnos *Git-TF* desde el [Download Center](http://www.microsoft.com/en-us/download/details.aspx?id=30474) de Microsoft. Luego lo descomprimimos en local y apuntamos el *path* de dicho directorio ya que nos servirá más adelante (en mi caso */Users/\*\*\*\*\*\*/Git-TF*).

Ahora tenemos que añadir unas rutas a la variable de entorno [PATH](http://www.bonillaware.com/configurar-variable-entorno-path-mac). El fichero **.profile** por defecto no está creado en OSX pero para curarnos en salud ejecutaremos este comando desde la *Terminal*:

    # El caracter ~ lo tenemos en OSX (Español ISO) con alt+ñ
    ls -la ~/ | grep profile
    
Si el resultado no devuelve nada significará que el *.profile* no esta creado, de lo contrario tendremos que editar el ya existente. Para crearlo ejecutamos en la terminal:

    # Si no nos encontramos en nuestro directorio HOME debemos ir a ella
    cd ~
    touch .profile
    
Una vez creado, lo abrimos con nuestro editor favorito. Nosotros ejecutaremos *TextEdit*:

    open .profile
    
Ahora añadiremos dos nuevas rutas a PATH, para ello guardamos *.profile* con este contenido:

    export PATH="/Applications/Xcode.app/Contents/Developer/usr/libexec/git-core/":$PATH
    export PATH="/Users/*****/Git-Tf/":$PATH
    
La segunda línea se corresponde a la ruta donde se encuentra *Git-TF* pero ¿y la primera?. Desde *Xcode* 4.3 la ruta del binario de *Git* no es añadido al user path y no tenemos todos los comandos disponibles de *Git* en la terminal. Hacer la prueba antes y después de añadirlo al *.profile* y vereis.

##Uso

Para poder usar *Git-TF* necesitamos tener instalado en nuestra máquina el runtime de Java. En mi caso al ejecutar por primera vez el comando *git-tf* me salió un *alert* dándome la opción de instalarlo.

<img src="http://objective-c.es/wp-content/uploads/2012/12/runtime_java-e1355950371268.png" alt="Instalar runtime Java" title="Instalar runtime Java" width="387" height="250" class="aligncenter size-full wp-image-738" />

Una vez instalado, ya podemos trabajar. Nos podemos encontrar en varias situaciones:

* Tenemos el proyecto ya creado en Xcode y queremos subirlo a TFS

<!-- -->

    # Configuramos este proyecto para que Git-TF suba los cambios a la colección y proyecto que le indicamos
    git-tf configure http://servidor.tfs.com:8080/tfs/iOSCollection $/Proyecto
     
    # Realizamos commit a nuestro repositorio local Git
    git commit -a -m "Commit para TFS"
     
    # Comprueba los cambios realizados en el repositorio Git y los sube a TFS
    git-tf checkin
    
* El proyecto ya existe en TFS y lo queremos clonar en nuestro equipo

<!-- -->

    git tf clone --deep http://servidor.tfs.com:8080/tfs/iOSCollection $/Proyecto
    # Introducimos nuestras credenciales y empezará la descarga del proyecto
    Username: ethomson
    Password: 
    Connecting to TFS...
    Cloning $/Proyecto into /Users/rafael/Proyecto: 100%, done.                   
    Cloned 14 changesets. Cloned last changeset 19 as fb65c36
    
Podeis encontrar el listado completo de comandos que te ofrece en el [manual oficial](http://webcache.googleusercontent.com/search?q=cache:o24wsDlL6RoJ:download.microsoft.com/download/A/E/2/AE23B059-5727-445B-91CC-15B7A078A7F4/Git-TF_GettingStarted.html+&cd=2&hl=es&ct=clnk&gl=es).

Fuente: [Ingens Blog](http://www.ingens-networks.com/blog/post/2012/12/12/Team-Foundation-Server-en-nuestros-proyectos-de-Xcode-gracias-a-Git-TF.aspx)