## Referencia a Nombres en Módulos Diferentes

Hemos explicado cómo llamar a las funciones definidas dentro de un módulo
utilizando el nombre del módulo como parte de la llamada, como en la llamada a la 
función `nested_modules` mostrada aquí en el Listado 7-7:

<span class="filename">Nombre de archivo: src/main.rs</span>

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

fn main() {
    a::series::of::nested_modules();
}
```

<span class="caption">Listado 7-7: Llamada a una función especificando completamente
la trayectoria de su módulo adjunto</span>

Como puedes ver, al referirse a un nombre totalmente calificado puede llegar a ser bastante largo.
Afortunadamente, Rust tiene una palabra clave para hacer estas llamadas más concisas.

### Cómo llevar los Nombres al Alcance de Aplicación con la Palabra Clave `use`

La palabra clave `use` de Rust acorta las largas llamadas de función al traer los módulos de
la función que deseas llamar al alcance. Aquí un ejemplo de cómo llevar el módulo 
`a::series::of` al alcance de la raíz de un crate binario:

<span class="filename">Nombre de archivo: src/main.rs</span>

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of;

fn main() {
    of::nested_modules();
}
```

La línea `use a::series::of; ` significa que en lugar de usar la ruta completa
`a::series::of` donde queramos referirnos al módulo `of`, podemos usar
`of`.

La palabra clave `use` sólo aporta lo que hemos especificado en el alcance: no 
lleva a los módulos hijo a scope. Por eso todavía tenemos que usar 
`of::nested_modules` cuando queremos llamar a la función `nested_modules`.

Podríamos haber optado por incluir la función en scope en lugar de especificar 
la función en `use` como se indica a continuación:

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of::nested_modules;

fn main() {
    nested_modules();
}
```

Esto nos permite excluir todos los módulos y referirnos directamente a la 
función.

Debido a que los enums también forman una especie de espacio de nombres como módulos, también podemos
poner las variantes de una lista en scope con `use`. Para cualquier tipo de declaración `use`,
si estás trayendo varios elementos de un espacio de nombres a scope, puedes listarlos
usando llaves y comas en la última posición, así:

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::{Red, Yellow};

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = TrafficLight::Green;
}
```

Todavía estamos especificando el espacio de nombres `TrafficLight` para la variante `Green`
porque no incluimos `Green` en la declaración de `use`.

### Llevando Todos los Nombres al Scope con Glob

Para poner todos los ítems en un espacio de nombres y a la vez en scope, podemos usar la sintaxis `*`, que se llama *el operador glob*. Este ejemplo pone todas las variantes de una enumeración en el scope sin tener que enumerar cada una específicamente:

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::*;

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = Green;
}
```

El `*` llevará a scope todos los elementos visibles en el espacio de nombres 
`TrafficLight`. Tu debes usar los globs con moderación: son convenientes, pero esto también
podría atraer más elementos de lo que esperabas y causar conflictos de nombres.

### Usando `super` para Acceder a un Módulo Padre

Como vimos al principio de este capítulo, cuando creas un crate de biblioteca,
Cargo hace un módulo de `tests` para ti. Vamos a entrar en más detalles sobre eso ahora. 
En tu proyecto `communicator`, abre *src/lib.rs*:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;

#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

El capítulo 11 explica más sobre las pruebas, pero partes de este ejemplo deben tener
sentido ahora: tenemos un módulo llamado `tests` que vive al lado de nuestros otros módulos
y contiene una función llamada `it_works`. A pesar de que hay anotaciones especiales,
el módulo `tests` es sólo otro módulo más! Así que nuestra jerarquía de módulos
se ve así:

```text
communicator
 ├── client
 ├── network
 |   └── client
 └── tests
```

Las pruebas son para ejercitar el código dentro de nuestra biblioteca, así que tratemos de llamar
a nuestra función `client::connect` desde esta función `it_works`, aunque ahora mismo no vamos
a comprobar ninguna funcionalidad. Esto no funcionará todavía:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        client::connect();
    }
}
```

Ejecuta las pruebas invocando el comando `cargo test`:

```text
$ cargo test
   Compiling communicator v0.1.0 (file:///projects/communicator)
error[E0433]: failed to resolve. Use of undeclared type or module `client`
 --> src/lib.rs:9:9
  |
9 |         client::connect();
  |         ^^^^^^ Use of undeclared type or module `client`
```

La compilación falló, pero ¿por qué? No necesitamos colocar `communicator::` 
delante de la función como hicimos en *src/main.rs* porque aquí estamos definitivamente
dentro del crate de la biblioteca de `communicator`. La razón es que las rutas son 
siempre relativas al módulo actual, que aquí es `tests`. La única 
excepción es en una declaración `use`, donde las rutas son por defecto relativas a la 
raíz del crate. Nuestro módulo `tests` necesita el módulo `client` en su scope!

Entonces, ¿cómo se puede recuperar un módulo de la jerarquía de módulos para llamar la 
función `client::connect` en el módulo `tests`? En el módulo `tests`, podemos usar
los dos puntos principales para hacerle saber a Rust que queremos empezar desde la raíz
y listar toda la ruta, así:

```rust,ignore
::client::connect();
```

O, podemos usar `super` para subir un módulo en la jerarquía de nuestro 
módulo actual, así:

```rust,ignore
super::client::connect();
```

Estas dos opciones no parecen tan diferentes en este ejemplo, pero si eres
más profundo en la jerarquía de un módulo, empezando desde la raíz cada vez, hará que
tu código sea más largo. En esos casos, usar `super` para pasar del módulo actual a 
los módulos hermanos es un buen atajo. Además, si has especificado la ruta desde la 
raíz en muchos lugares de tu código y luego has reordenado los módulos moviendo un
subárbol a otro lugar, tendrías que actualizar la ruta en varios lugares, 
lo cual sería tedioso.

También sería molesto tener que escribir `super::` en cada prueba, pero ya has
visto la herramienta para esa solución: `use`! La funcionalidad `super::`
cambia la ruta que le das a `use` para que sea relativa al módulo
padre en lugar del módulo raíz.

Por estas razones, en el módulo de `pruebas` especialmente, `use super::something` es
generalmente la mejor solución. Así que ahora nuestra prueba se ve así:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    use super::client;

    #[test]
    fn it_works() {
        client::connect();
    }
}
```

Cuando volvamos a ejecutar `cargo test`, la prueba pasará y la primera parte de la
salida del resultado de la prueba será la siguiente:

```text
$ cargo test
   Compiling communicator v0.1.0 (file:///projects/communicator)
     Running target/debug/communicator-92007ddb5330fa5a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

## Resumen

Ahora conoces algunas nuevas técnicas para organizar tu código! Utilizae estas técnicas
para agrupar las funciones relacionadas, evitar que los archivos se alarguen demasiado y 
presentar una API pública ordenada a los usuarios de la biblioteca.

A continuación, veremos algunas estructuras de recopilación de datos en la biblioteca estándar
que puedes utilizar en su genial y ordenado código!

