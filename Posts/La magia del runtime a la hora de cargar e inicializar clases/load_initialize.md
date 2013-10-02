[Hace unas semanas](http://objective-c.es/divirtiendonos-con-el-runtime-de-objective-c-metodo-swizzling/) empezamos a trabajar con el *runtime*. Para llevar a cabo nuestro ejemplo, utilizamos el método de clase `+load` pero no explicamos mucho sobre él. Creo que este método junto a `+initialize` se merecen un post dedicado especialmente a ellos.

###*+load*: Ejecutando código antes de *main*

El *runtime* de Objective-C nos ofrece una manera de poder ejecutar código antes de que la ejecución del programa entre en la función `main`. Se consigue a través del método de clase `+load` y será invocado por cada clase o categoría que lo implemente.

Esta funcionalidad es muy útil pero también muy peligrosa. Como hemos comentado antes, la lógica dentro de `+load` será ejecutada antes de `main`, lo que significa que nos encontramos en una fase muy temprana del ciclo de vida de la aplicación y el *runtime* no nos puede asegurar el orden en el que se cargarán las clases.

####Tenemos que tener en cuenta

#####Cosas que nos asegura el *runtime*

* Puedes escribir todo código C que tú quieras.
* Puedes "alocar" y enviar mensajes a objetos cuya clase esté implementada en el mismo fichero.
* Es seguro utilizar clases de frameworks que tengas enlazados a tu proyecto.
* Está garantizado que las superclases estarán cargadas antes.
* La implementación de `+load` de todas las superclases de una clase serán invocadas antes de la clase en cuestión.

#####Cosas que NO nos asegura el *runtime*

* "Alocar" o enviar mensajes a objetos arbitrarios ya que como hemos dicho antes, no sabemos el orden en el que se cargarán las clases. **JAMÁS DEBERÍAS HACER ESTO**


####Comportamiento especial en las Categorías

Si implementamos `+load` en una clase y en una categoría de esta, ambas serán invocadas y no se reemplazará una con la otra. Esto va en contra de todo lo que sabemos sobre las categorías pero es que `+load` no es un método normal.

La implementación de `+load` de una clase es ejecutada antes que la de cualquier categoría.


Como hemos visto hasta ahora, este método es de todo menos seguro y [muchos](http://nshipster.com/reader-submissions-new-years-2013/) no recomiendan su uso. Yo lo veo especialmente útil para implementar el [métodod "swizzling"](http://objective-c.es/divirtiendonos-con-el-runtime-de-objective-c-metodo-swizzling/) pero para cualquier otro tipo de uso el que deberíamos utilizar es `+initialize`.

###*+initialize*: Más vale tarde pero seguro

`+initialize` es muy similar a `+load` pero en este estamos en un entorno *seguro* (todas las clases ya han sido cargadas y puedes hacer casi todo lo que quieras). 

Este método será llamado la primera vez (*una sola vez*) que enviemos un mensaje a la clase pero hay ocasiones en el que puede llamarse más de una vez ya que dicho mensaje se envía usando el mecanismo normal, si tu clase no implementa `+initialize` pero hereda de una clase que si lo tiene, tu clase usará el `+initialize` de tu superclase.

Veamos un ejemplo:

    @interface Super: NSObject
    @end
    
    @implementation Super
    
    + (void)initialize
    {
        NSLog(@"Initializing %@", self);
    }
    
    + (void)load
    {
        NSLog(@"Loading");
    }
    
    @end
    
    @interface Sub : Super
    @end
    
    @implementation Sub
    @end
    
    int main(int argc, const char * argv[]) {
        [Sub class];
        return 0;
    }
    
Obtendremos lo siguiente:

    Test[4202:205] Loading
    Test[4202:205] Initializing Super
    Test[4202:205] Initializing Sub
    
Para solucionar esto, se recomienda implementar `+initialize` de la siguiente manera:

    + (void)initialize {
        if (self == [Someclass class]) {
            // Hacer lo que sea
        }
    }

####Categorías

Antes dijimos que si extendemos una clase e implementamos `+load` en las dos, ambas serán invocadas pero eso no pasa con `+initialize`, ya que el de la categoría reemplazaría al de la clase.

Vamos a extender la clase `Super` con la categoría `Foo`:

    @interface Super(Foo)
    @end
    
    @implementation Super(Foo)
    
    + (void)load
    {
        NSLog(@"Category +load");
    }
    + (void)initialize
    {
        NSLog(@"Category +initialize %@", self);
    }
    
    @end
    
Obtendremos esto:

    Test[4202:205] Loading
    Test[4202:205] Category +load
    Test[4202:205] Category +initialize Super
    Test[4202:205] Category +initialize Sub
    
###Conclusión

Tanto `+load` como `+initialize` son métodos que no usaremos la mayoría del tiempo pero es bueno que conozcamos de su existencia porque en ocasiones son útiles para asegurarnos que se cumplan ciertas condiciones antes de crear cualquier instancia de una clase.


Fuente: [mikeash.com](http://www.mikeash.com/pyblog/friday-qa-2009-05-22-objective-c-class-loading-and-initialization.html)