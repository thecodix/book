## What Is Ownership?

Rust’s central feature is *ownership*. Although the feature is straightforward
to explain, it has deep implications for the rest of the language.

All programs have to manage the way they use a computer’s memory while running.
Some languages have garbage collection that constantly looks for no longer used
memory as the program runs; in other languages, the programmer must explicitly
allocate and free the memory. Rust uses a third approach: memory is managed
through a system of ownership with a set of rules that the compiler checks at
compile time. No run-time costs are incurred for any of the ownership features.

Because ownership is a new concept for many programmers, it does take some time
to get used to. The good news is that the more experienced you become with Rust
and the rules of the ownership system, the more you’ll be able to naturally
develop code that is safe and efficient. Keep at it!

When you understand ownership, you’ll have a solid foundation for understanding
the features that make Rust unique. In this chapter, you’ll learn ownership by
working through some examples that focus on a very common data structure:
strings.

<!-- PROD: START BOX -->

> ### The Stack and the Heap
>
> In many programming languages, we don’t have to think about the stack and the
> heap very often. But in a systems programming language like Rust, whether a
> value is on the stack or the heap has more of an effect on how the language
> behaves and why we have to make certain decisions. We’ll describe parts of
> ownership in relation to the stack and the heap later in this chapter, so here
> is a brief explanation in preparation.
>
> Both the stack and the heap are parts of memory that is available to your code
> to use at runtime, but they are structured in different ways. The stack stores
> values in the order it gets them and removes the values in the opposite order.
> This is referred to as *last in, first out*. Think of a stack of plates: when
> you add more plates, you put them on top of the pile, and when you need a
> plate, you take one off the top. Adding or removing plates from the middle or
> bottom wouldn’t work as well! Adding data is called *pushing onto the stack*,
> and removing data is called *popping off the stack*.
>
> The stack is fast because of the way it accesses the data: it never has to
> search for a place to put new data or a place to get data from because that
> place is always the top. Another property that makes the stack fast is that all
> data on the stack must take up a known, fixed size.
>
> For data with a size unknown to us at compile time or a size that might change,
> we can store data on the heap instead. The heap is less organized: when we put
> data on the heap, we ask for some amount of space. The operating system finds
> an empty spot somewhere in the heap that is big enough, marks it as being in
> use, and returns to us a pointer to that location. This process is called
> *allocating on the heap*, and sometimes we abbreviate the phrase as just
> “allocating.” Pushing values onto the stack is not considered allocating.
> Because the pointer is a known, fixed size, we can store the pointer on the
> stack, but when we want the actual data, we have to follow the pointer.
>
> Think of being seated at a restaurant. When you enter, you state the number of
> people in your group, and the staff finds an empty table that fits everyone and
> leads you there. If someone in your group comes late, they can ask where you’ve
> been seated to find you.
>
> Accessing data in the heap is slower than accessing data on the stack because
> we have to follow a pointer to get there. Contemporary processors are faster if
> they jump around less in memory. Continuing the analogy, consider a server at a
> restaurant taking orders from many tables. It’s most efficient to get all the
> orders at one table before moving on to the next table. Taking an order from
> table A, then an order from table B, then one from A again, and then one from B
> again would be a much slower process. By the same token, a processor can do its
> job better if it works on data that’s close to other data (as it is on the
> stack) rather than farther away (as it can be on the heap). Allocating a large
> amount of space on the heap can also take time.
>
> When our code calls a function, the values passed into the function (including,
> potentially, pointers to data on the heap) and the function’s local variables
> get pushed onto the stack. When the function is over, those values get popped
> off the stack.
>
> Keeping track of what parts of code are using what data on the heap, minimizing
> the amount of duplicate data on the heap, and cleaning up unused data on the
> heap so we don’t run out of space are all problems that ownership addresses.
> Once you understand ownership, you won’t need to think about the stack and the
> heap very often, but knowing that managing heap data is why ownership exists
> can help explain why it works the way it does.
>
<!-- PROD: END BOX -->

### Ownership Rules

First, let’s take a look at the ownership rules. Keep these rules in mind as we
work through the examples that illustrate the rules:

