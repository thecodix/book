## Validating References with Lifetimes

When we talked about references in Chapter 4, we left out an important detail:
every reference in Rust has a *lifetime*, which is the scope for which that
reference is valid. Most of the time lifetimes are implicit and inferred, just
like most of the time types are inferred. Similarly to when we have to annotate
types because multiple types are possible, there are cases where the lifetimes
of references could be related in a few different ways, so Rust needs us to
annotate the relationships using generic lifetime parameters so that it can
make sure the actual references used at runtime will definitely be valid.

Yes, it’s a bit unusual, and will be different to tools you’ve used in other
programming languages. Lifetimes are, in some ways, Rust’s most distinctive
feature.

Lifetimes are a big topic that can’t be covered in entirety in this chapter, so
we’ll cover common ways you might encounter lifetime syntax in this chapter to
get you familiar with the concepts. Chapter 19 will contain more advanced
information about everything lifetimes can do.

### Lifetimes Prevent Dangling References

The main aim of lifetimes is to prevent dangling references, which will cause a
program to reference data other than the data we’re intending to reference.
Consider the program in Listing 10-18, with an outer scope and an inner scope.
The outer scope declares a variable named `r` with no initial value, and the
inner scope declares a variable named `x` with the initial value of 5. Inside
the inner scope, we attempt to set the value of `r` as a reference to `x`. Then
the inner scope ends, and we attempt to print out the value in `r`:

```rust,ignore
{
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```

<span class="caption">Listing 10-18: An attempt to use a reference whose value
has gone out of scope</span>

> #### Uninitialized Variables Cannot Be Used
>
> The next few examples declare variables without giving them an initial value,
> so that the variable name exists in the outer scope. This might appear to be
> in conflict with Rust not having null. However, if we try to use a variable
> before giving it a value, we’ll get a compile-time error. Try it out!

When we compile this code, we’ll get an error:

```text
error: `x` does not live long enough
   |
6  |         r = &x;
   |              - borrow occurs here
7  |     }
   |     ^ `x` dropped here while still borrowed
...
10 | }
   | - borrowed value needs to live until here
```

The variable `x` doesn’t “live long enough.” Why not? Well, `x` is going to go
out of scope when we hit the closing curly bracket on line 7, ending the inner
scope. But `r` is valid for the outer scope; its scope is larger and we say
that it “lives longer.” If Rust allowed this code to work, `r` would be
referencing memory that was deallocated when `x` went out of scope, and
anything we tried to do with `r` wouldn’t work correctly. So how does Rust
determine that this code should not be allowed?

#### The Borrow Checker

The part of the compiler called the *borrow checker* compares scopes to
determine that all borrows are valid. Listing 10-19 shows the same example from
Listing 10-18 with annotations showing the lifetimes of the variables:

```rust,ignore
{
    let r;                // -------+-- 'a
                          //        |
    {                     //        |
        let x = 5;        // -+-----+-- 'b
        r = &x;           //  |     |
    }                     // -+     |
                          //        |
    println!("r: {}", r); //        |
}                         // -------+
```

<span class="caption">Listing 10-19: Annotations of the lifetimes of `r` and
`x`, named `'a` and `'b` respectively</span>

<!-- Just checking I'm reading this right: the inside block is the b lifetime,
correct? I want to leave a note for production, make sure we can make that
clear -->
<!-- Yes, the inside block for the `'b` lifetime starts with the `let x = 5;`
line and ends with the first closing curly bracket on the 7th line. Do you
think the text art comments work or should we make an SVG diagram that has
nicer looking arrows and labels? /Carol -->

We’ve annotated the lifetime of `r` with `'a` and the lifetime of `x` with
`'b`. As you can see, the inner `'b` block is much smaller than the outer `'a`
lifetime block. At compile time, Rust compares the size of the two lifetimes
and sees that `r` has a lifetime of `'a`, but that it refers to an object with
a lifetime of `'b`. The program is rejected because the lifetime `'b` is
shorter than the lifetime of `'a`: the subject of the reference does not live
as long as the reference.

