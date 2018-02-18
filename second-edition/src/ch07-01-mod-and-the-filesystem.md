## `mod` y el Sistema de Archivos

Empezaremos nuestro ejemplo de módulo haciendo un nuevo proyecto con Cargo, pero en lugar
de crear un cajón binario, haremos un cajón de biblioteca: un proyecto que otras
personas pueden llevar a sus proyectos como una dependencia. Por ejemplo, la caja
`rand` discutida en el Capítulo 2 es una caja de biblioteca que usamos como dependencia 
en el proyecto del juego de adivinanzas.

Vamos a crear el esqueleto de una biblioteca que proporciona algo de funcionalidad 
a la red general; nos concentraremos en la organización de los módulos y 
funciones, pero no nos preocuparemos de qué código entra en los cuerpos funcionales. 
Llamaremos a nuestra biblioteca `communicator`. De forma predeterminada, Cargo creará una biblioteca a menos
que se especifique otro tipo de proyecto: si omitimos la opción `--bin` que hemos
estado usando en todos los capítulos anteriores, nuestro proyecto será una 
biblioteca:

```text
$ cargo new communicator
$ cd communicator
```

Nota que Cargo generó *src/lib.rs* en lugar de *src/main.rs*. Dentro de
*src/lib.rs* encontraremos lo siguiente:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Cargo crea un ejemplo de prueba para ayudarnos a poner en marcha nuestra biblioteca, en lugar 
de "¡Hola, mundo!" binario que conseguimos cuando usamos la opción `---bin`. Veremos
la sintaxis `#[]` y `mod tests` en la sección "Usando `super` para acceder a 
un módulo padre" más adelante en este capítulo, pero por ahora, deja este código en
la parte inferior de *src/lib.rs*.

Debido a que no tenemos un archivo *src/main.rs*, no hay nada para que Cargo 
ejecute con el comando `cargo run`. Por lo tanto, usaremos el comando `cargo build`
para compilar nuestro código del cajón de la biblioteca.

Examinaremos las diferentes opciones para organizar el código de tu biblioteca, que sean
adecuadas en una variedad de situaciones, dependiendo de la intención del código.

### Definiciones del Módulo

Para nuestra biblioteca de redes `communicator`, primero definiremos un módulo 
llamado `network` que contiene la definición de una función llamada `connect`. Cada 
definición de módulo en Rust comienza con la palabra clave `mod`. Añade este código al 
principio del archivo *src/lib.rs*, encima del código de prueba:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}
```

Después de la palabra clave `mod`, ponemos el nombre del módulo, `network`, y luego 
un bloque de código entre llaves. Todo dentro de este bloque está dentro del
espacio de nombres `network`. En este caso, tenemos una sola función, `connect`. Si 
quisiéramos llamar esta función desde un código fuera del módulo `network`, 
tendríamos que especificar el módulo y utilizar la sintaxis del espacio de nombres `::`, así:
`network::connect()` en lugar de simplemente `connect()`.

También podemos tener múltiples módulos, uno al lado del otro, en el mismo archivo *src/lib.rs*.
Por ejemplo, para tener también un módulo `client` que tenga una función llamada `connect`
también, podemos agregarlo como se muestra en eñ Listado 7-1:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}

mod client {
    fn connect() {
    }
}
```

<span class="caption">Listado 7-1: El módulo `network` y el módulo  `client`
definidos uno al lado del otro en *src/lib. rs*</span>

Ahora tenemos una función `network::connect` y una función `client::connect`.
Éstos pueden tener una funcionalidad completamente diferente, y los nombres de las funciones 
no entran en conflicto entre sí porque están en módulos diferentes.