> 1. Each value in Rust has a variable that’s called its *owner*.
> 2. There can only be one owner at a time.
> 3. When the owner goes out of scope, the value will be dropped.

### Variable Scope

We’ve walked through an example of a Rust program already in Chapter 2. Now
that we’re past basic syntax, we won’t include all the `fn main() {` code in
examples, so if you’re following along, you’ll have to put the following
examples inside a `main` function manually. As a result, our examples will be a
bit more concise, letting us focus on the actual details rather than
boilerplate code.

As a first example of ownership, we’ll look at the *scope* of some variables. A
scope is the range within a program for which an item is valid. Let’s say we
have a variable that looks like this:

```rust
let s = "hello";
```

The variable `s` refers to a string literal, where the value of the string is
hardcoded into the text of our program. The variable is valid from the point at
which it’s declared until the end of the current *scope*. Listing 4-1 has
comments annotating where the variable `s` is valid:

```rust
{                      // s is not valid here, it’s not yet declared
    let s = "hello";   // s is valid from this point forward

    // do stuff with s
}                      // this scope is now over, and s is no longer valid
```

<span class="caption">Listing 4-1: A variable and the scope in which it is
valid</span>

In other words, there are two important points in time here:

1. When `s` comes *into scope*, it is valid.
1. It remains so until it goes *out of scope*.

At this point, the relationship between scopes and when variables are valid is
similar to other programming languages. Now we’ll build on top of this
understanding by introducing the `String` type.

### The `String` Type

To illustrate the rules of ownership, we need a data type that is more complex
than the ones we covered in Chapter 3. All the data types we’ve looked at
previously are stored on the stack and popped off the stack when their scope is
over, but we want to look at data that is stored on the heap and explore how
Rust knows when to clean up that data.

We’ll use `String` as the example here and concentrate on the parts of `String`
that relate to ownership. These aspects also apply to other complex data types
provided by the standard library and that you create. We’ll discuss `String` in
more depth in Chapter 8.

We’ve already seen string literals, where a string value is hardcoded into our
program. String literals are convenient, but they aren’t always suitable for
every situation in which you want to use text. One reason is that they’re
immutable. Another is that not every string value can be known when we write
our code: for example, what if we want to take user input and store it? For
these situations, Rust has a second string type, `String`. This type is
allocated on the heap and as such is able to store an amount of text that is
unknown to us at compile time. You can create a `String` from a string literal
using the `from` function, like so:

```rust
let s = String::from("hello");
```

The double colon (`::`) is an operator that allows us to namespace this
particular `from` function under the `String` type rather than using some sort
of name like `string_from`. We’ll discuss this syntax more in the “Method
Syntax” section of Chapter 5 and when we talk about namespacing with modules in
Chapter 7.

This kind of string *can* be mutated:

```rust
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() appends a literal to a String

println!("{}", s); // This will print `hello, world!`
```

So, what’s the difference here? Why can `String` be mutated but literals
cannot? The difference is how these two types deal with memory.

### Memory and Allocation

In the case of a string literal, we know the contents at compile time so the
text is hardcoded directly into the final executable, making string literals
fast and efficient. But these properties only come from its immutability.
Unfortunately, we can’t put a blob of memory into the binary for each piece of
text whose size is unknown at compile time and whose size might change while
running the program.

With the `String` type, in order to support a mutable, growable piece of text,
we need to allocate an amount of memory on the heap, unknown at compile time,
to hold the contents. This means:

1. The memory must be requested from the operating system at runtime.
2. We need a way of returning this memory to the operating system when we’re
done with our `String`.

That first part is done by us: when we call `String::from`, its implementation
requests the memory it needs. This is pretty much universal in programming
languages.

However, the second part is different. In languages with a *garbage collector
(GC)*, the GC keeps track and cleans up memory that isn’t being used anymore,
and we, as the programmer, don’t need to think about it. Without a GC, it’s the
programmer’s responsibility to identify when memory is no longer being used and
call code to explicitly return it, just as we did to request it. Doing this
correctly has historically been a difficult programming problem. If we forget,
we’ll waste memory. If we do it too early, we’ll have an invalid variable. If
we do it twice, that’s a bug too. We need to pair exactly one `allocate` with
exactly one `free`.

