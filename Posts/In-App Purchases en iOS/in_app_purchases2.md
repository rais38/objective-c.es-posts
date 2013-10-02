En la [primera parte][1] se vió una introducción a las *In-App Purchases*, categorías y tipos. Ahora veremos un ejemplo práctico, en concreto, desarrollaremos una aplicación que comprará un archivo de audio que no se encuentra en el bundle. Una vez comprada y descargada, podremos reproducirla en nuestro dispositivo.

Antes de entrar a "picar código", necesitamos 3 cosas: *App ID / Provisioning Profile*, Usuario de pruebas para las *"In-App Purchases"* y productos que vender.

## APP ID / Provisioning Profile

Como ya he comentado en anteriores artículos. Para poder hacer uso de las In-App Purchases, notificaciones *Push* y/o *Game Center*, necesitamos que el App ID que estemos utilizando para firmar nuestra app no contenga comodines (wildcard).

**Incorrecto**: com.miempresa.*

[<img src="http://objective-c.es/wp-content/uploads/2013/05/appIDIncorrecto.jpeg" alt="App ID Incorrecto" width="793" height="52" class="aligncenter size-full wp-image-1428" />][2]

**Correcto**: com.miempresa.miapp

[<img src="http://objective-c.es/wp-content/uploads/2013/05/appIDCorrecto.jpeg" alt="App ID Correcto" width="798" height="54" class="aligncenter size-full wp-image-1432" />][3]

Para este ejemplo utilizaremos el App ID creado anteriormente "PushTest". Si tienes alguna duda de como crearlo en [este post][4] se explica con más detenimiento.

## Usuario de pruebas para las "In-App Purchases"

Debemos crear al menos una cuenta de prueba para cada región que la aplicación esté localizada. Las cuentas de prueba deben ser nuevas y únicas. No se pueden reutilizar las ya existentes. Solo los usuarios administradores y técnicos están autorizados para crearlos. Estos usuarios no tienen acceso a iTunes Connect, pero serán capaces de probar las In-App Purchases en un entorno de desarrollo de un dispositivo de prueba registrado.

Para crearlo seguiremos los siguientes pasos:

*   Logarnos en iTunes Connect.
*   En la página principal, seleccionamos "Manage Users".
*   Seleccionamos el tipo de usuario "Test User".
*   Ahora nos aparecerá el listado de usuarios, si no hemos creado ninguno previamente estará vacío. Para añadirlo tendremos que hacer click en "Add new user".
*   Metemos toda la información en el formulario de registro y guardamos.

Tenemos que asegurarnos que el correo de la nueva cuenta no está asociada a ninguna cuenta ya configurada.

## Productos para vender

Las compras a través de una aplicación (In-App) se pueden hacer en aplicaciones gratuitas o de pago, tanto en iOS como en OSX. Para registrarlas deberemos añadir antes dicha aplicación en iTunes Connect y nos aparecerá un menú como este:

[<img src="http://objective-c.es/wp-content/uploads/2013/05/menuItunesConnect.png" alt="Menu Itunes Connect" width="234" height="226" class="aligncenter size-full wp-image-1435" />][5]

Una vez que hayamos accedido al apartado que nos interesa (*Manage In-App Purchases -> Create new*), deberemos elegir el tipo que será, en este caso será "No Consumible". El siguiente apartado nos pedirá toda la información del producto, lo rellenamos todo a excepción del apartado "*Screenshot for review*" que no será necesario para entornos de desarrollo pero una vez que lo queramos publicar en la AppStore, sí que deberemos incluir este apartado.

Una vez creado, deberemos apuntar el campo *Product ID* que es el que utilizaremos con *StoreKit* para realizar las transacciones. Podemos crear hasta 10000 productos por aplicación, número más que suficiente para la inmensa mayoría.

Ya tenemos todo lo necesario para entrar en materia.

## Entremos en matería

Primero de todo deberemos importar los frameworks *StoreKit* (para trabajar con las transacciones de las *In-App Purchase*) y *AVFoundation* (es el que nos permitirá trabajar con audio).

Nuestro *.xib* tendrá 3 UIButtons: uno para hacer la petición de compra, otra para reproducir la canción comprada y la última para detener la reproducción de esta.

[<img src="http://objective-c.es/wp-content/uploads/2013/05/storyboard.png" alt="storyboard" width="391" height="548" class="aligncenter size-full wp-image-1438" />][6]

