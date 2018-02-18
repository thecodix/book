# Usando Módulos para Reutilizar y Organizar el Código

Cuando empiezas a escribir programas en Rust, tu código podría vivir únicamente en la
función `main`. A medida que tu código crezca, con el tiempo trasladarás la funcionalidad a otras
funciones para su reutilización y una mejor organización. Dividiendo tu código en 
trozos más pequeños, cada trozo es más fácil de entender por sí solo. Pero, ¿qué pasa 
si tienes demasiadas funciones? Rust tiene un sistema modular que permite la reutilización
del código de forma organizada.

Del mismo modo que extraes líneas de código en una función, puedes extraer
funciones (y otros códigos, como estructuras y enums) en diferentes módulos. Un 
*módulo* es un espacio de nombres que contiene definiciones de funciones o tipos, y
puedes elegir si esas definiciones son visibles fuera de su módulo
(público) o no (privado). A continuación se muestra un resumen de cómo funcionan los módulos:

* La palabra clave `mod` declara un nuevo módulo. El código dentro del módulo aparece
  inmediatamente después de esta declaración entre llaves
  o en otro archivo.
* Por defecto, las funciones, tipos, constantes y módulos son privados. La palabra clave
  `pub` hace que un elemento sea público y por lo tanto visible fuera de su espacio de nombre.
* La palabra clave `use` trae módulos, o las definiciones dentro de los módulos, al 
  alcance para que sea más fácil referirse a ellos.

Miraremos cada una de estas partes para ver cómo encajan en lo global.
