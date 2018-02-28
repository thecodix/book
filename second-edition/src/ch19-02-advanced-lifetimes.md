## Vida útil avanzada

De vuelta en el Capítulo 10 en la sección "Validación de referencias con tiempos de vida",
Aprendí a anotar referencias con parámetros de por vida para decirle a Rust cómo
vidas de diferentes referencias se relacionan. Vimos cómo cada referencia puede durar
de por vida, pero, la mayoría de las veces, Rust te permitirá olvidarlas. Aquí vamos a
mira tres características avanzadas de vidas que aún no hemos cubierto:


* Subtipo de por vida, una forma de asegurar que una vida sobreviva a otra para siempre.
* Límites de por vida, para especificar un tiempo de vida para una referencia a un tipo genérico.
* Duración de vida de los objetos, cómo se infieren y cuándo deben ser  especificados


<!-- ¿debería agregar un pequeño resumen de cada uno aquí? 
Eso nos permitiría aclarar directamente en ejemplos en la 
siguiente sección -> <! - He cambiado a viñetas y agregué un 
pequeño resumen / Carol -->

### La subtipificación de por vida garantiza que una vida sobrevive a otra

El subtipado de por vida es una forma de especificar que una vida 
debe sobrevivir a otra de por vida. Para explorar la subtipificación 
de por vida, imagina que queremos escribir un analizador sintáctico. 
Tendremos una estructura llamada `Context` que contiene una referencia
a la cadena que estamos analizando, Escribiremos un analizador que 
analizará esta cadena y devolverá éxito o fracaso. El analizador 
necesitará tomar prestado el contexto para hacer el analizando. 
Implementar esto se vería como el código en el listado 19-12, excepto
que este código no tiene las anotaciones de vida requeridas para que no se compile:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust,ignore
struct Context(&str);

struct Parser {
    context: &Context,
}

impl Parser {
    fn parse(&self) -> Result<(), &str> {
        Err(&self.context.0[1..])
    }
}
```

<span class="caption">Listing 19-12: Definiendo anotacines de un analizador 
  sin tiempo de vida</span>

La compilación del código da como resultado errores que indican que 
Rust espera una vida útil con parámetros en el segmento de cadena en  
`Context` y la referencia a `Context` en `Parser`.

<!-- ¿Cuál será el error de tiempo de compilación aquí? Creo que valdría la pena mostrar
eso para el lector ->
<! - Los errores solo dicen "parámetro de por vida esperado", son bastante aburridos.
Hemos mostrado mensajes de error como ese anteriormente, así que lo he explicado en palabras.
/ Carol -->


En aras de la simplicidad, nuestra función `parse` devuelve un
`Result <(), & str> `. Este, no hará nada en el éxito, y en el 
fracaso devolverá la parte de la segmento de cadena que no se 
analizó correctamente. Una implementación real tendría más información
de error que eso, y realmente devolvería algo al analizar si tiene éxito,
pero los dejaremos porque no son relevantes para el vidas que forman parte de este ejemplo.

Para mantener este código simple, no vamos a escribir ninguna lógica de análisis.
Es muy probable que en algún lugar de la lógica de análisis manejemos la entrada no válida por
devolver un error que hace referencia a la parte de la entrada que no es válida, y
esta referencia es lo que hace que el ejemplo de código sea interesante con respecto a las
vidas, Entonces vamos a pretender que la lógica de nuestro analizador es que
la entrada no es válida después del primer byte. Tenga en cuenta que este 
código puede entrar en pánico si el primer byte no está en un límite de 
caracteres válidos; de nuevo, estamos simplificando
ejemplo para concentrarse en las vidas involucradas.


<!-- ¿Por qué queremos siempre error después del primer byte? ->
<! - Por simplicidad del ejemplo para evitar saturar el código 
real Analizando la lógica, que no es el punto. He explicado un poco más arriba / Carol ->

Para obtener este código compilando, necesitamos completar los
parámetros de vida para el string slice en `Context` y la referencia
al` Context` en `Parser`. La forma más sencilla de hacer esto es usar
la misma vida en todas partes, como se muestra en el listado 19-13:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
struct Context<'a>(&'a str);

struct Parser<'a> {
    context: &'a Context<'a>,
}

impl<'a> Parser<'a> {
    fn parse(&self) -> Result<(), &str> {
        Err(&self.context.0[1..])
    }
}
```