Desde Interface Builder vamos a hacer que los UIButtons de *Play* y *Stop* tengan la propiedad de *hidden* activada (esto es porque solo aparecerá dichos botones cuando hayamos realizado la compra y descarga de la canción). En el UIButton que utilizaremos para realizar la acción de comprar, pondremos de texto predeterminado "Esperando a conectar..." y lo modificaremos por código una vez que hayamos recibido respuesta de la AppStore y también pondremos la propiedad *Enabled* deshabilitada (mientras no recibamos respuesta de la AppStore no podremos hacer dicha petición de compra).
 
Tenemos que hacer lo siguiente:
 
* Crear los IBAction / IBOutlet de los 3 UIButtons.
* Crear dos propiedades del tipo SKProduct y AVAudioPlayer.
* Implementar los protocolos *SKProductsRequestDelegate* y *SKPaymentTransactionObserver*.

De esta forma nuestro ViewController.h quedaría así:

    @interface ViewController : UIViewController <SKProductsRequestDelegate, SKPaymentTransactionObserver>
     
    @property (nonatomic, strong) SKProduct *song;
    @property (nonatomic, strong) AVAudioPlayer *player;
    @property (strong, nonatomic) IBOutlet UIButton *button;
    @property (strong, nonatomic) IBOutlet UIButton *stop;
    @property (strong, nonatomic) IBOutlet UIButton *play;
     
    - (IBAction)buy:(id)sender;
    - (IBAction)play:(id)sender;
    - (IBAction)stop:(id)sender;
     
    @end
    
Tenemos que controlar que no existe la restricción (control parental) de las "Compras integradas" en el dispositivo como también inicializar la solicitud con los productos que nos interesan. Para ello modificamos el método *viewDidLoad* del archivo ViewController.m:

    - (void)viewDidLoad
    {
        [super viewDidLoad];
         
        // Comprobamos si hay alguna restricción configurada en el device respecto a las In-App Purchases.
        if ([SKPaymentQueue canMakePayments]) {
             
            NSLog(@"Puedo hacer pagos In-App");
             
            // Inicializamos la solicitud con los productos que nos interesen. En este caso solo mandamos el único que tenemos creado.
            SKProductsRequest *productRequest = [[SKProductsRequest alloc] initWithProductIdentifiers:[NSSet setWithObject:@"com.miempresa.miapp.prueba"]];
            productRequest.delegate = self;
            [productRequest start];
        }
        else {
            NSLog(@"Control parental activado");
        }
    }
    
El identificador que le pasamos, se corresponde con el *Product ID* que creamos anteriormente en iTunes Connect.
 
Ahora debemos implementar los métodos delegados correspondiente al protocolo *SKProductsRequestDelegate*:
 
* productsRequest:didReceiveResponse: -> Obtiene la respuesta de la AppStore (obligatorio).
* request:didFailWithError: -> Este método es llamado cuando falla la solicitud a la AppStore.

<!-- -->


    #pragma mark SKProductsRequestDelegate
        
    -(void)productsRequest:(SKProductsRequest *)request didReceiveResponse:(SKProductsResponse *)response
    {
        NSArray *products = response.products;
            
        NSLog(@"Número de productos devueltos: %i", products.count);
            
        if (products.count > 0) {
            self.song = [products objectAtIndex:0];
            [self.button setTitle:@"Comprar canción 0.99€" forState:UIControlStateNormal];
            // Habilitamos el botón de comprar
            [self.button setEnabled:YES];
        }
        else {
            NSLog(@"Productos no disponibles");
        }
            
    }
        
    -(void)request:(SKRequest *)request didFailWithError:(NSError *)error
    {
        NSLog(@"Error al conectar a la AppStore para las In-App Purchase: %@", error);
    }

Controlamos el número de productos que recibimos en la respuesta ya que podríamos solicitar un producto que ya no esté en venta o lo hayamos borrado por cualquier causa.
 
Ahora tenemos que añadir a la cola de pago la transacción para adquirir nuestra canción. Para ello tendremos que añadir lo siguiente en el IBAction de nuestro botón:

    - (IBAction)buy:(id)sender 
    {
        // Añadimos el producto que recibimos en el método delegado productsRequest:didReceiveResponse:
        SKPayment *payment = [SKPayment paymentWithProduct:self.song];
        // Nos añadimos a nosotros mismos como observadores de la transacción.
        [[SKPaymentQueue defaultQueue] addTransactionObserver:self];
        [[SKPaymentQueue defaultQueue] addPayment:payment];
    }
    
