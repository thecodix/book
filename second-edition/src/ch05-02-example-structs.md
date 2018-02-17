## Un Programa de Ejemplo Utilizando Instrucciones

Para entender cuándo podríamos querer usar las estructuras, escribamos un programa que 
calcule el área de un rectángulo. Empezaremos con variables individuales y luego 
refactorizaremos el programa hasta que utilicemos estructuras.

Hagamos un nuevo proyecto binario con Cargo llamado *rectangles* que tomará 
el ancho y la altura de un rectángulo especificado en píxeles y calculará el 
área del rectángulo. El Listado 5-8 muestra un programa corto con una forma de hacer
eso en nuestro proyecto *src/main.rs*:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

<span class="caption">Listado 5-8: Cálculo del área de un rectángulo
especificado por su anchura y altura en variables separadas.</span>

Ahora, ejecuta este programa usando `cargo run`:

```text
The area of the rectangle is 1500 square pixels.
```

### Refactorizando con Tuplas

Aunque Listing 5-8 funciona y calcula el área del rectángulo 
llamando a la función `area` con cada dimensión, podemos hacerlo mejor. El ancho
y alto se relacionan entre sí porque juntas describen 
un rectángulo.

El problema con este código es evidente en la firma de `area`:

```rust,ignore
fn area(width: u32, height: u32) -> u32 {
```

La función `area` se supone que calcula el área de un rectángulo, pero la
función que escribimos tiene dos parámetros. Los parámetros están relacionados, pero eso 
no se expresa en ninguna parte de nuestro programa. Sería más legible y más
manejable agrupar el ancho y la altura juntos. Ya hemos discutido una forma de 
hacerlo en la sección "Agrupando valores en Tuplas" del Capítulo 3: 
usando tuplas. El Listado 5-9 muestra otra versión de nuestro programa que usa tuplas:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

<span class="caption">Listado 5-9: Especificación del ancho y altura del
 rectángulo con una tupla</span>

De alguna manera, este programa es mejor. Las tuplas añaden un poco de estructura, y
ahora estamos pasando sólo un argumento. Pero de otra forma esta versión es menos
clara: las tuplas no nombran sus elementos, así que nuestro cálculo se ha vuelto más
confuso porque tenemos que indexar en las partes de la tupla.

No importa si mezclamos ancho y alto para el cálculo del área, pero 
si queremos dibujar el rectángulo en la pantalla, ¡esto importaría! Deberíamos tener
en cuenta que `width` es el índice de tupla `0` y `height` es el índice de
tupla `1`. Si alguien más trabajó en este código, tendrían que averiguar esto
y tenerlo en cuenta. Sería fácil olvidar o confundir estos valores y 
causar errores, porque no hemos transmitido el significado de nuestros datos
en nuestro código.

### Refactorización con Estructuras: Añadiendo más Significado

Usamos estructuras para añadir significado al etiquetar los datos. Podemos transformar la tupla 
que estamos usando en un tipo de datos con un nombre para el conjunto, así como los nombres de las 
partes, como se muestra en Listado 5-10:

<span class="filename">Filename: src/main.rs</span>

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

<span class="caption">Listado 5-10: Definiendo una estructura de `Rectángulo`.</span>

Aquí hemos definido una estructura y la hemos llamado `Rectangle`. Dentro del `{}` 
definimos los campos como `width` y `height`, los cuales tienen el tipo `u32`. Luego,
en `main` creamos una instancia particular de un `Rectangle` que tiene una anchura de 
30 y una altura de 50. 

Nuestra función `área` se define ahora con un parámetro, al que hemos llamado
`rectangle`, cuyo tipo es un préstamo inmutable de una instancia estructural 
`Rectangle`. Como se menciona en el capítulo 4, queremos tomar prestada la estructura en
lugar de apropiarnos de ella. De esta manera, `main` retiene su posesión y puede continuar
usando `rect1`, que es la razón por la que usamos el `&` en la función firma 
y donde llamamos la función.

La función `area` accede a los campos `width` y `height` de la instancia 
`Rectangle`. Nuestra firma de función para `área` ahora dice exactamente lo que queremos decir:
calcula el área de un `Rectangle`, usando sus campos `width` y `height`.
Esto indica que la anchura y la altura están relacionadas entre sí, y da 
nombres descriptivos a los valores en lugar de utilizar los valores del índice de tuplas `0` 
y `1`. Esto es una victoria para la claridad.

