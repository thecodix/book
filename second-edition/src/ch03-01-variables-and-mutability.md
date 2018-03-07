##Variables y Mutabilidad

Como mencionamos en el capítulo 2, las variables por defecto son inmutables. Esta es una de las muchas motivaciones en Rust que te alienta a escribir su codigo de una forma que aproveche la seguridad y la fácil concurrencia que Rust ofrece. Sin embargo, sigues teniendo la opción de hacer que tus variables sean mutables. Exploremos como y 
porque Rust te alienta a favorecer la inmutabilidad, y en que casos sería mejor no usarla.

Cuando una variable es inmutable, eso significa que una vez que un valor está ligado a un nombre, no puedes cambiar ese valor. Para ilustrarlo, generemos un nuevo proyecto llamado variables en su directorio de proyectos mediante el uso de las nuevas variables cargo --bin.

Luego, en tu nuevo directorio de variables, abra src/main.rs y reemplace su código con el siguiente:

Nombre del archivo: src/main.rs

fn main() {
    let x = 5;
    println!("El valor de x es: {}", x);
    x = 6;
    println!("El valor de x es: {}", x);
}
Guarde y ejecute el programa usando el cargo run. Debería recibir un mensaje de error, como se muestra en esta salida:

error[E0384]:  cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |      - first assignment to `x`
-3 |     println!("The value of x is: {}", x);
4 |     x = 6;
  |     ^^^^^  cannot assign twice to immutable variable `x`

Este ejemplo muestra como el compilador te ayuda a encontrar errores en sus programas. Aunque los errores del compilador pueden ser frustrantes, solo significa que su programa aún no está haciendo lo que quieres que haga de forma segura; ¡esto no significa que no eres un buen programador! Los rustáceos experimentados todavía siguen teniendo errores de compilación.

El error indica que la causa del error es que no podemos asignar dos veces a la variable inmutable x, porque
intentamos de asignar un segundo valor a la variable inmutable x.

Es importante que tengamos errores de tiempo de compilación cuando intentamos cambiar un valor que previamente designamos como inmutable porque esta causa de errores puede ser díficil de ubicar después del hecho. Si una parte de nuestro código opera en la suposición de que un valor nunca cambiará y otra parte de nuestro código cambia ese valor, es posible que la primera parte del código no haga lo que fue diseñado para hacer. Esta causa de errores puede ser dificil de ubicar después del hecho, especialmente cuando la segunda pieza de código cambia el valor solo algunas veces.

En Rust, el compilador garantiza que cuando establecemos que un valor no cambie, realmente no cambiará. Eso significa que cuando estás leyendo y escribiendo código, no tienes que hacer un seguimiento de cómo y donde un valor puede cambiar, lo cual puede hacer que el código sea más fácil de razonar.

Pero la mutabilidad puede ser muy útil. Las variables son inmutables solo por defecto; podemos hacerlas mutables agregando mut adelante del nombre de la variable. Además de permitir que este valor cambie, transmite la intención a futuros lectores del código indicando que otras partes del código cambiarán el valor de esta variable.

Por ejemplo, cambie src/main.rs a lo siguiente:

Nombre del archivo: src/main.rs

fn main() {
    let mut x = 5;
    println!("El valor de x es: {}", x);
    x = 6;
    println!("El valor de x es: {}", x);
}
Cuando ejecutamos este programa, obtenemos lo siguiente:

