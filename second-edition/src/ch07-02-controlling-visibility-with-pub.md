## Controlar la Visibilidad con `pub`

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
crearemos un crate binario en el mismo directorio que nuestra biblioteca 
haciendo un archivo *src/main.rs* que contenga este código:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust,ignore
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```

Usamos el comando `extern crate` para llevar el crate de la biblioteca del `comunicador`
al alcance. Nuestro paquete ahora contiene *dos* cajones. Cargo trata *src/main.rs*
como el archivo raíz de un crate binario, que está separado del crate de la biblioteca existente
cuyo archivo raíz es *src/lib.rs*. Este patrón es bastante común para
proyectos ejecutables: la mayoría de las funciones se encuentran en un crate de biblioteca, y el crate
binario usa ese crate de biblioteca. Como resultado, otros programas también pueden usar el
crate de la biblioteca, y es una buena separación de preocupaciones.

Desde el punto de vista de un crate fuera de la biblioteca del `comunicator` 
mirando, todos los módulos que hemos estado creando están dentro de un módulo que tiene el mismo
nombre que el crate, `comunicator`. Llamamos al módulo de nivel superior de un crate el
*módulo raíz*.

También ten en cuenta que incluso si estamos usando un crate externa dentro de un submódulo de
nuestro proyecto, el `crate extern` debería ir en nuestro módulo raíz (también en *src/main.rs*
o *src/lib.rs*). A continuación, en nuestros submódulos, podemos referirnos a los artículos de cajones
externos como si se tratara de módulos de primer nivel.

En este momento, nuestr crate binario sólo llama a la función `connect` de
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
se permitirá nuestro llamado a esa función desde nuestra crate binario, sino que 
la advertencia de que la función no se utiliza desaparecerá. Marcar una función como pública
le permite a Rust saber que la función será usada por código fuera de nuestro programa.
Rust considera el uso externo teórico que ahora es posible como la 
función "en uso". Por lo tanto, cuando una función está marcada como pública, Rust no 
requerirá que se utilice en nuestro programa y dejará de advertir que la función no se 
utiliza.

### Haciendo una Función Pública

Para decirle a Rust que haga pública una función, añadimos la palabra clave `pub` al inicio
de la declaración. Nos centraremos en arreglar la advertencia que indica 
`cliente::connect` ha quedado sin usar por ahora, así como el `` módule `client` is
private `` de nuestr crate binario. Modifica *src/lib.rs* para hacer que
el módulo `client` sea público, así:

<span class="filename">Nombre del archivo: src/lib.rs</span>

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


¡Hurrah! ¡Tenemos un error diferente! Si, los mensajes de error diferentes son causa
de celebración. El nuevo error muestra `` function `connect` is private `` (la función
`connect` es privada), así que editemos *src/client.rs* para hacer `client::connect`
pública también:

<span class="filename">Filename: src/client.rs</span>

```rust
pub fn connect() {
}
```

Ahora ejecute `cargo build` de nuevo:

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

¡El código compiló, y la advertencia sobre `client::connect` no siendo usada
se ha ido!

Las advertencias de código no usado no siempre indican que un objeto en su código
necesita hacerse público: si usted *no* quiso que estas funciones sean parte de su
API público, las advertencias de código no usado podrían estar advirtiéndole de que
el código que ya no necesita puede ser borrado sin problemas. También pudieran estar
advirtiéndole de un error si usted borró accidentalmente todos los lugares en su
librería en los cuales se llama esta función.

Pero en este caso, nosostro *si* necesitamos que las otras dos funciones sean parte
de nuestro API público de nuestro crate, así que marquémoslas como `pub` también para
deshacernos de las advertencias restantes. Modifique *src/network/mod.rs* para que se
vea así:

<span class="filename">Filename: src/network/mod.rs</span>

```rust,ignore
pub fn connect() {
}

mod server;
```

Luego compile el código:

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

Hmmm, aún seguimos recibiendo una advertencia de una función no usada, incluso
cuando `network::connect` fue establecido como `pub`. La razón es que la función
es pública dentro del módulo, pero el módulo `network` en el que la función
está no es público. Esta vez estamos trabajando desde el interior de la librería 
hacia afuera, donde con `client::connect` trabajamos desde afuera hacia adentro.
Necesitamos cambiar *src/lib.rs* para hacer `network` pública también, así:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;
```

Ahora cuando compilamos, la advertencia se fue:

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

¡Sólo queda una advertencia! ¡Intente resolverla por usted mismo!

### Reglas de Privacidad

En general, estas son las reglas para la visibilidad de un objeto:

1. Si un objeto es público, puede ser accedido desde cualquiera de sus módulos padre.
2. Si un objeto es privado, puede ser accedido sólo por su módulo padre inmediato
   y cualquiera de los módulos hijos de ese padre.

### Ejemplos de Privacidad

Veamos algunos ejemplos más para practicar. Crée un nuevo proyecto de librería e
introduzca el código en el Listado 7-6 en el *src/lib.rs* de su proyecto nuevo:

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

Antes de que intente compilar este código, intente adivinar cuales lineas
en la función `try_me` tendrán errores. ¡Luego, intente compilar el código para
ver si tenía razón, y siga leyendo para la discusión de los errores!

#### Viendo los Errores

La función `try_me` está en el módulo raíz de nuestro proyecto. El módulo nombrado
`outermost` es privado, pero la segunda regla de privacidad dice que la función
`try_me` tiene permitido acceder al módulo `outermost` ya que `outermost` está en
el módulo (raíz) actual, al igual que `try_me`.

La llamada a `outermost::middle_secret_function` causará un error de compilación.
`middle_secret_function` es privada, así que la segunda regla aplica. El módulo
raíz no es el módulo actual del `middle_secret_function` (`outermost`
lo es), ni es el módulo hijo del módulo actual de `middle_secret_function`.

El módulo nombrado `inside` es privado y no tiene módulos hijo, así que sólo puede
ser accedido por su módulo actual `outermost`. Eso significa que la función `try_me`
no tiene permitido llamar a `outermost::inside::inner_function` o
`outermost::inside::secret_function`.

#### Arreglando los Errores

Aquí hay algunas sugerencias para cambiar el código intentando arreglar los errores.
Antes de que intente cada una, intente adivinar si esto arreglará los errores, y
luego compile el código para ver si tenía razón o no, usando las reglas de privacidad
para entender el porqué.

* ¿Y si el módulo `inside` fuese público?
* ¿Y si `outermost` fuese público e `inside` fuese privado?
* ¿Y si, en el cuerpo de `inner_function`, usted llamase 
  `::outermost::middle_secret_function()`? (Los dos puntos dobles al principio indican
  que queremos referirnos a los módulos comenzando desde el módulo raíz).

¡Siéntase libre de diseñar más experimentos e intentarlos!

A continuación, hablemos de traer objetos al ambiente con la palabra reservada `use`.
