Como pasó en el artículo de [las notificaciones Push][1], hay mucha información y por ello he tenido que dividir el artículo en dos partes. En esta primera haré una introducción a las *In-App Purchases*.

## Introducción

Mucha gente piensa que los únicos modelos de negocios existentes en el mundo de las app son dos: Aplicación gratuita o de pago. Esa idea es errónea porque existen muchas más. Una de las más interesantes podrían ser las *In-App Purchases*. ¿Porqué este modelo es especial al resto? algunas razones:

*   Aparte del dinero que ganes con la venta de tu app, puedes conseguir dinero extra ofreciendo ciertos servicios/contenidos a tu app.
*   Puedes poner tu aplicación gratuita (llegará a muchos más usuarios) y luego puedes restringir ciertas funcionalidades a que solo estén disponibles si pagan previamente.

Las In-App Purchase ofrecen a los usuarios contenidos digitales adicionales, funcionalidad, servicios y suscripciones, incluso dentro de una aplicación de pago o gratuita.

[<img src="http://objective-c.es/wp-content/uploads/2013/05/confirmar_compra-300x200.png" alt="Confirmar compra" width="300" height="200" class="aligncenter size-medium wp-image-1408" />][2]

Algunos ejemplos de In-App Purchase podrían ser:

*   Libros digitales
*   Elementos adicionales a juegos
*   Suscripción a revistas digitales

[<img src="http://objective-c.es/wp-content/uploads/2013/05/inapppurchase.jpeg" alt="In App Purchase" width="297" height="268" class="aligncenter size-full wp-image-1411" />][3]

**StoreKit** es el framework que nos proporciona la funcionalidad para procesar los pagos de los artículos ofrecidos en la aplicación. In-App Purchase utiliza los mismos términos comerciales utilizados por las aplicaciones que se venden en la App Store y Mac App Store. Apple se lleva el 30% de los beneficios del precio de compra de cada artículo vendido dentro de la app y los pagos se harán mensualmente.

## Categorías y tipos

Hay cuatro categorías de producto soportadas:

*   Contenido
*   Funcionalidad
*   Servicios
*   Suscripciones

Estas categorías deben estar dentro de uno de los tipos de compra siguientes:

*   Consumibles
*   No consumibles
*   Suscripciones auto-renovables
*   Suscripciones gratuitas
*   Suscripciones no renovables

Al implementar las In-App Purchase en nuestra aplicación tenemos que tener en cuenta lo siguiente:

*   Solo se pueden vender bienes digitales dentro de la misma aplicación. Jamás se podrá vender bienes/servicios del mundo real.
*   Los productos deben estar disponibles en todos los dispositivos que tenga el usuario configurado con su Apple ID.
*   Los productos adquiridos no se pueden compartir con otras aplicaciones o plataformas.
*   Los productos no pueden contener o estar relacionado con: Pornografía, difamación...

## Tipos de In-App Purchase

Tenemos que asegurarnos cual es el tipo **adecuado** para cada producto antes de crearlo en iTunes Connect.

### Consumibles

Se deben comprar cada vez que el usuario necesite ese elemento. Ejemplo: Consumibles en un juego (munición, vida extra, puntos de salud…)

### No Consumibles

Son los que se compran solo una vez y están disponibles en todos los dispositivos que el usuario tenga registrado con su Apple ID. Este tipo de compra se suele utilizar en servicios que no expiran. Ejemplos: Niveles extra en un juego, libros o revistas.

### Suscripciones auto-renovables

Permite al usuario comprar contenido por episodios o acceso a cierto contenido por tiempo limitado. Al final de cada período, la suscripción se renovará, hasta que el usuario lo cancele.

[<img src="http://objective-c.es/wp-content/uploads/2013/05/PurchaseCredit-200x300.png" alt="Purchase Credit" width="200" height="300" class="aligncenter size-medium wp-image-1414" />][4]

### Suscripciones gratuitas