En este caso, como estamos construyendo una biblioteca, el archivo que sirve como
punto de entrada para construir nuestra biblioteca es *src/lib.rs*. Sin embargo, con respecto a
la creación de módulos, no hay nada especial en *src/lib.rs*. También podríamos
crear módulos en *src/main.rs* para un cajón binario de la misma manera que 
creamos módulos en *src/lib.rs* para el cajón de la biblioteca. De hecho, podemos colocar 
módulos dentro de los módulos, lo que puede ser útil a medida que los módulos crecen para mantener
las funciones relacionadas organizadas y separadas entre sí. La 
elección de cómo organizar tu código depende de cómo pienses sobre la
relación entre las partes de tu código. Por ejemplo, el código `client` 
y su función `conect` podrían tener más sentido para los usuarios de nuestra biblioteca si 
estuvieran dentro del espacio de nombres `network`, como en el Listado 7-2:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}
```

<span class="caption">Listado 7-2: Moviendo el módulo `client` dentro del
módulo `network`</span>

En tu archivo *src/lib.rs*, reemplaza las definiciones existentes de `mod network` y `mod client`
con las definiciones del Listado 7-2, que tienen el módulo `client` como módulo 
interno de `network`. Ahora tenemos las funciones `network::connect` y 
`network::client::connect`: de nuevo, las dos funciones llamadas `connect` no
entran en conflicto entre sí porque están en diferentes espacios de nombres.

De este modo, los módulos forman una jerarquía. El contenido de *src/lib.rs* se encuentra en
el nivel superior, y los submódulos en los niveles inferiores. Aquí se muestra cómo se 
ve la organización de nuestro ejemplo en el Listado 7-1 cuando se piensa que es una
jerarquía:

```text
communicator
 ├── network
 └── client
```

Y aquí está la jerarquía correspondiente al ejemplo del Listado 7-2:

```text
communicator
 └── network
     └── client
```

La jerarquía muestra que en el Listado 7-2, `client` es un hijo del módulo 
`network` en lugar de un hermano. Los proyectos más complicados pueden tener muchos módulos,
y necesitarán organizarse lógicamente para poder hacer un seguimiento de ellos. Lo que
"lógicamente" significa que tu proyecto depende de ti y depende de cómo tu y los
usuarios de tu biblioteca piensen sobre el dominio de tu proyecto. Utiliza las técnicas mostradas
aquí para crear módulos lado a lado y módulos anidados en cualquier estructura
que desees.

### Moviendo Módulos a Otros Archivos

Los módulos forman una estructura jerárquica, muy parecida a otra estructura de computación
a la que estás acostumbrado: ¡los sistemas de archivos! Podemos usar el sistema de módulos de Rust junto con
múltiples archivos para dividir los proyectos de Rust para que no todos vivan en
*src/lib.rs* o *src/main.rs*. Para este ejemplo, comencemos con el código en
Listado 7-3:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod client {
    fn connect() {
    }
}

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Listado 7-3: Tres módulos,`client`,`network` y 
`network::server`, todos definidos en *src/lib.rs*.</span>

El fichero *src/lib.rs* tiene esta jerarquía de módulos:

```text
communicator
 ├── client
 └── network
     └── server
