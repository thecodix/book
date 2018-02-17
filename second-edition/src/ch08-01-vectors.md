## ## Los Vectores Almacenan Listas de Valores

El primer tipo de colección que veremos es `Vec <T>`, también conocido como * vector *.
Los vectores nos permiten almacenar más de un valor en una única estructura de datos que
pone todos los valores uno al lado del otro en la memoria. Los vectores solo pueden almacenar valores
del mismo tipo. Son útiles en situaciones en las que tiene una lista de
elementos, como las líneas de texto en un archivo o los precios de los artículos en una carro 
de compra.

### Creando un Nuevo Vector

Para crear un nuevo vector vacío, podemos llamar a la función `Vec :: new` como se muestra en
Listado 8-1:

```rust
let v: Vec<i32> = Vec::new();
```

<span class="caption">Listando 8-1: Creando un nuevo vector vacío para guardar valores
del tipo `i32`</span>

Tenga en cuenta que agregamos una anotación de tipo aquí. Porque no estamos insertando ningun
valores en este vector, Rust no sabe qué tipo de elementos pretendemos
almacenar. Éste es un punto importante. Los vectores se implementan usando genéricos;
cubriremos cómo usar genéricos con sus propios tipos en el Capítulo 10. Por ahora,
sepa que el tipo `Vec <T>` proporcionado por la biblioteca estándar puede contener cualquier tipo,
y cuando un vector específico tiene un tipo específico, el tipo se especifica dentro de
paréntesis angulares. En el listado 8-1, le hemos dicho a Rust que el `Vec <T>` en `v`
contiene elementos del tipo `i32`.

En un código más realista, Rust a menudo puede inferir el tipo de valor que queremos almacenar
una vez que insertemos los valores, rara vez necesita hacer esta anotación de tipo. Es más
comunmente para crear un `Vec <T>` que tiene valores iniciales, y Rust proporciona el
macro `vec!` por conveniencia. La macro creará un nuevo vector que contiene el
valores que le damos. El listado 8-2 crea un nuevo `Vec <i32>` que contiene los valores
`1`, `2`, y `3`:

```rust
let v = vec![1, 2, 3];
```

<span class="caption">Listando 8-2:Creando un nuevo vector que contiene
values</span>

Debido a que hemos dado valores iniciales de `i32`, Rust puede inferir que el tipo de` v`
es `Vec <i32>`, y la anotación de tipo no es necesaria. A continuación, veremos cómo
para modificar un vector.

### Actualizando un Vector

Para crear un vector y luego agregarle elementos, podemos usar el método `push` como
mostrado en el Listado 8-3:

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

<span class="caption">Lisando 8-3: Usando el método `push` para agregar valores a un
vector</span>

Como con cualquier variable, como se discutió en el Capítulo 3, si queremos ser capaces de
cambiar su valor, tenemos que hacerlo mutable usando la palabra clave `mut`. 
los números que colocamos dentro son todos de tipo `i32`, y Rust infiere esto de los
datos, por lo que no necesitamos la anotación `Vec <i32>`.

### Dejar caer un vector Cae sus elementos

Al igual que cualquier otra `struct`, un vector se liberará cuando salga del alcance, como
anotandolo en el Listado 8-4:

```rust
{
    let v = vec![1, 2, 3, 4];

    // do stuff with v

} // <- v goes out of scope and is freed here
```

<span class="caption">Listando 8-4: Mostrando donde el vector y sus elementos
son descartados</span>

Cuando el vector se descarta, todos sus contenidos también se descartarán, lo que significa que
esos enteros que contiene serán limpiados. Esto puede parecer como un
punto directo, pero puede ser un poco más complicado cuando comenzamos a
introducir referencias a los elementos del vector. ¡Vamos a abordar eso enseguida!

### Lectura de elementos de vectores

Ahora que sabe cómo crear, actualizar y destruir vectores, saber cómo
leer sus contenidos es un buen paso siguiente. Hay dos formas de hacer referencia a un
valor almacenado en un vector. En los ejemplos, hemos anotado los tipos de
valores que se devuelven de estas funciones para mayor claridad.

El Listado 8-5 muestra ambos métodos para acceder a un valor en un vector con
sintaxis de indexación o el método `get`:

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
let third: Option<&i32> = v.get(2);
```

<span class="caption">Listando 8-5: Usando la sintaxis de índices o el método `get` para
acceder a un elemento en un vector</span>

Tenga en cuenta dos detalles aquí. Primero, usamos el valor de índice de `2` para obtener el tercer
elemento: los vectores están en los índices por números, comenzando por cero. En segundo lugar, los dos tienen
diferentes formas de obtener el tercer elemento usando `&` y `[]`, que da
una referencia, o usando el método `get` con el índice pasado como
argumento, que nos da una `Opción <& T>`.

La razón por la que Rust tiene dos formas de hacer referencia a un elemento es para que pueda elegir cómo
el programa se comporta cuando intenta usar un valor de índice que el vector no
tiene un elemento para ello. Como ejemplo, veamos qué hará un programa si tiene
un vector que contiene cinco elementos y luego intenta acceder a un elemento en el índice
100, como se muestra en el Listado 8-6:

```rust,should_panic
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

