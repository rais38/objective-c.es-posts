En **Cocoa** hay varias formas de iterar en el contenido de una colección de objetos (*NSArray*, *NSDictionary* y *NSSet*). El que se utilizaba hasta hace unos años era la clase *NSEnumerator*.

	NSEnumerator *enumerator = [myMutableDictionary objectEnumerator];
	id obj;
	while (obj = [enumerator nextObject]) {
	    // Hacer lo que sea
	}
	
Sin embargo, esta clase ya apenas se utiliza a favor de la sintaxis *for/in*.

##Fast Enumeration

Desde **OSX 10.5**, *Apple* resolvió varios problemas que teníamos con la clase *NSEnumerator*:

* *Fast Enumeration* mucho más eficiente.
*  La sintaxis es mucho más concisa.
*  Se levanta una excepción si intentamos modificar la colección durante la iteración (*NSFastEnumerationMutationHandler*).

<!-- -->

	for (id obj in collection) {
	    // Hacer algo con obj
	}

*Fast Enumeration* no se comporta igual en función del tipo de colección. *NSArray* y *NSSet* enumera su contenido pero cuando es *NSDictionary* enumera sus *keys*.


##Blocks-Based Enumeration

Desde **OSX 10.6**, como todos sabemos, *Apple* introdujo los bloques en *Objective-C* pero también añadió una nueva forma de iterar en una colección usando estos (*Blocks-Based Enumeration*).

    [myArray enumerateObjectsUsingBlock: ^(id obj, NSUInteger index, BOOL *stop) {
            // Hacer algo con obj
        }];
        
> "¿Porqué debería usar *Blocks-Based Enumeration* si la sintaxis de *Fast Enumeration* es mucho más amigable y fácil de entender?"

La primera diferencia que vemos a simple vista entre estos dos tipos de iteración es en la sintaxis pero *Blocks-Based Enumeration* nos ofrece alguna opción más:

* Iteración concurrente

<!-- -->

    [myArray enumerateObjectsWithOptions:NSEnumerationConcurrent usingBlock:^(id obj, NSUInteger index, BOOL *stop) {
            NSLog(@"Object %@ with index %lu", obj, index);
        }];
        
* Iteración inversa

<!-- -->

    [myArray enumerateObjectsWithOptions:NSEnumerationReverse usingBlock:^(id obj, NSUInteger index, BOOL *stop) {
                NSLog(@"Object %@ with index %lu", obj, index);
            }];
            
* Cuando iteramos sobre un *NSDictionary* podemos obtener la *key* y el *value* de una sola vez. Con *Fast Enumeration* obtenemos el *value* con otro mensaje.

<!-- -->

    // Blocks-Based Enumeration
    [dictionary enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
            // Hacer algo con key y obj
        }];
        
    // Fast Enumeration
    for (id key in dictionary)
    {
        id obj = [dictionary objectForKey: key];
        // Hacer algo con key y obj
    }
    
* Parar la iteración.

<!-- -->

    [myArray enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
        if(idx == 1) {
            *stop = YES;
        }
    }];
    
    
###En definitiva, ¿Cúal hay que usar?

En la red hay una gran disputa donde unos opinan que *Fast Enumeration* es más eficiente que *Blocks-Based Enumeration*; en cambio, hay otros que opinan lo contrario.
Personalmente pienso que a nivel de rendimiento son muy similares y que nuestra elección debe estar influida por la que mejor se adapte a nuestro código y cúal es más fácil de leer y mantener. Yo utilizo más *Blocks-Based Enumeration* aunque cuando quiero hacer iteraciones simples me decanto por *Fast Enumeration*.


##Extra

El *runtime* de *blocks* de Objective-C es *open source* y podéis encontrarlo [aquí](http://llvm.org/svn/llvm-project/compiler-rt/trunk/).


###Fuentes:

* [Enumeration: Traversing a Collection’s Elements](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/Collections/Articles/Enumerators.html)
* [Blocks Programming Topics](http://developer.apple.com/library/ios/#documentation/cocoa/Conceptual/Blocks/Articles/00_Introduction.html)