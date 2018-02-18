## ## Los Vectores Almacenan Listas de Valores

El primer tipo de colección que veremos es `Vec <T>`, también conocido como * vector *.
Los vectores nos permiten almacenar más de un valor en una única estructura de datos que
pone todos los valores uno al lado del otro en la memoria. Los vectores solo pueden almacenar valores
del mismo tipo. Son útiles en situaciones en las que tiene una lista de
elementos, como las líneas de texto en un archivo o los precios de los artículos en una carro 
de compra.

### Creando un Nuevo Vector

Para crear un nuevo vector vacío, podemos llamar a la función `Vec :: new` como se muestra en
Listado 8-1:

```rust
let v: Vec<i32> = Vec::new();
```

<span class="caption">Listando 8-1: Creando un nuevo vector vacío para guardar valores
del tipo `i32`</span>

Tenga en cuenta que agregamos una anotación de tipo aquí. Porque no estamos insertando ningun
valores en este vector, Rust no sabe qué tipo de elementos pretendemos
almacenar. Éste es un punto importante. Los vectores se implementan usando genéricos;
cubriremos cómo usar genéricos con sus propios tipos en el Capítulo 10. Por ahora,
sepa que el tipo `Vec <T>` proporcionado por la biblioteca estándar puede contener cualquier tipo,
y cuando un vector específico tiene un tipo específico, el tipo se especifica dentro de
paréntesis angulares. En el listado 8-1, le hemos dicho a Rust que el `Vec <T>` en `v`
contiene elementos del tipo `i32`.

En un código más realista, Rust a menudo puede inferir el tipo de valor que queremos almacenar
una vez que insertemos los valores, rara vez necesita hacer esta anotación de tipo. Es más
comunmente para crear un `Vec <T>` que tiene valores iniciales, y Rust proporciona el
macro `vec!` por conveniencia. La macro creará un nuevo vector que contiene el
valores que le damos. El listado 8-2 crea un nuevo `Vec <i32>` que contiene los valores
`1`, `2`, y `3`:

```rust
let v = vec![1, 2, 3];
```

<span class="caption">Listando 8-2:Creando un nuevo vector que contiene
values</span>

Debido a que hemos dado valores iniciales de `i32`, Rust puede inferir que el tipo de` v`
es `Vec <i32>`, y la anotación de tipo no es necesaria. A continuación, veremos cómo
para modificar un vector.

### Actualizando un Vector

Para crear un vector y luego agregarle elementos, podemos usar el método `push` como
mostrado en el Listado 8-3:

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

<span class="caption">Lisando 8-3: Usando el método `push` para agregar valores a un
vector</span>

Como con cualquier variable, como se discutió en el Capítulo 3, si queremos ser capaces de
cambiar su valor, tenemos que hacerlo mutable usando la palabra clave `mut`. 
los números que colocamos dentro son todos de tipo `i32`, y Rust infiere esto de los
datos, por lo que no necesitamos la anotación `Vec <i32>`.

### Dejar caer un vector Cae sus elementos

Al igual que cualquier otra `struct`, un vector se liberará cuando salga del alcance, como
anotandolo en el Listado 8-4:

```rust
{
    let v = vec![1, 2, 3, 4];

    // do stuff with v

} // <- v goes out of scope and is freed here
```

<span class="caption">Listando 8-4: Mostrando donde el vector y sus elementos
son descartados</span>

Cuando el vector se descarta, todos sus contenidos también se descartarán, lo que significa que
esos enteros que contiene serán limpiados. Esto puede parecer como un
punto directo, pero puede ser un poco más complicado cuando comenzamos a
introducir referencias a los elementos del vector. ¡Vamos a abordar eso enseguida!

### Lectura de elementos de vectores

Ahora que sabe cómo crear, actualizar y destruir vectores, saber cómo
leer sus contenidos es un buen paso siguiente. Hay dos formas de hacer referencia a un
valor almacenado en un vector. En los ejemplos, hemos anotado los tipos de
valores que se devuelven de estas funciones para mayor claridad.

El Listado 8-5 muestra ambos métodos para acceder a un valor en un vector con
sintaxis de indexación o el método `get`:

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
let third: Option<&i32> = v.get(2);
```

<span class="caption">Listando 8-5: Usando la sintaxis de índices o el método `get` para
acceder a un elemento en un vector</span>

Tenga en cuenta dos detalles aquí. Primero, usamos el valor de índice de `2` para obtener el tercer
elemento: los vectores están en los índices por números, comenzando por cero. En segundo lugar, los dos tienen
diferentes formas de obtener el tercer elemento usando `&` y `[]`, que da
una referencia, o usando el método `get` con el índice pasado como
argumento, que nos da una `Opción <& T>`.

La razón por la que Rust tiene dos formas de hacer referencia a un elemento es para que pueda elegir cómo
el programa se comporta cuando intenta usar un valor de índice que el vector no
tiene un elemento para ello. Como ejemplo, veamos qué hará un programa si tiene
un vector que contiene cinco elementos y luego intenta acceder a un elemento en el índice
100, como se muestra en el Listado 8-6:

```rust,should_panic
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

<span class="caption">Listando 8-6: Intentando acceder al elemento en el índice
100 en un vector que contiene 5 elementos </ span>

Cuando ejecutes este código, el primer método `[]` causará `panic!` por que este
hace referencia a un elemento inexistente. Este método podría usarse de una mejor manera
si quiere que su programa pueda considerar el acceder a un elemento que está más allá del final del vector
ya que sería un grave error si el programa llegara a bloquearse.

