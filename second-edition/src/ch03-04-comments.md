## Comentarios

Todos los programadores se esfuerzan en hacer sus códigos fáciles de entender,
pero a veces la explicación extra está justificada. En estos casos, los
programadores dejan notas, o *comentarios*, en sus códigos fuentes que podría
ser útil para las personas que los lean y será ignorado por el compilador.

Aquí tenemos un comentario simple:

```rust
// Hello, world.
```

En Rust, los comentarios deben de comenzar con dos slashes (barras diagonales)
y continuar hasta el final de la línea. Para los comentarios que sean más largos
que una línea, usted necesitará incluir `//` en cada línea, de esta manera:

```rust
// ¡Lo que estamos haciendo aquí es complicado, es tan largo que necesitamos
// múltiples líneas de comentarios para hacerlo! Esperamos que este comentario
// explique lo que está pasando.
```

Los comentarios también se pueden colocar al final de líneas que contengan código:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let lucky_number = 7; // Hoy me siento con suerte.
}
```

Pero usted los verá, más seguido, siendo usados de esta forma, con el comentario
en una línea separada encima del código al cual se está refiriendo:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    // Hoy me siento con suerte.
    let lucky_number = 7;
}
```

Rust también tiene otro tipo de comentarios, los comentarios de documentación,
los cuales vamos a discutir en el Capítulo 14.
