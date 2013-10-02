Para el seguimiento correcto de esta segunda parte se presupone que se tienen unas nociones básicas en el sistema de notificaciones de Apple. Si no es así, te recomiendo que leas antes [la primera parte][1].

Sé que a los desarrolladores *Cocoa*, el asunto de los "provisioning profiles", "App ID´s"...nos da muchos dolores de cabeza pero es algo necesario para las notificaciones Push. Así que, vamos a dar un repaso para poder configurarlo correctamente.

### Provisioning Portal

Para enviar notificaciones Push a una aplicación/dispositivo necesitamos el "device token" de dicho dispositivo, un certificado para nuestro servidor y poder firmar la aplicación con un "provisioning profile" que tenga habilitada las push.

#### Certificado

*   Logarnos en el *Provisioning portal*.
*   Creamos un App ID para hacer pruebas con las notificaciones Push.
*   **(IMPORTANTE)** En el *Bundle identifier* del App ID, no debe contener comodines (wildcard). Deberemos colocar el bundle identifier completo, ejemplo: `com.miempresa.miapp`.
*   Hacemos click en *Configure* y luego presionamos el botón para empezar el asistente para crear un nuevo certificado Push (Existen dos tipos, uno de desarrollo y otro de producción. Para las pruebas utilizaremos el de desarrollo).
*   Descargamos el certificado y hacemos doble click en el fichero `aps_developer_identity.cer` para importar a nuestro llavero.
*   Lanzamos "Acceso a llaveros" (En Aplicaciones -> Utilidades o lo buscamos en Spotlight) y hacemos click en la categoría "Mis certificados".
*   Expandimos *Apple Development iOS Push Services*, seleccionamos este apartado y el elemento que despliega (tu key privada).
*   Botón derecho y elegimos "Exportar 2 items…" y lo guardamos como `server_certificates_bundle_sandbox.p12`.
*   Abrimos la Terminal y cambiamos de directorio a donde hayamos guardado `server_certificates_bundle_sandbox.p12` y convertimos el fichero p12 (PKCS12) a PEM utilizando este comando (cuando os pida la password, introducís el indicado en el paso anterior):

<!-- -->

    openssl pkcs12 -in server_certificates_bundle_sandbox.p12 -out server_certificates_bundle_sandbox.pem -nodes -clcerts
    

##### ¿Porqué este último paso?

Para este ejemplo vamos a utilizar como *Provider* nuestra propia máquina local y para ello vamos a utilizar una librería PHP que nos facilitará mucho la tarea [ApnsPHP][2] (trabaja mejor con el formato PEM).

Desde el 22 de diciembre del 2010, el servicio de notificaciones Push de Apple empezaría a utilizar certificados TLS/SSL de 2048 bits que proporcionará más seguridad en las conexiones entre nuestro servidor (provider) al servidor de Apple.

> To ensure you can continue to validate your server's connection to the Apple Push Notification service, you will need to update your push notification server with a copy of the 2048-bit root certificate from Entrust's website.

Para ello tan solo deberemos ejecutar lo siguiente en la Terminal:

    wget https://www.entrust.net/downloads/binary/entrust_2048_ca.cer -O - > entrust_root_certification_authority.pem
    

### Provisioning Profile

Una vez que tenemos la App ID creada y configurada para las notificaciones Push, podemos crear el provisioning profile (development) correspondiente como lo hemos hecho hasta ahora.

## Creamos el proyecto

Creamos un proyecto nuevo, yo lo he llamado "pushTest". Veremos que esta parte es la más fácil.

En el método `application:didFinishLaunchingWithOptions:` del *AppDelegate.m*, incluiremos lo siguiente:

    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
    {
        self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
        // Override point for customization after application launch.
        self.viewController = [[ViewController alloc] initWithNibName:@"ViewController" bundle:nil];
        self.window.rootViewController = self.viewController;
        [self.window makeKeyAndVisible];
    
        // Nos registramos para recibir las notificaciones Push de los tipos especificados
        [[UIApplication sharedApplication] registerForRemoteNotificationTypes:(UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound)];
    
        return YES;
    }
    

Esta llamada le dirá al OS que nuestra app quiere recibir notificaciones Push. Deberemos arrancarla en un dispositivo real y tener instalado el "provisioning profile" que generamos en el punto anterior. Nos deberá aparecer una pantalla similar a esta:

Ahora deberemos implementar los siguientes métodos:

- *application:didRegisterForRemoteNotificationsWithDeviceToken:* -> Cuando el dispositivo recibe el token por parte de Apple.
 
- *application:didFailToRegisterForRemoteNotificationsWithError:* -> Cuando ha habido un error al registrar el dispositivo para el token.
 
- *application:didReceiveRemoteNotification:* -> Cuando se ha recibido una notificación

<!-- -->

    #pragma mark Push Notifications Methods
     
    - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
    {
        NSLog(@"Mi device token es %@", deviceToken);
    }
     
    // Lo podemos comprobar en el simulador ya que en este no podemos probar las notificaciones Push
    - (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
    {
        NSLog(@"Error al obtener el token. Error: %@", error);
    }
     
    - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
    {
        NSLog(@"Contenido del JSON: %@", userInfo);
    }
    
Cuando arranquemos la app, nos deberá aparecer algo similar a esto en la consola de Xcode:

    Mi device token es:
    <9aff4707 a47cf74f 887c25d4 8e326894 5f6ba01d a5dab387 147c7eec 81ca78fc>
    
Este es el código que utilizaremos con [ApnsPHP](http://code.google.com/p/apns-php/) (sin espacios '9aff4707a47cf74f887c25d48e3268945f6ba01da5dab387147c7eec81ca78fc').

##Enviar Notificación con APNSPHP

Nos descargamos las fuentes de [ApnsPHP](http://code.google.com/p/apns-php/) y podemos utilizar este código de ejemplo:

    <?php
    /**
     * @file
     * sample_push.php
     *
     * Push demo
     *
     * LICENSE
     *
     * This source file is subject to the new BSD license that is bundled
     * with this package in the file LICENSE.txt.
     * It is also available through the world-wide-web at this URL:
     * http://code.google.com/p/apns-php/wiki/License
     * If you did not receive a copy of the license and are unable to
     * obtain it through the world-wide-web, please send an email
     * to aldo.armiento@gmail.com so we can send you a copy immediately.
     *
     * @author (C) 2010 Aldo Armiento (aldo.armiento@gmail.com)
     * @version $Id$
     */
     
    // Adjustar tu timezone
    date_default_timezone_set('Europe/Madrid');
     
    // Reportar todos los errores de PHP
    error_reporting(-1);
     
    // Usando Autoload.php, cargamos todas las clases necesarias
    require_once 'ApnsPHP/Autoload.php';
     
    // Crear instancia de un nuevo objeto ApnsPHP_Push
    $push = new ApnsPHP_Push(
            ApnsPHP_Abstract::ENVIRONMENT_SANDBOX,
            'server_certificates_bundle_sandbox.pem'
    );
     
    // Set the Root Certificate Autority to verify the Apple remote peer
    $push->setRootCertificationAuthority('entrust_root_certification_authority.pem');
     
    // Conectar al APNS
    $push->connect();
     
    // Crear una instancia de un nuevo mensaje con un solo destinatario
    $message = new ApnsPHP_Message('9aff4707a47cf74f887c25d48e3268945f6ba01da5dab387147c7eec81ca78ad');
     
    // Establecer el identificador de la notificación
    $message->setCustomIdentifier("identificador-mensaje");
     
    // Establecer badge "3"
    $message->setBadge(3);
     
    // Texto de la notificacion
    $message->setText('Texto prueba!');
     
    // Reproducir sonido por defecto
    $message->setSound();
     
    // Establecer una propiedad fuera del 'aps'
    $message->setCustomProperty('acme2', array('bang', 'whiz'));
     
    // La notificacion expira en 30 segundos
    $message->setExpiry(30);
     
    // Añade el mensaje. Se pueden añadir varios
    $push->add($message);
     
    // Se envían todos los mensajes
    $push->send();
     
    // Desconectar del APNS
    $push->disconnect();
     
    // Examinamos si ha existido algún error
    $aErrorQueue = $push->getErrors();
    if (!empty($aErrorQueue)) {
            var_dump($aErrorQueue);
    }

Ya solo faltaría ejecutar el script anterior. Lo podemos hacer directamente desde la Terminal y si todo ha salido bien nos tendría que llegar la notificación a nuestro terminal en unos segundos:

    php miScript.php
    

Fuente: [Ingens Blog](http://www.ingens-networks.com/blog/post/2012/05/24/Envio-de-notificaciones-en-iOS-2.aspx)


 [1]: http://objective-c.es/envio-de-notificaciones-en-ios-parte-1/
 [2]: http://code.google.com/p/apns-php/