Let’s look at an example in Listing 10-20 that doesn’t try to make a dangling
reference and compiles without any errors:

```rust
{
    let x = 5;            // -----+-- 'b
                          //      |
    let r = &x;           // --+--+-- 'a
                          //   |  |
    println!("r: {}", r); //   |  |
                          // --+  |
}                         // -----+
```

<span class="caption">Listing 10-20: A valid reference because the data has a
longer lifetime than the reference</span>

Here, `x` has the lifetime `'b`, which in this case is larger than `'a`. This
means `r` can reference `x`: Rust knows that the reference in `r` will always
be valid while `x` is valid.

Now that we’ve shown where the lifetimes of references are in a concrete
example and discussed how Rust analyzes lifetimes to ensure references will
always be valid, let’s talk about generic lifetimes of parameters and return
values in the context of functions.

### Generic Lifetimes in Functions

Let’s write a function that will return the longest of two string slices. We
want to be able to call this function by passing it two string slices, and we
want to get back a string slice. The code in Listing 10-21 should print `The
longest string is abcd` once we’ve implemented the `longest` function:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

<span class="caption">Listing 10-21: A `main` function that calls the `longest`
function to find the longest of two string slices</span>

Note that we want the function to take string slices (which are references, as
we talked about in Chapter 4) since we don’t want the `longest` function to
take ownership of its arguments. We want the function to be able to accept
slices of a `String` (which is the type of the variable `string1`) as well as
string literals (which is what variable `string2` contains).

<!-- why is `a` a slice and `b` a literal? You mean "a" from the string "abcd"? -->
<!-- I've changed the variable names to remove ambiguity between the variable
name `a` and the "a" from the string "abcd". `string1` is not a slice, it's a
`String`, but we're going to pass a slice that refers to that `String` to the
`longest` function (`string1.as_str()` creates a slice that references the
`String` stored in `string1`). We chose to have `string2` be a literal since
the reader might have code with both `String`s and string literals, and the way
most readers first get into problems with lifetimes is involving string slices,
so we wanted to demonstrate the flexibility of taking string slices as
arguments but the issues you might run into because string slices are
references.
All of the `String`/string slice/string literal concepts here are covered
thoroughly in Chapter 4, which is why we put two back references here (above
and below). If these topics are confusing you in this context, I'd be
interested to know if rereading Chapter 4 clears up that confusion.
/Carol -->

Refer back to the “String Slices as Parameters” section of Chapter 4 for more
discussion about why these are the arguments we want.

If we try to implement the `longest` function as shown in Listing 10-22, it
won’t compile:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

<span class="caption">Listing 10-22: An implementation of the `longest`
function that returns the longest of two string slices, but does not yet
compile</span>

Instead we get the following error that talks about lifetimes:

```text
error[E0106]: missing lifetime specifier
   |
1  | fn longest(x: &str, y: &str) -> &str {
   |                                 ^ expected lifetime parameter
   |
   = help: this function's return type contains a borrowed value, but the
   signature does not say whether it is borrowed from `x` or `y`
```

The help text is telling us that the return type needs a generic lifetime
parameter on it because Rust can’t tell if the reference being returned refers
to `x` or `y`. Actually, we don’t know either, since the `if` block in the body
of this function returns a reference to `x` and the `else` block returns a
reference to `y`!

As we’re defining this function, we don’t know the concrete values that will be
passed into this function, so we don’t know whether the `if` case or the `else`
case will execute. We also don’t know the concrete lifetimes of the references
that will be passed in, so we can’t look at the scopes like we did in Listings
10-19 y 10-20 para determinar que la referencia que devolvemos siempre
sea válida. El comprobador tampoco puede determinar esto, porque no le hace
saber cómo las vidas de `x` y `y` se relacionan con la vida del retorno
del valor. Vamos a agregar parámetros genéricos de por vida que definirán las
relaciones entre las  referencias para que el comprobador pueda realizar su
análisis.