Rust takes a different path: the memory is automatically returned once the
variable that owns it goes out of scope. Here’s a version of our scope example
from Listing 4-1 using a `String` instead of a string literal:

```rust
{
    let s = String::from("hello"); // s es válido a partir de este momento

    // hacer cosas con s
}                                  // este alcance ya ha terminado, y s no es
                                   // más extensa
```

There is a natural point at which we can return the memory our `String` needs
to the operating system: when `s` goes out of scope. When a variable goes out
of scope, Rust calls a special function for us. This function is called `drop`,
and it’s where the author of `String` can put the code to return the memory.
Rust calls `drop` automatically at the closing `}`.

> Note: In C++, this pattern of deallocating resources at the end of an item's
> lifetime is sometimes called *Resource Acquisition Is Initialization (RAII)*.
> The `drop` function in Rust will be familiar to you if you’ve used RAII
> patterns.

This pattern has a profound impact on the way Rust code is written. It may seem
simple right now, but the behavior of code can be unexpected in more
complicated situations when we want to have multiple variables use the data
we’ve allocated on the heap. Let’s explore some of those situations now.

#### Ways Variables and Data Interact: Move

Multiple variables can interact with the same data in different ways in Rust.
Let’s look at an example using an integer in Listing 4-2:

```rust
let x = 5;
let y = x;
```

<span class="caption">Listing 4-2: Assigning the integer value of variable `x`
to `y`</span>

We can probably guess what this is doing based on our experience with other
languages: “Bind the value `5` to `x`; then make a copy of the value in `x` and
bind it to `y`.” We now have two variables, `x` and `y`, and both equal `5`.
This is indeed what is happening because integers are simple values with a
known, fixed size, and these two `5` values are pushed onto the stack.

Now let’s look at the `String` version:

```rust
let s1 = String::from("hello");
let s2 = s1;
```

This looks very similar to the previous code, so we might assume that the way
it works would be the same: that is, the second line would make a copy of the
value in `s1` and bind it to `s2`. But this isn’t quite what happens.

To explain this more thoroughly, let’s look at what `String` looks like under
the covers in Figure 4-3. A `String` is made up of three parts, shown on the
left: a pointer to the memory that holds the contents of the string, a length,
and a capacity. This group of data is stored on the stack. On the right is the
memory on the heap that holds the contents.

<img alt="String in memory" src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-3: Representation in memory of a `String`
holding the value `"hello"` bound to `s1`</span>

The length is how much memory, in bytes, the contents of the `String` is
currently using. The capacity is the total amount of memory, in bytes, that the
`String` has received from the operating system. The difference between length
and capacity matters, but not in this context, so for now, it’s fine to ignore
the capacity.

When we assign `s1` to `s2`, the `String` data is copied, meaning we copy the
pointer, the length, and the capacity that are on the stack. We do not copy the
data on the heap that the pointer refers to. In other words, the data
representation in memory looks like Figure 4-4.

<img alt="s1 and s2 pointing to the same value" src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-4: Representation in memory of the variable `s2`
that has a copy of the pointer, length, and capacity of `s1`</span>

The representation does *not* look like Figure 4-5, which is what memory would
look like if Rust instead copied the heap data as well. If Rust did this, the
operation `s2 = s1` could potentially be very expensive in terms of runtime
performance if the data on the heap was large.

<img alt="s1 and s2 to two places" src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-5: Another possibility of what `s2 = s1` might
do if Rust copied the heap data as well</span>

Earlier, we said that when a variable goes out of scope, Rust automatically
calls the `drop` function and cleans up the heap memory for that variable. But
Figure 4-4 shows both data pointers pointing to the same location. This is a
problem: when `s2` and `s1` go out of scope, they will both try to free the
same memory. This is known as a *double free* error and is one of the memory
safety bugs we mentioned previously. Freeing memory twice can lead to memory
corruption, which can potentially lead to security vulnerabilities.

To ensure memory safety, there’s one more detail to what happens in this
situation in Rust. Instead of trying to copy the allocated memory, Rust
considers `s1` to no longer be valid and therefore, Rust doesn’t need to free
anything when `s1` goes out of scope. Check out what happens when you try to
use `s1` after `s2` is created:

