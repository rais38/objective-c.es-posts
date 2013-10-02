Los programadores experimentados (y no tan experimentados) en Cocoa, estamos muy acostumbrados a implementar [patrones de diseño](http://goo.gl/NAIun) en nuestros desarrollos.

Un patrón de diseño que nos podemos encontrar de manera habitual es el "[delegado](http://goo.gl/5Wyw0)". Este lo hemos utilizado en artículos anteriores como puede ser el de "[Detectar toques fuera de los límites de una UIView](http://objective-c.es/detectar-toques-fuera-de-los-limites-de-una-uiview/)" cuando implementábamos nuestro propio protocolo delegado *INGTouchViewDelegate* para poder detectar los toques fuera de una *UIView*.

El delegado es una opción muy buena cuando nos encontramos en el caso de que tenemos una relación de "uno a uno" entre el objeto que delega y su delegado pero ¿qué pasaría si necesitáramos que hubiera más de un objeto delegado (múltiples observadores)? pues deberíamos implementar el patrón "[observador](http://goo.gl/oi9C)".

##Patrón observador

Este define un patrón donde un objeto (o varios) se puede registrar como observador de otro. Cuando suceda un evento en concreto, el remitente (objeto emisor) envía al centro de notificaciones dicha notificación y este se encargará de notificar a los objetos que previamente se hayan dado de alta como observadores. La implementación del patrón observador en Objective-C se realiza utilizando la clase *NSNotificationCenter* con la que proporcionaremos un sistema de envío global. Visto así puede resultar algo lioso pero vamos a ver un ejemplo básico de cómo implementarlo.

##Show me the CODE

Vamos a ver un ejemplo en el cual contaremos el número de veces que el usuario sale de nuestra aplicación a través del botón home de nuestro dispositivo.
 
Primero de todo, el objeto emisor debe estar preparado para enviar notificaciones al *NSNotificationCenter* cuando ocurra el evento. Para ello, añadiremos el siguiente código a nuestro *AppDelegate.m*:

    // A este método no solamente se le llamará cuando 
    // pulsemos el botón home de nuestro dispositivo, 
    // sino también cuando se interrumpa temporalmente 
    // nuestra app como puede ser el caso de llamadas entrantes
     
    - (void)applicationWillResignActive:(UIApplication *)application
    {
        // El parámetro NSString es el que especifica el nombre del mensaje 
        // a enviar, seguido del objeto que manda el aviso
        [[NSNotificationCenter defaultCenter] postNotificationName:@"ButtonHomeNotification" object:self];
    }
    
Ahora tendremos que registrar el observador en el centro de notificaciones y también implementar el método que mostrará el total de veces que se ha marchado el usuario de nuestra app, para ello incluiremos lo siguiente en nuestro *ViewController*:

    @interface ViewController () {
    // Variable de instancia que contará el número 
    // de veces que el usuario sale de nuestra app
    int count;
    }
    
    @end
     
    @implementation ViewController
     
    - (void)viewDidLoad
    {
        [super viewDidLoad];
         
        // Especificamos el nombre de la notificación que
        // queremos observar y opcionalmente podemos
        // indicar el objeto que deseamos contemplar.
        // Si especificamos 'nil' como parámetro de objeto, 
        // recibiremos notificaciones de todos los elementos que envia
        // dicha notificación.
         
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(receiveButtonHomeNotification:) name:@"ButtonHomeNotification" object:nil];
    }
     
    - (void)viewDidUnload
    {
        [super viewDidUnload];
        // Release any retained subviews of the main view.
    }
     
    -(void)receiveButtonHomeNotification:(NSNotification*) notification
    {
        count++;
        NSLog(@"Valor de contador es: %i", count);
    }
     
    @end    

Lo ejecutamos en nuestro terminal y cada vez que presionemos el botón *home*, nos aparecerá lo siguiente en la ventana del debugger de Xcode:

<a href="http://objective-c.es/wp-content/uploads/2013/02/debugger_xcode_nsnotificationcenter.png"><img src="http://objective-c.es/wp-content/uploads/2013/02/debugger_xcode_nsnotificationcenter.png" alt="Debugger Xcode" width="721" height="206" class="aligncenter size-full wp-image-1026" /></a>

##Eliminarnos como observador

¿Porqué es tan importante? Si no lo hacemos, podemos generar un error que hará que la aplicación se cierre ya que tenemos un objeto liberado pero todavía sigue referenciado en NSNotificationCenter. Para ello en nuestro *ViewController* reemplazaremos el método *dealloc* como el que muestro a continuación:

    - (void)dealloc {
        // Eliminarnos como observador  
        [[NSNotificationCenter defaultCenter] removeObserver:self];
     
        [super dealloc];
    }
    

Fuente: [Ingens Blog](http://www.ingens-networks.com/blog/post/2012/07/10/Patrones-de-diseno-en-Objective-C-Observador-NSNotificationCenter.aspx)