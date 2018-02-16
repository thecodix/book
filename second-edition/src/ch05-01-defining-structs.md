## Definición e Instanciación de Estructuras

Las estructuras son similares a las tuplas, que fueron discutidas en el Capítulo 3. Al igual que las tuplas,
las piezas de una estructura pueden ser de diferentes tipos. A diferencia de las tuplas, nombramos cada
dato para que quede claro el significado de los valores. Como resultado de estos nombres, 
las estructuras son más flexibles que las tuplas: no tenemos que depender del orden de
los datos para especificar o acceder a los valores de una instancia.

Para definir una estructura, introducimos la palabra clave `struct` y nombramos toda la estructura. El
nombre de una estructura debe describir el significado de las piezas de datos que se están 
agrupando. Luego, dentro de las llaves, definimos los nombres y tipos de las piezas de
datos, que llamamos *campos*. Por ejemplo, el Listado 5-1 muestra una
estructura para almacenar información sobre una cuenta de usuario:

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

<span class="caption">Listado 5-1: Una definición de estructura de `User`</span>

Para utilizar una estructura después de haberla definido, creamos una *instancia* de esa estructura 
especificando valores concretos para cada uno de los campos. Creamos una instancia 
indicando el nombre de la estructura, y luego añadimos llaves que contienen pares 
`key: value` donde las teclas son los nombres de los campos y los valores son los
datos que queremos almacenar en esos campos. No es necesario especificar los campos en 
el mismo orden en el que los declaramos en la estructura. En otras palabras, la definición 
de estructura es como una plantilla general para el tipo, y las instancias completan 
esa plantilla con datos particulares para crear valores del tipo. Por
ejemplo, podemos declarar un usuario particular como se muestra en la lista 5-2:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
``` 

<span class="caption">Listado 5-2: Creación de una instancia de la estructura
`User`</span>

Para obtener un valor específico de una estructura, podemos usar la notación de puntos. Si sólo queríamos
la dirección de correo electrónico de este usuario, podemos usar `user1. email` donde queramos
usar este valor. Para cambiar un valor en una estructura, si la instancia es mutable, podemos
usar la notación de puntos y asignarla a un campo en particular. El listado 5-3 muestra
cómo cambiar el valor en el campo `email` de una instancia de `User` mutable:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");
``` 

<span class="caption">Listado 5-3: Cambiar el valor del campo `email` de una instancia
de `User`</span>

### Abreviatura de Campo Inicial cuando las Variables Tienen el Mismo Nombre que los Campos

Si tienes variables con los mismos nombres que los campos de estructura, puedes utilizar el *campo en 
forma abreviatura*. Esto puede hacer que las funciones que crean nuevas instancias de estructuras
sean más concisas. La función llamada `build_user` mostrada aquí en la lista 5-4 tiene 
parámetros llamados `email` y `username`. La función crea y devuelve una instancia
de `User`:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

 <span class="caption">Listado 5-4:  Una función `build_user` que toma un correo electrónico
 y un nombre de usuario y devuelve una instancia de `User`.</span>

Debido a que los nombres de parámetro `email` y `username` son los mismos que los nombres 
de campo de la estructura `User`, `email` y `username`, podemos escribir `build_user` sin 
la repetición de `email` y `username` como se muestra en el Listado 5-5. Esta versión
de `build_user` se comporta de la misma manera que la de Listing 5-4. La sintaxis del
campo puede hacer que casos como este sean más cortos de escribir, especialmente cuando las estructuras
tienen muchos campos.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

<span class="caption">Listado 5-5: Una función `build_user` que utiliza la sintaxis del campo 
ya que los parámetros `email` y `username` tienen el mismo nombre que los campos
de estructura</span>

### Creación de Instancias Desde otras Instancias con la Sintaxis de Actualización de Estructura

