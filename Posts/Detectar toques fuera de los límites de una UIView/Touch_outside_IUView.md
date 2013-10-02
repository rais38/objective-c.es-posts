Pongámonos en el caso que tenemos una subvista con un tamaño inferior a la vista padre y queremos hacer una acción determinada cuando pulsamos fuera de dicha subvista. Para que lo entendáis, sería algo similar al "popUp" de elección de cuenta de la app de **Tweetbot** para iPhone. Hay una manera de cerrar dicha ventana: presionamos fuera de los límites (*bounds*) de dicha vista.

<img src="http://objective-c.es/wp-content/uploads/2012/12/tweetbot_ejemplo-e1356462406548.png" alt="Tweetbot ejemplo" title="Tweetbot ejemplo" width="300" height="450" class="aligncenter size-full wp-image-763" />

¿Cómo detectar cuando el usuario toca fuera de una *UIView* en concreto? A continuación os propongo una solución muy fácil de incluir en cualquier proyecto.

##Comenzamos

Para este artículo, tenemos un *UIViewController* con un *UIButton*. Cuando presionamos el *UIButton* nos aparecerá la subvista con una imagen centrada.

<img src="http://objective-c.es/wp-content/uploads/2012/12/antes_de_pulsar-e1356462797566.png" alt="Antes de pulsar" title="Antes de pulsar" width="300" height="450" class="aligncenter size-full wp-image-769" />

<img src="http://objective-c.es/wp-content/uploads/2012/12/despues_de_pulsar-e1356462881468.png" alt="Después de pulsar" title="Después de pulsar" width="300" height="450" class="aligncenter size-full wp-image-772" />

##Subclase de UIView

Para continuar, necesitamos crear una subclase de *UIView*. En la cual incluiremos el método que comprobará si estamos tocando fuera o dentro de la *UIView*:

*INGTouchView.h*

    #import <UIKit/UIKit.h>
     
    // En el protocolo delegado "INGTouchViewDelegate", se definen los métodos que nuestra UIView llamará cuando haya algún toque.
     
    @protocol INGTouchViewDelegate
     
    // El método que usaremos para decirle a nuestro delegado si el toque ha sido dentro o fuera de la UIView.
     
    - (void) viewTouched:(BOOL)wasInside;
     
    @end
     
    @interface INGTouchView : UIView
     
    @property (nonatomic, weak) id delegate;
     
    @end
    
*INGTouchView.m*

    #import "INGTouchView.h"
     
    @implementation INGTouchView
     
    #pragma mark - Touches
    - (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event
    {
         
        if( point.x > 0 && point.x < self.frame.size.width && point.y > 0 && point.y < self.frame.size.height )
        {
            [_delegate viewTouched:YES ];
            return YES;
        }
         
        [_delegate viewTouched:NO ];
        return NO;
    }
     
    @end
    
Ahora editaremos nuestra subvista para que cumpla con el protocolo *INGTouchDelegate* e implementaremos el método delegado *viewTouched:*

*INGPopUpView.h*

    #import <UIKit/UIKit.h>
    #import "INGTouchView.h"
     
    @interface INGPopUpView : UIView <INGTouchViewDelegate>
     
    @property ( nonatomic , strong ) INGTouchView *touchView;
     
    @end
    
*INGPopUpView.m*

    #import "INGPopUpView.h"
     
    @implementation INGPopUpView
     
    -(id)init
    {
        self = [super initWithFrame:[UIScreen mainScreen].bounds];
        if (self) {
            // Oscurecemos el fondo y utilizamos otra UIView para que la propiedad alpha no nos afecte a toda la vista
            UIView *backgroundView = [[UIView alloc] initWithFrame:self.frame];
            backgroundView.backgroundColor = [UIColor blackColor];
            backgroundView.alpha = 0.5;
             
            [self addSubview:backgroundView];
             
            // Añadimos PopUp
            UIImageView *imageView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"popover.png"]];
            imageView.center = self.center;
             
            [self addSubview:imageView];
             
            // Le indicamos el mismo frame que el objeto del PopUp para poder controlar si estamos tocando fuera o dentro de la misma
            _touchView = [[INGTouchView alloc ] initWithFrame:imageView.frame];
            [self addSubview:_touchView];
            [_touchView setDelegate:self];
        }
         
        return self;
    }
     
    #pragma mark - INGTouchView Delegate Methods
     
    - (void) viewTouched:(BOOL)wasInside
    {
        if(wasInside) {
            NSLog(@"Toca dentro");
        }
        else {
            NSLog(@"Toca fuera");
        }
    }
     
     
    @end
    
Si ahora lo arrancamos en nuestro terminal o en el simulador, comprobaremos que cuando tocamos dentro o fuera de nuestro popUp nos saldrá un mensaje similar a este en la consola:

<img src="http://objective-c.es/wp-content/uploads/2012/12/NSLog_debugger.png" alt="NSLog debugger" title="NSLog debugger" width="571" height="142" class="aligncenter size-full wp-image-777" />

¿Notas algo raro? tranquilo, no necesitas ir al oculista. El que se llame tres veces al método *pointInside:withEvent:* se debe a la jerarquía de vistas y a la cadena de respuestas de eventos. Este tema lo tocaré en un próximo artículo pero si eres impaciente ;) aquí os dejo un par de enlaces con más información sobre la misma:

- [Handle Touch Events in UIWebView](http://www.idryman.org/blog/2012/06/18/handle-touch-events-in-uiwebview/)
- [Event Handling Guide for iOS](http://developer.apple.com/library/ios/#documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/EventsiPhoneOS/EventsiPhoneOS.html#//apple_ref/doc/uid/TP40009541-CH2-SW1)

Podéis descargar el proyecto [DetectTouches.zip](http://objective-c.es/wp-content/uploads/2012/12/DetectTouches.zip)

Fuente: [Ingens Blog](http://www.ingens-networks.com/blog/post/2012/10/19/Detectar-toques-fuera-de-los-limites-de-una-UIView.aspx)