<span class="caption">Listado 19-13: Anotando todas las referencias en `Context` 
  y `Parser` con el mismo parámetro de vida</span>

Esto compila bien y le dice a Rust que un `Parser` contiene una referencia 
a un `Context` con lifetime `a`, y que `Context` contiene un segmento de 
cadena que también vive tanto tiempo como la referencia al `Context` en `Parser` 
.El compilador de Rust envía un mensaje de error con dichos parámetros de por
vida que fueron necesarios para estas referencias, y ahora hemos agregado parámetros de por vida.

<!-- ¿Puede dejar que el lector sepa que deberían estar tomando distancia 
de este anterior ¿ejemplo? No tengo muy claro por qué agregar vidas aquí 
guardado el código -> <! - Done -->

Luego, en el listado 19-14, agreguemos una función que toma una instancia
de `Context`, usa un` Parser` para analizar ese contexto, y devuelve `parse`. 
Esto no funcionará del todo:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust,ignore
fn parse_context(context: Context) -> Result<(), &str> {
    Parser { context: &context }.parse()
}
```

<span class="caption">Listado 19-14: Un intento de agregar una funcion `parse_context` que 
  toma un `Contexto` y utiliza un `Parser` </span>

Obtenemos dos errores bastante detallados cuando tratamos de compilar el 
código con el Además de la función `parse_context`:

```text
error[E0597]: borrowed value does not live long enough
  --> src/lib.rs:14:5
   |
14 |     Parser { context: &context }.parse()
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ does not live long enough
15 | }
   | - temporary value only lives until here
   |
note: borrowed value must be valid for the anonymous lifetime #1 defined on the function body at 13:1...
  --> src/lib.rs:13:1
   |
13 | / fn parse_context(context: Context) -> Result<(), &str> {
14 | |     Parser { context: &context }.parse()
15 | | }
   | |_^

error[E0597]: `context` does not live long enough
  --> src/lib.rs:14:24
   |
14 |     Parser { context: &context }.parse()
   |                        ^^^^^^^ does not live long enough
15 | }
   | - borrowed value only lives until here
   |
note: borrowed value must be valid for the anonymous lifetime #1 defined on the function body at 13:1...
  --> src/lib.rs:13:1
   |
13 | / fn parse_context(context: Context) -> Result<(), &str> {
14 | |     Parser { context: &context }.parse()
15 | | }
   | |_^
```

Estos errores dicen que tanto la instancia `Parser` que se creó como
`context` parameter live only desde cuando se crea el` Parser` hasta el final
de la función `parse_context`, necesitan vivir en su totalidad para la
vida de la función.


En otras palabras, `Parser` y` context` necesitan * sobrevivir * a la 
función completa y ser válido antes de que la función comience tan bien 
como después de que termine en orden para que todas las referencias en 
este código siempre sean válidas. Tanto el `Parser` que estamos
creando y el parámetro `context` sale fuera del alcance al final de
función, sin embargo (porque `parse_context` toma posesión de` context`).


<!-- Oh interesante, ¿por qué necesitan sobrevivir a la función?, simplemente
para aseguran absolutamente de que vivirán mientras dure la función --> 
<!-- Sí, es lo que creo que hemos dicho en la primera frase del párrafo anterior.
¿Hay algo que no está claro? / Carol -->

Para descubrir por qué estamos recibiendo estos errores, veamos las 
definiciones en Listado 19-13 nuevamente, específicamente 
las referencias en la firma del Método `parse`:

```rust,ignore
    fn parse(&self) -> Result<(), &str> {
```

<!-- What exactly is it the reader should be looking at in this signature? -->
<!-- Added above /Carol -->

¿Recuerdas las reglas de elision? Si anotamos las vidas de las 
referencias en lugar de elidir, la firma sería:

```rust,ignore
    fn parse<'a>(&'a self) -> Result<(), &'a str> {