### Sintaxis de anotación de por vida

Las anotaciones de por vida no cambian la duración de ninguna de las referencias involucradas.
De la misma manera que las funciones pueden aceptar cualquier tipo cuando la firma especifique
un parámetro de tipo genérico, 
las funciones pueden aceptar referencias con cualquier duración si es que
las anotaciones relacionan las vidas de referencias múltiples entre sí.

Las anotaciones de vida tienen una sintaxis levemente inusual: los nombres de los parametros
de la vida deben empezar con el apóstrofe `'`. Los nombres de los parámetros de vida
generalmente son en minúsculas y, como tipos genéricos, sus nombres suelen ser muy
cortos. `'a` es el nombre que la mayoría de la gente usa como valor predeterminado. Las anotaciones de parametros de 
vida van después del `&` de cada referencia, y un espacio separa la anotación
de la vida del tip ode referencia.

Aquí hay algunos ejemplos: tenemos una referencia a un `i32` sin un parametro de
vida, una referencia a un `i32` que tiene un parámetro de vida llamado`'a`,
y una referencia mutable en `i32` que tiene la vida de `'a`:

```rust,ignore
&i32        // una referencia
&'a i32     // una referencia con el tiempo de vida explícito
&'a mut i32 // una referencia mutable con el tiempo de vida explícito
```

Una anotación de por vida por sí misma no tiene mucho significado: duración de las antocaciones
de vida le dicen a Rust cómo los parámetros de vida genéricos de múltiples
referencias se relacionan entre sí. Si tenemos una función con el parámetro
`primero` que es una referencia a `i32`que tiene la vida de `'a`, y la
función tiene otro parámetro llamado `segundo` que es otra referencia a
`i32` que tambien tiene la vida de `'a`, estas dos anotaciones de vida que tienen
el mismo nombre indica que la referencia `primera` y `segunda` deberían vivir
siempre que tengan la misma vida genérica.

### Anotaciones de Vida en las Firmas de Funciones

Veamos las anotaciones de por vida en el contexto de la función más `larga`
que estamos trabajando. Al igual que los parámetros de tipo genérico, los parámetros de
vida genérica deben declararse entre paréntesis angulares entre el nombre de la función
y el listado de parámetros. La restricción que queremos decirle a Rust sobre las
referencias en los parámetros y el valor de retorno es que todos deben tener
el mismo tiempo de vida, al que llamaremos `'a` y agregaremos a cada referencia como se muestra en el
listado 10-23:

<span class="filename">Nombre de archívo: src/main.rs</span>

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

<span class="caption">Listado 10-23: Definición de la función más `larga` function que
especifíca que todas las referencias en la firma deben tener el mismo tiempo de vida de
`'a`</span>

Esto compilará y producirá el resultado que queremos cuando se usa con la función 
`principal` en el listado 10-21.

La firma de la función ahora dice que durante algún tiempo de vida de `'a`, la función
tendrá dos parámetors, ambos son segmentos de cadena que viven por lo menos tan largos
como la vida de `'a`. La función devolverá un segmento de cadena que también
dura al menos tanto como la vida `'a`. Este es el contrato que estamos haciendo que
Rust lo haga cumplir.

Al especificar los parámetros de duración en esta firma de función, no estamos
cambiando la vida útil de cualquier valor pasado o devuelto, pero estamos diciendo
que cualquier valor que no se adhiera a este contrato debe ser rechazado por el
comprobador. Esta función no sabe (o necesita saber) exactamente cuan largo
`x` y `y` vivirán, pero solo necesita saber que hay algún alcance que
puede ser sustituído por `'a` que satisfacerá a la firma.

