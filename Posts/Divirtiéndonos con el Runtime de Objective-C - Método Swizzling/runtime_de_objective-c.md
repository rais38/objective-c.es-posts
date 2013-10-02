El *Runtime* es una de las características de *Objective-C* que olvidamos en un principio. Cuando somos *newbie* en Cocoa, no tardamos mucho en hacernos con el lenguaje, empezar a pintar nuestros primeros *"Hello World!"* y cosas "algo" más complejas con *Interface Builder*. Todos sabemos que *Objective-C* es un superconjunto de C pero no todos saben que cuando realmente envías el mensaje

    [array insertObject:foo atIndex:2];
    

el compilador lo traduce a

    objc_msgSend(array, @selector(insertObject:atIndex:), foo, 2);
    

Conocer más a fondo el *runtime*, nos ayudará a comprender cómo y porqué *Objective-C* y *Cocoa* trabajan de la manera que lo hacen.

## ¿Qué es el Runtime de Objective-C?

Es una librería escrita principalmente en C y ensamblador que es la culpable de añadir las capacidades OO, herencia, KVC, UI con *Interface Builder*… a Objective-C.

Vamos a ver una técnica en la que comprobaremos lo divertido y dinámico que es el *runtime*.

### Método Swizzling (Conversión de referencias)

Cuando trabajamos con categorías, si intentamos sobreescribir algún método de la clase que estamos extendiendo nos aparecería un *warning* diciendo *"category is implementing a method which will also be implemented by its primary class"*.

Sin embargo, esto funcionaría si el método que intentamos reemplazar está implementado en una superclase de la que estamos sobreescribiendo (podemos llamar a *super*) sino nos surge un gran problema:

*   Es imposible llamar a la implementación original porque la sobreescribiríamos y la perderíamos. La mayoría de ocasiones lo que nos interesa es añadir funcionalidad y no eliminar la original.

Para solucionar esto, podemos utilizar el método *swizzling*. Esto básicamente lo que hace es reemplazar un método por otro.

[<img src="http://objective-c.es/wp-content/uploads/2013/03/method_swizzling.png" alt="Método Swizzling" width="798" height="561" class="aligncenter size-full wp-image-1121" />][1]

Vamos a realizar una prueba en la que añadiremos un borde rojo a todas las *UIViews* generadas desde un *xib*.

*   Lo primero será crearnos una categoría de *UIView* la que llamaremos *BorderColor*. 
*   Debemos añadir la librería *<objc/runtime.h>* para poder trabajar con las funciones del *runtime*.
*   Implementamos nuestro método modificado para que nos pinte los bordes en rojo.

<!-- -->

    // initWithNibName:bundle: llamará a initWithCoder: para instanciar tu vista desde el xib
    
    - (id)swizzled_initWithCoder:(NSCoder *)decoder {
        // Llamamos al original
        id result = [self swizzled_initWithCoder:decoder];
    
        // Nos aseguramos que tenemos UIView (responde al mensaje layer)
        if ([result respondsToSelector:@selector(layer)]) {
            // Obtenemos la capa de esta vista
            CALayer *layer = [result layer];
            layer.borderWidth = 2;
            layer.borderColor = [[UIColor redColor] CGColor];
        }
    
        // Devolvemos la UIView modificada
        return result;
    }
    

*   Añadimos la función que se encargará de hacer el intercambio.

<!-- -->

    void MethodSwizzle(Class c, SEL origSEL, SEL overrideSEL) {
        Method origMethod = class_getInstanceMethod(c, origSEL);
        Method overrideMethod = class_getInstanceMethod(c, overrideSEL);
    
        // Intercambiamos las implementaciones
        method_exchangeImplementations(origMethod, overrideMethod);
    }
    

*   Tendremos que realizar la llamada de la función *MethodSwizzle* desde el método de clase [*load*][2]. Este método se invoca cada vez que se añade una clase o una función al *runtime* (se llama incluso antes que la función *main*).

<!-- -->

    + (void)load {
        // "+ load" se invoca cada vez que una clase o categoría es añadida al runtime
        // Se llama antes incluso que la función "main"
        MethodSwizzle(self, @selector(initWithCoder:), @selector(swizzled_initWithCoder:));
    }
    

*   Ya sólo nos faltará añadir varios elementos al xib en cuestión y tendremos algo similar a esto.

[<img src="http://objective-c.es/wp-content/uploads/2013/03/IMG_0839-169x300.png" alt="Resultado" width="169" height="300" class="aligncenter size-medium wp-image-1131" />][3]

La parte que suele costar entender es la llamada recursiva dentro de nuestro método modificado. Como estamos haciendo *swapping*, en realidad se está llamando al método original y trabajamos con lo que este nos devuelve.

En la última [NSExperience](http://nsexperience.tumblr.com/post/44556154778/objective-runtime) se habló sobre este tema e [@ileider](https://twitter.com/ileider) recomendó que utilicemos este [*wrapper*](https://github.com/rentzsch/jrswizzle) para implementar el método *swizzling* correctamente en diferentes versiones del *runtime*.

Podéis descargaros el proyecto de ejemplo [aquí](https://github.com/Objective-C-es/demo-swizzling).


### Extra: Objetos asociados

Otro problema que nos podemos encontrar con las categorías es que no podemos añadir variables de instancia a la misma. Para solucionar esto tenemos los "Objetos asociados" (a partir de iOS 4 y OSX 10.6) el cual podremos asociar un objeto dado a otro en tiempo de ejecución.

Sobre este tema, ya habló [@sendoaportuondo](https://twitter.com/sendoaportuondo) en esta [entrada](http://www.punteroavoid.com/blog/2013/01/29/sustituyendo-delegados-por-bloques-introduccion-a-los-objetos-asociados/), el cual os lo recomiendo encarecidamente.

 [1]: http://objective-c.es/wp-content/uploads/2013/03/method_swizzling.png
 [2]: http://developer.apple.com/library/ios/#documentation/Cocoa/Reference/Foundation/Classes/nsobject_Class/Reference/Reference.html#//apple_ref/occ/clm/NSObject/load
 [3]: http://objective-c.es/wp-content/uploads/2013/03/IMG_0839.png