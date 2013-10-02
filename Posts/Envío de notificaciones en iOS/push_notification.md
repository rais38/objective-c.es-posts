En *iOS*, las aplicaciones no pueden hacer mucho en segundo plano. Sólo se les permite hacer un número determinado de acciones para que la batería dure más.

> ¿Qué pasa si sucede algo muy interesante y desea que el usuario sea notificado, incluso si no está usando la aplicación?

La **solución** para esto es utilizar las notificaciones. ¿Qué es una notificación? Información que se le muestra al usuario respecto a un evento en concreto.

Tenemos 3 tipos:

*   *Badge* en el icono de la aplicación 
*   Reproducir Sonido
*   Mensajes de texto

Los cuales pueden ser combinados como nos plazca.

## Push Vs Locales

Las notificaciones pueden ser *Push* o locales. La principal diferencia es que las primeras son externas, es decir, provienen de un servidor de Apple (*APNS*), mientras que las segundas son, como su nombre indica, locales. Es decir, las genera la propia aplicación.

### Notificaciones Push

*   Se envían a través de un servidor
*   Su entrega es inmediata (si es posible la entrega sino se almacenará por un tiempo limitado)
*   Disponible para iOS y OSX (>= 10.7)

### Notificaciones Locales

*   Se generan en el mismo dispositivo y solo le llega al dispositivo que la ha creado
*   Su entrega puede ser programada
*   Solo para iOS

En este artículo nos centraremos en las notificaciones Push pero si queréis saber más sobre las locales, os recomiendo que leáis el artículo [Usando Local Notifications en iOS][1] de Fernando Rodríguez ([@frr149][2]).

## Ciclo de vida

[<img src="http://objective-c.es/wp-content/uploads/2013/04/ciclo_vida.png" alt="Ciclo vida" width="849" height="631" class="aligncenter size-full wp-image-1342" />][3]

1 – La aplicación tiene habilitada las notificaciones Push. El usuario tiene que confirmar que quiere recibir estas notificaciones.

2 – La aplicación recibe el “device token” del servidor de Apple, a partir de ahora APNS. Se puede ver este “device token” como la dirección que utilizará Apple para hacer llegar las notificaciones al dispositivo.

3 – La aplicación envía el “device token” a tu servidor y este lo almacenará.

4 – Cuando suceda algo de interés en tu aplicación o relacionado con ella, el servidor externo le enviará la notificación Push al APNS.

5 – APNS envía la notificación Push al dispositivo del usuario.

## ¿Qué necesitamos?

*   Dispositivo iOS: Las notificaciones Push solo funcionan en dispositivos reales y en ningún caso en el simulador.
*   Miembro del "iOS Developer Program": Es necesario crear una nueva App ID y “provisioning profile” para cada aplicación que utiliza Push, así como un certificado SSL para el servidor. Para ello, debemos tener acceso al “Provisioning portal”.
*   Servidor (en la documentación oficial lo encontrarás como "Provider")

## Estamos encargado de...

Apple nos facilita mucho el trabajo con las notificaciones pero nosotros como *Provider* somos responsables de estos aspectos:

*   Nos encargamos del contenido del "Payload"
*   Suministramos el número adecuado de los "badges"
*   Periódicamente deberemos conectarnos con el servidor web de “feedbacks” y buscar la lista actual de los dispositivos que no les llega en reiteradas veces estas notificaciones. Eso significará que el usuario a bloqueado las notificaciones de nuestra aplicación a su dispositivo así que deberemos dejar de enviarlos a esos dispositivos porque sino Apple nos podría Banear por “malas prácticas”.

## Anatomía de las notificaciones push

Como hemos comentado anteriormente, el *Provider* es responsable de crear los mensajes de notificación Push. Una notificación Push es un mensaje corto, que contiene el *device token*, Payload, y algunos bytes de información para el APNS. Existen dos tipos:

*   Formato de notificación simple

[<img src="http://objective-c.es/wp-content/uploads/2013/04/notificacion_simple.jpeg" alt="Notificación simple" width="522" height="111" class="aligncenter size-full wp-image-1347" />][4]

<!-- -->

*   Formato de notificación mejorada

[<img src="http://objective-c.es/wp-content/uploads/2013/04/notificacion_mejorada.jpeg" alt="Notificación mejorada" width="654" height="92" class="aligncenter size-full wp-image-1348" />][5]

La única diferencia entre ambas es que la "mejorada" tiene dos campos más:

* Identifier -> Un valor aleatorio con el cual se podrá identificar la notificación.
* Expiry -> Es una fecha UNIX (UTC), expresada en segundos e indica cuando la notificación no es válida y puede ser desechada. Por defecto Apple tiene ya un tiempo estimado para este campo pero si el nuestro no supera al de Apple este dará prioridad al que le indiquemos.
Si el valor es positivo, APNS intentará entregar la notificación al menos una vez. Podemos especificar cero o un valor menor que cero para solicitar que APNS no almacene la notificación.

##Payload

¿Qué es eso del Payload?. Es la parte que más nos interesa de la notificación. Aquí es donde construiremos la información de nuestra notificación.

Tiene las siguientes características:

* Debe ser un diccionario JSON
* Mapeado fácil a NSDictionary
* 256 bytes máximo
* Las keys fuera de "aps" no se utilizan en las notificaciones pero si podremos hacerlo para nuestro propio uso en la aplicación.

A continuación escribiremos un Payload de ejemplo:

    {
    "aps" : {
        "alert" : "Tienes 3 mensajes.",
        "badge" : 3,
        "sound" : "ringring.aiff"
    },
    "acme1" : "prueba",
    "acme2" : 14
    }
    
El sonido que se indica en el Payload deberá encontrarse en el bundle de la aplicación aunque podemos indicar un sonido por defecto(default).

    {
    "aps" : {
        "alert" : "Tienes 3 mensajes.",
        "badge" : 3,
        "sound" : "default"
    },
    "acme1" : "prueba",
    "acme2" : 14
    }
    
Como se ha indicado antes, esta información no debe superar los 256 bytes, así que se recomienda eliminar espacios, tabulaciones… etc. Ejemplo anterior:

    {"aps":{"alert":"Tienes 3 mensajes.","badge":3,"sound":"ringring.aiff"},"acme1":"prueba","acme2":14}
    
En este primer artículo, he intentado hacer un resumen de los conceptos teóricos. En la segunda parte entramos en materia y veremos un ejemplo práctico de como implementar las notificaciones Push.

Fuente: [Ingens Blog](http://www.ingens-networks.com/blog/post/2012/05/23/Envio-de-notificaciones-en-iOS-1.aspx)


 [1]: http://www.cocoaosx.com/2012/02/17/usando-local-notifications-en-ios/
 [2]: http://twitter.com/#!/frr149
 [3]: http://objective-c.es/wp-content/uploads/2013/04/ciclo_vida.png
 [4]: http://objective-c.es/wp-content/uploads/2013/04/notificacion_simple.jpeg
 [5]: http://objective-c.es/wp-content/uploads/2013/04/notificacion_mejorada.jpeg