Al anotar vidas en funciones, las anotaciones van a la función
de la firma, y no en ninguno de los códigos en el cuerpo de la función. Esto es porque
Rust es capaz de analizar el código dentro de la función sin ayuda, pero cuando
una función tiene referencias hacia o desde el código fuera de esa función, las vidas
de los argumentos o valores de retorno serán potencialmente diferentes cada vez que
la función es ejecutada. Esto sería increíblemente costoso y a menudo imposible para
Rust averiguarlo. En este caso, necesitamos anotar las vidas nosotros mismos.

Cuando las referencias concretas se pasan a `larga`, la vida útil concreta que
es sustituída por `'a` es la parte almacenada de `x` que se superpone con
el alcance de `y`. Como los ámbitos siempre anidan, otra forma de decir esto es que
la vida útil genérica `'a` obtendrá la vida útil concreta igual a la más pequeña de
de las vidas de `x` y `y`. Porque hemos anotado la referencia devuelta
con el mismo parámetro de vida `'a`, la referencia devuelta será, por lo tanto,
se garantiza que será válido siempre que sea el más corto de las vidas de `x` y` y`.

Veamos cómo esto restringe el uso de la función `más larga` al pasar
referencias que tienen diferentes tiempos de vida concretos. El listado 10-24 es un
ejemplo directo que debe coincidir con su intuición desde cualquier idioma:
`cadena1` es válido hasta el final del alcance externo, `cadena2` es válido hasta
el final del alcance interno, y `resultado` hace referencia a algo que es válido
hasta el final del alcance interno. El corrector de préstamos aprueba este código; eso
compilará e imprimirá `La secuencia más larga es la cadena larga es larga` cuando se ejecuta:

<span class="filename">Nombre de archivo: src/main.rs</span>

```rust
# fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
#     if x.len() > y.len() {
#         x
#     } else {
#         y
#     }
# }
#
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

<span class="caption">Listado 10-24: Usando la función más `larga` con
referencias a valores de `Cadena` que tienen diferentes tiempos de vida concretos</span>

A continuación, probemos con un ejemplo que mostrará que la vida útil de la referencia en
`resultado` debe ser la vida más pequeña de los dos argumentos. Vamos a mover la
declaración de la variable `resultado` fuera del alcance interno, pero deje la
asignación del valor a la variable `resultado` dentro del alcance con
`cadena2`. A continuación, moveremos el `println!` Que usa `resultado` fuera del
alcance interno, después de que haya terminado. El código en el listado 10-25 no se compilará:

<span class="filename">Nombre de archivo: src/main.rs</span>

```rust,ignore
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

<span class="caption">Listado 10-25: Intentando usar `resultado` después de que `Cadena2`
salió del alcance y no se compiló</span>

Si tratamos de compilar esto, obtendremos este error:

```text
error: `string2` does not live long enough
   |
6  |         result = longest(string1.as_str(), string2.as_str());
   |                                            ------- borrow occurs here
7  |     }
   |     ^ `string2` dropped here while still borrowed
8  |     println!("The longest string is {}", result);
9  | }
   | - borrowed value needs to live until here
```

El error es decir que para que `result` sea válido para` println! `,
`cadena2` necesitaría ser válido hasta el final del alcance externo. Rust sabe
esto porque anotamos las vidas de los parámetros de la función y el retorno
valores con el mismo parámetro de vida, `'a`.

We can look at this code as humans and see that `string1` is longer, and
therefore `result` will contain a reference to `string1`. Because `string1` has
not gone out of scope yet, a reference to `string1` will still be valid for the
`println!`. However, what we’ve told Rust with the lifetime parameters is that
the lifetime of the reference returned by the `longest` function is the same as
the smaller of the lifetimes of the references passed in. Therefore, the borrow
checker disallows the code in Listing 10-25 as possibly having an invalid
reference.

Try designing some more experiments that vary the values and lifetimes of the
references passed in to the `longest` function and how the returned reference
is used. Make hypotheses about whether your experiments will pass the borrow
checker or not before you compile, then check to see if you’re right!