### Añadir Funcionalidad ütil con Rasgos Derivados

Sería bueno poder imprimir una instancia de nuestro `Rectangle` mientras estamos 
depurando nuestro programa y ver los valores para todos sus campos. El listado 5-11 prueba
la macro `println!` tal y como la hemos usado en los capítulos 2, 3 y 4:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {}", rect1);
}
```

<span class="caption">Listado 5-11: Intentar imprimir una instancia de 
`Rectangle`</span>

Cuando ejecutamos este código, obtenemos un error con este mensaje central:

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Display` is not satisfied
```

La macro `println!` puede hacer muchas clases de formateo, y por defecto, `{}`
le indica a `println!` que utilice el formato conocido como `Display`: salida destinado al
consumo directo del usuario final. Los tipos primitivos que hemos visto hasta ahora implementan 
`Display` por defecto, porque sólo hay una forma de mostrar un `1` o 
cualquier otro tipo primitivo a un usuario. Pero con structuras, la forma en que `println! ` debería
formatear la salida es menos clara porque hay más posibilidades de visualización: 
¿quieres o no quieres comas? ¿Quieres imprimir las llaves? ¿Deberían 
mostrarse todos los campos? Debido a esta ambigüedad, Rust no intenta adivinar lo que
queremos y las estructuras no tienen una implementación provista de `Display`.

Si continuamos leyendo los errores, encontraremos esta útil nota:

```text
`Rectangle` cannot be formatted with the default formatter; try using
`:?` instead if you are using a format string
```

Probémoslo! La llamada a la macro `println!` ahora parecerá `println!("rect1 es
{:?}", rect1);`. Poniendo el especificador `:?` dentro del `{}` le dice a `imprintln!` que 
queremos usar un formato de salida llamado `Debug`. `Debug` es un rasgo que nos permite
imprimir nuestra estructura de una manera que es útil para los desarrolladores para que podamos
ver su valor mientras depuramos nuestro código.

Ejecuta el código con este cambio. ¡Maldición! Todavía tenemos un error:

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Debug` is not satisfied
```

Pero de nuevo, el compilador nos da una nota útil:

```text
`Rectangle` cannot be formatted using `:?`; if it is defined in your
crate, add `#[derive(Debug)]` or manually implement it
```

Rust *hace* que se incluya la funcionalidad para imprimir la información de depuración, pero
tenemos que optar explícitamente por hacer que esa funcionalidad esté disponible para nuestra estructura.
Para ello, añadimos la anotación `#[derive(Debug)]` justo antes de la definición de la 
estructura, como se muestra en el Listado 5-12:

<span class="filename">Filename: src/main.rs</span>

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {:?}", rect1);
}
```

<span class="caption">Listado 5-12: Añadiendo la anotación para derivar el rasgo `Debug`
 e imprimir la instancia `Rectangle` usando el formato de depuración</span>

Ahora cuando ejecutamos el programa, no tendremos ningún error y veremos la
siguiente salida:

```text
rect1 is Rectangle { width: 30, height: 50 }
```

¡Bien! No es la salida más bonita, pero muestra los valores de todos los campos
para esta instancia, lo que definitivamente ayudaría durante la depuración. Cuando tenemos
estructuras más grandes, es útil tener una salida que sea un poco más fácil de leer; en
esos casos, podemos usar `{:#?}`en lugar de `{:?}`en la cadena `println!`.
Cuando usamos el estilo `{:#?}` en el ejemplo, la salida se verá así:

```text
rect1 is Rectangle {
    width: 30,
    height: 50
}
```

Rust nos ha proporcionado una serie de rasgos que podemos utilizar con la anotación `derive`
que puede añadir un comportamiento útil a nuestros tipos personalizados. Estos rasgos y sus
comportamientos se enumeran en el Apéndice C. En el Capítulo 10 explicaremos cómo 
implementar estos rasgos con un comportamiento personalizado, y también de cómo crear tus propios rasgos.

Nuestra función `area` es muy específica: sólo calcula el área de rectángulos.
Sería útil vincular este comportamiento más estrechamente a nuestra estructura del
`Rectangle`, porque no funcionará con ningún otro tipo. Veamos cómo podemos 
continuar refactorizando este código al convertir la función de `área` en un *método* 
de `área` definido en nuestro tipo de `Rectangle`.
