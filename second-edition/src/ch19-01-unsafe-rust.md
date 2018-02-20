## Unsafe Rust (Rust Inseguro)

Todo el código que hemos discutido hasta ahora ha tenido las garantías
de seguridad de la memoria de Rust aplicado en tiempo de compilación. 
Sin embargo, Rust tiene un segundo idioma escondido dentro, que no hace 
cumplir estas garantías de seguridad de la memoria: Unsafe Rust. Este
funciona como el Rust normal, pero te da superpoderes adicionales.


Unsafe Rust existe porque, por naturaleza, el análisis estático es conservador.
Cuando el compilador está tratando de determinar si el código mantiene las 
garantías o no, es mejor rechazar algunos programas que son válidos que aceptar
algunos programas que son inválidos. Eso inevitablemente significa que hay 
algunas veces que su código podría estar bien, ¡pero Rust piensa que no! 
En estos casos, puede utilizar el código inseguro para decirle al compilador,
"créeme, sé lo que estoy haciendo". El inconveniente es que estás solo; si obtiene
un código no seguro incorrecto, tendrá problemas con la memoria debido a la inseguridad,
puede ocurrir, por ejemplo la desreferenciación de indicadores nulos, 


Hay otra razón por la cual Rust tiene un alter ego inseguro: el hardware subyacente de
las computadoras no son intrínsecamente seguras. Si Rust no te permitió hacer 
operaciones inseguras, habrá algunas tareas que simplemente no podras hacer. 
Rust necesita permitirte hacer programación de sistemas de bajo nivel, como 
interactuar directamente con su sistema operativo, o incluso escribir su 
propio sistema operativo! Ese es uno de los objetivos del lenguaje. 
Veamos qué puedes hacer con Unsafe Rust y cómo hacerlo.


### Superpoderes inseguros

Para cambiar a Unsafe Rust usamos la palabra clave `Unsafe`, 
y luego podemos comenzar un nuevo bloque que contiene el código inseguro.
Hay cuatro acciones que puedes tomar en Unsafe Rust que no se puede en 
Rust seguro que llamamos "superpotencias inseguras".Esos superpoderes 
son la capacidad de:

1. Desreferenciar un indicador sin formato 
2. Llamar a una función o método inseguro 
3. Acceder o modificar una variable estática mutable 
4. Implementar un rasgo inseguro

Es importante entender que `Unsafe` no apaga el comprobador de préstamos
o deshabilita cualquier otra verificación de seguridad de Rust: si utiliza
una referencia en código inseguro, aún será verificado. La palabra clave 
`Unsafe` solo le da acceso a estas cuatro características que el compilador
no verifica la memoria para la seguridad. ¡Todavía obtienes cierto grado de 
seguridad dentro de un bloque inseguro!

Además, `Unsafe` no significa que el código dentro del bloque sea necesariamente
peligroso o que definitivamente tendrá problemas de seguridad en la memoria: 
la intención es que usted como programador se asegurará de que el código dentro
de un bloque `Unsafe` acceda a la memoria de una manera válida.

Las personas son falibles y los errores ocurrirán, pero al requerir estas cuatro
operaciones inseguras para estar dentro de bloques anotados como `Unsafe`, 
sabrá que cualquier error relacionado con la seguridad de la memoria debe estar
dentro de un bloque "inseguro". Al mantener 'inseguro' te agradecerás más tarde cuando vayas a
investigar errores de memoria.


Para aislar el código inseguro tanto como sea posible, es una buena idea encerrar el
código inseguro dentro de una abstracción segura y proporcionar una API segura, que estaremos
discutiendo una vez que entremos en funciones y métodos inseguros. Partes del estándar de la
biblioteca se implementan como abstracciones seguras sobre el código inseguro que ha sido
auditado Esta técnica evita que los usos de "inseguro" se filtren en todos los
lugares que usted o sus usuarios pueden querer hacer uso de la funcionalidad
implementado con el código `Unsafe`, porque usar una abstracción segura es seguro.

Vamos a hablar sobre cada una de las cuatro superpotencias inseguras a su vez,
y en el camino veremos algunas abstracciones que proporcionan una interfaz 
segura para el código inseguro.

### Desreferenciando un indicador sin formato

De regreso en el Capítulo 4, en la sección "Referencias que cuelgan", cubrir
al compilador asegura que las referencias son siempre válidas. Unsafe Rust 
tiene dos tipos nuevos similares a las referencias llamadas * indicadores 
crudos *. Al igual que con las referencias, sin procesar los punteros pueden 
ser inmutables o mutables, escritos como `* const T` y` * mut T`,
respectivamente. El asterisco no es el operador de desreferencia; es parte de como
escribir un nombre. En el contexto de punteros crudos, "inmutable" significa que el puntero
no se puede asignar directamente después de haber sido desreferenciado.


