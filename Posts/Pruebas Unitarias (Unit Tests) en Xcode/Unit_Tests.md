> "Eso de Unit Tests es lo que sale nada más al crear un nuevo proyecto en Xcode junto a ARC y Storyboards y no debo marcar ya que no tengo ni idea de lo que hace ¿no?"

[caption id="attachment_223" align="aligncenter" width="352"]<img src="http://objective-c.es/wp-content/uploads/2012/11/Captura-de-pantalla-2012-11-24-a-las-16.20.42.png" alt="Menú Xcode" title="Menu Xcode" width="352" height="88" class="size-full wp-image-223" />Menu Xcode[/caption]

¿Te sientes identificado con esta afirmación? Si tu respuesta es afirmativa, este artículo está especialmente dedicado para tí ya que después de leer este artículo, te maldecirás por no haberlo sabido antes y desearás implementarlo en la mayoría de tus proyectos actuales y futuros ;).

## ¿Qué son Pruebas Unitarias?

Son pruebas creadas por nosotros mismos en el que comprobamos que nuestra lógica desarrollada está funcionando de la manera correcta. Estas pruebas están especialmente indicadas para proyectos grandes en el que es muy complicado tener un control total de toda la lógica y al añadir nuevas funcionalidades podríamos **"cargarnos"** algo desarrollado anteriormente.<!--more-->

Con ellas tenemos más confianza al realizar cambios (en código propio o ajeno) e incluso las pruebas nos servirían como documentación.

### Tipos

Hay dos tipos de pruebas:

*   **Pruebas de lógica** (*Logic tests*): Se refiere a la prueba directa de la lógica de clases. Prueba la lógica de los métodos que no interactúan con elementos UI "[White-box testing][1]". Por ejemplo, tenemos un método muy sencillo que elimina elementos duplicados en un *NSArray* y una prueba sería comprobar que este método devuelva un *NSArray*. **Solo se pueden lanzar en el simulador**.
*   **Pruebas de aplicación** (*Application tests*): Comprueba que las interacciones con los controles UI y la propia UI de la aplicación te da los resultados esperados "[Black-box testing][2]". **Solo se pueden lanzar en un dispositivo**. 

Existen varios frameworks *Unit Testing* en *Objective-C*: *GHUnit*, *CATCH*, *OCUnit*… Desde [aquí][3] podeis ver el listado completo.

En este artículo trabajaremos con **Logic Tests** y **OCUnit**. He elegido este framework ya que es el que viene integrado con Xcode 4 aunque *GHUnit* es uno a tener muy en cuenta ya que nos proporciona características muy interesantes que hablaremos de ellas en un próximo post.

## Empecemos

Como he comentado antes, *OCUnit* viene integrado en Xcode así que implementarlo en nuestro proyecto es tan fácil como activar la casilla *Include Unit Tests* cuando lo creamos.

Una vez hecho esto, vemos que Xcode nos crea el proyecto como hasta ahora, a excepción de un nuevo grupo y Target que tienen el nombre de nuestro proyecto con la terminación *Tests*. En nuestro ejemplo es "PruebaUnitTestingTests".

<img src="http://objective-c.es/wp-content/uploads/2012/12/nuevo_grupo.png" alt="Nuevo grupo" title="Nuevo grupo" width="271" height="129" class="aligncenter size-full wp-image-414" />

Por defecto, Xcode nos crea un test de prueba así que lo vamos a lanzar. Para ello iremos a *Product* -> *Test*.

<img src="http://objective-c.es/wp-content/uploads/2012/12/Menu_Xcode.png" alt="Menu Xcode" title="Menu Xcode" width="331" height="436" class="aligncenter size-full wp-image-444" />

El resultado del test nos devuelve un error *"Unit tests are not implemented yet in PruebaUnitTestingTests"*. Esto es debido a *STFail* ya que este generará un fallo sí o sí.

<img src="http://objective-c.es/wp-content/uploads/2012/12/test_ejemplo.png" alt="Test ejemplo" title="Test ejemplo" width="811" height="100" class="aligncenter size-full wp-image-490" />

Veamos en profundidad el código.

#### *PruebaUnitTestingTests.h*

    #import <SenTestingKit/SenTestingKit.h>
    
    @interface PruebaUnitTestingTests : SenTestCase
    
    @end
    

Lo primero que vemos es que importamos el framework *SenTestingKit*.Este framework es en el que se basa *OCUnit*, así que siempre que queramos realizar pruebas unitarias tendremos que importar este framework. Cada *Test Suite* será una clase que hereda de *SenTestCase*.

#### *PruebaUnitTestingTests.m*

    #import "PruebaUnitTestingTests.h"
    
    @implementation PruebaUnitTestingTests
    
    - (void)setUp {
        [super setUp];
    
        // Set-up code here.
    }
    
    - (void)tearDown {
        // Tear-down code here.
    
        [super tearDown];
    }
    
    - (void)testExample {
        STFail(@"Unit tests are not implemented yet in PruebaUnitTestingTests");
    }
    

