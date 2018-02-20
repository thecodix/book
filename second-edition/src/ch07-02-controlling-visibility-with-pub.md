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

La palabra clave `pub` se coloca justo antes de `mod`. Intentemos construir de nuevo:

```text
error[E0603]: function `connect` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

¡Hurra! Tenemos un error diferente! Sí, diferentes mensajes de error son motivo
de celebración. El nuevo error muestra que ``función `connect` is private``, así 
que editemos *src/client.rs* para hacer que `client::connect` también sea público:

<span class="filename">Filename: src/client.rs</span>

```rust
pub fn connect() {
}
```

Ahora vuelve a ejecutar `cargo build`:

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

El código ha compilado, y la advertencia de `client::connect` que no se está usando
ha desaparecido!

Las advertencias de código no utilizado no siempre indican que un elemento de tu código debe
hacerse público: si tu *no* deseas que estas funciones formen parte de tu API
pública, las advertencias de código no utilizadas podrían estar alertándote sobre el código que ya no necesitas
para poder eliminarlo de forma segura. También podrían estar alertándote de un error si acabas
de eliminar accidentalmente todos los lugares de tu biblioteca donde se llama 
esta función.

Pero en este caso, *queremos* que las otras dos funciones formen parte de la
API pública de nuestro cajón, así que marquémoslas como `pub` y deshagámonos de las 
advertencias restantes. Modifica *src/network/mod.rs* para que luzca de la siguiente forma:

<span class="filename">Nombre del archivo: src/network/mod.rs</span>

```rust,ignore
pub fn connect() {
}

mod server;
```

Luego compila el código:

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

Hmmm, todavía estamos recibiendo una advertencia de función no utilizada, aunque
`network::connect` está configurada en `pub`. La razón es que la función es pública
dentro del módulo, pero el módulo de `network` en el que reside la función no es
público. Estamos trabajando desde el interior de la biblioteca esta vez, mientras que
con `client::connect` trabajamos desde el exterior. Necesitamos cambiar
*src/lib.rs* para hacer también a `network` pública, así:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;
```

Ahora cuando compilamos, esa advertencia desaparece:

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

Sólo queda una advertencia! ¡Intenta arreglar esto por tu cuenta!

### Reglas de Privacidad

En general, estas son las reglas para la visibilidad de los artículos:

1. Si un elemento es público, se puede acceder a él a través de cualquiera de sus módulos padre.
2. Si un elemento es privado, sólo puede ser accedido por su módulo padre inmediato
y por cualquiera de los módulos hijo del padre.

### Ejemplos de Privacidad

Veamos algunos ejemplos más de privacidad para tener un poco de práctica. Crea un nuevo
proyecto de biblioteca e introduce el código en el Listado 7-6 en el archivo *src/lib.rs*
de tu nuevo proyecto:

<span class="filename">Nombre del archivo: src/lib.rs</span>

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

<span class="caption">Listado 7-6: Ejemplos de funciones privadas y públicas,
algunas de las cuales son incorrectas</span>

Antes de intentar compilar este código, adivina qué líneas de la función
`try_me` tendrán errores. Luego, intenta compilar el código para ver si
tenías razón, y sigue leyendo para la discusión de los errores!

#### Analizando los Errores

La función `try_me` está en el módulo raíz de nuestro proyecto. El módulo llamado
`outermost` es privado, pero la segunda regla de privacidad establece que la función
`try_me` permite el acceso al módulo `outermost` porque `outermost` está en
el módulo (raíz) actual, como es `try_me`.

La llamada a `outermost::middle_function` funcionará porque `middle_function` es 
pública, y `try_me` está accediendo a `middle_function` a través de su módulo
padre `outermost`. Hemos determinado en el párrafo anterior que este módulo
es accesible.

La llamada a la función `outermost::middle_secret_function` causará un error de compilación.
`middle_secret_function` es privada, por lo que se aplica la segunda regla. El módulo 
raíz no es ni el módulo actual de la función `middle_secret_function` (es `outermost`),
ni es un módulo hijo del módulo actual de la función `middle_secret_function`.

El módulo llamado `inside` es privado y no tiene módulos hijo, por lo que sólo
se puede acceder al módulo actual `outermost`. Esto significa que la función `try_me`
no puede llamar a la función `outermost::inside::inner_function` o 
`outermost::inside::secret_function`.

#### Corrección de los Errores

Aquí están algunas sugerencias para cambiar el código en un intento de corregir los
errores. Antes de probar cada una de ellas, adivina si corregirá los 
errores y luego compila el código para ver si tienes razón o no, usando las
reglas de privacidad para entender por qué.

* ¿Y si el módulo `inside` fuera público?
* ¿Qué pasaría si `outermost` fuera público e `inside` fuera privado?
* ¿Qué pasa si, en el cuerpo de `inner_function`, llamaste a
  `::outermost::middle_secret_function()`? (Los dos puntos al principio significan
  que queremos referirnos a los módulos empezando por el módulo raíz.)

Siéntete libre de diseñar más experimentos y probarlos!

A continuación, hablemos de poner los artículos en scope con la palabra clave `use`.
