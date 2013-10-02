Cuando hablamos de constantes en *Objective-C*, tenemos varias formas para hacerlo:

*   Enum
*   *#define*
*   Constantes privadas
*   Contantes públicas

## Enum

Los *enum* se usan para agrupar constantes relacionadas que tengan enteros como valor.

    typedef NS_ENUM(NSUInteger, RAMMatrixMode) {
        RAMRadioModeMatrix           = 0,
        RAMHighlightModeMatrix       = 1,
        RAMListModeMatrix            = 2,
        RAMTrackModeMatrix           = 3
    };
    

No voy a hablar en detalle de los *enum* porque ya lo hizo en un estupendo [post][1] mi compañero [@pepito][2].<!--more-->

## #define

El proceso de compilación consta de varias partes, la primera de ellas es el preprocesamiento. Antes de que el código es compilado, el preprocesador busca en el código directivas especiales (precedidas del caracter *#*) y los convierte en código que puede ser manejado por el compilador.

En este caso, la directiva que nos importa es *#define*.

    #define NUM_SECTIONS          3
    #define NUM_ITEMS_1           2
    #define NUM_ITEMS_2           14
    #define NUM_ITEMS_3           5
    

Las variables inicializadas con *#define* son sustituídas en el código por su valor en todos los lugares donde aparece su nombre.

Esta es la forma más común de definir una constante pero no es la más recomendada.

> Don’t use the preprocessor to do something the language itself provides.

## Constantes privadas

Antes de la implementación, declaramos la constante de la siguiente manera:

    static NSString *const kConstant = @"something";
    

*   *static* -> Puede tener dos significados. Si está dentro de una función/método,  el valor se mantendrá a través de las llamadas a dicha función/método. Será inicializado la primera vez que se llame y una vez hecho, su valor será tomado en las siguientes llamadas.
Si está fuera de una función/método (*global scope*), sólo será accesible dentro del ámbito (scope) del fichero que lo declara.
* *const* -> Esta *keyword* nos indica que no podemos modificar su valor (constante).

## Constantes públicas

Para estas utilizaremos la *keyword* *extern*. En el *header* declaramos las constantes.

    // En el .h declaramos nuestras constantes como punteros a objetos NSString
    extern NSString *const MyConstant;
    extern NSString *const AnotherConstant;
    

Y luego las definimos en nuestro *.m*.

    NSString *const MyConstant = @"myConstantValue";
    NSString *const AnotherConstant = @"anotherConstantValue";
    

En algunas ocasiones, podremos encontrarnos [código de terceros][3] que en vez de utilizar *extern* utilizan macros como *FOUNDATION_EXPORT*, *OBJC_EXPORT*, *APPKIT_EXPORT*… Si vemos la clase [**NSObjCRuntime.h**][4] podemos observar lo siguiente:

    #if defined(__cplusplus)
    #define FOUNDATION_EXPORT extern "C"
    #define FOUNDATION_IMPORT extern "C"
    #endif
    
    #if !defined(FOUNDATION_EXPORT)
    #define FOUNDATION_EXPORT extern
    #endif
    
    #if !defined(FOUNDATION_IMPORT)
    #define FOUNDATION_IMPORT extern
    #endif
    

Lo que hace es compilar a *extern* en C y *extern "C"* en C++.

Las "best practices" recomiendan utilizar estas últimas en vez de los *#define*. Algunas de las ventajas es que el valor de la variable puede ser vista en el depurador como también puedes usar la comparación *(stringInstance == MyConstant)* que es mucho más rápida que *([stringInstance isEqualToString:MyConstant])*.

## Posición de la keyword const

Antes hemos dicho que la *keyword* *const* nos indicaba que no podemos modificar su valor.

    const int x = 42;
    // Esta asignación nos dará un error tipo "Read-only variable is not assignable"
    x           = 43;
    

Es fácil de entender, ¿verdad? pero el problema viene con los punteros. La posición de dicha *keyword* es determinante y veremos las diferentes opciones.

    // Puntero-a-char constante
    const char* someString;
    
    // Puntero-constante-a-char
    char* const someString;
    
    // Puntero-constante-a-char constante
    const char* const someString;
    

Si no te has aclarado aún, no te preocupes vamos a ver un ejemplo práctico donde se entiende muy bien.

    // valor de char
    char value = 'A';
    // puntero a char
    char *valuePointer = &value;
    
    // Puntero-a-char constante
    const char* charPointer1;
    charPointer1 = valuePointer; // cambiar la dirección del puntero
    *charPointer1 = 'B';         // cambiar el valor del puntero
    
    // Puntero-constante-a-char
    char* const charPointer2;
    charPointer2 = valuePointer; // cambiar la dirección del puntero
    *charPointer2 = 'C';         // cambiar el valor del puntero
    
    // Puntero-constante-a-char constante
    const char* const charPointer3;
    charPointer3 = valuePointer; // cambiar la dirección del puntero
    *charPointer3 = 'D';         // cambiar el valor del puntero
    

Si intentamos compilar este código, tendremos los siguientes errores.

[<img src="http://objective-c.es/wp-content/uploads/2013/02/Captura-de-pantalla-2013-02-18-a-las-13.37.32.png" alt="const code" width="795" height="281" class="aligncenter size-full wp-image-1091" />][5]

*   const char* charPointer1: Sólo se puede cambiar la dirección del puntero.
*   char* const charPointer2: Sólo se puede modificar el valor que contiene la dirección que apunta el puntero.
*   const char* const charPointer3: No se puede cambiar ni la dirección ni el valor.

PD: La cita que incluyo en el artículo, lo leí en un blog que descubrí en Twitter hace un mes y algo. Os recomiendo mucho que lo sigáis de cerca "[Quality Coding][6]".

 [1]: http://objective-c.es/modern-objective-c-enum/
 [2]: http://twitter.com/pepito
 [3]: https://github.com/steeleforge/iOS-Railtime/blob/master/Railtime/Constants.h
 [4]: https://gist.github.com/mattt/4469665
 [5]: http://objective-c.es/wp-content/uploads/2013/02/Captura-de-pantalla-2013-02-18-a-las-13.37.32.png
 [6]: http://qualitycoding.org/preprocessor