```rust,ignore
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
```

Recibirás un error como este porque Rust te impide usar la
referencia invalidada:

```text
error[E0382]: use of moved value: `s1`
 --> src/main.rs:4:27
  |
3 |     let s2 = s1;
  |         -- value moved here
4 |     println!("{}, world!", s1);
  |                            ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`,
which does not implement the `Copy` trait
```

Si has escuchado los términos "copia superficial" y "copia profunda" mientras trabajas
con otros lenguajes, el concepto de copiar el puntero, la longitud y la capacidad
sin copiar los datos probablemente suena como una copia superficial. Pero debido a que Rust 
también invalida la primera variable, en lugar de llamarla copia superficial,
se conoce como *move*. Aquí leíamos esto diciendo que `s1` era *movido*
en `s2`. Así que lo que realmente sucede se muestra en la Figura 4-6.

<img alt="s1 moved to s2" src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 4-6: Representación en la memoria después de que se haya
invalidado `s1`</span>


Eso resuelve nuestro problema! Con sólo `s2` válido, cuando salga del alcance, solo
liberará la memoria, y ya está.

Además, hay una elección de diseño que implica esto: Rust nunca 
creará automáticamente copias "profundas" de sus datos. Por lo tanto, se puede suponer
que cualquier copia *automática* es barata en términos de rendimiento en tiempo de ejecución.

### Variables de Vías e Interacción de Datos: Clonación

Si queremos copiar  *do* con profundidad de datos del monton del `String`, no sólo los 
datos de pila, podemos usar un método común llamado `clone`. Discutiremos la sintaxis 
del método en el Capítulo 5, pero como los métodos son una característica común en muchos
lenguajes de programación, probablemente los hayas visto antes.

He aquí un ejemplo del método `clone` en acción:

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

Esto funciona perfectamente y es cómo se puede producir explícitamente el comportamiento que se muestra 
en la Figura 4-5, donde los datos de montones *se copian*.

Cuando ves una llamada a `clone`, sabes que se está ejecutando algún código arbitrario
y ese código puede ser caro. Es un indicador visual de que algo
diferente está pasando.

#### Datos de Sólo-Pila: Copiar

Hay otra arruga de la que aún no hemos hablado. Este código utilizando números enteros,
parte de los cuales se mostraron anteriormente en el Listado 4-2, funciona y es válido:

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

Pero este código parece contradecir lo que acabamos de aprender: no tenemos una llamada a
`clone`, pero `x` sigue siendo válido y no fue movido a `y`.

La razón es que los tipos como enteros que tienen un tamaño conocido en el momento de compilar
se almacenan enteramente en la pila, por lo que las copias de los valores reales se pueden hacer 
rápidamente. Eso significa que no hay ninguna razón por la que queramos evitar que `x` sea 
válido después de crear la variable `y`. En otras palabras, no hay diferencia 
entre la copia profunda y superficial aquí, por lo que llamar `clone` no haría nada diferente
de la copia superficial habitual y podemos dejarla fuera.

Rust tiene una anotación especial llamada el rasgo de "Copy" que podemos colocar en
tipos como números enteros que se almacenan en la pila (hablaremos más sobre rasgos
en el Capítulo 10). Si un tipo tiene el rasgo `Copy`, una variable antigua sigue 
siendo utilizable después de la asignación. Rust no nos permite anotar un tipo con el rasgo `Copy`
si el tipo, o cualquiera de sus partes, ha implementado el rasgo `Drop`. Si 
el tipo necesita algo especial para que suceda cuando el valor salga del alcance y 
agreguemos la anotación `Copy` a ese tipo, tendremos un error de tiempo de compilación. Para 
obtener más información sobre cómo agregar la anotación `Copy` a tu tipo, consulta el Apéndice C sobre
Rasgos derivables.

Entonces, ¿qué tipos son `Copy`? Puede comprobar la documentación del tipo dado para 
estar seguro, pero como regla general, cualquier grupo de valores escalares simples puede ser
`Copy`, y nada que requiera asignación o sea alguna forma de recurso es
`Copy`. Éstos son algunos de los tipos que son `Copy`:

* Todos los tipos enteros, como `u32`.
* El tipo booleano,`bool`, con valores `true` y `false`.
* Todos los tipos de coma flotante, como `f64`.
* Tuples, pero sólo si contienen tipos que también son `Copy`. ` (i32, i32)` es
`Copy`, pero `(i32, String)` no lo es.

### Propiedad y Funciones

La semántica para pasar un valor a una función que es similar a la asignación de
un valor a una variable. Pasar una variable a una función se moverá o copiará, al igual
que la asignación. El Listado 4-7 tiene un ejemplo con algunas anotaciones que muestran donde
las variables entran y salen del alcance:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s = String::from("hello");  // s entra a alcance.

    takes_ownership(s);             // el valor de s se mueve dentro de la función...
                                    // ... y por lo tanto ya no es válido aquí.
    let x = 5;                      // x entra a alcance.

    makes_copy(x);                  // x pasaría a la función,
                                    // pero i32 es Copy, así que está bien seguir
                                    // uso x después.

} // Aquí, X se sale del alcance, luego s. Pero como el valor de s fue movido, nada
  // especial pasa.

fn takes_ownership(some_string: String) { // some_string entra a alcance.
    println!("{}", some_string);
} // Aquí, some_string se sale del alcance y se llama `drop`. El respaldo
  // de la memoria se libera.

fn makes_copy(some_integer: i32) { // some_integer entra a alcance.
    println!("{}", some_integer);
} // Aquí, some_integer entero queda fuera del alcance. No pasa nada especial.
```

