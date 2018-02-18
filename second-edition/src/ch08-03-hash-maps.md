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