A menudo es útil crear una nueva instancia a partir de una instancia antigua, usando la mayoría
de los valores de la instancia antigua pero cambiando algunos. El Listado 5-6 muestra un ejemplo de
creación de una nueva instancia de `User` en `user2` al establecer los valores de `email` y
`usuario` pero usando los mismos valores para el resto de los campos de la instancia 
`user1` que creamos en el Listado 5-2:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("someone@example.com"),
#     username: String::from("someusername123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    active: user1.active,
    sign_in_count: user1.sign_in_count,
};
``` 

<span class="caption">Listado 5-6: Creando una nueva instancia de `User`, `user2`, y
poniendo algunos campos a los valores de los mismos campos de `user1`</span>

La *struct update syntax* logra el mismo efecto que el código en Listing 5-6 
usando menos código. La sintaxis de actualización de estructura utiliza `... ` para especificar que 
los campos restantes no configurados explícitamente deben tener el mismo valor que los campos de la
instancia en cuestión. El código en Listado 5-7 también crea una instancia en `user2`
que tiene un valor diferente para los campos `email` y `username` pero tiene los mismos valores
para los campos `active` y `sign_in_count` que `user1`:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("someone@example.com"),
#     username: String::from("someusername123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```

<span class="caption">Listado 5-7: Usando la sintaxis de actualización de estructura para establecer un nuevo valor de
`email` y `username` para una instancia `User` pero usa el resto de los
valores de los campos de la instancia en la variable `user1`.</span>

### Estructuras Tupla sin campos especificados para crear tipos diferentes

También podemos definir estructuras que parecen similares a tuplas, llamadas *estructuras tupla*,
que tienen el significado añadido que proporciona el nombre de la estructura, pero no tienen nombres
asociados con sus campos, sólo los tipos de campos. La definición de una
estructura tupla todavía comienza con la palabra clave `struct` y el nombre de la estructura, que son
seguidos por los tipos en la tupla. Por ejemplo, aquí están las definiciones y 
usos de las estructuras dobles llamadas `Color` y `Point`:

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

Ten en cuenta que los valores `black` y `origin` son diferentes tipos, ya que son 
instancias de diferentes estructuras tuplas. Cada estructura que definimos es su propio tipo,
aunque los campos dentro de la estructura tienen los mismos tipos. De lo contrario, 
las instancias de estructura tupla se comportan como tuplas, que hemos tratado en el Capítulo 3.

### Unit-Like Structs without Any Fields

We can also define structs that don't have any fields! These are called
*unit-like structs* since they behave similarly to `()`, the unit type.
Unit-like structs can be useful in situations such as when you need to
implement a trait on some type, but you don't have any data that you want to
store in the type itself. We'll be discussing traits in Chapter 10.

> ### Ownership of Struct Data
>
> In the `User` struct definition in Listing 5-1, we used the owned `String`
> type rather than the `&str` string slice type. This is a deliberate choice
> because we want instances of this struct to own all of its data and for that
> data to be valid for as long as the entire struct is valid.
>
> It’s possible for structs to store references to data owned by something else,
> but to do so requires the use of *lifetimes*, a Rust feature that is discussed
> in Chapter 10. Lifetimes ensure that the data referenced by a struct is valid
> for as long as the struct is. Let’s say you try to store a reference in a
> struct without specifying lifetimes, like this:
>
> <span class="filename">Filename: src/main.rs</span>
>
> ```rust,ignore
> struct User {
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
>     active: bool,
> }
>
> fn main() {
>     let user1 = User {
>         email: "someone@example.com",
>         username: "someusername123",
>         active: true,
>         sign_in_count: 1,
>     };
> }
> ```
>
> The compiler will complain that it needs lifetime specifiers:
>
> ```text
> error[E0106]: missing lifetime specifier
>  -->
>   |
> 2 |     username: &str,
>   |               ^ expected lifetime parameter
>
> error[E0106]: missing lifetime specifier
>  -->
>   |
> 3 |     email: &str,
>   |            ^ expected lifetime parameter
> ```
>
> We’ll discuss how to fix these errors so you can store references in structs
> in Chapter 10, but for now, we’ll fix errors like these using owned types like
> `String` instead of references like `&str`.
