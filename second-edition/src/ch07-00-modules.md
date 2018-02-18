# Uso de módulos para reutilizar y organizar código

Cuando comiences a escribir programas en Rust, tu código podría vivir únicamente en la función `main`. A medida que su código crezca, eventualmente moverá la funcionalidad a otras funciones para su reutilización y una mejor organización. Al dividir el código en fragmentos más pequeños, cada fragmento es más fácil de entender por sí mismo. Pero, ¿qué sucede si tienes demasiadas funciones? Rust tiene un sistema modular que permite la reutilización de código de manera organizada.

De la misma manera que extraes líneas de código en una función, puedes extraer funciones (y otro código, como estructuras y enumeraciones) en diferentes módulos. Un *módulo* es un espacio de nombres que contiene definiciones de funciones o tipos, y usted puede elegir si esas definiciones son visibles fuera de su módulo (público) o no (privado). Aquí hay una descripción general de cómo funcionan los módulos:

* La palabra clave `mod` declara un nuevo módulo. El código dentro del módulo aparece inmediatamente después de esta declaración entre llaves o en otro archivo.
* Por defecto, las funciones, tipos, constantes y módulos son privados. La palabra clave `pub` hace que un elemento sea público y, por lo tanto, visible fuera de su espacio de nombres.
* La palabra clave `use` lleva los módulos, o las definiciones dentro de los módulos, al alcance, por lo que es más fácil referirse a ellos.

Veremos cada una de estas partes para ver cómo encajan en el todo.