<span class="caption">Listado 4-7: Funciones con propiedad y alcance
anotados</span>

Si intentáramos usar `s` después de la llamada a `takes_ownership`, Rust arrojaría un 
error de compilación de tiempo. Estas comprobaciones estáticas nos protegen de los errores. Intenta agregar
código a `main` que usa `s` y `x` para ver dónde puedes usarlas y dónde 
las reglas de propiedad te impiden hacerlo.

### Valores de Retorno y Alcance

Los valores devueltos también pueden transferir la propiedad. Aquí hay un ejemplo con anotaciones similares
a las del Listado 4-7:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership mueve su retorno
                                        // valor en s1.

    let s2 = String::from("hello");     // s2 entra a alcance.

    let s3 = takes_and_gives_back(s2);  // s2 se traslada a
                                        // takes_and_gives_back, que también
                                        // mueve su valor de retorno a s3.
} // Aquí, s3 se sale del alcance y es eliminado. s2 se sale del ámbito de aplicación pero fue
  // movido, así que no pasa nada. s1 se sale del alcance y se abandona.

fn gives_ownership() -> String {             // gives_ownership moverá su
                                             // valor de retorno a la función
                                             // que lo llama.

    let some_string = String::from("hello"); // some_string entra en alcance.

    some_string                              // some_string es devuelto y
                                             // se traslada a la función de
                                             // llamado.
}

// takes_and_gives_back tomará una cadena y la devolverá
fn takes_and_gives_back(a_string: String) -> String { // a_string entra en
                                                      // alcance.

    a_string  // a_string se devuelve y pasa a la función de llamado.
}
```

La propiedad de una variable sigue el mismo patrón cada vez: asignar un 
valor a otra variable la mueve. Cuando una variable que incluye datos en el 
monton se sale del alcance, el valor será limpiado por `drop` a menos que los datos
hayan sido movidos para ser propiedad de otra variable.

Tomar posesión y luego devolver la propiedad con cada función es un poco 
tedioso. ¿Qué pasa si queremos que una función utilice un valor pero no tome posesión? 
Es bastante molesto que cualquier cosa que pasemos también tenga que ser devuelta si 
queremos volver a usarla, además de cualquier dato que resulte del cuerpo de la
función que queramos devolver también.

Es posible devolver múltiples valores usando un tuple, así:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() devuelve la longitud de una cadena.

    (s, length)
}
```
Pero esto es demasiada ceremonia y mucho trabajo para un concepto que debería ser
común. Afortunadamente para nosotros, Rust tiene una característica para este concepto, y se llama
*referencias*.