### Thinking in Terms of Lifetimes

The exact way to specify lifetime parameters depends on what your function is
doing. For example, if we changed the implementation of the `longest` function
to always return the first argument rather than the longest string slice, we
wouldn’t need to specify a lifetime on the `y` parameter. This code compiles:

<span class="filename">Filename: src/main.rs</span>

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

In this example, we’ve specified a lifetime parameter `'a` for the parameter
`x` and the return type, but not for the parameter `y`, since the lifetime of
`y` does not have any relationship with the lifetime of `x` or the return value.

When returning a reference from a function, the lifetime parameter for the
return type needs to match the lifetime parameter of one of the arguments. If
the reference returned does *not* refer to one of the arguments, the only other
possibility is that it refers to a value created within this function, which
would be a dangling reference since the value will go out of scope at the end
of the function. Consider this attempted implementation of the `longest`
function that won’t compile:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

Even though we’ve specified a lifetime parameter `'a` for the return type, this
implementation fails to compile because the return value lifetime is not
related to the lifetime of the parameters at all. Here’s the error message we
get:

```text
error: `result` does not live long enough
  |
3 |     result.as_str()
  |     ^^^^^^ does not live long enough
4 | }
  | - borrowed value only lives until here
  |
note: borrowed value must be valid for the lifetime 'a as defined on the block
at 1:44...
  |
1 | fn longest<'a>(x: &str, y: &str) -> &'a str {
  |                                             ^
```

The problem is that `result` will go out of scope and get cleaned up at the end
of the `longest` function, and we’re trying to return a reference to `result`
from the function. There’s no way we can specify lifetime parameters that would
change the dangling reference, and Rust won’t let us create a dangling
reference. In this case, the best fix would be to return an owned data type
rather than a reference so that the calling function is then responsible for
cleaning up the value.

Ultimately, lifetime syntax is about connecting the lifetimes of various
arguments and return values of functions. Once they’re connected, Rust has
enough information to allow memory-safe operations and disallow operations that
would create dangling pointers or otherwise violate memory safety.

### Lifetime Annotations in Struct Definitions

Up until now, we’ve only defined structs to hold owned types. It is possible
for structs to hold references, but we need to add a lifetime annotation on
every reference in the struct’s definition. Listing 10-26 has a struct named
`ImportantExcerpt` that holds a string slice:

<span class="filename">Filename: src/main.rs</span>

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

<span class="caption">Listing 10-26: A struct that holds a reference, so its
definition needs a lifetime annotation</span>

This struct has one field, `part`, that holds a string slice, which is a
reference. Just like with generic data types, we have to declare the name of
the generic lifetime parameter inside angle brackets after the name of the
struct so that we can use the lifetime parameter in the body of the struct
definition.

The `main` function here creates an instance of the `ImportantExcerpt` struct
that holds a reference to the first sentence of the `String` owned by the
variable `novel`.

### Lifetime Elision

In this section, we’ve learned that every reference has a lifetime, and we need
to specify lifetime parameters for functions or structs that use references.
However, in Chapter 4 we had a function in the “String Slices” section, shown
again in Listing 10-27, that compiled without lifetime annotations:

<span class="filename">Filename: src/lib.rs</span>

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

<span class="caption">Listing 10-27: A function we defined in Chapter 4 that
compiled without lifetime annotations, even though the parameter and return
type are references</span>

The reason this function compiles without lifetime annotations is historical:
in early versions of pre-1.0 Rust, this indeed wouldn’t have compiled. Every
reference needed an explicit lifetime. At that time, the function signature
would have been written like this:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

After writing a lot of Rust code, the Rust team found that Rust programmers
were typing the same lifetime annotations over and over in particular
situations. These situations were predictable and followed a few deterministic
patterns. The Rust team then programmed these patterns into the Rust compiler’s
code so that the borrow checker can infer the lifetimes in these situations
without forcing the programmer to explicitly add the annotations.