```

Es decir, la parte de error del valor de retorno de `parse` tiene una vida que esta
atada a la vida de la instancia `Parser` (la de` & self` en `parse`
firma del método). Eso tiene sentido: el segmento de cadena devuelto hace referencia a los
segmento de cadena en la instancia `Context` sostenida por `Parser`, y la definición
de la estructura `Parser` especifica que la duración de la referencia a
`Context` y el tiempo de vida del segmento de cadena que contiene `Context` deben ser
lo mismo.


El problema es que la función `parse_context` devuelve el valor devuelto
por `parse`, por lo que el tiempo de vida del valor de retorno de` parse_context` está vinculado a
la vida útil del `Parser` también. Pero la instancia `Parser` creada en
La función `parse_context` no sobrevivirá al final de la función (es
temporal), y `context` saldrá del alcance al final de la función
(`parse_context` se apropia de él).


Rust piensa que estamos tratando de devolver una referencia a un valor
que sale de alcance al final de la función, porque anotamos todas las 
vidas con el mismo parámetro, Eso le dijo a Rust que la duración del 
segmento de cadena que contiene `Context` es la misma que la duración 
de la referencia al `Context` que contiene el `Parser`.

La función `parse_context` no puede ver eso dentro de la función` parse`,
El segmento de cadena devuelto sobrevivirá a `Context` y` Parser`, y que el
la referencia `parse_context` returns se refiere al segmento de cadena, no a` Context`
o `Parser`.

Al saber qué hace la implementación de `parse`, sabemos que el único
razón por la cual el valor de retorno de `parse` está vinculado al `Parser` 
es porque está haciendo referencia al `Context` del` Parser`, que hace 
referencia al segmento de cadena, por lo es realmente el tiempo de vida 
del segmento de cadena que `parse_context` necesita cuidar. Necesitamos una
manera de decirle a Rust que el corte de la cadena en `Context` y la referencia 
al `Context` en` Parser` tiene diferentes tiempos de vida y que el valor de
retorno de `parse_context` está vinculado al tiempo de vida del segmento de 
cadena en `Context`.


Primero trataremos de darle a `Parser` y` Context` diferentes parámetros
de por vida como se muestra en el listado 19-15. Usaremos `'s` y` 'c` como
nombres de parámetros de por vida para tener claro qué vida va con el 
segmento de cadena en `Context` y cuál va con la referencia a `Context` en `Parser`.
Tenga en cuenta que esto no soluciona completamente el problema, 
pero es un comienzo y veremos por qué esto no es suficiente cuando intentamos compilar.

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust,ignore
struct Context<'s>(&'s str);

struct Parser<'c, 's> {
    context: &'c Context<'s>,
}

impl<'c, 's> Parser<'c, 's> {
    fn parse(&self) -> Result<(), &'s str> {
        Err(&self.context.0[1..])
    }
}

fn parse_context(context: Context) -> Result<(), &str> {
    Parser { context: &context }.parse()
}
```

<span class="caption">> Listado 19-15: Especificación de diferentes parámetros 
    de vida útil para las referencias al segmento de cadena y al `Context`</span>

Hemos anotado las vidas de las referencias en todos los lugares que
se anotaron en el listado 19-13, pero usaron diferentes parámetros dependiendo de
si la referencia va con el segmento de cadena o con `Context`. Nosotros también
agregamos una anotación a la parte del corte de cadena del valor de retorno de `parse` 
e indica que va con el tiempo de vida del segmento de cadena en `Context`.


El siguiente es el error que obtenemos ahora cuando intentamos compilar:

```text
error[E0491]: in type `&'c Context<'s>`, reference has a longer lifetime than the data it references
 --> src/lib.rs:4:5
  |
4 |     context: &'c Context<'s>,
  |     ^^^^^^^^^^^^^^^^^^^^^^^^
  |
note: the pointer is valid for the lifetime 'c as defined on the struct at 3:1
 --> src/lib.rs:3:1
  |
3 | / struct Parser<'c, 's> {
4 | |     context: &'c Context<'s>,
5 | | }
  | |_^
note: but the referenced data is only valid for the lifetime 's as defined on the struct at 3:1
 --> src/lib.rs:3:1
  |
3 | / struct Parser<'c, 's> {
4 | |     context: &'c Context<'s>,
5 | | }
  | |_^
```