El método *setUp* se utiliza para inicializar objetos y *tearDown* para liberación de memoria. Estos dos métodos son opcionales. **Muy importante**, cada *"Test case"* debe ser un método que el prefijo del nombre del mismo debe ser *test*, sin parámetros.

El flujo general de una subclase de *SenTestCase* es la siguiente:

*   Llama +setUp de la clase
*   Por cada test: 
    *   Llama -setUp
    *   Llama -test_
    *   Llama -tearDown
*   Llama +tearDown

### *OCUnit* Macros

Anteriormente ya hemos visto una de las Macros *STFail* pero hay algunas más.

| Macro                        | Descripción                                                                                                    |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------- |
| STFail                       | Fallo incondicional                                                                                            |
| STAssertEqualObjects         | Fallo cuando dos objetos son diferentes                                                                        |
| STAssertEquals               | Fallo cuando dos valores son diferentes                                                                        |
| STAssertEqualsWithAccuracy   | Fallo cuando la diferencia entre los dos valores es mayor que el valor dado                                    |
| STAssertNil                  | Fallo cuando una expresión dada no es *nil*                                                                    |
| STAssertNotNil               | Fallo cuando una expresión dada es *nil*                                                                       |
| STAssertTrue                 | Fallo cuando una expresión dada es *false*                                                                     |
| STAssertFalse                | Fallo cuando una expresión dada es *true*                                                                      |
| STAssertThrows               | Fallo cuando una expresión dada no levanta una excepción                                                       |
| STAssertThrowsSpecific       | Fallo cuando una expresión dada no levanta una excepción de una clase en particular                            |
| STAssertThrowsSpecificNamed  | Fallo cuando una expresión dada no levanta una excepción de una clase en particular con el nombre especificado |
| STAssertNoThrow              | Fallo cuando una expresión dada levanta una excepción                                                          |
| STAssertNoThrowSpecific      | Fallo cuando una expresión dada levanta una excepción de una clase en particular                               |
| STAssertNoThrowSpecificNamed | Fallo cuando una expresión dada levanta una excepción de una clase en particular con el nombre especificado    |
| STAssertTrueNoThrow          | Fallo cuando una expresión dada es *false* o levanta una excepción                                             |
| STAssertFalseNoThrow         | Fallo cuando una expresión dada es *true* o levanta una excepción                                              |

Vamos a escribir un test muy sencillo en el que veremos algunas de estas Macros.

    - (void)testExample {
        NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://objective-c.es"]];
        NSURLResponse *response = nil;
        NSError *err = nil;
    
        NSData *data = [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&err];
    
        STAssertNotNil(response, @"Deberíamos tener respuesta");
        STAssertNil(err, @"No deberíamos tener error");
        STAssertTrue([data length] > 100, @"Deberíamos tener datos");
    }
    

Si tenemos conectividad, el resultado del test nos tendría que devolver 0 errores.

<img src="http://objective-c.es/wp-content/uploads/2012/12/log_sin_errores.png" alt="Log sin errores" title="Log sin errores" width="550" height="261" class="aligncenter size-full wp-image-528" />

En cambio, si quitamos nuestra conexión de datos obtendremos lo siguiente.

<img src="http://objective-c.es/wp-content/uploads/2012/12/log_con_errores.png" alt="Log con errores" title="Log con errores" width="787" height="115" class="aligncenter size-full wp-image-533" />

### Code Coverage

*Code Coverage* nos ayuda a saber que parte de nuestro código estamos contemplando en nuestras pruebas. Para algunos desarrolladores es una motivación ya que tenemos un objetivo, conseguir el mayor porcentaje posible de código contemplado.

A partir de Xcode 4.2 es muy fácil tener *Code Coverage* en nuestras pruebas. Tan sólo tendremos que hacer unas modificaciones en *Build Settings* del target de nuestro test. Tenemos que modificar *Generate Test Coverage Files* (Yes), *Instrument Program Flow* (Yes) y *Other Linker Flags* (-ObjC) como se muestra en la imagen.

<img src="http://objective-c.es/wp-content/uploads/2012/12/Habilitar_Code_Coverage.png" alt="Habilitar Code Coverage" title="Habilitar Code Coverage" width="652" height="259" class="aligncenter size-full wp-image-547" />

Para ver los datos de los *Coverage Files*, yo uso [CoverStory][4]. Solo tenemos que abrir la app e indicarle la carpeta donde se encuentran estos (a partir de la ruta del *Derived Data* Build/Intermediates/${Project Target}.build/Coverage-iphonesimulator/${Project Target}.build/Objects-normal/i386. Reemplaza ${Project Target} con el nombre del target de tu proyecto).

 [1]: http://en.wikipedia.org/wiki/White-box_testing
 [2]: http://en.wikipedia.org/wiki/Black-box_testing
 [3]: http://en.wikipedia.org/wiki/List_of_unit_testing_frameworks#Objective-C
 [4]: http://code.google.com/p/coverstory/