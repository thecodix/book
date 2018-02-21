## Vectors Store Lists of Values

The first collection type we’ll look at is `Vec<T>`, also known as a *vector*.
Vectors allow us to store more than one value in a single data structure that
puts all the values next to each other in memory. Vectors can only store values
of the same type. They are useful in situations in which you have a list of
items, such as the lines of text in a file or the prices of items in a shopping
cart.

### Creating a New Vector

To create a new, empty vector, we can call the `Vec::new` function as shown in
Listing 8-1:

```rust
let v: Vec<i32> = Vec::new();
```

<span class="caption">Listing 8-1: Creating a new, empty vector to hold values
of type `i32`</span>

Note that we added a type annotation here. Because we aren’t inserting any
values into this vector, Rust doesn’t know what kind of elements we intend to
store. This is an important point. Vectors are implemented using generics;
we’ll cover how to use generics with your own types in Chapter 10. For now,
know that the `Vec<T>` type provided by the standard library can hold any type,
and when a specific vector holds a specific type, the type is specified within
angle brackets. In Listing 8-1, we’ve told Rust that the `Vec<T>` in `v` will
hold elements of the `i32` type.

In more realistic code, Rust can often infer the type of value we want to store
once we insert values, so you rarely need to do this type annotation. It’s more
common to create a `Vec<T>` that has initial values, and Rust provides the
`vec!` macro for convenience. The macro will create a new vector that holds the
values we give it. Listing 8-2 creates a new `Vec<i32>` that holds the values
`1`, `2`, and `3`:

```rust
let v = vec![1, 2, 3];
```

<span class="caption">Listing 8-2: Creating a new vector containing
values</span>

Because we’ve given initial `i32` values, Rust can infer that the type of `v`
is `Vec<i32>`, and the type annotation isn’t necessary. Next, we’ll look at how
to modify a vector.

### Updating a Vector

To create a vector and then add elements to it, we can use the `push` method as
shown in Listing 8-3:

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

<span class="caption">Listing 8-3: Using the `push` method to add values to a
vector</span>

As with any variable, as discussed in Chapter 3, if we want to be able to
change its value, we need to make it mutable using the `mut` keyword. The
numbers we place inside are all of type `i32`, and Rust infers this from the
data, so we don’t need the `Vec<i32>` annotation.

### Dropping a Vector Drops Its Elements

Like any other `struct`, a vector will be freed when it goes out of scope, as
annotated in Listing 8-4:

```rust
{
    let v = vec![1, 2, 3, 4];

    // do stuff with v

} // <- v goes out of scope and is freed here
```

<span class="caption">Listing 8-4: Showing where the vector and its elements
are dropped</span>

When the vector gets dropped, all of its contents will also be dropped, meaning
those integers it holds will be cleaned up. This may seem like a
straightforward point but can get a bit more complicated when we start to
introduce references to the elements of the vector. Let’s tackle that next!

### Reading Elements of Vectors

Now that you know how to create, update, and destroy vectors, knowing how to
read their contents is a good next step. There are two ways to reference a
value stored in a vector. In the examples, we’ve annotated the types of the
values that are returned from these functions for extra clarity.

Listing 8-5 shows both methods of accessing a value in a vector either with
indexing syntax or the `get` method:

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
let third: Option<&i32> = v.get(2);
```

<span class="caption">Listing 8-5: Using indexing syntax or the `get` method to
access an item in a vector</span>

Note two details here. First, we use the index value of `2` to get the third
element: vectors are indexed by number, starting at zero. Second, the two
different ways to get the third element are by using `&` and `[]`, which gives
us a reference, or by using the `get` method with the index passed as an
argument, which gives us an `Option<&T>`.

The reason Rust has two ways to reference an element is so you can choose how
the program behaves when you try to use an index value that the vector doesn’t
have an element for. As an example, let’s see what a program will do if it has
a vector that holds five elements and then tries to access an element at index
100, as shown in Listing 8-6:

```rust,should_panic
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

<span class="caption">Listing 8-6: Attempting to access the element at index
100 in a vector containing 5 elements</span>

When you run this code, the first `[]` method will cause a `panic!` because it
references a nonexistent element. This method is best used when you want your
program to consider an attempt to access an element past the end of the vector
to be a fatal error that crashes the program.

When the `get` method is passed an index that is outside the vector, it returns
`None` without panicking. You would use this method if accessing an element
beyond the range of the vector happens occasionally under normal circumstances.
Your code will then have logic to handle having either `Some(&element)` or
`None`, as discussed in Chapter 6. For example, the index could be coming from
a person entering a number. If they accidentally enter a number that’s too
large and the program gets a `None` value, you could tell the user how many
items are in the current vector and give them another chance to enter a valid
value. That would be more user-friendly than crashing the program due to a typo!

#### Invalid References

