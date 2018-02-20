## Controlando La Visibilidad con `pub`

Resolvimos los mensajes de error mostrados en el Listado 7-5 moviendo
el código de `network` y `network::server` a los archivos *src/network/mod.rs*
y *src/network/server.rs*,respectivamente. En ese punto, `cargo build`
era capaz de construir nuestro proyecto, pero aún recibimos mensajes de
advertencia sobre las funciones `client::connect`, `network::connect`, y
`network::server::connect` no siendo usadas:

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

¿Entonces por qué recibimos estas advertencias? Despues de todo, estamos
construyendo una librería con funciones que fueron hechas para ser usadas
por nuestros *usuarios*, no necesariamente por nosotros en nuestro propio
proyecto, así que no debería importar que estas funciones `connect` no sean
usadas. El punto de crearlas es que ellas serán usadas por otro proyecto,
no por el nuestro.


Para entender el porqué este programa invoca estas advertencias, intentemos
usando la librería `connect` de otro proyecto, llamándola externamente.
Para hacerlo, crearemos un crate binario en el mismo directorio que nuestro
crate de librería haciendo un archivo *src/main.rs* que contenga este código:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```

Usamos el comando `extern crate` para traer el crate de librería `communicator`
al ambiente. Nuestro paquete ahora contiene *dos* crates. Cargo trata *src/main.rs*
como el archivo raíz de un crate binario, el cual está separado del crate de
librería existente cuyo archivo raíz es *src/lib.rs*. Este patrón es bastante
común para proyecto ejecutables: la mayoría de la funcionalidad está en un crate
de librería, y el crate binario usa ese crate de librería. Como resultado, otros
programas también pueden usar el crate de librería, y es una buena separación de
preocupaciones.

Desde el punto de vista de un crate que está fuera de la librería `communicator`
viendo hacia adentro, todos los módulos que hemos estado creando están dentro de
un módulo que tiene el mismo nombre que el crate, `communicator`. Llamamos *módulo
raíz* al módulo en el nivel tope de un crate.

También note que incluso si estamos usando un crate externo dentro del submódulo
de nuestro proyecto, el `extern crate` debería ir en nuestro módulo raíz (entonces
dentro de *src/main.rs* o *src/lib.rs*). Entonces, en nuestros submódulos, podemos
referirnos a los objetos de crates externos como objetos que son módulos de nivel
tope.

Justo ahora, nuestro crate binario llama a la función de `connect` de nuestra
librería desde el módulo `client`. Sin embargo, invocar `cargo build` ahora nos dará
un error luego de las advertencias:

```text
error[E0603]: module `client` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

¡Ah ha! Este error nos dice que el módulo `cliente` es privado, el cual es el
centro de las advertencias. También es la primera vez que nos topamos con los
conceptos de *público* y *privado* en nuestro contexto de Rust. El estado por
defecto de todo código en Rust es privado: más nadie tiene permitido usar el
código. Si usted no usa una función privada dentro de su programa, ya que su
programa es el único código al cual se le permite usar esa función, entonces
Rust le advertirá que esa función no está siendo usada.

Despues de que especificamos que la función como `client::connect` es pública,
no sólo nuestra llamada a esa función desde nuestro crate binario será permitida,
sino también la advertencia que decía que esa función no estaba siendo usada se
irá. Marcando una función como pública le permite a Rust saber que esa función
será usada por código fuera de nuestro programa. Rust considera el uso externo
teórico que ahora es posible como que la función "esté siendo usada". En
consecuencia, cuando una función es marcada como pública, Rust no requerirá que
sea usada en nuestro programa y no dará advertencias de que la función no está
siendo usada.

### Haciendo Pública una Función

Para decirle a Rust que haga una función pública, añadimos la palabra reservada
`pub` al principio de la declaración. Nos concentraremos en arreglar la advertencia
que indica que `client::connect` se ha vuelto no usada, así como el error `` module
`client` is private `` (módulo `cliente` es privado) de nuestro crate binario.
Modifique *src/lib.rs* para que el módulo `client` sea público, así:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub mod client;

mod network;
```

La palabra reservada `pub` se coloca justo antes de `mod`. Intentemos construyendo
denuevo:

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