Cuando el método `get` se pasa un índice que está fuera del vector, regresa al
`None` sin entrar en pánico. Deberías usar este método si accedes a un elemento
más allá del rango del vector ocurre ocasionalmente bajo circunstancias normales.
Su código tendrá lógica para manejar tener `Some (& element)` o
`None`, como se discutió en el Capítulo 6. Por ejemplo, el índice podría venir de
una persona que ingresa un número. Si ingresan accidentalmente un número que también es
grande y el programa obtiene un valor 'Ninguno', podría decirle al usuario cuántos
artículos están en el vector actual y les da otra oportunidad de ingresar un valor
válido. Eso sería más fácil de usar que estrellar el programa debido a un error tipográfico.

#### Referencias inválidas

Cuando el programa tiene una referencia válida, el comprobador de préstamos impone las
reglas de propiedad y de endeudamiento (cubiertas en el Capítulo 4) para asegurar esta referencia
y cualquier otra referencia a los contenidos del vector sigue siendo válida. Recuerda la
regla que establece que no podemos tener referencias mutables e inmutables en el mismo
alcance. Esa regla se aplica en el Listado 8-7 donde tenemos una referencia inmutable a
el primer elemento en un vector e intenta agregar un elemento al final, que no
hace el trabajo:

```rust,ignore
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);
```

<span class="caption">Listando 8-7: Intentando agregar un elemento a un vector
mientras mantiene una referencia a un elemento </ span>

Compilar este código dará como resultado este error:

```text
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 -->
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^ mutable borrow occurs here
7 |
8 | }
  | - immutable borrow ends here
```

El código en el Listado 8-7 podría parecer que debería funcionar: ¿por qué una referencia
al primer elemento le importa lo que cambia al final del vector? la
razón detrás de este error se debe a la forma en que funcionan los vectores: agregar un nuevo elemento
en el extremo del vector podría requerir asignar nueva memoria y copiar los
elementos antiguos al nuevo espacio si no hay espacio suficiente para poner todos los
elementos uno al lado del otro donde estaba el vector. En ese caso, la referencia
al primer elemento estaría apuntando a la memoria desasignada. El préstamo de
las reglas impiden que los programas terminen en esa situación.

> Nota: Para obtener más información sobre los detalles de implementación del tipo `Vec <T>`, consulte "
> Rustonomicon” en https://doc.rust-lang.org/stable/nomicon/vec.html.

### Iteración sobre los valores en un vector

Si queremos acceder a cada elemento en un vector a su vez, podemos iterar a través de
todos los elementos en lugar de utilizar índices para acceder uno a la vez. El listado
8-8 muestra como usar `for` bucle para obtener referencias inmutables a cada elemento
en un vector de valores `i32` e imprímalos:

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
```

<span class="caption">Listado 8-8: Imprimir cada elemento en un vector por el
iterando sobre los elementos usando un bucle `for` </ span>

También podemos iterar sobre referencias mutables para cada elemento en un vector mutable
para hacer cambios a todos los elementos. El bucle `for` en el Listado 8-9
agregará `50` a cada elemento:

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

<span class="caption">Listado 8-9: Iterando sobre referencias mutables a
elementos en un vector </span>

Para cambiar el valor al que se refiere la referencia mutable, tenemos que usar el
operador de referencia (`*`) para obtener el valor en `i` antes de que podamos usar el
operador `+=`.

### Usando un Enum para almacenar múltiples tipos

Al comienzo de este capítulo, dijimos que los vectores solo pueden almacenar valores
que son del mismo tipo. Esto puede ser un inconveniente; definitivamente hay uso de
casos por la necesidad de almacenar una lista de artículos de diferentes tipos. Afortunadamente, las
variantes de una enumeración se definen bajo el mismo tipo de enumeración, por lo que cuando necesitamos
almacenar elementos de un tipo diferente en un vector, podemos definir y usar una enumeración!

Por ejemplo, supongamos que queremos obtener valores de una fila en una hoja de cálculo donde
algunas de las columnas en la fila contienen enteros, algunos números de coma flotante,
y algunas cuerdas. Podemos definir una enumeración cuyas variantes sostendrán los diferentes
tipos de valor, y luego todas las variantes enum se considerarán del mismo tipo:
el de la enumeración. Entonces podemos crear un vector que contenga esa enumeración y así,
finalmente, tiene diferentes tipos. Hemos demostrado esto en el Listado 8-10:

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

<span class="caption">Listado 8-10: Definir un `enum` para almacenar valores de
diferentes tipos en un vector </span>

La razón por la que Rust necesita saber qué tipos estarán en el vector en tiempo de compilación
es así que sabe exactamente cuánta memoria en el montón será necesaria para almacenar cada
elemento. Una ventaja secundaria es que podemos ser explícitos sobre qué tipos son
permitido en este vector. Si Rust permitiera que un vector tuviera cualquier tipo, habría
ser una posibilidad de que uno o más de los tipos causaría errores con las
operaciones realizadas en los elementos del vector. Usando una enumeración más 
la expresión `match` significa que Rust asegurará en tiempo de compilación que siempre
maneja todos los casos posibles, como se discutió en el Capítulo 6.

Si no conoce el conjunto exhaustivo de tipos que el programa obtendrá en tiempo de ejecución
para almacenar en un vector cuando estás escribiendo un programa, la técnica enum no
trabaja. En cambio, puede usar un objeto de rasgo, que veremos en el Capítulo 17.

Ahora que hemos discutido algunas de las formas más comunes de usar vectores, asegúrese
para revisar la documentación API para todos los muchos métodos útiles definidos en
`Vec <T>` por la biblioteca estándar. Por ejemplo, además de `push`,` pop`
este método elimina y devuelve el último elemento. Pasemos al siguiente
tipo de colección: `String`!