We mention this piece of Rust history because it’s entirely possible that more
deterministic patterns will emerge and be added to the compiler. In the future,
even fewer lifetime annotations might be required.

The patterns programmed into Rust’s analysis of references are called the
*lifetime elision rules*. These aren’t rules for programmers to follow; the
rules are a set of particular cases that the compiler will consider, and if
your code fits these cases, you don’t need to write the lifetimes explicitly.

The elision rules don’t provide full inference: if Rust deterministically
applies the rules but there’s still ambiguity as to what lifetimes the
references have, it won’t guess what the lifetime of the remaining references
should be. In this case, the compiler will give you an error that can be
resolved by adding the lifetime annotations that correspond to your intentions
for how the references relate to each other.

First, some definitions: Lifetimes on function or method parameters are called
*input lifetimes*, and lifetimes on return values are called *output lifetimes*.

Now, on to the rules that the compiler uses to figure out what lifetimes
references have when there aren’t explicit annotations. The first rule applies
to input lifetimes, and the second two rules apply to output lifetimes. If the
compiler gets to the end of the three rules and there are still references that
it can’t figure out lifetimes for, the compiler will stop with an error.

1. Each parameter that is a reference gets its own lifetime parameter. In other
   words, a function with one parameter gets one lifetime parameter: `fn
   foo<'a>(x: &'a i32)`, a function with two arguments gets two separate
   lifetime parameters: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`, and so on.

2. If there is exactly one input lifetime parameter, that lifetime is assigned
   to all output lifetime parameters: `fn foo<'a>(x: &'a i32) -> &'a i32`.

3. If there are multiple input lifetime parameters, but one of them is `&self`
   or `&mut self` because this is a method, then the lifetime of `self` is
   assigned to all output lifetime parameters. This makes writing methods much
   nicer.

Let’s pretend we’re the compiler and apply these rules to figure out what the
lifetimes of the references in the signature of the `first_word` function in
Listing 10-27 are. The signature starts without any lifetimes associated with
the references:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Then we (as the compiler) apply the first rule, which says each parameter gets
its own lifetime. We’re going to call it `'a` as usual, so now the signature is:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

On to the second rule, which applies because there is exactly one input
lifetime. The second rule says the lifetime of the one input parameter gets
assigned to the output lifetime, so now the signature is:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Now all the references in this function signature have lifetimes, and the
compiler can continue its analysis without needing the programmer to annotate
the lifetimes in this function signature.

Let’s do another example, this time with the `longest` function that had no
lifetime parameters when we started working with in Listing 10-22:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Pretending we’re the compiler again, let’s apply the first rule: each parameter
gets its own lifetime. This time we have two parameters, so we have two
lifetimes:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Looking at the second rule, it doesn’t apply since there is more than one input
lifetime. Looking at the third rule, this also does not apply because this is a
function rather than a method, so none of the parameters are `self`. So we’re
out of rules, but we haven’t figured out what the return type’s lifetime is.
This is why we got an error trying to compile the code from Listing 10-22: the
compiler worked through the lifetime elision rules it knows, but still can’t
figure out all the lifetimes of the references in the signature.

Because the third rule only really applies in method signatures, let’s look at
lifetimes in that context now, and see why the third rule means we don’t have
to annotate lifetimes in method signatures very often.

### Lifetime Annotations in Method Definitions

<!-- Is this different to the reference lifetime annotations, or just a
finalized explanation? -->
<!-- This is about lifetimes on references in method signatures, which is where
the 3rd lifetime elision rule kicks in. It can also be confusing where lifetime
parameters need to be declared and used since the lifetime parameters could go
with the struct's fields or with references passed into or returned from
methods. /Carol -->

