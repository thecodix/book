# Un proyecto de E/S: Construcción de un programa de línea de comandos

Este capítulo es una recapitulación de las muchas habilidades que ha aprendido hasta ahora y 
una exploración de algunas características más de la biblioteca estándar. Construiremos una herramienta
de línea de comandos que interactúa con la entrada/salida de la línea de comandos y archivos para practicar
algunos de los conceptos de Rust que ahora tienes bajo tu cinturón.

La velocidad de Rust, la seguridad, la salida *single binary* y el soporte multiplataforma lo convierten 
en un lenguaje ideal para crear herramientas de línea de comandos, por lo que para nuestro proyecto, crearemos
nuestra propia versión de la clásica herramienta de línea de comandos `grep` (**g**lobally 
search a **r**egular **e**xpression and **p**rint) (buscar globalmente una expresión regular e imprimir). En el caso
de uso más sencillo, `grep` busca un archivo especificado para una cadena específica. Para ello, `grep` toma 
como argumentos un nombre de archivo y una cadena, y luego lee el archivo y busca las líneas
en ese archivo que contienen el argumento de la cadena. Luego imprime esas líneas.

A lo largo del camino, mostraremos cómo hacer que nuestra herramienta de línea de comandos utilice 
las características de la terminal que muchas herramientas de línea de comandos utilizan. Leeremos el valor
de una variable de entorno para permitir al usuario configurar el comportamiento de nuestra herramienta.
También imprimiremos en la consola de errores estándar (`stderr`) en lugar de la salida 
estándar (`stdout`), de modo que, por ejemplo, el usuario puede redirigir la salida correcta
a un archivo mientras sigue viendo mensajes de error en pantalla.

Un miembro de la comunidad de Rust, Andrew Gallant, ya ha creado una versión completa
y muy rápida de `grep`, llamada `ripgrep`. En comparación, nuestra versión 
de `grep` será bastante simple, pero este capítulo te dará algunos de los 
conocimientos básicos que necesitas para entender un proyecto del mundo real como
`ripgrep`.

Nuestro proyecto `grep` combinará una serie de conceptos que has aprendido hasta ahora:

* Código de organización (utilizando lo aprendido en los módulos, Capítulo 7)
* Usando vectores y cadenas (colecciones, Capítulo 8)
* Manejo de errores (Capítulo 9)
* Utilizar rasgos y vidas útiles cuando sea apropiado (Capítulo 10)
* Pruebas de escritura (Capítulo 11)

También presentaremos brevemente los cierres, iteradores y objetos característicos, que
en los capítulos 13 y 17 se tratarán en detalle.
