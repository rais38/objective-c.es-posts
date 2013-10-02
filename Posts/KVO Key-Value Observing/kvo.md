Como comenté en el post sobre [KVC](http://objective-c.es/kvc-key-value-coding/), **KVO** proporciona un mecanismo de notificación similar al que proporciona las clases [NSNotification y NSNotificationCenter](http://www.ingens-networks.com/blog/post/2012/07/10/Patrones-de-diseno-en-Objective-C-Observador-NSNotificationCenter.aspx) pero con una **diferencia principal**. En lugar de un objeto central(*NSNotificationCenter*) que difunde las notificaciones a todos los objetos que han sido registrados como observadores, **KVO** notifica directamente a los objetos que están observando cuando ocurre un cambio en la propiedad del objeto observado.

Para poder utilizar **KVC** y **KVO** necesitamos que nuestros *setters* y *getters* cumplan con el estándar de [KVC](http://developer.apple.com/library/ios/#documentation/cocoa/conceptual/KeyValueCoding/Articles/AccessorConventions.html). No asustarse con el enlace anterior porque (de nuevo) **Apple** nos pone las cosas más fáciles a los desarrolladores Cocoa, ya que si utilizamos las *properties* de Objective-C 2.0 no tendremos problemas porque ya cumplen el estándar que exige **KVC**.

##Registrarnos como observadores

Registrarse como observador es muy sencillo. Sólo tenemos que llamar al método *addObserver:forKeyPath:options:context:*.

    [self.car addObserver:self forKeyPath:@"model" options:(NSKeyValueObservingOptionNew) context:nil];
    
En este caso, el observador es el objeto desde la que estamos llamando a *addObserver:forKeyPath:options:context:* y el objeto observado es *self.car*. La *keyPath* especifica el atributo que queremos observar, en *options* establecemos unas indicaciones a **KVO** de cómo queremos que nos envíe las notificaciones. Podemos establecer más de una separándolas por el operador "|". A continuación, os indico una tabla con los posibles valores:

Valor | Descripción
------------ | -------------
*NSKeyValueObservingOptionNew* | Envía el nuevo valor.
*NSKeyValueObservingOptionOld* | Envía el antiguo valor.
*NSKeyValueObservingOptionInitial* | Se envía una notificación al observador inmediatamente.
*NSKeyValueObservingOptionPrior* | Se envía actualizaciones separadas antes y después de que se haya ejecutado dicha modificación, en vez de solo una cuando se haya modificado.

##Implementar Callbacks en KVO

Lo siguiente que tenemos que hacer para utilizar **KVO** es escribir el método *callback* del observador *observeValueForKeyPath:ofObject:change:context:*.

    - (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
        NSString *newValue = [change objectForKey:NSKeyValueChangeNewKey];
         
        NSLog(@"Valor %@ --> %@", keyPath, newValue);
    }
    
Podemos ver que el parámetro *change* es de tipo *NSDictionary*, el cual contiene las claves y los valores asociados con la información modificada que solicitamos cuando nos registramos como observador. Aquí tenemos la lista completa de dichas claves y valores:

Clave | Valor
------------ | -------------
*NSKeyValueChangeKindKey* | *NSNumber* que especifica el tipo de cambio.
*NSKeyValueChangeNewKey* | Nuevo valor.
*NSKeyValueChangeOldKey* | Antiguo valor.
*NSKeyValueChangeIndexesKey* | Si *NSKeyValueChangeKindKey* es uno de estos: *NSKeyValueChangeInsertion*, *NSKeyValueChangeRemoval* o *NSKeyValueChangeReplacement*, este tendrá los indices de los valores cambiados.
*NSKeyValueChangeNotificationIsPriorKey* | Se utiliza junto con *NSKeyValueChangeOptionPrior* para indicar la notificación previa.

La clave *NSKeyValueChangeKindKey* especifica el tipo de cambio que se nos está notificando. Aquí la lista con los posibles valores:

Clave | Valor
------------ | -------------
*NSKeyValueChangeSetting* | Especifica que el valor se está definiendo.
*NSKeyValueChangeInsertion* | Especifica que los valores se están insertando, como en una colección o en una relación "uno a varios".
*NSKeyValueChangeRemoval* | Especifica que los valores se están eliminando de una relación "uno a varios".
*NSKeyValueChangeReplacement* | Especifica que los valores se están sustituyendo en una relación "uno a varios".

##Eliminarnos como observadores

Como pasa con *NSNotificationCenter*, cuando ya no necesitamos observar más cambios de un objeto tenemos que eliminarnos como observadores sino podríamos generar un error que haría que la aplicación se cerrase.

    - (void)dealloc {
        [self.car removeObserver:self forKeyPath:@"model"];
    }
    
##SHOW ME THE CODE

Vamos a poner en práctica la teoría anterior. Primero crearemos una sencilla clase(*INGCar*) con dos *properties*, las cuales observaremos con una segunda clase(*INGCarObserver*).

*INGCar.h*

    #import <Foundation/Foundation.h>
     
    @interface INGCar : NSObject
     
    @property (nonatomic, retain) NSString *model;
    @property (nonatomic, retain) NSString *color;
     
    @end
    
*INGCarObserver.h*

    #import <Foundation/Foundation.h>
     
    @class INGCar;
     
    @interface INGCarObserver : NSObject
     
    @property (retain) INGCar *car;
     
    - (id)initWithCar:(INGCar *)inCar;
     
    @end
    
*INGCarObserver.m*

    #import "INGCarObserver.h"
    #import "INGCar.h"
     
    @implementation INGCarObserver
     
    - (id)initWithCar:(INGCar *)inCar
    {
        if (self = [super init]) {
            self.car = inCar;
             
            // Nos añadimos como observadores de ambas propiedades y nos notificará con el antiguo y nuevo valor
            [self.car addObserver:self forKeyPath:@"model" options:(NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld) context:nil];
            [self.car addObserver:self forKeyPath:@"color" options:(NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld) context:nil];
        }
         
        return self;
    }
     
    -(void)dealloc
    {
        // Nos eliminamos como observadores de ambas propiedades
        [self.car removeObserver:self forKeyPath:@"model"];
        [self.car removeObserver:self forKeyPath:@"color"];
    }
     
    #pragma mark - Callback KVO
     
    - (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
    {
        NSString *newValue = [change objectForKey:NSKeyValueChangeNewKey];
        NSString *oldValue = [change objectForKey:NSKeyValueChangeOldKey];
         
        NSLog(@"Valor %@ --> Antes: %@ - Después: %@", keyPath, oldValue, newValue);
    }
     
    @end
    
Ya solo nos faltará añadir nuevos *imports* a nuestro *Appdelegate.m* y modificar *application:didFinishLaunchingWithOptions:*.

*INGAppDelegate.m*

    #import "INGCarObserver.h"
    #import "INGCar.h"
     
    @implementation INGAppDelegate
     
    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
    {
        INGCar *car = [[INGCar alloc] init];
        car.model = @"Modelo1";
        car.color = @"Rojo";
         
        INGCarObserver *carObserver = [[INGCarObserver alloc] initWithCar:car];
         
        car.model = @"Modelo2";
        car.color = @"Verde";
    }
    
Si ejecutamos este código, en la terminal tendríamos algo similar a esto:

<img src="http://objective-c.es/wp-content/uploads/2013/01/image-1.png" alt="Resultado en terminal" width="719" height="105" class="size-full wp-image-950" /></a>

##Notificaciones Manuales

Todas estas notificaciones se generan de manera automática pero hay ocasiones que nos podría interesar hacerlo de manera manual. Por ejemplo, cuando tenemos muchas notificaciones y nos gustaría agruparlas y enviarlas una sola vez.
Para hacer esto, primero de todo tenemos que sobrescribir el método de clase *automaticallyNotifiesObserversForKey:* indicando que deseamos hacerlo de manera manual.

    + (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key {
        // Podemos hacer que las notificaciones manuales sean solo para una determinada propiedad
        if ([key isEqualToString:@"someVar"]) {
            return NO;
        }
         
        return YES;
    }
    
Luego de esto, tenemos que llamar a *willChangeValueForKey:* antes de cambiar el valor y *didChangeValueForKey:* después del cambio.

    [carObserver willChangeValueForKey:@"model"];
    car.model = @"Modelo2";
    [carObserver didChangeValueForKey:@"model"];
    
Fuente: [Ingens Blog](http://www.ingens-networks.com/blog/post/2012/11/15/KVO-Key-Value-Observing.aspx)