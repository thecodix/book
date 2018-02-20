# Patrones y Matching

Los patrones son una sintaxis especial de Rust para que coincidan con la estructura de
tipos, tanto complejos como simples. El uso de patrones en conjunto con expresiones 
`match` y otras construcciones te da más control sobre el flujo de control 
de un programa. Un patrón se compone de alguna de estas combinaciones:

- literales
- arrays desestructurados, enums, structuras, o tuplas
- variables
- comodines
- marcadores de posición

Estas piezas describen la forma de los datos con los que estamos trabajando, que luego
comparamos con los valores para determinar si nuestro programa tiene los datos correctos para
seguir ejecutando un bit de código en particular.

<!-- I think we need a concise description of what we use patterns for here,
what they provide the programmer. Hopefully you can see what I've trying to do,
above! But I think you'll agree it's not quite right, can you have a whack, try
to give the reader that explanation? -->
<!-- We tweaked the wording a bit, how's this? /Carol -->

Para usar un patrón lo comparamos con algún valor. Si el patrón coincide con nuestro valor, 
usamos las partes del valor en nuestro código. Recuerda nuestras expresiones `match` del Capítulo
6 que usaban patrones como una máquina de clasificación de monedas. Si el valor se ajusta a la forma
del patrón, podemos usar las piezas nombradas. Si no lo hace, el código asociado
con el patrón no se ejecutará.

Este capítulo es una referencia sobre todas las cosas relacionadas con los patrones. Cubriremos los 
lugares válidos para usar patrones, la diferencia entre los patrones *refutables* e
*irrefutables*, y los diferentes tipos de sintaxis de patrones que 
podrías ver. Al final, verás cómo usar patrones para crear un código 
potente y claro.
