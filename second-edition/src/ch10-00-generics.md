# Tipos de genéricos, rasgos y tiempos de vida

Cada lenguaje de traducción tiene herramientas para lidiar efectivamente con la duplicación
de conceptos; en Rust, una de esas herramientas es *Genéricos*. Genéricos son conceptos
sustitutos para tipos concretos u otras propiedades. Cuando estámos escribiendo y
compilando el código podemos expresar propiedades de genéricos, como su
comportamiento o como se relacionan con otros generics, sin necesidad de saber qué 
serán en su lugar.

En la misma manera en la que una función toma unos parametros cuyos valores no sabemos en
orden para escribir un codigo que correrá en multiples valores concretos, nosotros podemos
escribir funciones que tomarán parámetros de algún tipo genérico en vez de uno concreto.
Tipos como `i32` o `String`. Ya hemos usado genéricos en el capítulo 6 con 
`Option<T>`, en el capítulo 8 con `Vec<T>` y `HashMap<K, V>`, y en el capítulo 9 con
`Result<T, E>`. En éste capítulo, exploraremos como definir nuestros propios tipos,
funciones y metodos con los genéricos!

Primero, vamos a revisar las mecánicas de extraer una función que
reduce la duplicación de códigos. Entonces usaremos las mismas mecánicas para generar una función
genérica de dos funciones que solamente difieren en los tipos de sus 
parametros. Analizaremos el uso de los tipos genéricos en una estructura y definiciones enumeradas
también.

*Después de eso, discutiremos *rasgos*, los cuales son una manera de definir un comportamiento en una
manera genérica. Los rasgos pueden ser combinados con tipos genéricos para limitar un
tipo generico para esos tipos que tienen un comportamiento particular, en vez de 
cualquier otro tipo.

Finalmente, discutiremos los *los tiempos de vida*, los cuales son un tipo de genérico que nos deja
darle al compilador información sobre cómo las referencias están relacionadas entre sí
Los tiempos de vida son las característica en Rust que nos deja tomar prestados valores en varias 
situaciones y tener la prueba del compilador de que las referencias serán válidas.

## Retirando la duplicación al extraer una función

Antes de entrar de lleno con la sintaxis de los genéricos, aprendamos primero una técnica para lidiar
con la duplicación que no usa tipos genéricos: extraer una función. Una vez
que eso esté fresco en nuestras mentes, usaremos las mismas mecánicas con los genéricos para
extraer una función genérica! De la misma manera en la que reconoces un código duplicado
para extraer en una función, comenzarás a reconocer el código duplicado que puede
Usar un genérico

considera un pequeño programa que encuentra el número más largo en una lista. Se muestra en
el listado 10-1: 

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
#  assert_eq!(largest, 100);
}
```

<span class="caption">Listado 10-1: Código para encontrar el número más largo en una lista 
de números</span>

Éste código toma una lista de números enteros, guardados en la variable `number_list`. 
Pone el primer objeto en la lista en una variable llamada `largest`. Entonces revisa
todos los números en la lista, y si el valor actual es mayor a
el número guardado en `largest`, reemplaza el valor en `largest`. Si el
valor actual es menor que el valor más grande que se ha visto hasta ahora, `largest` no será
cambiado. Cuando todos los objetos en la lista han sido considerados, `largest` se quedará
con el valor más largo, el cual en este caso es 100.

Si necesitasemos encontrar el número más largo en dos listas de números distintas, podríamos
duplicar el código en el listado 10-1 y tendría la misma lógica que existiese en dos
lugares en el programa, como en el listado 10-2:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```

<span class="caption">Listado 10-2: Código para encontrar el número más grande en *dos* 
listas de números</span>

Aunque éste código funciona, la duplicación de códigos es tediosa y susceptible a errores, y significa que
tenemos varios lugares donde actualizar la lógica si necesitamos cambiarlo.

<!-- Are we safe assuming the reader will be familiar with the term
"abstraction" in this context, or do we want to give a brief definition? -->
<!-- Yes, our audience will be familiar with this term. /Carol -->

Para eliminar ésta duplicación, podemos crear una abstracción, lo que en este caso
será en la forma de una función que opera en cualquier lista de números enteros que se le hayan dado
a la función en un parámetro. Esto incrementará la claridad de nuestro código y
nos deja comunicar y razonar el concepto de encontrar el número más grande
en una lista independientemente de los lugares específicos en los que este concepto sea usado.

En el programa del listado 10-3, hemos extraido el código que encuentra el número
más grande en una función llamada  `largest`. Éste programa puede encontrar el número
más grande en dos listas diferentes de números, pero el código del listado 10-1 solo
existe en un sitio:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
fn largest(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
#    assert_eq!(result, 100);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
#    assert_eq!(result, 6000);
}
```

<span class="caption">Listado 10-3: Código de abstracción para encontrar el número más grande
en dos listas</span>

La función tiene un parametro, `list`, que representa cualquier pedazo concreto de los valores de
`i32` que podríamos pasar a la función. El código en la definición
de la función opera en `list` la representación de cualquier `&[i32]`. Cuando llamamos la
función `largest`, el código de hecho corre en los valores específicos
que le pasemos.

Las mecánicas que revisamos para ir desde el listado 10-2 to al listado 10-3 fueron 
estos tres pasos:

1. Nos dimos cuenta de que había un código de duplicación.
2. Extrajimos el código duplicado en el cuerpo de la función, y especificamos
   los resultados y valores de respuesta en ese código en la firma de la función.
3. Reemplazamos los dos lugares completos que tenían el código duplicado para en cambio llamar
   una función.

Podemos usar estos mismos pasos con los genéricos para reducir la duplicación de códigos de 
diferentes maneras en diferentes escenarios. De la misma manera que el cuerpo de la función
está operando ahora en un abstracto `list` en ves de valores concretos, un código usando 
genéricos operará en tipos abstractos. Los conceptos que le dan poder a los genéricos son los
mismos conceptos que sabes que le dan poder a las funciones, sólo que aplicados
de forma distinta.

¿Qué si tuvieramos dos funciones, una que encontrase el objeto más grande en un pedazo de valores
`i32` y uno que encontrase el objeto más grande en un pedazo de valores `char`?
¿Cómo nos podríamos deshacer de esa duplicación? ¡Vamos a averiguarlo!