Antes implementamos el protocolo delegado *SKPaymentTransactionObserver* y algunos os preguntareis ¿Exactamente qué hacemos con él?. Es el que se encarga de ver como ha ido el proceso de la transacción de compra: si se ha comprado satisfactoriamente, si ha habido algún error en el proceso de compra.... Exactamente tenemos 5 estados en una transacción en StoreKit pero para este ejemplo solo utilizaremos *SKPaymentTransactionStateFailed* y *SKPaymentTransactionStatePurchased*.

    #pragma mark SKPaymentTransactionObserver
     
    -(void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions
    {
        for (SKPaymentTransaction *transaction in transactions) {
            switch (transaction.transactionState) {
                case SKPaymentTransactionStatePurchasing:
                     
                    break;
                     
                case SKPaymentTransactionStatePurchased:
                     
                    // Nos descargamos la canción
                    [self downloadFromUrl:[NSURL URLWithString:@"http://miweb.com/pruebas/cancion.mp3"]];
                     
                    NSLog(@"Comprado");
                     
                    // mostramos los botones
                    [self.play setHidden:NO];
                    [self.stop setHidden:NO];
                     
                    [[SKPaymentQueue defaultQueue] finishTransaction:transaction];
                     
                    break;
                     
                     
                case SKPaymentTransactionStateRestored:
                    [[SKPaymentQueue defaultQueue] finishTransaction:transaction];
                     
                    break;
                 
                case SKPaymentTransactionStateFailed:
                     
                    if (transaction.error.code != SKErrorPaymentCancelled) {
                        NSLog(@"Error en la transacción: %@", transaction.error.localizedDescription);
                    }
                     
                    [[SKPaymentQueue defaultQueue] finishTransaction:transaction];
                     
                    break;
            }
        }
    }
    
Ya solo faltaría implementar los métodos que nos permitirá descargarnos la canción desde un servidor externo, reproducir la canción y detener su reproducción.

    -(void)downloadFromUrl:(NSURL *)url
    {
        NSData *data = [NSData dataWithContentsOfURL:url];
        NSString *pathString = [self retrieveStringOfFile];
    
        [data writeToFile:pathString atomically:YES];
    }
     
    - (IBAction)play:(id)sender 
    {
        NSString *pathString = [self retrieveStringOfFile];
        NSError *error;
         
        self.player = [[AVAudioPlayer alloc] initWithContentsOfURL:[NSURL fileURLWithPath:pathString] error:&error];
         
        if (self.player == nil) {
            NSLog(@"Error al reproducir: %@", [error description]);
        }
        else {
            [self.player play];
        }
    }
     
    - (IBAction)stop:(id)sender 
    {
        NSString *pathString = [self retrieveStringOfFile];
        NSError *error;
         
        self.player = [[AVAudioPlayer alloc] initWithContentsOfURL:[NSURL fileURLWithPath:pathString] error:&error];
         
        if (self.player == nil) {
            NSLog(@"Error al reproducir: %@", [error description]);
        }
        else {
            [self.player stop];
        }
    }
    
    - (NSString *)retrieveStringOfFile
    {
        NSArray *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
        NSString *documentDirectory = [path objectAtIndex:0];
        NSString *file = [NSString stringWithFormat:@"%@/cancion.mp3", documentDirectory];
    
        return file;
    }
    
El resultado lo podréis ver [aquí](https://www.youtube.com/watch?v=KLR0p5EOhQY).

## Extra

[@jmoreno78](https://twitter.com/jmoreno78) ha escrito un [artículo](http://javi.zinkinapps.com/blog/2013/05/22/helios-iii-piratas-y-bucaneros/) sobre como trabajar con IAP en [Helios](http://helios.io/), concretamente con la librería [CargoBay](https://github.com/mattt/CargoBay) de [@mattt](https://twitter.com/mattt). Os recomiendo mucho que le echéis un vistazo.


Fuente: [Ingens Blog](http://www.ingens-networks.com/blog/post/2012/06/04/In-App-Purchases-en-iOS-Parte-2.aspx)

 [1]: http://objective-c.es/in-app-purchases-en-ios-parte-1/
 [2]: http://objective-c.es/wp-content/uploads/2013/05/appIDIncorrecto.jpeg
 [3]: http://objective-c.es/wp-content/uploads/2013/05/appIDCorrecto.jpeg
 [4]: http://objective-c.es/envio-de-notificaciones-en-ios-parte-2/
 [5]: http://objective-c.es/wp-content/uploads/2013/05/menuItunesConnect.png</avfoundation></storekit>
 [6]: http://objective-c.es/wp-content/uploads/2013/05/storyboard.png