```

Si estos módulos tuvieran muchas funciones, y esas funciones se estuvieran haciendo largas,
sería difícil desplazarse por este archivo para encontrar el código con el que 
queríamos trabajar. Debido a que las funciones están anidadas dentro de uno o más bloques `mod`,
las líneas de código dentro de las funciones también empezarán a alargarse.
Estas serían buenas razones para separar los módulos `client`,`network` y `server`
de *src/lib.rs* y colocarlos en sus propios archivos.

En primer lugar, reemplaza el código de módulo `client` por sólo la declaración del 
módulo `client`, de modo que tu *src/lib.rs* se parezca al código mostrado en el Listado 7-4:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod client;

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Listado 7-4: Extracción del contenido del módulo `client` pero dejando la declaración en *src/lib.rs*</span>

Todavía estamos *declarando* el módulo `client` aquí, pero reemplazando el bloque
por un punto y coma, le estamos diciendo a Rust que busque en otra ubicación el código 
definido dentro del alcance del módulo `client`. En otras palabras, la línea `mod
cliente`; significa:

```rust,ignore
mod client {
    // contents of client.rs
}
```

Ahora tenemos que crear el archivo externo con ese nombre de módulo. Crea un
archivo *client.rs* en tu directorio *src/* y ábrelo. A continuación, introduce
lo siguiente, que es la función `connect` en el módulo `client` que
eliminamos en el paso anterior:

<span class="filename">Filename: src/client.rs</span>

```rust
fn connect() {
}
```

Ten en cuenta que no necesitamos una declaración `mod` en este archivo porque ya hemos
declarado el módulo `client` con `mod` en *src/lib.rs*. Este archivo sólo
proporciona los *contenidos* del módulo `client`. Si pusiéramos un `mod client` aquí, 
estaríamos dando al módulo de `client` su propio submódulo llamado `client`!

Rust por defecto sólo sabe mirar en *src/lib.rs*. Si queremos añadir más
archivos a nuestro proyecto, necesitamos decirle a Rust en *src/lib.rs* que busque en otros 
archivos; por eso es necesario definir `mod client` en *src/lib.rs* y no se puede
definir en *src/client.rs*.

Ahora el proyecto debería compilar con éxito, aunque obtendrás algunas
advertencias. Recuerde usar `cargo build` en lugar de `cargo run` porque tenemos
un cajón de biblioteca en lugar de un cajon binario:

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
warning: function is never used: `connect`
 --> src/client.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/lib.rs:4:5
  |
4 | /     fn connect() {
5 | |     }
  | |_____^

warning: function is never used: `connect`
 --> src/lib.rs:8:9
  |
8 | /         fn connect() {
9 | |         }
  | |_________^
```

Estas advertencias nos dicen que tenemos funciones que nunca se utilizan. No te preocupes
por estas advertencias por ahora; las trataremos más adelante en este capítulo en la
sección "Controlar la visibilidad con `pub`". La buena noticia es que son sólo 
advertencias, ¡nuestro proyecto se construyó con éxito!

A continuación, extraigamos el módulo `network` en su propio archivo usando el mismo
patrón. En *src/lib.rs*, borra el cuerpo del módulo `network` y añade un
punto y coma a la declaración, así:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod client;

mod network;
```

A continuación, crea un nuevo archivo *src/network.rs* e introduce lo siguiente:

<span class="filename">Filename: src/network.rs</span>

```rust
fn connect() {
}

mod server {
    fn connect() {
    }
}
```

Nota que todavía tenemos una declaración `mod` dentro de este archivo de módulo; esto es
porque todavía queremos que el `server` sea un submódulo de `network`.

Ejecuta `cargo build` de nuevo. Éxito! Tenemos un módulo más para extraer: `server`.
Porque es un submódulo, es decir,  un módulo dentro de un módulo, nuestra táctica
actual de extraer un módulo en un archivo nombrado después de ese módulo no funcionará. Lo
intentaremos de todos modos para que puedas ver el error. Primero, cambia *src/network.rs* para tener 
`mod server;` en lugar del contenido del módulo `server`:

<span class="filename">Filename: src/network.rs</span>

```rust,ignore
fn connect() {
}

mod server;
```

Luego crea un archivo *src/server.rs* e introduce el contenido del módulo `server` 
que hemos extraído:

<span class="filename">Filename: src/server.rs</span>

```rust
fn connect() {
}
```

Cuando intentemos `cargo build`, tendremos el error mostrado en Listing 7-5:

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
error: cannot declare a new module at this location
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
  |
note: maybe move this module `src/network.rs` to its own directory via `src/network/mod.rs`
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
note: ... or maybe `use` the module `server` instead of possibly redeclaring it
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
```

<span class="caption">Listado 7-5: Error al intentar extraer el submódulo `server`
en *src/server.rs</span>

El error dice que `cannot declare a new module at this location` y está
apuntando a `mod server; `linea en *src/network.rs*. Así que *src/network.rs* es
diferente a *src/lib.rs* de alguna manera: sigue leyendo para entender por qué.

La nota en medio del Listado 7-5 es realmente muy útil porque 
señala algo por hacer de lo que que aún no hemos hablado:

```text
note: maybe move this module `network` to its own directory via
`network/mod.rs`
```

