# Escribiendo Pruebas Automatizadas

En su ensayo de 1972, "El Programador Humilde", Edsger W. Dijkstra afirmó que 
"La prueba de programas puede ser una forma muy efectiva de mostrar la presencia de bugs,
pero es irremediablemente inefectiva para mostrar su ausencia". ¡Esto no quiere decir que
no deberíamos intentar hacer tantas pruebas como podamos! La exactitud en nuestros programas
depende de la medida en la que nuestro código haga lo nosotros queremos que haga. Rust es un lenguaje de
programación diseñado con una gran preocupación por de la exactitud de los
programas, pero la exactitud es compleja y difícil de probar. El sistema de tipos de Rust
lleva una gran parte de esta carga, pero el sistema de tipos no puede atrapar todo
tipo de inexactitud. Debido a esto, Rust incluye soporte para escribir pruebas
de software automatizadas incorporadas en el lenguaje.

Como ejemplo, digamos que escribimos una función llamada `add_two` que añade dos
a cualquier número que pase por ella. La firma de esta función acepta un integral de
parámetro y regresa un integral como resultado. Cuando implementamos y
compilamos esta función, Rust realiza todos los chequeos de tipos y de préstamos que
has aprendido hasta ahora para asegurarse de que, por ejemplo, no estemos pasando un
valor del tipo `string` o una referencia inválida a esta función. Pero Rust *no puede* verificar
si esta función hará precisamente lo que se quiere que haga, que es regresar el 
parámetro más dos en lugar de, por decir, el parámetro más 10 ¡o el parámetro
menos 50! Aquí es donde entran las pruebas.

Podemos escribir pruebas que evalúen, por ejemplo, si cuando pasamos `3` a la
función `add_two`, el valor que regrese sea `5`. Podemos ejecutar estas pruebas cada vez
que hagamos cambios en nuestro código para asegurarnos que cualquier comportamiento correcto no haya
cambiado.

Hacer pruebas es una habilidad compleja: aunque no podemos abarcar cada detalle acerca de cómo
escribir pruebas efectivas en un capítulo, discutiremos las mecánicas de las instalaciones de prueba
de Rust. Vamos a hablar acerca de las anotaciones y macros disponibles al
escribir las pruebas, el comportamiento predeterminado y las opciones que se ofrecen para correr tus
pruebas, y cómo organizar pruebas por pruebas de unidad y pruebas de integración.