Rust no sabe de ninguna relación entre `'c` y`' s`. Con el fin de ser
válido, los datos a los que se hace referencia en `Context` con `s` de por vida deben ser
restringidos, para garantizar que viva más tiempo que la referencia con tiempo de vida
`'c`. Si `'s` no es más largo que`' c`, la referencia a `Context` podría no ser
válida.

Lo que nos lleva al punto de esta sección: la función Rust *lifetime subtyping* 
es una forma de especificar que un parámetro de por vida vive al menos como siempre
y cuando sea otro. En los corchetes angulares donde declaramos la vida de los 
parámetros, podemos declarar una vida `'a` como de costumbre, y declarar una vida
`'b` que vive al menos tanto como`'a` al declarar `'b` con la sintaxis`' b: 'a'.

In our definition of `Parser`, in order to say that `'s` (the lifetime of the
string slice) is guaranteed to live at least as long as `'c` (the lifetime of
the reference to `Context`), we change the lifetime declarations to look like
this:

<span class="filename">Nombre del archivo:: src/lib.rs</span>

```rust
# struct Context<'a>(&'a str);
#
struct Parser<'c, 's: 'c> {
    context: &'c Context<'s>,
}
```

Ahora, la referencia a `Context` en` Parser` y la referencia a la cadena
cortada en el `Context` tienen diferentes tiempos de vida, y nos hemos asegurado de que
la vida útil del segmento de la cadena es más larga que la referencia al `Context`.



Ese fue un ejemplo muy largo, pero como mencionamos al comienzo de este capítulo, 
estas características son bastante nicho. A menudo no necesitarás esta sintaxis, 
pero puede aparecer en situaciones como esta, donde necesita consultar 
algo a lo que tienes una referencia.


### Límites de por vida en las referencias a los tipos genéricos

En la sección "Trait Bounds" del Capítulo 10, discutimos el uso de límites de rasgos en
tipos genéricos, También podemos agregar parámetros de por vida como restricciones en tipos 
genéricos, y estos se llaman *límites de por vida*. Los límites de por vida ayudan a Rust a verificar
las referencias en tipos genéricos que no sobrevivirán a los datos a los que hacen referencia.


<!-- Can you say up front why/when we use these? -->
<!-- Done -->

Por ejemplo, considere un tipo que sea un contenedor de referencias. 
Recuerda el `RefCell <T>` escriba desde "` RefCell <T> `y el Patrón 
de Mutabilidad Interior" sección del Capítulo 15: sus métodos `borrow` y` borrow_mut` 
devuelven los tipos `Ref` y` RefMut`, respectivamente. Estos tipos 
son envoltorios sobre referencias que Mantengan un registro de las 
reglas de endeudamiento en tiempo de ejecución. La definición de `Ref` 
struct se muestra en el listado 19-16, sin límites de por vida por ahora:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust,ignore
struct Ref<'a, T>(&'a T);
```

<span class="caption">Listado 19-16: Definición de una estructura para envolver 
    una referencia a un tipo genérico; sin límites de por vida para comenzar</span>

Sin restringir explícitamente el tiempo de vida `a` en relación con el 
genérico parámetro `T`, Rust dara un error porque no sabe por cuánto 
tiempo el genérico tipo `T` va a vivir:

```text
error[E0309]: the parameter type `T` may not live long enough
 --> src/lib.rs:1:19
  |
1 | struct Ref<'a, T>(&'a T);
  |                   ^^^^^^
  |
  = help: consider adding an explicit lifetime bound `T: 'a`...
note: ...so that the reference type `&'a T` does not outlive the data it points at
 --> src/lib.rs:1:19
  |
1 | struct Ref<'a, T>(&'a T);
  |                   ^^^^^^
```

Debido a que `T` puede ser de cualquier tipo,` T` podría ser una referencia 
o un tipo que contiene una o más referencias, cada una de las cuales podría 
tener sus propias vidas. Rust no puede estar seguro de que `T` viva tanto tiempo como `'a`.

