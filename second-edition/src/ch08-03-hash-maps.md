## Hash Maps Store claves asociadas con valores

La última de nuestras colecciones comunes es el * hash map *. El tipo `HashMap <K, V>`
almacena una asignación de claves de tipo `K` a valores de tipo` V`. Lo hace a través de 
* hashing function *, que determina cómo coloca estas claves y valores en la
memoria. Muchos lenguajes de programación diferentes admiten este tipo de datos
estructura, pero a menudo usan un nombre diferente, como hash, mapa, objeto, mesa
hash, o matriz asociativa, solo por mencionar algunos

Los mapas hash son útiles para cuando quiere buscar datos no por un índice, ya que
puede con vectores, pero usando una clave que puede ser de cualquier tipo. Por ejemplo, en un
juego, puedes hacer un seguimiento del puntaje de cada equipo en un mapa hash donde cada clave es
el nombre de un equipo y los valores son el puntaje de cada equipo. Dado el nombre de un equipo, puedes
recuperar su puntaje

Repasaremos la API básica de los mapas hash en esta sección, pero muchos más detalles
se esconden en las funciones definidas en `HashMap <K, V>` por la biblioteca estándar.
Como siempre, consulte la documentación estándar de la biblioteca para obtener más información.

### Crear un nuevo hash map

Podemos crear un mapa hash vacío con `new` y agregar elementos con` insert`. En
Listado 8-20, estamos haciendo un seguimiento de los puntajes de dos equipos cuyos nombres son
Azul y amarillo. El equipo azul comenzará con 10 puntos, y el equipo amarillo
comienza con 50:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

<span class="caption">Listado 8-20: Creando un nuevo mapa hash e insertando algunos
llaves y valores</span>

Tenga en cuenta que necesitamos primero `use` el` HashMap` de la porción de colecciones de
la biblioteca estándar. De nuestras tres colecciones comunes, esta es la menos siendo
de uso frecuente, por lo que no está incluido en las características incluidas en el alcance
automático del preludio. Los mapas Hash también tienen menos soporte de la
biblioteca estándar; no hay una macro incorporada para construirlos, por ejemplo.

Al igual que los vectores, los mapas hash almacenan sus datos en el montón. Este `HashMap` tiene
teclas de tipo `String` y valores de tipo` i32`. Como vectores, los mapas hash son
homogéneo: todas las claves deben tener el mismo tipo y todos los valores
debe tener el mismo tipo.

Otra forma de construir un mapa hash es usar el método `collect` en un
vector de tuplas, donde cada tupla consiste en una clave y su valor. los
métodos `collect` reúnen datos en una cantidad de tipos de colección, incluyendo
`HashMap`. Por ejemplo, si tuviéramos los nombres de los equipos y puntajes iniciales en dos
vectores separados, podemos usar el método `zip` para crear un vector de tuplas
donde "Azul" está emparejado con 10, y así sucesivamente. Entonces podemos usar el método `collect`
para convertir ese vector de tuplas en un `HashMap` como se muestra en el Listado 8-21:

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

<span class="caption">Listado 8-21: Crear un mapa hash a partir de una lista de equipos
y una lista de puntajes</span>

La anotación tipo `HashMap <_, _>` es necesaria aquí porque es posible
`recoger` en muchas estructuras de datos diferentes, y Rust no sabe cuál 
escoger a menos que especifiques. Para los parámetros de tipo para los tipos de clave y valor,
sin embargo, usamos caracteres de subrayado, y Rust puede inferir los tipos que el mapa de hash
contiene en base a los tipos de datos en los vectores.

### hash map y Propiedad