A diferencia de las referencias y los indicadores inteligentes, 
tenga en cuenta que a los indicadores sin formato:

- Se les permite ignorar las reglas de préstamo y tener tanto indicadores 
inmutables como mutables, o múltiples indicadores mutables en la misma ubicación 
- No se garantiza que apunte a la memoria válida 
- Se les permite ser nulo 
- No implementa ninguna limpieza automática


Al optar por que Rust no haga cumplir estas garantías, puede hacer la
compensación de renunciar a la seguridad garantizada para obtener rendimiento o la capacidad de
interfaz con otro idioma o hardware donde las garantías de Rust no se aplican.


<!-- ¿Puede decir aquí qué beneficios ofrecen estos, sobre indicadores inteligentes y
referencias, y el uso de los aspectos en estas viñetas? ->
<! - No hay realmente beneficios para cada uno de estos individualmente. Estas son las
advertencias que el lector debe tener en cuenta cuando se trabaja con indicadores crudos.
Elegiría usar indicadores sin procesar para hacer algo que no puede hacer con
Indicadores crudos o referencias. He tratado de aclarar arriba / Carol -->


El listado 19-1 muestra cómo crear un indicador sin procesar inmutable 
y mutable de referencias.

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;
```

<span class="caption">Listing 19-1: Crear indicadores crudos a partir de referencias</span>

<!-- ¿Entonces creamos un indicador sin formato usando el operador de desreferencia? 
¿Es el mismo operador? ¿Vale la pena tocar, por qué? -> <! - No es el operador de 
desreferencia, el * es parte del tipo. Trató de aclarar arriba donde se introducen 
los tipos / Carol ->

Tenga en cuenta que no incluimos la palabra clave `unsafe` aquí 
--- usted puede * crear * sin procesar indicadores en código seguro,
simplemente no puede * desreferenciar * indicadores crudos fuera 
de un bloque inseguro, como veremos en un momento.

Hemos creado indicadores crudos mediante el uso de `as` para convertir
una referencia inmutable y un mutable en sus correspondientes tipos 
de indicadores sin procesar. Porque los creamos directamente de las 
referencias que se garantiza que son válidas, podemos saber que estos
indicadores crudos particulares son válidos, pero no podemos hacer 
esa suposición sobre cualquier indicador sin formato.

A continuación, crearemos un indicador sin formato de cuya validez
no podemos estar tan seguros. El Listado 19-2 muestra cómo crear 
un indicador sin formato en una ubicación arbitraria en la
memoria. Tratar de usar memoria arbitraria no está definido: 
puede haber datos en ese momento de dirección o no, el compilador
podría optimizar el código para que este no tenga acceso a memoria,
o su programa puede fallar. No suele haber una buena
razón para escribir un código como este, pero es posible:


```rust
let address = 0x012345usize;
let r = address as *const i32;
```

<span class="caption">Listing 19-2: Crear un indicador sin formato a una dirección arbitraria de memoria</span>

Recuerde que dijimos que puede crear indicadores crudos en 
código seguro, pero no puede * desreferenciar * punteros sin 
indicadores y leer los datos que se apuntan. Lo haremos ahora
utilizando el operador de desreferencia, `*`, en un indicador 
sin formato, que requiere un bloque `unsafe`, como se muestra 
en el Listado 19-3:


```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```

<span class="caption">Listing 19-3: Desreferencia indicadores sin procesar 
    dentro de un `unsafe`bloqueado </span>

Crear un indicador no puede hacer ningún daño; solo cuando se accede al valor que
apunta a que podrías terminar lidiando con un valor inválido.

Tenga en cuenta también que en el listado 19-1 y 19-3 creamos `* const i32` y` * mut i32`
indicadores crudos que apuntan a la misma ubicación de memoria, la de `num`. Si
en cambio, intentamos crear una referencia inmutable y mutable a `num`,
esto no se habría compilado porque las reglas de propiedad de Rust no permiten una
referencia mutable al mismo tiempo que cualquier referencia inmutable. 
Con un indicador crudo podemos crear un indicador mutable y uno inmutable para
la misma ubicación, y cambiar los datos a través del indicador mutable, potencialmente
creando una carrera de datos. ¡Ten cuidado!


Con todos estos peligros, ¿por qué alguna vez usaríamos indicadores crudos? Un uso principal
 es cuando se conecta con el código C, como veremos en la siguiente sección
funciones inseguras. Otro caso es cuando se construyen abstracciones seguras que el
el prestamista no entiende. Vamos a introducir funciones inseguras y luego mirar
en un ejemplo de una abstracción segura que usa código inseguro.


### Llamar a una función o método inseguro

El segundo tipo de operación que requiere un bloque inseguro es llamado
funciones inseguras. Las funciones y métodos inseguros se ven exactamente 
como las funciones regulares y métodos, pero tienen un frente `unsafe` extra.
Eso `unsafe` indica que la función tiene requisitos que nosotros como 
programadores necesitamos mantener cuando llamamos esta función, porque 
Rust no puede garantizar que hemos cumplido con estos requisitos.
Llamando una función insegura dentro de un bloque `unsafe`, estamos diciendo
que hemos leído estas documentaciones de la función y asumimos la responsabilidad
de mantener la función de contratos nosotros mismos.

<!-- Above -- so what is the difference, when and why would we ever use the
unsafe function? -->
<!-- Tried to clarify /Carol -->

Aquí hay una función insegura llamada `dangerous` que no hace nada en su cuerpo:

```rust
unsafe fn dangerous() {}

