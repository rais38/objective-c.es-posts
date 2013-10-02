Este tipo de *feature* la utilizamos a diario la gran mayoría. Gracias a ellas podemos compartir recursos multiplataforma (*iOS*, *Android*, *HTML5*, *Desktop*…) y lanzar aplicaciones pudiendo delegar en ellas determinadas acciones. Un ejemplo podría ser cuando copiamos el enlace de una canción en Spotify y se lo enviamos a un amigo por medio de un correo o un cliente de mensajería y cuando haga click se le abrirá dicha canción para que pueda escucharla.

[caption id="attachment_641" align="aligncenter" width="476"][<img src="http://objective-c.es/wp-content/uploads/2012/12/copiar_uri.png" alt="Copiar Uri" title="Copiar Uri" width="476" height="285" class="size-full wp-image-641" />Copiar Uri[/caption]

[caption id="attachment_652" align="aligncenter" width="538"]<img src="http://objective-c.es/wp-content/uploads/2012/12/cont_uri.png" alt="Contenido de la URI anteriormente copiada" title="Contenido de la URI anteriormente copiada" width="538" height="91" class="size-full wp-image-652" />Contenido de la URI anteriormente copiada[/caption]

Es muy fácil crear nuestras propias “**URI Scheme**” en *iOS* y vamos a ver un ejemplo de cómo implementarlo en nuestro propio proyecto.

## Registrar nuestra propia URL Scheme personalizada

Primero de todo, tendremos que añadir nuestra **URL Scheme** personalizada al proyecto y eso lo haremos en el fichero *Info.plist*:

*   Añadimos una nueva fila e indicamos la key "*URL Types*".

<img src="http://objective-c.es/wp-content/uploads/2012/12/url_types.png" alt="URL types" title="URL types" width="518" height="258" class="aligncenter size-full wp-image-656" />

*   Vemos que tenemos el elemento 0 que contiene la key *URL Identifier*. Aquí podremos introducir el valor que queramos pero yo utilizaré el mismo que el *Bundle identifier*.

<img src="http://objective-c.es/wp-content/uploads/2012/12/url_identifier.png" alt="URL identifier" title="URL identifier" width="576" height="115" class="aligncenter size-full wp-image-661" />

*   Añadimos una nueva fila pero esta vez le pondremos de key *URL Schemes* con el valor (esquema) que queramos que empiece nuestra *URL Scheme*. Ejemplo: miapp://

<img src="http://objective-c.es/wp-content/uploads/2012/12/url_schemes.png" alt="URL Schemes" title="URL Schemes" width="518" height="113" class="aligncenter size-full wp-image-664" />

Ya solo tenemos que guardar y arrancarlo en nuestro dispositivo. Veremos que con la app instalada (no hace falta ni que esté arrancada), cuando escribimos en Safari "miapp://loquesea" o pulsamos un enlace en un correo que tenga en el *href* eso mismo, veremos que se abrirá nuestra aplicación.

Pero ¿qué pasaría si le pasamos unos parámetros para que cuando se abra el enlace haga algo determinado? (como el ejemplo anterior de Spotify). Ahora mismo no haría nada pero si incluimos el método *application:openURL:sourceApplication:annotation:* en el AppDelegate.m si que haría algo:

    #pragma mark -
    #pragma mark URL Schemes
    
    -(BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
    {
      if (!url) {
        return NO;
      }
    
      // Nuestra URL Scheme de ejemplo es miapp://ejemplo/loquesea
    
      NSString *hostPath = [url host]; // Valor = ejemplo
      NSArray *arrayUrlPath = [url pathComponents];
      NSString *urlPath = [arrayUrlPath objectAtIndex:1]; // Valor = loquesea
    
      if ([hostPath isEqualToString:@"ejemplo"]) {
        // Hacer lo que sea
      }
    
      return YES;
    }
    

Para comprobar si podemos abrir una *URL Scheme* personalizada:

    // Si la app de Facebook está instalada
    if([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"fb://"]]) {
        // do something...
    }
    // Si la app de Ebay está instalada
    if([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"ebay://"]]) {
        // do something...
    }
    

Una de las principales utilidades de esta funcionalidad es la posibilidad de integración de nuestra app con la de terceros. En este aspecto nos es de mucha ayuda la existencia de la web que os muestro a continuación: [handleOpenURL][1].

Esta web te muestra un listado de los *URL Schemes* conocidos hasta la fecha, entre las que podrían encontrarse las desarrolladas por nosotros ahora o en un futuro. Para ello, tan sólo tendremos que ponernos en contacto con los administradores del sitio.

## Averiguar URL Scheme desde servidor externo

Ya hemos visto como averiguar si podemos abrir una URL Scheme desde el dispositivo pero y ¿si queremos averiguarlo desde un servidor web (Ej: mostrar una web u otra al usuario dependiendo si tiene instalada nuestra app o no)? Esta cuestión se la planteó nuestro amigo [@ValentiGoClimb][2] y encontramos una posible solución [aquí][3].


Fuente: [Ingens Blog](http://www.ingens-networks.com/blog/post/2012/06/08/Lanzar-aplicaciones-iOS-a-traves-de-URL-personalizada-URL-Schemes.aspx)

 [1]: http://handleopenurl.com/
 [2]: https://twitter.com/ValentiGoClimb
 [3]: http://suhinini.me/2011/07/04/installing-ios-application-via-custom-scheme-url/