Este tipo se implementa igual que las “auto-renovables”, la única excepción es que no lleva cargo al usuario. Estas suscripciones no tienen caducidad pero el usuario puede cancelarlas en cualquier momento.

### Suscripciones no renovables

Permite la venta de servicios con una duración limitada. Este tipo debe ser utilizado para las compras que ofrecen el acceso a contenido basado en el tiempo. Ejemplo: Suscripción de una semana a la funcionalidad de guía por voz dentro de una aplicación de navegación.

[<img src="http://objective-c.es/wp-content/uploads/2013/05/NoRenovable-300x215.jpeg" alt="No renovable" width="300" height="215" class="aligncenter size-medium wp-image-1417" />][5]

## Categorías de las In-App Purchase

### Contenido

Los productos que se encuentran dentro de esta categoría podrían ser libros digitales, fotografías, revistas, niveles de juego y todo aquel producto que pueda ser entregado a través de tu aplicación.
 
Tenemos que asegurarnos que los artículos comprados están disponibles en todas las instancias de tu aplicación en todos los dispositivos que tenga el usuario, incluso después que la aplicación sea eliminada, reinstalada o descargada en un dispositivo nuevo. Para restaurar los ítems comprados en un nuevo dispositivo o cuando se vuelve a instalar la aplicación después de borrarla, utilizaremos de nuevo el framework **StoreKit**. Para ello deberemos llamar al método *restoreCompletedTransactions*. Una transacción será creada por cada artículo que tengamos comprado y la procesaremos de manera similar a cuando se realiza una nueva solicitud de pago.
 
Los artículos consumibles son la única excepción a la exigencia de que estén disponibles en todos los dispositivos del usuario. Estos artículos desaparecen después de su uso y no pueden ser reutilizados. Estos artículos no serán devueltos cuando se llame al método *restoreCompletedTransactions*.
 
Podemos elegir dos maneras con la que mandar al usuario estos contenidos:

* Se encuentra dentro del bundle de la aplicación y se activará una vez que el usuario haga la compra.
* Descargar el contenido desde la app a un servidor externo (en el ejemplo de la parte II, utilizaremos este).

Los de esta categoría, por norma general serán del tipo “No consumibles” a excepción que se espere que sean utilizados una única vez lo que serán “Consumibles”.

### Funcionalidad
 
Podemos vender y desbloquear funcionalidad extra en nuestras aplicaciones con las “In-App Purchase”. Estas funcionalidades “extras” normalmente se deberán considerar “no consumibles”.
 
### Servicios
 
Estos deben ser normalmente “No consumibles” o “Suscripción no renovable”. La diferencia está en que si el servicio está limitado a un periodo de tiempo concreto.
 
### Suscripciones
 
Las renovaciones de las suscripciones son manejadas automáticamente por la AppStore y la facturación también.
El usuario será recordado de la renovación de la suscripción poco antes que finalice el mismo.
 
Para las suscripciones “no renovables”, si deseamos permitir a los usuarios renovarlas tendremos que estar pendiente de la expiración de los mismos y hacerlo de forma manual. Si el usuario elige renovarla, tendremos que iniciar una nueva petición de compra en **StoreKit**.

En la segunda parte, crearemos una app en la que reproduciremos un fichero de audio (mp3) que deberemos comprar y se descargará desde un servidor externo.


Fuente: [Ingens Blog](http://www.ingens-networks.com/blog/post/2012/05/31/In-App-Purchases-en-iOS-Parte-I.aspx)


 [1]: http://objective-c.es/envio-de-notificaciones-en-ios-parte-1/
 [2]: http://objective-c.es/wp-content/uploads/2013/05/confirmar_compra.png
 [3]: http://objective-c.es/wp-content/uploads/2013/05/inapppurchase.jpeg
 [4]: http://objective-c.es/wp-content/uploads/2013/05/PurchaseCredit.png
 [5]: http://objective-c.es/wp-content/uploads/2013/05/NoRenovable.jpeg