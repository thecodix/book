## Method Syntax

*Methods* are similar to functions: they’re declared with the `fn` keyword and
their name, they can have parameters and a return value, and they contain some
code that is run when they’re called from somewhere else. However, methods are
different from functions in that they’re defined within the context of a struct
(or an enum or a trait object, which we cover in Chapters 6 and 17,
respectively), and their first parameter is always `self`, which represents the
instance of the struct the method is being called on.

### Defining Methods

Let’s change the `area` function that has a `Rectangle` instance as a parameter
and instead make an `area` method defined on the `Rectangle` struct, as shown
in Listing 5-13:

<span class="filename">Filename: src/main.rs</span>

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

<span class="caption">Listing 5-13: Defining an `area` method on the
`Rectangle` struct</span>

To define the function within the context of `Rectangle`, we start an `impl`
(*implementation*) block. Then we move the `area` function within the `impl`
curly brackets and change the first (and in this case, only) parameter to be
`self` in the signature and everywhere within the body. In `main` where we
called the `area` function and passed `rect1` as an argument, we can instead
use *method syntax* to call the `area` method on our `Rectangle` instance.
The method syntax goes after an instance: we add a dot followed by the method
name, parentheses, and any arguments.

In the signature for `area`, we use `&self` instead of `rectangle: &Rectangle`
because Rust knows the type of `self` is `Rectangle` due to this method being
inside the `impl Rectangle` context. Note that we still need to use the `&`
before `self`, just like we did in `&Rectangle`. Methods can take ownership of
`self`, borrow `self` immutably as we’ve done here, or borrow `self` mutably,
just like any other parameter.

Hemos escogido el `self` por la misma razón que usamos el `&Rectangle` en la 
versión funcional: no queremos tomar posesión, y sólo queremos leer los 
datos en la estructura, no escribirle. Si quisiéramos cambiar la instancia en la
que hemos llamado al método como parte de lo que hace el método, usaríamos `&mut
self` como el primer parámetro. Tener un método que se adueña de la 
instancia usando sólo `self` como el primer parámetro es raro; esta técnica se 
suele utilizar cuando el método transforma `self` en algo más y queremos 
evitar que el llamante utilice la instancia original después de la transformación.

El principal beneficio de usar métodos en vez de funciones, además de usar 
la sintaxis del método y no tener que repetir el tipo de `self` en la firma de 
cada método, es para la organización. Hemos puesto todas las cosas que podemos hacer con una
instancia de un tipo en un bloque `impl` en lugar de hacer que los futuros usuarios de nuestro
código busquen capacidades de `Rectangle` en varios lugares de la biblioteca 
que proporcionamos.

> ### ¿Dónde está el  operador `->`?
>
> En lenguajes como C++, se utilizan dos operadores diferentes para llamar a métodos: 
> usas `.` si se llama a un método en el objeto directamente y `->` si
> llamas al método en un puntero al objeto y necesitas desreferenciar 
> el puntero primero. En otras palabras, si `object`  es un puntero,
> `object->something()` es similar a `(*objeto).something()`.
>
> Rust no tiene un equivalente al operador `->`; en su lugar, Rust tiene una
> característica llamada *referenciamiento automático y dereferenciación*. Llamar métodos es
> uno de los pocos lugares en Rust que tiene este comportamiento.
>
> Así es como funciona: cuando llamas a un método con `object.something()`, Rust
> agrega automáticamente `&`, `&mut`, o `*` así que `object` coincide con la firma del
> método. En otras palabras, lo siguiente es lo mismo:
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> La primera parece mucho más limpia. Este comportamiento de referenciación automática funciona
> porque los métodos tienen un receptor claro, el tipo de `self`. Dado el receptor
> y el nombre de un método, Rust puede determinar definitivamente si el método está
> leyendo (`&self`), mutando (`&mut self`) o consumiendo (`&self`). El hecho
> de que Rust haga que el préstamo implícito para receptores de métodos sea implícit
> o es una gran parte de hacer que la propiedad ergonómica en la práctica.

### Métodos con Más Parámetros

Practiquemos usando métodos implementando un segundo método en la estructura del
`Rectangle`. Esta vez, queremos una instancia de `Rectangle` para tomar otra instancia
de `Rectangle` y retornar `true` si el segundo `Rectangle` puede encajar completamente
dentro de `self`; de lo contrario debería retornar `falso`. Es decir, queremos poder
escribir el programa mostrado en el Listado 5-14, una vez que hayamos definido el método 
`can_hold`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    let rect2 = Rectangle { width: 10, height: 40 };
    let rect3 = Rectangle { width: 60, height: 45 };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

