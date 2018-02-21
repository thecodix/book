## Hash Maps Store Keys Associated with Values

The last of our common collections is the *hash map*. The type `HashMap<K, V>`
stores a mapping of keys of type `K` to values of type `V`. It does this via a
*hashing function*, which determines how it places these keys and values into
memory. Many different programming languages support this kind of data
structure, but often use a different name, such as hash, map, object, hash
table, or associative array, just to name a few.

Hash maps are useful for when you want to look up data not by an index, as you
can with vectors, but by using a key that can be of any type. For example, in a
game, you could keep track of each team’s score in a hash map where each key is
a team’s name and the values are each team’s score. Given a team name, you can
retrieve its score.

We’ll go over the basic API of hash maps in this section, but many more goodies
are hiding in the functions defined on `HashMap<K, V>` by the standard library.
As always, check the standard library documentation for more information.

### Creating a New Hash Map

We can create an empty hash map with `new` and add elements with `insert`. In
Listing 8-20, we’re keeping track of the scores of two teams whose names are
Blue and Yellow. The Blue team will start with 10 points, and the Yellow team
starts with 50:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

<span class="caption">Listing 8-20: Creating a new hash map and inserting some
keys and values</span>

Note that we need to first `use` the `HashMap` from the collections portion of
the standard library. Of our three common collections, this one is the least
often used, so it’s not included in the features brought into scope
automatically in the prelude. Hash maps also have less support from the
standard library; there’s no built-in macro to construct them, for example.

Just like vectors, hash maps store their data on the heap. This `HashMap` has
keys of type `String` and values of type `i32`. Like vectors, hash maps are
homogeneous: all of the keys must have the same type, and all of the values
must have the same type.

Another way of constructing a hash map is by using the `collect` method on a
vector of tuples, where each tuple consists of a key and its value. The
`collect` method gathers data into a number of collection types, including
`HashMap`. For example, if we had the team names and initial scores in two
separate vectors, we can use the `zip` method to create a vector of tuples
where “Blue” is paired with 10, and so forth. Then we can use the `collect`
method to turn that vector of tuples into a `HashMap` as shown in Listing 8-21:

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

<span class="caption">Listing 8-21: Creating a hash map from a list of teams
and a list of scores</span>

The type annotation `HashMap<_, _>` is needed here because it’s possible to
`collect` into many different data structures, and Rust doesn’t know which you
want unless you specify. For the type parameters for the key and value types,
however, we use underscores, and Rust can infer the types that the hash map
contains based on the types of the data in the vectors.

### Hash Maps and Ownership

