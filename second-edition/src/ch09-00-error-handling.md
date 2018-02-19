# Manejo de errores

El compromiso de Rust con la confiabilidad se extiende al manejo de errores. Los errores son un hecho de la vida en el 
software, por lo que Rust tiene una serie de características para manejar situaciones en las que algo sale mal. En
muchos casos, Rust requiere que reconozcas la posibilidad de que se produzca un error y actúes antes de compilar el 
código. Este requisito hace que su programa sea más robusto al garantizar que descubrirá los errores y los manejará 
adecuadamente antes de implementar su código en producción.

Rust agrupa los errores en dos categorías principales: *recuperables* y *errores irrecuperables*. Los errores
recuperables son situaciones en las que es razonable informar el problema al usuario y volver a intentar la operación,
como un error de archivo no encontrado. Los errores irrecuperables son siempre síntomas de errores, como intentar 
acceder a una ubicación más allá del final de una matriz.

La mayoría de los idiomas no distinguen entre estos dos tipos de errores y manejan ambos de la misma manera usando
mecanismos como excepciones. Rust no tiene excepciones. En cambio, tiene el tipo `Result<T,E>` para los errores
recuperables y la macro `panic!` Que detiene la ejecución cuando detecta errores irrecuperables. Este capítulo cubre
la invocación de `panic!` Primero y luego habla acerca de cómo devolver los valores `Result<T,E>`. Además, exploraremos consideraciones a tener en cuenta al decidir si intentamos recuperar un error o detener la ejecución.