When we implement methods on a struct with lifetimes, the syntax is again the
same as that of generic type parameters that we showed in Listing 10-11: the
place that lifetime parameters are declared and used depends on whether the
lifetime parameter is related to the struct fields or the method arguments and
return values.

Lifetime names for struct fields always need to be declared after the `impl`
keyword and then used after the struct’s name, since those lifetimes are part
of the struct’s type.

In method signatures inside the `impl` block, references might be tied to the
lifetime of references in the struct’s fields, or they might be independent. In
addition, the lifetime elision rules often make it so that lifetime annotations
aren’t necessary in method signatures. Let’s look at some examples using the
struct named `ImportantExcerpt` that we defined in Listing 10-26.

First, here’s a method named `level`. The only parameter is a reference to
`self`, and the return value is just an `i32`, not a reference to anything:

```rust
# struct ImportantExcerpt<'a> {
#     part: &'a str,
# }
#
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

The lifetime parameter declaration after `impl` and use after the type name is
required, but we’re not required to annotate the lifetime of the reference to
`self` because of the first elision rule.

Here’s an example where the third lifetime elision rule applies:

```rust
# struct ImportantExcerpt<'a> {
#     part: &'a str,
# }
#
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

There are two input lifetimes, so Rust applies the first lifetime elision rule
and gives both `&self` and `announcement` their own lifetimes. Then, because
one of the parameters is `&self`, the return type gets the lifetime of `&self`,
and all lifetimes have been accounted for.

### The Static Lifetime

There is *one* special lifetime we need to discuss: `'static`. The `'static`
lifetime is the entire duration of the program. All string literals have the
`'static` lifetime, which we can choose to annotate as follows:

```rust
let s: &'static str = "I have a static lifetime.";
```

The text of this string is stored directly in the binary of your program and
the binary of your program is always available. Therefore, the lifetime of all
string literals is `'static`.

<!-- How would you add a static lifetime (below)? -->
<!-- Just like you'd specify any lifetime, see above where it shows `&'static str`. /Carol -->

You may see suggestions to use the `'static` lifetime in error message help
text, but before specifying `'static` as the lifetime for a reference, think
about whether the reference you have is one that actually lives the entire
lifetime of your program or not (or even if you want it to live that long, if
it could). Most of the time, the problem in the code is an attempt to create a
dangling reference or a mismatch of the available lifetimes, and the solution
is fixing those problems, not specifying the `'static` lifetime.

### Generic Type Parameters, Trait Bounds, and Lifetimes Together

Let’s briefly look at the syntax of specifying generic type parameters, trait
bounds, and lifetimes all in one function!

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

This is the `longest` function from Listing 10-23 that returns the longest of
two string slices, but with an extra argument named `ann`. The type of `ann` is
the generic type `T`, which may be filled in by any type that implements the
`Display` trait as specified by the `where` clause. This extra argument will be
printed out before the function compares the lengths of the string slices,
which is why the `Display` trait bound is necessary. Because lifetimes are a
type of generic, the declarations of both the lifetime parameter `'a` and the
generic type parameter `T` go in the same list within the angle brackets after
the function name.

## Summary

We covered a lot in this chapter! Now that you know about generic type
parameters, traits and trait bounds, and generic lifetime parameters, you’re
ready to write code that isn’t duplicated but can be used in many different
situations. Generic type parameters mean the code can be applied to different
types. Traits and trait bounds ensure that even though the types are generic,
those types will have the behavior the code needs. Relationships between the
lifetimes of references specified by lifetime annotations ensure that this
flexible code won’t have any dangling references. And all of this happens at
compile time so that run-time performance isn’t affected!

Believe it or not, there’s even more to learn in these areas: Chapter 17 will
discuss trait objects, which are another way to use traits. Chapter 19 will be
covering more complex scenarios involving lifetime annotations. Chapter 20 will
get to some advanced type system features. Up next, though, let’s talk about
how to write tests in Rust so that we can make sure our code using all these
features is working the way we want it to!