<span class="caption">Listando 8-6: Intentando acceder al elemento en el índice
100 en un vector que contiene 5 elementos </ span>

Cuando ejecutes este código, el primer método `[]` causará `panic!` por que este
hace referencia a un elemento inexistente. Este método podría usarse de una mejor manera
si quiere que su programa pueda considerar el acceder a un elemento que está más allá del final del vector
ya que sería un grave error si el programa llegara a bloquearse.

Cuando el método `get` se pasa un índice que está fuera del vector, regresa al
`None` sin entrar en pánico. Deberías usar este método si accedes a un elemento
más allá del rango del vector ocurre ocasionalmente bajo circunstancias normales.
Su código tendrá lógica para manejar tener `Some (& element)` o
`None`, como se discutió en el Capítulo 6. Por ejemplo, el índice podría venir de
una persona que ingresa un número. Si ingresan accidentalmente un número que también es
grande y el programa obtiene un valor 'Ninguno', podría decirle al usuario cuántos
artículos están en el vector actual y les da otra oportunidad de ingresar un valor
válido. Eso sería más fácil de usar que estrellar el programa debido a un error tipográfico.

#### Referencias inválidas

Cuando el programa tiene una referencia válida, el comprobador de préstamos impone las
reglas de propiedad y de endeudamiento (cubiertas en el Capítulo 4) para asegurar esta referencia
y cualquier otra referencia a los contenidos del vector sigue siendo válida. Recuerda la
regla que establece que no podemos tener referencias mutables e inmutables en el mismo
alcance. Esa regla se aplica en el Listado 8-7 donde tenemos una referencia inmutable a
el primer elemento en un vector e intenta agregar un elemento al final, que no
hace el trabajo:

```rust,ignore
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);
```

<span class="caption">Listing 8-7: Attempting to add an element to a vector
while holding a reference to an item</span>

Compiling this code will result in this error:

```text
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 -->
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^ mutable borrow occurs here
7 |
8 | }
  | - immutable borrow ends here
```

The code in Listing 8-7 might look like it should work: why should a reference
to the first element care about what changes at the end of the vector? The
reason behind this error is due to the way vectors work: adding a new element
onto the end of the vector might require allocating new memory and copying the
old elements to the new space if there isn’t enough room to put all the
elements next to each other where the vector was. In that case, the reference
to the first element would be pointing to deallocated memory. The borrowing
rules prevent programs from ending up in that situation.

> Note: For more on the implementation details of the `Vec<T>` type, see “The
> Rustonomicon” at https://doc.rust-lang.org/stable/nomicon/vec.html.

### Iterating Over the Values in a Vector

If we want to access each element in a vector in turn, we can iterate through
all of the elements rather than use indexes to access one at a time. Listing
8-8 shows how to use a `for` loop to get immutable references to each element
in a vector of `i32` values and print them out:

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
```

<span class="caption">Listing 8-8: Printing each element in a vector by
iterating over the elements using a `for` loop</span>

We can also iterate over mutable references to each element in a mutable vector
in order to make changes to all the elements. The `for` loop in Listing 8-9
will add `50` to each element:

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

<span class="caption">Listing 8-9: Iterating over mutable references to
elements in a vector</span>

To change the value that the mutable reference refers to, we have to use the
dereference operator (`*`) to get to the value in `i` before we can use the
`+=` operator .

### Using an Enum to Store Multiple Types

At the beginning of this chapter, we said that vectors can only store values
that are the same type. This can be inconvenient; there are definitely use
cases for needing to store a list of items of different types. Fortunately, the
variants of an enum are defined under the same enum type, so when we need to
store elements of a different type in a vector, we can define and use an enum!

For example, let’s say we want to get values from a row in a spreadsheet where
some of the columns in the row contain integers, some floating-point numbers,
and some strings. We can define an enum whose variants will hold the different
value types, and then all the enum variants will be considered the same type:
that of the enum. Then we can create a vector that holds that enum and so,
ultimately, holds different types. We’ve demonstrated this in Listing 8-10:

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

<span class="caption">Listing 8-10: Defining an `enum` to store values of
different types in one vector</span>

The reason Rust needs to know what types will be in the vector at compile time
is so it knows exactly how much memory on the heap will be needed to store each
element. A secondary advantage is that we can be explicit about what types are
allowed in this vector. If Rust allowed a vector to hold any type, there would
be a chance that one or more of the types would cause errors with the
operations performed on the elements of the vector. Using an enum plus a
`match` expression means that Rust will ensure at compile time that we always
handle every possible case, as discussed in Chapter 6.

If you don’t know the exhaustive set of types the program will get at runtime
to store in a vector when you’re writing a program, the enum technique won’t
work. Instead, you can use a trait object, which we’ll cover in Chapter 17.

Now that we’ve discussed some of the most common ways to use vectors, be sure
to review the API documentation for all the many useful methods defined on
`Vec<T>` by the standard library. For example, in addition to `push`, a `pop`
method removes and returns the last element. Let’s move on to the next
collection type: `String`!