En lugar de seguir el mismo patrón de nombres de archivo que usamos 
anteriormente, podemos hacer lo que la nota sugiere:

1. Haz un nuevo *directory* llamado *network*, el nombre del módulo padre.
2. Mueve el archivo *src/network.rs* en el nuevo directorio *network* y 
   renómbralo a *src/network/mod.rs*.
3. Mueve el archivo del submódulo *src/server.rs* en el directorio *network*.

Aquí están los comandos para llevar a cabo estos pasos:

```text
$ mkdir src/network
$ mv src/network.rs src/network/mod.rs
$ mv src/server.rs src/network
```

Ahora, cuando intentemos ejecutar `cargo build`, la compilación funcionará (aunque seguiremos 
teniendo advertencias). Nuestro diseño de módulo todavía se ve así, que es exactamente
igual que cuando teníamos todo el código en *src/lib.rs* en Listado 7-3:

```text
communicator
 ├── client
 └── network
     └── server
```

El diseño de archivo correspondiente ahora se ve así:

```text
├── src
│   ├── client.rs
│   ├── lib.rs
│   └── network
│       ├── mod.rs
│       └── server.rs
```

Así que cuando quisimos extraer el módulo `network::server`, ¿por qué tuvimos que 
cambiar también el archivo *src/network.rs* al archivo *src/network/mod.rs* y poner el
código para `network::server` en el directorio *network* en 
*src/network/server.rs* en lugar de simplemente poder extraer el
módulo `network::server` en *src/server.rs*? La razón es que Rust no sería 
capaz de reconocer que el `server` debía ser un submódulo de `network`
si el archivo *server.rs* estuviera en el directorio *src*. Para aclarar el comportamiento de Rust 
aquí, vamos a considerar un ejemplo diferente con la siguiente jerarquía de módulos, 
donde todas las definiciones están en *src/lib.rs*:

```text
communicator
 ├── client
 └── network
     └── client
```

En este ejemplo, tenemos de nuevo tres módulos: `client`, `network` y 
`network::client`. Siguiendo los mismos pasos que hicimos anteriormente para extraer 
módulos en archivos, crearíamos *src/client.rs* para el módulo `client`.
Para el módulo `network`, crearíamos *src/network.rs*. Pero no podríamos
extraer el módulo `network::client` en un archivo *src/client.rs*
porque ya existe para el módulo `client` de alto nivel! Si pudiéramos poner
el código para *ambos* módulos `client` y `network::client` en el
archivo *src/client.rs*, Rust no tendría forma de saber si el código era 
para `client` o para `network::client`.

Por lo tanto, para extraer un archivo para el submódulo `networl::client` del
módulo `network`, necesitábamos crear un directorio para el módulo `network`
en lugar de un archivo *src/network.rs*. El código que se encuentra en el módulo `network`
entra en el archivo *src/network/mod.rs*, y el submódulo 
`network::client` puede tener su propio archivo *src/network/client.rs*. Ahora el
nivel superior *src/client.rs* es inequívocamente el código que pertenece al
módulo `client`.

### Reglas de los Sistemas de Ficheros del Módulo

Resumamos las reglas de los módulos con respecto a los archivos:

* Si un módulo llamado `foo` no tiene submódulos, debes poner las declaraciones
para `foo` en un archivo llamado *foo.rs*.
* Si un módulo llamado `foo` tiene submódulos, debes poner las declaraciones
para `foo` en un archivo llamado *foo/mod.rs*.

Estas reglas se aplican recursivamente, así que si un módulo llamado `foo` tiene un submódulo llamado
`bar` y `bar` no tiene submódulos, deberías tener los siguientes archivos
en tu directorio *src*:

```text
├── foo
│   ├── bar.rs (contains the declarations in `foo::bar`)
│   └── mod.rs (contains the declarations in `foo`, including `mod bar`)
```

Los módulos deben declararse en el archivo del módulo padre utilizando la palabra 
clave `mod`.

A continuación, hablaremos de la palabra clave `pub` y nos desharemos de esas advertencias!