unsafe {
    dangerous();
}
```

Debemos llamar a la función `dangerous` dentro de un bloque `unsafe` separado. 
Si nosotros intentamos llamar a `dangerous` sin el bloque `unsafe`, obtendremos un error:


```text
error[E0133]: call to unsafe function requires unsafe function or block
 -->
  |
4 |     dangerous();
  |     ^^^^^^^^^^^ call to unsafe function
```

Al insertar el bloque `unsafe` alrededor de nuestra llamada a `dangerous`, 
estamos afirmandoa Rust que hemos leído la documentación para esta función, 
entendemos cómo usarlo correctamente, y hemos verificado que todo este correcto.


Los cuerpos de funciones `unsafe` son efectivamente bloques "inseguros", 
por lo que para realizar otras operaciones inseguras dentro de una función 
insegura, no necesitamos agregar otro bloque `unsafe`.


#### Creando una abstracción segura sobre un código inseguro

El hecho de que una función contenga un código inseguro no significa que toda la función
debe marcarse como insegura. De hecho, envolver un código inseguro en una función segura
es una abstracción común. Como, por ejemplo, vamos a ver una función de la
biblioteca estándar, `split_at_mut`, que requiere un código inseguro y explorar
cómo podríamos implementarlo. Este método seguro se define en rebanadas mutables:
toma una porción y la divide en dos, en el índice dado
como un argumento Usando `split_at_mut` se muestra en el Listado 19-4:


```rust
let mut v = vec![1, 2, 3, 4, 5, 6];

let r = &mut v[..];

let (a, b) = r.split_at_mut(3);

assert_eq!(a, &mut [1, 2, 3]);
assert_eq!(b, &mut [4, 5, 6]);
```

<span class="caption">Listing 19-4: Using the safe `split_at_mut`
function</span>

Esta función no puede implementarse utilizando solo Rust seguro. Un intento podría verse
algo así como el Listado 19-5, que no compilará. Para simplificar, estamos
implementando `split_at_mut` como una función en lugar de un método, y solo para
segmentos de valores `i32` en lugar de un tipo genérico` T`.


```rust,ignore
fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();

    assert!(mid <= len);

    (&mut slice[..mid],
     &mut slice[mid..])
}
```

<span class="caption">Listing 19-5: Un intento de implementación de `split_at_mut` usando solo Rust seguro</span>

Esta función primero obtiene la longitud total de la porción y luego afirma que el
índice dado como parámetro está dentro del segmento al verificar que es menor que
o igual a la longitud. La afirmación significa que si pasamos un índice que es
mayor que la porción que dividimos, la función entrará en pánico antes
intentar usar ese índice.


Luego devolvemos dos rebanadas mutables en una tupla: 
una desde el comienzo de la rebanada original en el índice `mid`, 
y otra desde `mid` hasta el final de la rebanada. 

Si tratamos de compilar esto, obtendremos un error:

```text
error[E0499]: cannot borrow `*slice` as mutable more than once at a time
 -->
  |
6 |     (&mut slice[..mid],
  |           ----- first mutable borrow occurs here
7 |      &mut slice[mid..])
  |           ^^^^^ second mutable borrow occurs here
8 | }
  | - first borrow ends here