$ cargo run
   Compilando variables v0.1.0 (file:///projects/variables)
    Finalizado dev [unoptimized + debuginfo] destino(s) en 0.30 segundos
     Ejecutando `target/debug/variables`
El valor de x es: 5
El valor de x es: 6
Usando ```mut```, podemos cambiar el valor que se une a x de 5 a 6. En algunos casos, usted querrá hacer una variable mutable porque hace que escribir código sea más conveniente que usar una implementación únicamente con variables inmutables.

Hay múltiples intercambios para considerar, además de la prevención de errores. Por ejemplo, en casos donde está usando estructuras de grandes datos, mutar una instancia en su lugar puede ser más rapido que copiar y devolver instancias asignadas recientemente. Con estructuras de datos más pequeñas, escribir nuevas instancias y escribir en un estilo de programación más funcional puede ser más fácil de razonar, por lo que el rendimiento más bajo podría ser un hándicap que sirva para ganar esa claridad.

Diferencias Entre Variables y Constantes
No poder cambiar el valor de una variable podría haberle hecho acordar de otro concepto de programación que la mayoría de los otros lenguajes tiene: las constantes. Como las variables inmutables, las constantes también son valores que se unen a un nombre y no se pueden cambiar, pero hay algunas diferencias entre constantes y variables.

Primero, no podemos usar mut con constantes: las constantes no son sólo immutables por defecto, son siempre immutables.


Declaramos constantes usando la palabra clave const en lugar de la palabra clave let, y el tipo del valor debe ser anotado. Estamos prontos a cubrir tipos y tipos de anotaciones en la siguiente sección, “Tipos de Datos,” por lo que no te preocupes por los detalles ahora mismo, solo sepa que siempre debemos anotar el tipo.


Constants can be declared in any scope, including the global scope, which makes them useful for values that many parts of code need to know about.

The last difference is that constants may only be set to a constant expression, not the result of a function call or any other value that could only be computed at runtime.

Here’s an example of a constant declaration where the constant’s name is MAX_POINTS and its value is set to 100,000. (Rust constant naming convention is to use all upper case with underscores between words):

const MAX_POINTS: u32 = 100_000;
Constants are valid for the entire time a program runs, within the scope they were declared in, making them a useful choice for values in your application domain that multiple parts of the program might need to know about, such as the maximum number of points any player of a game is allowed to earn or the speed of light.

Naming hardcoded values used throughout your program as constants is useful in conveying the meaning of that value to future maintainers of the code. It also helps to have only one place in your code you would need to change if the hardcoded value needed to be updated in the future.

Shadowing
As we saw in the guessing game tutorial in Chapter 2, we can declare a new variable with the same name as a previous variable, and the new variable shadows the previous variable. Rustaceans say that the first variable is shadowed by the second, which means that the second variable’s value is what we’ll see when we use the variable. We can shadow a variable by using the same variable’s name and repeating the use of the let keyword as follows:

Filename: src/main.rs

fn main() {
    let x = 5;

    let x = x + 1;

    let x = x * 2;

    println!("The value of x is: {}", x);
}
This program first binds x to a value of 5. Then it shadows x by repeating let x =, taking the original value and adding 1 so the value of x is then 6. The third let statement also shadows x, taking the previous value and multiplying it by 2 to give x a final value of 12. When you run this program, it will output the following:

$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/variables`
The value of x is: 12
This is different than marking a variable as mut, because unless we use the let keyword again, we’ll get a compile-time error if we accidentally try to reassign to this variable. We can perform a few transformations on a value but have the variable be immutable after those transformations have been completed.

The other difference between mut and shadowing is that because we’re effectively creating a new variable when we use the let keyword again, we can change the type of the value, but reuse the same name. For example, say our program asks a user to show how many spaces they want between some text by inputting space characters, but we really want to store that input as a number:

let spaces = "   ";
let spaces = spaces.len();
This construct is allowed because the first spaces variable is a string type, and the second spaces variable, which is a brand-new variable that happens to have the same name as the first one, is a number type. Shadowing thus spares us from having to come up with different names, like spaces_str and spaces_num; instead, we can reuse the simpler spaces name. However, if we try to use mut for this, as shown here, we’ll get a compile-time error:

let mut spaces = "   ";
spaces = spaces.len();
The error says we’re not allowed to mutate a variable’s type:

error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected &str, found usize
  |
  = note: expected type `&str`
             found type `usize`
Now that we’ve explored how variables work, let’s look at more data types they can have.
