# Colecciones comunes

La biblioteca estándar de Rust incluye varias estructuras de datos muy útiles llamadas *colecciones*. La mayoría de los otros tipos de datos representan un valor específico, pero las colecciones pueden contener múltiples valores. A diferencia de los tipos de matriz y tupla incorporados, los datos a los que apuntan estas colecciones se almacenan en el montón, lo que significa que no es necesario conocer la cantidad de datos en tiempo de compilación y pueden crecer o reducirse a medida que se ejecuta el programa. Cada tipo de colección tiene diferentes capacidades y costos, y elegir uno apropiado para su situación actual es una habilidad que desarrollará con el tiempo. En este capítulo, analizaremos tres colecciones que se utilizan muy a menudo en los programas de Rust:

* Un *vector* nos permite almacenar un número variable de valores uno al lado del otro.
* Una *string* es una colección de caracteres. Ya hemos discutido el tipo `String` anteriormente, pero en este capítulo hablaremos en profundidad.
* Un *hash map* nos permite asociar un valor con una clave particular. Es una implementación particular de la estructura de datos más general llamada *map*.

Para conocer los otros tipos de colecciones proporcionadas por la biblioteca estándar, consulte [la documentación][collections].

[collections]: ../../std/collections/index.html

Discutiremos cómo crear y actualizar vectores, cadenas y mapas hash, así como también lo que hace que cada especial.