```

El inspector de préstamos de Rust no puede entender que estamos 
tomando prestadas diferentes partes de la rebanada; solo sabe que 
tomamos prestado de la misma porción dos veces. Pedir prestado 
diferentes partes de una rebanada está fundamentalmente bien porque 
nuestras dos rebanadas no se superponen, pero Rust no es lo suficientemente
inteligente como para saber esto. Cuando nosotros sabemos que algo está bien, 
pero Rust no, es hora de buscar un código inseguro.


Listing 19-6 shows how to use an `unsafe` block, a raw pointer, and some calls
to unsafe functions to make the implementation of `split_at_mut` work:

```rust
use std::slice;

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (slice::from_raw_parts_mut(ptr, mid),
         slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid))
    }
}
```

<span class="caption">Listing 19-6: Using unsafe code in the implementation of
the `split_at_mut` function</span>

Recall from the “Slices” section in Chapter 4 that slices are a pointer to some
data and the length of the slice. We use the `len` method to get the length of
a slice, and the `as_mut_ptr` method to access the raw pointer of a slice. In
this case, because we have a mutable slice to `i32` values, `as_mut_ptr`
returns a raw pointer with the type `*mut i32`, which we’ve stored in the
variable `ptr`.

We keep the assertion that the `mid` index is within the slice. Then we get to
the unsafe code: the `slice::from_raw_parts_mut` function takes a raw pointer
and a length and creates a slice. We use this function to create a slice that
starts from `ptr` and is `mid` items long. Then we call the `offset` method on
`ptr` with `mid` as an argument to get a raw pointer that starts at `mid`, and
we create a slice using that pointer and the remaining number of items after
`mid` as the length.

The function `slice::from_raw_parts_mut` is unsafe because it takes a raw
pointer and must trust that this pointer is valid. The `offset` method on raw
pointers is also unsafe, because it must trust that the offset location is also
a valid pointer. We therefore had to put an `unsafe` block around our calls to
`slice::from_raw_parts_mut` and `offset` to be allowed to call them. We can
tell, by looking at the code and by adding the assertion that `mid` must be
less than or equal to `len`, that all the raw pointers used within the `unsafe`
block will be valid pointers to data within the slice. This is an acceptable
and appropriate use of `unsafe`.

Note that we don’t need to mark the resulting `split_at_mut` function as
`unsafe`, and we can call this function from safe Rust. We’ve created a safe
abstraction to the unsafe code with an implementation of the function that uses
`unsafe` code in a safe way because it creates only valid pointers from the
data this function has access to.

In contrast, the use of `slice::from_raw_parts_mut` in Listing 19-7 would
likely crash when the slice is used. This code takes an arbitrary memory
location and creates a slice ten thousand items long:

```rust
use std::slice;

let address = 0x012345usize;
let r = address as *mut i32;

let slice = unsafe {
    slice::from_raw_parts_mut(r, 10000)
};
```

<span class="caption">Listing 19-7: Creating a slice from an arbitrary memory
location</span>

We don’t own the memory at this arbitrary location, and there’s no guarantee
that the slice this code creates contains valid `i32` values. Attempting to use
`slice` as if it was a valid slice would result in undefined behavior.

#### Using `extern` Functions to Call External Code

Sometimes, your Rust code may need to interact with code written in another
language. For this, Rust has a keyword, `extern`, that facilitates the creation
and use of a *Foreign Function Interface* (FFI). A Foreign Function Interface
is a way for a programming language to define functions and enable a different
(foreign) programming language to call those functions.

<!-- Can you give a definition for FFI? -->
<!-- Done /Carol -->

Listing 19-8 demonstrates how to set up an integration with the `abs` function
from the C standard library. Functions declared within `extern` blocks are
always unsafe to call from Rust code, because other languages don`t enforce
Rust's rules and guarantees and Rust can't check them, so responsibility falls
on the programmer to ensure safety:

<span class="filename">Filename: src/main.rs</span>

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

<span class="caption">Listing 19-8: Declaring and calling an `extern` function
defined in another language</span>

Within the `extern "C"` block, we list the names and signatures of external
functions from another language we want to be able to call. The `"C"` part
defines which *application binary interface* (ABI) the external function
uses---the ABI defines how to call the function at the assembly level. The
`"C"` ABI is the most common, and follows the C programming language’s ABI.

<!-- PROD: START BOX -->

##### Calling Rust Functions from Other Languages

