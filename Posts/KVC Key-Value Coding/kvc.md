¿Qué es eso de **Key-Value Coding**?, simplificando mucho *KVC* es un mecanismo que nos da la posibilidad de obtener y definir propiedades de una clase especificando identificadores (*key*) que representan los nombres de los atributos a los que queremos acceder.

Los métodos para **KVC** están implementados en el protocolo *NSKeyValueCoding*, el cual añade métodos a la clase raíz *NSObject* con lo que todos los objetos en Objective-C pueden acceder fácilmente a esta funcionalidad.

Los dos métodos más importantes que implementa este protocolo para el acceso **KVC** son *valueForKey:* y *setValue:forKey:*. El primero permite obtener el valor y el segundo permite asignarlo. Veamos un ejemplo:
 
Tenemos la clase Version:

	@interface Version : NSObject
	 
	@property (nonatomic, retain) NSString *thing;
	 
	@end
	
Accedemos a las propiedades de la clase utilizando **KVC**.

	Version *version1 = [[Version alloc] init];
	// Asignamos el valor
	[version1 setValue:@"valueThing" forKey:@"thing"];
	 
	// Leemos el valor
	NSString *thing = [version1 valueForKey:@"thing"];
	

##Relaciones de objetos

El protocolo *NSKeyValueCoding* también define los métodos *setValue:forKeyPath:* y *valueForKeyPath:* con la que podremos asignar y leer el atributo de un atributo. Vamos a crear una segunda clase llamada *Map* y añadiremos a la clase *Version* una relación a esta.

*Map*

	@interface Map : NSObject
	 
	@property (nonatomic, retain) NSString *type;
	 
	@end

*Version*

	@interface Version : NSObject
	 
	@property (nonatomic, retain) NSString *thing;
	@property (nonatomic, retain) Map *map;
	 
	@end
	
Ahora veremos como acceder a las propiedades de *Map* a través de *Version* con la sintaxis del "punto".

	Version *version1 = [[Version alloc] init];
	[version1 setValue:@"valueThing" forKey:@"thing"];
	     
	NSString *thing = [version1 valueForKey:@"thing"];
	     
	Map *map1 = [[Map alloc] init];
	[version1 setValue:map1 forKey:@"map"];
	     
	[version1 setValue:@"valueType" forKeyPath:@"map.type"];
	     
	NSString *cadena3 = [version1 valueForKeyPath:@"map.type"];
	
##Simplificando nuestro código

Ahora mismo pensarás que con **KVC** generamos más código que utilizando los métodos tradicionales (acceso directo a los *getters* y *setters* de la clase), y tenemos más posibilidades de error al poder escribir mal el nombre de los atributos. Estás en lo cierto pero hay ocasiones que necesitamos acceder a los atributos de una clase dinámicamente en tiempo de ejecución en vez de en tiempo de compilación. 
 
*NSTableView* (OSX) asocia un identificador (*key*) a cada una de sus columnas. Veamos como nos quedaría el código sin **KVC** y con él.

	// Implementación del método delegado "data-source" de NSTableView sin KVC
	- (id)tableView:(NSTableView *)tableview
	      objectValueForTableColumn:(id)column row:(NSInteger)row {
	  
	    ChildObject *child = [childrenArray objectAtIndex:row];
	    if ([[column identifier] isEqualToString:@"name"]) {
	        return [child name];
	    }
	    if ([[column identifier] isEqualToString:@"age"]) {
	        return [child age];
	    }
	    if ([[column identifier] isEqualToString:@"favoriteColor"]) {
	        return [child favoriteColor];
	    }
	    // And so on.
	}
	
	// Implementación del método delegado "data-source" de NSTableView con KVC
	- (id)tableView:(NSTableView *)tableview
	      objectValueForTableColumn:(id)column row:(NSInteger)row {
	  
	    ChildObject *child = [childrenArray objectAtIndex:row];
	    return [child valueForKey:[column identifier]];
	}
	
Imaginemos que tenemos una *NSTableView* con 50 columnas, nuestro código sin **KVC** sería un auténtico infierno en cambio con él, nuestro código es muchísimo más eficiente, más flexible y más resistente a errores.

##Funciones en colecciones de objetos

Gracias también a **KVC** nos permite realizar determinadas acciones (funciones) a los elementos de una colección de objetos (*NSArray* - *NSSet*).

Función | Descripción
------------ | -------------
*@avg* | Devuelve el valor medio de todos los elementos del conjunto.
*@count* | Devuelve el número de elementos del conjunto
*@max* | Devuelve el valor máximo de todos los elementos del conjunto
*@min* | Devuelve el valor mínimo de todos los elementos del conjunto
*@sum* | Devuelve la suma de los valores de todos los elementos del conjunto
*@unionOfArrays / @distinctUnionOfArrays* | Dada una colección de arrays, devuelve un array que contiene todos los de la colección / Si hay repetidos, solo devolverá uno de estos.
*@unionOfSets / @distinctUnionOfSets* | Dada una colección de *NSSet*, devuelve un conjunto que contiene todos los de la colección / Si hay repetidos, solo devolverá uno de estos.
*@unionOfObjects / @distinctUnionOfObjects* | Dada una colección de arrays o conjuntos, devuelve un array que contiene todos ellos / Si hay repetidos, solo devolverá uno de estos

Para esta prueba vamos a añadir una nueva propiedad a nuestra clase *Version* ("number" -> *NSNumber*).

	Version *version1 = [[Version alloc] init];
	[version1 setValue:@"valueThing" forKey:@"thing"];
	[version1 setValue:@"10" forKey:@"number"];
	     
	 Version *version2 = [[Version alloc] init];
	 [version2 setValue:@"valueThing2" forKey:@"thing"];
	 [version2 setValue:@"100" forKey:@"number"];
	     
	 NSArray *arrayVersion = [NSArray arrayWithObjects:version1, version2, nil];
	     
	 // Resultado 100
	 NSNumber *maxVersion = [arrayVersion valueForKeyPath:@"@max.number"];

##Utilizar KVC con estructuras y escalares

**KVC** es capaz de detectar propiedades que sean estructuras y escalares para encapsularlas automáticamente en tipos *NSNumber* o *NSValue*. NSNumber permite encapsular los tipos numéricos primitivos de C en objetos de Objective-C y *NSValue* estructuras como *NSPoint*, *NSRect* o *NSRange*.

Este mecanismo está directamente relacionado con **KVO**. El cual nos permite implementar el "patrón observador" en **KVC** y os hablaré más en detalle en un próximo post.


Fuente: [Ingens Blog](http://www.ingens-networks.com/blog/post/2012/11/02/KVC-Key-Value-Coding.aspx)