Para los tipos que implementan el rasgo `Copiar`, como` i32`, los valores se copian
en el mapa hash. Para valores propios como `String`, los valores se moverán y
el mapa hash será el propietario de esos valores como se muestra en el Listado 8-22:

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are invalid at this point, try using them and
// see what compiler error you get!
```

<span class="caption">Listado 8-22: Mostrar que las claves y los valores son propiedad del
mapa hash una vez que se insertan</span>

No podemos usar las variables `field_name` y` field_value` después de que
se han movido al mapa hash con la llamada a `insert`.

Si insertamos referencias a valores en el mapa hash, los valores no se moverán
en el mapa hash. Los valores a los que apuntan las referencias deben ser válidos al
menos mientras el mapa hash sea válido. Hablaremos más sobre estos temas en
la sección "Validación de referencias con tiempos de vida" en el Capítulo 10.

### Acceso a valores en un hash map

Podemos obtener un valor del mapa hash proporcionando su clave para el método `get`
como se muestra en el Listado 8-23:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

<span class="caption">Listado 8-23: Accediendo al puntaje del equipo azul
almacenado en el mapa hash</span>

Aquí, `score` tendrá el valor que está asociado con el equipo azul, y el
resultado será `Some(&10)`. El resultado está envuelto en `Some` porque `get`
devuelve a `Option<&V>`; si no hay ningún valor para esa clave en el mapa hash,
`get` regresará a `None`. El programa necesitará manejar la `Option` en una de 
las maneras que cubrimos en el capítulo 6.

Podemos iterar sobre cada par clave / valor en un mapa hash de manera similar a como lo hacemos
hacer con vectores, usando un bucle `for`:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

Este código imprimirá cada par en un orden arbitrario:

```text
Yellow: 50
Blue: 10
```

### Actualización de un hash map

Aunque la cantidad de claves y valores puede crecer, cada clave solo puede tener un
valor asociado con él a la vez. Cuando queremos cambiar los datos en un mapa
hash, tenemos que decidir cómo manejar el caso cuando una clave ya tiene un valor
asignado. Podríamos reemplazar el valor anterior por el nuevo valor, completamente
sin tener en cuenta el valor anterior. Podríamos mantener el valor anterior e ignorar el nuevo
valor, y solo agregue el nuevo valor si la clave * no * ya tiene un valor. O
podríamos combinar el valor anterior y el nuevo valor. ¡Veamos cómo hacer cada
de estos!

#### Sobrescribir un valor

Si insertamos una clave y un valor en un mapa hash, y luego insertamos esa misma clave
con un valor diferente, el valor asociado con esa clave será reemplazado.
Aunque el código en el Listado 8-24 llama `insert` dos veces, el hash map
solo contiene un par de key/value porque estamos insertando el valor para el equipo azul
en ambas ocasiones:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

<span class="caption">Listing 8-24: Replacing a value stored with a particular
key</span>

This code will print `{"Blue": 25}`. The original value of `10` has been
overwritten.

#### Only Insert If the Key Has No Value

It’s common to check whether a particular key has a value, and if it doesn’t,
insert a value for it. Hash maps have a special API for this called `entry`
that takes the key we want to check as a parameter. The return value of the
`entry` function is an enum called `Entry` that represents a value that might
or might not exist. Let’s say we want to check whether the key for the Yellow
team has a value associated with it. If it doesn’t, we want to insert the value
50, and the same for the Blue team. Using the `entry` API, the code looks like
Listing 8-25:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

<span class="caption">Listing 8-25: Using the `entry` method to only insert if
the key does not already have a value</span>

The `or_insert` method on `Entry` is defined to return the value for the
corresponding `Entry` key if that key exists, and if not, inserts the parameter
as the new value for this key and returns the modified `Entry`. This technique
is much cleaner than writing the logic ourselves, and in addition, plays more
nicely with the borrow checker.

Running the code in Listing 8-25 will print `{"Yellow": 50, "Blue": 10}`. The
first call to `entry` will insert the key for the Yellow team with the value
`50` because the Yellow team doesn’t have a value already. The second call to
`entry` will not change the hash map because the Blue team already has the
value `10`.

#### Updating a Value Based on the Old Value

Another common use case for hash maps is to look up a key’s value and then
update it based on the old value. For instance, Listing 8-26 shows code that
counts how many times each word appears in some text. We use a hash map with
the words as keys and increment the value to keep track of how many times we’ve
seen that word. If it’s the first time we’ve seen a word, we’ll first insert
the value `0`:

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

<span class="caption">Listing 8-26: Counting occurrences of words using a hash
map that stores words and counts</span>

This code will print `{"world": 2, "hello": 1, "wonderful": 1}`. The
`or_insert` method actually returns a mutable reference (`&mut V`) to the value
for this key. Here we store that mutable reference in the `count` variable, so
in order to assign to that value we must first dereference `count` using the
asterisk (`*`). The mutable reference goes out of scope at the end of the `for`
loop, so all of these changes are safe and allowed by the borrowing rules.

### Hashing Function

By default, `HashMap` uses a cryptographically secure hashing function that can
provide resistance to Denial of Service (DoS) attacks. This is not the fastest
hashing algorithm available, but the trade-off for better security that comes
with the drop in performance is worth it. If you profile your code and find
that the default hash function is too slow for your purposes, you can switch to
another function by specifying a different *hasher*. A hasher is a type that
implements the `BuildHasher` trait. We’ll talk about traits and how to
implement them in Chapter 10. You don’t necessarily have to implement your own
hasher from scratch; [crates.io](https://crates.io) has libraries shared by
other Rust users that provide hashers implementing many common hashing
algorithms.

## Summary

Vectors, strings, and hash maps will provide a large amount of functionality
that you need in programs where you need to store, access, and modify data.
Here are some exercises you should now be equipped to solve:

* Given a list of integers, use a vector and return the mean (average), median
  (when sorted, the value in the middle position), and mode (the value that
  occurs most often; a hash map will be helpful here) of the list.
* Convert strings to pig latin. The first consonant of each word is moved to
  the end of the word and “ay” is added, so “first” becomes “irst-fay.” Words
  that start with a vowel have “hay” added to the end instead (“apple” becomes
  “apple-hay”). Keep in mind the details about UTF-8 encoding!
* Using a hash map and vectors, create a text interface to allow a user to add
  employee names to a department in a company. For example, “Add Sally to
  Engineering” or “Add Amir to Sales.” Then let the user retrieve a list of all
  people in a department or all people in the company by department, sorted
  alphabetically.

The standard library API documentation describes methods that vectors, strings,
and hash maps have that will be helpful for these exercises!

We’re getting into more complex programs in which operations can fail; so, it’s
a perfect time to discuss error handling next!
