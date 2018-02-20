## Flujo de control conciso con `if let`

La sintaxis `if let` te permite combinar `if` y `let` de una manera menos detallada
para manejar valores que coinciden con un patrón e ignora el resto. Considera el programa
en el Listado 6-6 que coincide con un valor `Option<u8>` pero solo quiere ejecutar
código si el valor es tres:


```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```


<span class="caption">Listado 6.6: Un `match` que solo se preocupa de ejecutar
código cuando el valor es `Some(3)`</span>

Queremos hacer algo con la coincidencia `Some(3)` pero no hacemos nada con ningún
otro valor `Some<u8>` o el valor `None`. Para satisfacer la expresión `match`, tenemos
que agregar `_ => ()` después de procesar solo una variante, que es mucho código
para agregar.

En su lugar, podemos escribir esto de una manera más corta usando `if let`. El siguiente
código se comporta igual que el `match` en el Listado 6-6:

```rust
# let some_u8_value = Some(0u8);
if let Some(3) = some_u8_value {
    println!("three");
}
```

`if let` toma un patrón y una expresión separada por un `=`. Esto trabaja de la
manera que un `match`, donde la expresión se asigna al `match` y el patrón
es su primer brazo.

El uso de `if let` significa que tienes menos para escribir, menos indexáción, y
menos código repetido. Sin embargo, hemos perdido la comprobación exhaustiva que `match`
ejecuta. Eligiendo entre `match` e `if let` depende de que estes haciendo en
tu situación particular y si gana precisión es un intercambio por perder una 
comprobación exhaustiva.

En otras palabras, puedes pensar en `if let` como sintaxis dulce para un `match` que
ejecuta código cuanto el valor coincide con un patrón y luego ignora todos los otros valores.

Podemos incluir un `else` con un `if let`. El código de bloque que va con el
`else` es el mismo bloque de código que podría ir con el caso `_` en la expresión
`match` que es equivalente al `if let` y `else`. Recuerda la definición de enumeración
`Coin` en el Listado 6-4, donde la variante `Quarter` también tenía un valor
`UsState`. Si quisieramos contar todas las monedas que no son cuartos vemos también
mientras anunciamos el estado de los cuartos, podríamos hacer eso con una expresión `match`
como esta:

```rust
# #[derive(Debug)]
# enum UsState {
#    Alabama,
#    Alaska,
# }
#
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter(UsState),
# }
# let coin = Coin::Penny;
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}
```

O podríamos usar una expresión `if let` y `else` como esta:

```rust
# #[derive(Debug)]
# enum UsState {
#    Alabama,
#    Alaska,
# }
#
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter(UsState),
# }
# let coin = Coin::Penny;
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

Si tienes una situación en la cual tu programa tiene una lógica que es muy verbosa 
para expresar usando un `match`, recuerda que `if let` está en tu caja de herramientas
Rust también.

## Resúmen

Ahora hemos cubierto como usar enumeraciones para crear tipos personalizados
que pueden ser uno de un conjunto de valores enumerados. Hemos mostrado como el tipo
de biblioteca estándar `Option<T>` te ayuda el sistema de tipos para prevenir errores. Cuando
los valoesr de enumeración tiene datos dentro de ellos, puedes usar `match` o `if let` para extraer
y usar esos valores, dependiendo de cuantos casos necesites manejar.

Tus programas de Rust pueden expresar conceptos ahora en tu dominio usando estructuras y
enumeraciones. Creando tipos personalizados para usar en tu API garantiza la seguridad del
equipo: el compilador se asegurará de que tus funciones solo obtengan valores del tipo que cada
función espera.

Con el fin de proveer una API bien organizada a tus usuarios que sea clara
para usar y solo muestre exactamente lo que tus usuarios necesitarán, vayamos ahora
a los módulos de Rust.