<span class="caption">Listing 5-14: Listado 5-14: Demostración del uso del método
`can_hold` aún no escrito</span>

Y la salida esperada sería similar a la siguiente, porque ambas dimensiones
de `rect2` son más pequeñas que las dimensiones de `rect1`, pero `rect3` es más ancha
que `rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Sabemos que queremos definir un método, por lo que estará dentro del bloque 
`impl Rectangle`. El nombre del método será `can_hold`, y tomará un préstamo inmutable
de otro `Rectangle` como parámetro. Podemos saber cuál será el tipo de
parámetro al mirar el código que llama al método:
`rect1.can_hold(&rect2)` pasa en `&rect2`, que es un préstamo inmutable a 
`rect2`, una instancia de `Rectangle`. Esto tiene sentido porque sólo necesitamos
leer `rect2` (en lugar de escribir, lo que significaría que necesitaríamos un préstamo mutable),
y queremos que `main` conserve la propiedad de `rect2` para que podamos volver a usarlo después
de llamar al método `can_hold`. El valor de retorno de `can_hold` será
Booleano, y la implementación comprobará si la anchura y la altura del 
`self` son mayores que la anchura y la altura del otro `Rectangle`, 
respectivamente. Agreguemos el nuevo método `can_hold` al bloque `impl` del
Listado 5-13, mostrado en Listado 5-15:

<span class="filename">Filename: src/main.rs</span>

```rust
# #[derive(Debug)]
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

<span class="caption">Listado 5-15: Implementación del método `can_hold` en el 
`Rectangle` que toma otra instancia de `Rectangle` como un parámetro</span>

Cuando ejecutamos este código con la función `main` en Listado 5-14, obtendremos 
la salida deseada. Los métodos pueden tomar múltiples parámetros que añadimos a la
firma después del parámetro `self`, y esos parámetros funcionan igual que los
parámetros en las funciones.

### Funciones asociadas

Otra característica útil de los bloques `impl` es que se nos permite definir 
funciones dentro de bloques `impl` que *no* se toman como un parámetro `self`. Estas 
funciones se denominan *funciones asociadas* porque están asociadas a la estructura.
Siguen siendo funciones, no métodos, porque no tienen una instancia de 
la estructura con la que trabajar. Ya has utilizado la función `String::from`
asociada.

Las funciones asociadas se utilizan a menudo para constructores que devolverán una nueva 
instancia de la estructura. Por ejemplo, podríamos proporcionar una función asociada
que tendría un parámetro de una dimensión y usarlo como ancho y altura,
haciendo así más fácil crear un cuadrado `Rectangle` en lugar de tener que
especificar el mismo valor dos veces:

<span class="filename">Filename: src/main.rs</span>

```rust
# #[derive(Debug)]
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

Para llamar a esta función asociada, utilizamos la sintaxis `::` con el nombre de la estructura, 
como `let sq = Rectangle::square(3); `, por ejemplo. Esta función está
espaciada por el nombre de la estructura: la sintaxis `::` se utiliza tanto para las funciones asociadas
como para los espacios de nombres creados por los módulos, que discutiremos en el Capítulo 7.

### Múltiples Bloques `impl`

Cada estructura puede tener múltiples bloques `impl`. Por ejemplo, el Listado
5-15 es equivalente al código mostrado en el Listado 5-16, que tiene cada método
en su propio bloque `impl`:

```rust
# #[derive(Debug)]
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

<span class="caption">Listado 5-16: Reescritura Listado 5-15 usando múltiples bloques 
`impl`</span>

No hay razón para separar estos métodos en bloques múltiples `impl` aquí,
pero es sintaxis válida. Veremos un caso cuando los bloques múltiples `impl` son útiles 
en el Capítulo 10 cuando discutamos tipos y rasgos genéricos.

## Resumen

Las estructuras nos permiten crear tipos personalizados que son significativos para nuestro dominio. Mediante
el uso de estructuras, podemos mantener las piezas de datos asociadas conectadas entre sí y nombrar
cada pieza para hacer que nuestro código sea claro. Los métodos nos permiten especificar el comportamiento 
que tienen las instancias de nuestras estructuras, y las funciones asociadas nos permiten la funcionalidad
del espacio de nombres que es particular a nuestra estructura sin tener una instancia
disponible.

Pero las estructuras no son la única forma en que podemos crear tipos personalizados: pasemos a la 
función de enumeración de Rust para añadir otra herramienta a nuestra caja de herramientas.
