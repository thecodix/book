<! - Este es un capítulo largo! Estaba considerando dividirlo, y
si es así, donde --- la única solución que pude encontrar fue dividirlo en
cinco temas principales: Inseguro, Vidas, Rasgos, Tipos, Funciones y
Cierres Sin embargo, no estoy convencido de que sea ideal, así que pensé que podríamos
incluir un ToC en la parte superior de este capítulo para que el lector pueda usarlo como
referencia cuando encuentren algo que no puedan descifrar. Qué es lo que tú
piensas? ->
<! - Un ToC para hacer que este capítulo sea más fácil de usar como referencia
suena bien, Sin embargo, ¿sería redundante el ToC este al comienzo de todo el libro?
¿O este ToC sería más detallado que el comienzo del libro? Verdad solo agregue 
números de página a los puntos de viñeta después del primer párrafo? Tenemos 
curiosidad acerca de la implementación :) / Carol ->

# Características avanzadas

¡Hemos recorrido un largo camino! Por ahora, has aprendido el 99% de las
cosas que necesitarás para saber escribir Rust. Antes de hacer un proyecto
más en el Capítulo 20, hablemos sobre algunas cosas con las que te puedes
encontrar en ese último 1% del tiempo. Siéntase libre de usar este capítulo
como referencia para cuando te encuentras con algo desconocid; las características
que aprenderá a usar aquí son útiles en muy específicas situaciones, 
no queremos dejar estas características afuera, porque no las encontrará
usted mismo muy a menudo tratando de buscarlas.

En este capítulo, vamos a cubrir:

* Unsafe Rust (Rust inseguro): para cuando tiene que optar por salir de algunas de 
las garantías de Rust y hacerse responsable de mantener las garantías en su lugar 
* Advanced Lifetimes (Vida útil avanzada): sintaxis para situaciones complejas de por vida 
* Advanced Traits (Rasgos avanzados): Tipos asociados, parámetros de tipo por defecto, 
completamente calificado sintaxis, supertraits y el nuevo patrón de tipo en relación con los rasgos 
* Advanced Types (Tipos avanzados): algunos más sobre el patrón newtype, tipo alias, 
el   Tipo "never" y tipos de tamaño dinámico 
* Advanced Functions and Closures (Funciones y cierres avanzados): indicadores de función y cierres de retorno

¡Es una colección de características de Rust con algo para todos! ¡Vamos a sumergirnos!
