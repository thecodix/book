## Controlar la visibilidad con `pub`

Resolvimos los mensajes de error mostrados en el Listado 7-5 moviendo el código `network` y 
`network::server` a los archivos *src/network/mod.rs* y 
*src/network/server.rs*, respectivamente. En ese momento, `cargo build` fue
capaz de construir nuestro proyecto, pero aún así recibimos mensajes de advertencia sobre las
funciones `client::connect`, `network::connect`, y `network::server::connect` 
que no se usaban:

```text
warning: function is never used: `connect`
 --> src/client.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

¿Por qué recibimos estas advertencias? Después de todo, estamos construyendo una biblioteca
con funciones que están pensadas para ser usadas por nuestros *usuarios*, no necesariamente por
nosotros dentro de nuestro propio proyecto, así que no debería importar que estas funciones de
`connect` no se utilicen. El punto de crearlas es que serán utilizadas por
otro proyecto, no por el nuestro.

Para entender por qué este programa invoca estas advertencias, intentemos usar la
biblioteca `connect` de otro proyecto, llamándola externamente. Para ello, 
crearemos un cajón binario en el mismo directorio que nuestra biblioteca 
haciendo un archivo *src/main.rs* que contenga este código:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```

Usamos el comando `extern crate` para llevar el cajón de la biblioteca del `comunicador`
al alcance. Nuestro paquete ahora contiene *dos* cajones. Cargo trata *src/main.rs*
como el archivo raíz de un cajón binario, que está separado del cajón de la biblioteca existente
cuyo archivo raíz es *src/lib.rs*. Este patrón es bastante común para
proyectos ejecutables: la mayoría de las funciones se encuentran en un cajón de biblioteca, y el cajón
binario usa esa cajón de biblioteca. Como resultado, otros programas también pueden usar el
cajón de la biblioteca, y es una buena separación de preocupaciones.

Desde el punto de vista de un cajón fuera de la biblioteca del `comunicator` 
mirando, todos los módulos que hemos estado creando están dentro de un módulo que tiene el mismo
nombre que el cajón, `comunicator`. Llamamos al módulo de nivel superior de un cajón el
*módulo raíz*.

También ten en cuenta que incluso si estamos usando un cajón externa dentro de un submódulo de
nuestro proyecto, el `cajón externo` debería ir en nuestro módulo raíz (también en *src/main.rs*
o *src/lib.rs*). A continuación, en nuestros submódulos, podemos referirnos a los artículos de cajones
externos como si se tratara de módulos de primer nivel.

En este momento, nuestr cajón binario sólo llama a la función `connect` de
nuestra biblioteca desde el módulo `client`. Sin embargo, invocando `cargo build` ahora nos dará un error
después de las advertencias:

```text
error[E0603]: module `client` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

¡Ah ha! Este error nos dice que el módulo `client` es privado, que es el punto
crucial de las advertencias. Es también la primera vez que nos encontramos con los conceptos de
*public* y *private* en el contexto de Rust. El estado predeterminado de todo el código en 
Rust es privado: nadie más puede usar el código. Si tu no usas una 
función privada dentro de tu programa, porque tu programa es el único código
permitido para usar esa función, Rust te advertirá que la función no se ha
usado.

Después de especificar que una función como `client::connect` es pública, no sólo 
se permitirá nuestro llamado a esa función desde nuestra cajón binario, sino que 
la advertencia de que la función no se utiliza desaparecerá. Marcar una función como pública
le permite a Rust saber que la función será usada por código fuera de nuestro programa.
Rust considera el uso externo teórico que ahora es posible como la 
función "en uso". Por lo tanto, cuando una función está marcada como pública, Rust no 
requerirá que se utilice en nuestro programa y dejará de advertir que la función no se utiliza.

### Haciendo una Función Pública

Para decirle a Rust que haga pública una función, añadimos la palabra clave `pub` al inicio
de la declaración. Nos centraremos en arreglar la advertencia que indica 
`cliente::connect` ha quedado sin usar por ahora, así como el `` módule `client` is
private `` de nuestr cajón binario. Modifica *src/lib.rs* para hacer que
el módulo `client` sea público, así:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub mod client;

mod network;
```