You can also use `extern` to create an interface that allows other languages to
call Rust functions. Instead of an `extern` block, we add the `extern` keyword
and specify the ABI to use just before the `fn` keyword. We also need to add a
`#[no_mangle]` annotation to tell the Rust compiler not to mangle the name of
this function. Mangling is when a compiler changes the name we’ve given a
function to a different name that contains more information for other parts of
the compilation process to consume but is less human readable. Every
programming language compiler mangles names slightly differently, so for a Rust
function to be nameable from other languages, we have to disable the Rust
compiler’s name mangling.

<!-- have we discussed mangling before this? It doesn't ring a bell with me,
though it may have been in an early chapter that I forgot --- if not could you
give a quick explanation here? -->
<!-- I've tried, without going into too much detail! /Carol -->

In this example we make the `call_from_c` function accessible from C code, once
it’s compiled to a shared library and linked from C:

```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

This usage of `extern` does not require `unsafe`.

<!-- PROD: END BOX -->

### Accessing or Modifying a Mutable Static Variable

We’ve managed to go this entire book without talking about *global variables*,
which Rust does support, but which can be problematic with Rust’s ownership
rules. If you have two threads accessing the same mutable global variable, it
can cause a data race.

Global variables are called *static* variables in Rust. Listing 19-9 shows an
example declaration and use of a static variable with a string slice as a value:

<span class="filename">Filename: src/main.rs</span>

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```

<span class="caption">Listing 19-9: Defining and using an immutable static
variable</span>

`static` variables are similar to constants, which we discussed in the
“Differences Between Variables and Constants” section in Chapter 3. The names
of static variables are in `SCREAMING_SNAKE_CASE` by convention, and we *must*
annotate the variable’s type, which is `&'static str` in this case. Static
variables may only store references with the `'static` lifetime, which means
the Rust compiler can figure out the lifetime by itself and we don’t need to
annotate it explicitly. Accessing an immutable static variable is safe.

Constants and immutable static variables may seem similar, but a subtle
difference is that values in a static variable have a fixed address in memory.
Using the value will always access the same data. Constants, on the other hand,
are allowed to duplicate their data whenever they are used.

Another difference between constants and static variables is that static
variables can be mutable. Both accessing and modifying mutable static variables
is *unsafe*. Listing 19-10 shows how to declare, access, and modify a mutable
static variable named `COUNTER`:

<span class="filename">Filename: src/main.rs</span>

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

<span class="caption">Listing 19-10: Reading from or writing to a mutable
static variable is unsafe</span>

Just like with regular variables, we specify mutability using the `mut`
keyword. Any code that reads or writes from `COUNTER` must be within an
`unsafe` block. This code compiles and prints `COUNTER: 3` as we would expect
because it’s single threaded. Having multiple threads access `COUNTER` would
likely result in data races.

With mutable data that’s globally accessible, it’s difficult to ensure there
are no data races, which is why Rust considers mutable static variables to be
unsafe. Where possible, it’s preferable to use the concurrency techniques and
thread-safe smart pointers we discussed in Chapter 16, so the compiler checks
that data accessed from different threads is done safely.

### Implementing an Unsafe Trait

Finally, the last action that only works with `unsafe` is implementing an
unsafe trait. A trait is unsafe when at least one of its methods has some
invariant that the compiler can’t verify. We can declare that a trait is
`unsafe` by adding the `unsafe` keyword before `trait`, and then implementation
of the trait must be marked as `unsafe` too, as shown in Listing 19-11:

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```

<span class="caption">Listing 19-11: Defining and implementing an unsafe
trait</span>

By using `unsafe impl`, we’re promising that we’ll uphold the invariants that
the compiler can’t verify.

As an example, recall the `Sync` and `Send` marker traits from the “Extensible
Concurrency with the `Sync` and `Send` Traits” section of Chapter 16, and that
the compiler implements these automatically if our types are composed entirely
of `Send` and `Sync` types. If we implement a type that contains something
that’s not `Send` or `Sync`, such as raw pointers, and we want to mark that
type as `Send` or `Sync`, we must use `unsafe`. Rust can’t verify that our type
upholds the guarantees that it can be safely sent across threads or accessed
from multiple threads, so we need to do those checks ourselves and indicate as
such with `unsafe`.

### When to Use Unsafe Code

Using `unsafe` to take one of these four actions isn’t wrong or even frowned
upon, but it is trickier to get `unsafe` code correct because the compiler isn’t
able to help uphold memory safety. When you have a reason to use `unsafe` code,
it is possible to do so, and having the explicit `unsafe` annotation makes it
easier to track down the source of problems if they occur.
