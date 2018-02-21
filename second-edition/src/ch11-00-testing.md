# Programando pruebas automatizadas

En su ensayo de 1972, “The Humble Programmer,” Edsger W. Dijkstra dijo que
“Programar pruebas puede ser una manera muy efectiva de mostrar la presencia de "bugs", pero
es irremediablemente inadecuado para mostrar su ausencia.” ¡Eso no significa que nosotros
no deberíamos intentar hacer pruebas tanto como podamos! La corrección en nuestros programas es el
grado en el que nuestro código hace para lo que fue planeado. Rust es un lenguaje
de programación diseñado con un alto grado de preocupación por la exactitud de los
programas, pero la exactitud es compleja y difícil de probar. El sistema de tipo de Rust
asume una gran parte de esta carga, pero el sistema de tipo no puede atrapar todos
los tipos de errores. De por sí, Rust soporta la escritura automática de
pruebas de software dentro del lenguaje.

Por ejemplo, digamos que escribimos una función llamada `add_two` que añade dos a
cualquier número que es puesto. La firma de esta función acepta un entero
como un parámetro y retorna un entero como resultado. Cuando implementados y
compilamos esta función, Rust hace topos los tipos de revisiones y revisiones prestadas que
has aprendido hasta el momento para asegurar eso, por ejemplo, no estamos pasando un
valor `String` o una referencia inválida a esta función. Pero Rust *can’t* revisar
eso, esta función hará precisamente lo que teníamos planeado, lo cual es retornar el
parámetro más dos en lugar de, digamos, ¡el parámetro más 10 o el parámetro
menos 50! Ahí es cuando las pruebas entran.

Podemos programar pruebas que afirmen, por ejemplo, que cuando ponemos `3` a la
función `add_two`, el valor retornado sea `5`. Podemos ejecutar estas pruebas cuando quiera que
hagamos cambios a nuestro código para asegurarnos que cualquier comportamiento correcto no ha
cambiado.

Hacer pruebas es una habilidad compleja: aunque no podamos cubrir cada detalle sobre cómo
programar buenas pruebas en un capítulo, discutiremos las mecánicas de las instalanciones de prueba
de Rust. Hablaremos sobre las anotaciones y macros disponibles para ti cuando
programes tus pruebas, el comportamiento por defecto y las opciones proporcionadas para ejecutar tus
pruebas, y cómo organizar pruebas en pruebas de unidad y pruebas de integración.