For types that implement the `Copy` trait, like `i32`, the values are copied
into the hash map. For owned values like `String`, the values will be moved and
the hash map will be the owner of those values as demonstrated in Listing 8-22:

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are invalid at this point, try using them and
// see what compiler error you get!
```

<span class="caption">Listing 8-22: Showing that keys and values are owned by
the hash map once they’re inserted</span>

We aren’t able to use the variables `field_name` and `field_value` after
they’ve been moved into the hash map with the call to `insert`.

If we insert references to values into the hash map, the values won’t be moved
into the hash map. The values that the references point to must be valid for at
least as long as the hash map is valid. We’ll talk more about these issues in
the “Validating References with Lifetimes” section in Chapter 10.

### Accessing Values in a Hash Map

We can get a value out of the hash map by providing its key to the `get` method
as shown in Listing 8-23:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

<span class="caption">Listing 8-23: Accessing the score for the Blue team
stored in the hash map</span>

Here, `score` will have the value that’s associated with the Blue team, and the
result will be `Some(&10)`. The result is wrapped in `Some` because `get`
returns an `Option<&V>`; if there’s no value for that key in the hash map,
`get` will return `None`. The program will need to handle the `Option` in one
of the ways that we covered in Chapter 6.

We can iterate over each key/value pair in a hash map in a similar manner as we
do with vectors, using a `for` loop:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

This code will print each pair in an arbitrary order:

```text
Yellow: 50
Blue: 10
```

### Updating a Hash Map

Although the number of keys and values is growable, each key can only have one
value associated with it at a time. When we want to change the data in a hash
map, we have to decide how to handle the case when a key already has a value
assigned. We could replace the old value with the new value, completely
disregarding the old value. We could keep the old value and ignore the new
value, and only add the new value if the key *doesn’t* already have a value. Or
we could combine the old value and the new value. Let’s look at how to do each
of these!

#### Overwriting a Value

If we insert a key and a value into a hash map, and then insert that same key
with a different value, the value associated with that key will be replaced.
Even though the code in Listing 8-24 calls `insert` twice, the hash map will
only contain one key/value pair because we’re inserting the value for the Blue
team’s key both times:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

<span class="caption">Listado 8-24: Reemplazar un valor almacenado con una clave
particular</span>

Este código pondrá `{"Blue": 25}`. El valor original de `10` ha sido
reescrito.

#### Solo inserte si la clave no tiene valor

Es común comprobar si una clave en particular tiene un valor, y si no lo hace,
inserta un valor para eso. Los mapas Hash tienen una API especial para esto llamado`entry`
eso toma la clave que queremos verificar como parámetro. El valor de retorno del
`entry` función es una enumeración llamado `Entry`eso representa un valor que podría
o podría no existir. Digamos que queremos comprobar si la clave para el equipo
amarillo tiene un valor asociado con eso. Si no es así, queremos insertar el valor
50, y lo mismo para el equipo azul. Utilizando el `entry` API, el código se ve como
Listado 8-25:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

<span class="caption">Listado 8-25: Usando el método `entry` para insertar solo si
la clave aún no tiene un valor</span>

EL `or_insert` método en `Entry` se define para devolver el valor para la
correspondiente llave `Entry` si esa clave existe, y si no, inserta el parámetro
como el nuevo valor de esta clave y devuelve la modificación `Entry`. Esta tecnica
es mucho más limpia que escribir la lógica nosotros mismos, y además, hace mejor
el trabajo con el prestamista.

Se imprimirá el código en el Listado 8-25. `{"Yellow": 50, "Blue": 10}`. La
primera llamada al `entry` insertará la clave para el equipo amarillo con el valor
`50`porque el equipo amarillo ya no tiene un valor. La segunda llamada a
`entry` no cambiará el mapa hash porque el equipo azul ya tiene el
valor `10`.

#### Actualización de un valor basado en el valor anterior

Otro caso de uso común para los mapas hash es buscar el valor de una clave y luego
actualizarlo en base al valor anterior. Por ejemplo, el Listado 8-26 muestra un código que
cuenta cuántas veces aparece cada palabra en algún texto. Usamos un mapa hash con
las palabras como claves e incrementar el valor para realizar un seguimiento de cuántas veces hemos
visto esa palabra. Si es la primera vez que vemos una palabra, primero la insertaremos
con el valor de `0`:

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

<span class="caption">Listado 8-26: Recuento de ocurrencias de palabras usando un hash
map que almacena palabras y las cuenta</span>

este código pondrá `{"world": 2, "hello": 1, "wonderful": 1}`. El
método `or_insert` de hecho devuelve una referencia mutable (`&mut V`) al valor
para esta clave. Aquí almacenamos esa referencia mutable en la variable `count`, de manera que
para asignar a ese valor primero debemos eliminar la referencia `count` usando el
asterisco (`*`). La referencia mutable sale del alcance al final de la vuelta
`for`,entonces todos estos cambios son seguros y están permitidos por las reglas de préstamo.

### Función Hashing

Por defecto, `HashMap` utiliza una función hashing criptográficamente segura que puede
proporcionar resistencia a los ataques de denegación de servicio (DoS). Este no es el más rápido
algoritmo hashing disponible, pero la compensación para una mejor seguridad que viene
con la caída en el rendimiento, lo vale. Si perfila su código y encuentra
que la función hash predeterminada es demasiado lenta para sus propósitos, puede cambiar a
otra función especificando un * hasher * diferente. Un hasher es un tipo que
implementa el rasgo `BuildHasher`. Hablaremos sobre los rasgos y cómo
implementarlos en el Capítulo 10. No necesariamente tiene que implementar su propio
hasher desde cero; [crates.io](https://crates.io) tiene bibliotecas compartidas por
otros usuarios de Rust que proporcionan hashers implementando muchos algoritmos 
hash comúnes.

## Resúmen

Los vectores, cadenas y mapas hash proporcionarán una gran cantidad de funcionalidades
que necesita en programas donde necesita almacenar, acceder y modificar datos.
Aquí hay algunos ejercicios que ahora debes estar preparado para resolver:

* Dada una lista de enteros, usa un vector y devuelve la media (promedio), mediana
 (cuando está ordenado, el valor en la posición media) y modo (el valor que
  ocurre más a menudo; un mapa hash será útil aquí) de la lista.
* Convierte cadenas para cerdo latino. La primera consonante de cada palabra se mueve a
  se agrega el final de la palabra y "ay", por lo que "primero" se convierte en "irst-fay". Palabras
  que comienzan con una vocal tienen "hay" agregado al final en su lugar ("apple" se convierte
  “apple-hay”). ¡Tenga en cuenta los detalles sobre la codificación UTF-8!
* Usando un mapa hash y vectores, crea una interfaz de texto para permitir que un usuario agregue
  nombres de empleados a un departamento en una empresa. Por ejemplo, "Agregar a Sally a
  Ingeniería "o" Agregar Amir a Ventas ". Luego, deje que el usuario recupere una lista de todas las
  personas en un departamento o todas las personas en la empresa por departamento, ordenadas
  alfabeticamente.

La documentación estándar de la API de la biblioteca describe métodos que vectorizan, cadenas,
y hash maps, ¡que serán útiles para estos ejercicios!

Estamos entrando en programas más complejos en los que las operaciones pueden fallar; entonces, es
¡un momento perfecto para discutir el manejo de errores a continuación!