When the program has a valid reference, the borrow checker enforces the
ownership and borrowing rules (covered in Chapter 4) to ensure this reference
and any other references to the contents of the vector remain valid. Recall the
rule that states we can’t have mutable and immutable references in the same
scope. That rule applies in Listing 8-7 where we hold an immutable reference to
the first element in a vector and try to add an element to the end, which won’t
work:

```rust,ignore
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);
```

<span class="caption">Listando 8-7: Intentando agregar un elemento a un vector
mientras mantiene una referencia a un elemento </ span>

Compilar este código dará como resultado este error:

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

El código en el Listado 8-7 podría parecer que debería funcionar: ¿por qué una referencia
al primer elemento le importa lo que cambia al final del vector? la
razón detrás de este error se debe a la forma en que funcionan los vectores: agregar un nuevo elemento
en el extremo del vector podría requerir asignar nueva memoria y copiar los
elementos antiguos al nuevo espacio si no hay espacio suficiente para poner todos los
elementos uno al lado del otro donde estaba el vector. En ese caso, la referencia
al primer elemento estaría apuntando a la memoria desasignada. El préstamo de
las reglas impiden que los programas terminen en esa situación.

> Nota: Para obtener más información sobre los detalles de implementación del tipo `Vec <T>`, consulte "
> Rustonomicon” en https://doc.rust-lang.org/stable/nomicon/vec.html.

### Iteración sobre los valores en un vector

Si queremos acceder a cada elemento en un vector a su vez, podemos iterar a través de
todos los elementos en lugar de utilizar índices para acceder uno a la vez. El listado
8-8 muestra como usar `for` bucle para obtener referencias inmutables a cada elemento
en un vector de valores `i32` e imprímalos:

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
```

<span class="caption">Listado 8-8: Imprimir cada elemento en un vector por el
iterando sobre los elementos usando un bucle `for` </ span>

También podemos iterar sobre referencias mutables para cada elemento en un vector mutable
para hacer cambios a todos los elementos. El bucle `for` en el Listado 8-9
agregará `50` a cada elemento:

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

<span class="caption">Listado 8-9: Iterando sobre referencias mutables a
elementos en un vector </span>

Para cambiar el valor al que se refiere la referencia mutable, tenemos que usar el
operador de referencia (`*`) para obtener el valor en `i` antes de que podamos usar el
operador `+=`.

### Usando un Enum para almacenar múltiples tipos

Al comienzo de este capítulo, dijimos que los vectores solo pueden almacenar valores
que son del mismo tipo. Esto puede ser un inconveniente; definitivamente hay uso de
casos por la necesidad de almacenar una lista de artículos de diferentes tipos. Afortunadamente, las
variantes de una enumeración se definen bajo el mismo tipo de enumeración, por lo que cuando necesitamos
almacenar elementos de un tipo diferente en un vector, podemos definir y usar una enumeración!

Por ejemplo, supongamos que queremos obtener valores de una fila en una hoja de cálculo donde
algunas de las columnas en la fila contienen enteros, algunos números de coma flotante,
y algunas cuerdas. Podemos definir una enumeración cuyas variantes sostendrán los diferentes
tipos de valor, y luego todas las variantes enum se considerarán del mismo tipo:
el de la enumeración. Entonces podemos crear un vector que contenga esa enumeración y así,
finalmente, tiene diferentes tipos. Hemos demostrado esto en el Listado 8-10:

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

<span class="caption">Listado 8-10: Definir un `enum` para almacenar valores de
diferentes tipos en un vector </span>

La razón por la que Rust necesita saber qué tipos estarán en el vector en tiempo de compilación
es así que sabe exactamente cuánta memoria en el montón será necesaria para almacenar cada
elemento. Una ventaja secundaria es que podemos ser explícitos sobre qué tipos son
permitido en este vector. Si Rust permitiera que un vector tuviera cualquier tipo, habría
ser una posibilidad de que uno o más de los tipos causaría errores con las
operaciones realizadas en los elementos del vector. Usando una enumeración más 
la expresión `match` significa que Rust asegurará en tiempo de compilación que siempre
maneja todos los casos posibles, como se discutió en el Capítulo 6.

Si no conoce el conjunto exhaustivo de tipos que el programa obtendrá en tiempo de ejecución
para almacenar en un vector cuando estás escribiendo un programa, la técnica enum no
trabaja. En cambio, puede usar un objeto de rasgo, que veremos en el Capítulo 17.

Ahora que hemos discutido algunas de las formas más comunes de usar vectores, asegúrese
para revisar la documentación API para todos los muchos métodos útiles definidos en
`Vec <T>` por la biblioteca estándar. Por ejemplo, además de `push`,` pop`
este método elimina y devuelve el último elemento. Pasemos al siguiente
tipo de colección: `String`!