Afortunadamente, ese error nos brindó consejos útiles sobre 
cómo especificar el tiempo de vida obligado en este caso:


```text
consider adding an explicit lifetime bound `T: 'a` so that the reference type
`&'a T` does not outlive the data it points at
```

El listado 19-17 muestra cómo aplicar este consejo especificando el 
límite de por vida cuando declaramos el tipo genérico `T`.

```rust
struct Ref<'a, T: 'a>(&'a T);
```

<span class="caption">Listado 19-17: Agregar límites de por vida en `T` 
    para especificar que cualquier referencia en `T` viva al menos tanto como `'a`</span>

Este código ahora se compila porque la sintaxis `T: 'a` 
especifica que `T` puede ser cualquier tipo, pero si contiene 
alguna referencia, las referencias deben vivir al menos como `'a`.



Podríamos resolver esto de una manera diferente, que se muestra en la 
definición de `StaticRef`en el listado 19-18, agregando el límite de
tiempo de vida `'static` enlazando `T`. Esto significa que si `T` 
contiene referencias, deben tener el `'static` toda la vida:

```rust
struct StaticRef<T: 'static>(&'static T);
```

<span class="caption">Listado 19-18: Agregar una vida útil `'static` ligada a
    `T` restringir `T` a tipos que tienen solo referencias `'static` o no tengan referencias</span>


Porque `'static` significa que la referencia debe vivir tanto como todo el programa,
un tipo que no contiene referencias cumple los criterios de todas las referencias que viven
siempre y durante todo el programa (porque no hay referencias). no hay real
distinción entre un tipo que no tiene referencias y un tipo que tiene
referencias que viven para siempre; ambos son iguales para el propósito de
determinar si una referencia tiene o no una vida más corta que la que tiene.


### Inferencia en los tiempos de vida del objeto de rasgo


En el Capítulo 17 el "Uso de objetos de rasgo que permiten valores de diferentes
Tipos de sección”, discutimos los objetos de rasgo, que consisten en un rasgo 
detrás de una referencia y nos permiten usar el despacho dinámico. 
Todavía no hemos discutido qué sucede si el tipo que implementa el rasgo en 
el objeto de rasgo tiene vida propia. Considere el Listado 19-19, donde tenemos
un rasgo `Red` y una estructura `Ball`. `Ball` contiene una referencia 
(y por lo tanto tiene un parámetro de por vida) y también implementa el rasgo 
`Red`. Queremos usar una instancia de `Ball` como un objeto de rasgo `Box<Red>`:


<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
trait Red { }

struct Ball<'a> {
    diameter: &'a i32,
}

impl<'a> Red for Ball<'a> { }

fn main() {
    let num = 5;

    let obj = Box::new(Ball { diameter: &num }) as Box<Red>;
}
```

<span class="caption">Listado 19-19: Usar un tipo que tiene un parámetro de 
por vida con un objeto de rasgo</span>

Este código compila sin ningún error, aunque no hayamos dicho nada 
explícito sobre las vidas involucradas en `obj`. Esto funciona porque
hay reglas que tienen que ver con el tiempo de vida y los objetos de rasgo:

* La vida útil por defecto de un objeto de rasgo es `'static'. 
* Con `& 'a Trait` o` &' a mut Trait`, la duración predeterminada es `'a`. 
* Con una sola cláusula `T: 'a`, la duración predeterminada es `'a`. 
* Con múltiples cláusulas tipo `T: 'a`, no habrá ningún defecto; debemos ser explícitos.

Cuando debemos ser explícitos, podemos agregar un límite de por vida en un objeto de rasgo como 
`Box <Red>` con la sintaxis `Box <Red + 'a>` o `Box <Red +' static>`, 
dependiendo de lo que se necesita, al igual que con los otros límites, 
esto significa que cualquier implementador del rasgo `Rojo` que tiene 
referencias internas debe tener la misma duración especificada en los 
límites del objeto de rasgo como esas referencias.

A continuación, echemos un vistazo a algunas otras características avanzadas relacionadas con los rasgos.