The `pub` keyword is placed right before `mod`. Let’s try building again:

```text
error[E0603]: function `connect` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Hooray! We have a different error! Yes, different error messages are a cause
for celebration. The new error shows `` function `connect` is private ``, so
let’s edit *src/client.rs* to make `client::connect` public too:

<span class="filename">Filename: src/client.rs</span>

```rust
pub fn connect() {
}
```

Now run `cargo build` again:

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

The code compiled, and the warning about `client::connect` not being used is
gone!

Unused code warnings don’t always indicate that an item in your code needs to
be made public: if you *didn’t* want these functions to be part of your public
API, unused code warnings could be alerting you to code you no longer need that
you can safely delete. They could also be alerting you to a bug if you had just
accidentally removed all places within your library where this function is
called.

But in this case, we *do* want the other two functions to be part of our
crate’s public API, so let’s mark them as `pub` as well to get rid of the
remaining warnings. Modify *src/network/mod.rs* to look like the following:

<span class="filename">Filename: src/network/mod.rs</span>

```rust,ignore
pub fn connect() {
}

mod server;
```

Then compile the code:

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / pub fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

Hmmm, we’re still getting an unused function warning, even though
`network::connect` is set to `pub`. The reason is that the function is public
within the module, but the `network` module that the function resides in is not
public. We’re working from the interior of the library out this time, whereas
with `client::connect` we worked from the outside in. We need to change
*src/lib.rs* to make `network` public too, like so:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;
```

Now when we compile, that warning is gone:

```text
warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default
```

Only one warning is left! Try to fix this one on your own!

### Privacy Rules

Overall, these are the rules for item visibility:

1. If an item is public, it can be accessed through any of its parent modules.
2. If an item is private, it can be accessed only by its immediate parent
   module and any of the parent’s child modules.

### Privacy Examples

Let’s look at a few more privacy examples to get some practice. Create a new
library project and enter the code in Listing 7-6 into your new project’s
*src/lib.rs*:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod outermost {
    pub fn middle_function() {}

    fn middle_secret_function() {}

    mod inside {
        pub fn inner_function() {}

        fn secret_function() {}
    }
}

fn try_me() {
    outermost::middle_function();
    outermost::middle_secret_function();
    outermost::inside::inner_function();
    outermost::inside::secret_function();
}
```

<span class="caption">Listing 7-6: Examples of private and public functions,
some of which are incorrect</span>

Before you try to compile this code, make a guess about which lines in the
`try_me` function will have errors. Then, try compiling the code to see whether
you were right, and read on for the discussion of the errors!

#### Looking at the Errors

The `try_me` function is in the root module of our project. The module named
`outermost` is private, but the second privacy rule states that the `try_me`
function is allowed to access the `outermost` module because `outermost` is in
the current (root) module, as is `try_me`.

The call to `outermost::middle_function` will work because `middle_function` is
public, and `try_me` is accessing `middle_function` through its parent module
`outermost`. We determined in the previous paragraph that this module is
accessible.

The call to `outermost::middle_secret_function` will cause a compilation error.
`middle_secret_function` is private, so the second rule applies. The root
module is neither the current module of `middle_secret_function` (`outermost`
is), nor is it a child module of the current module of `middle_secret_function`.

The module named `inside` is private and has no child modules, so it can only
be accessed by its current module `outermost`. That means the `try_me` function
is not allowed to call `outermost::inside::inner_function` or
`outermost::inside::secret_function`.

#### Fixing the Errors

Here are some suggestions for changing the code in an attempt to fix the
errors. Before you try each one, make a guess as to whether it will fix the
errors, and then compile the code to see whether or not you’re right, using the
privacy rules to understand why.

* What if the `inside` module was public?
* What if `outermost` was public and `inside` was private?
* What if, in the body of `inner_function`, you called
  `::outermost::middle_secret_function()`? (The two colons at the beginning mean
  that we want to refer to the modules starting from the root module.)

Feel free to design more experiments and try them out!

Next, let’s talk about bringing items into scope with the `use` keyword.
