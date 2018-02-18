## Cómo está Hecho Rust y "Rust Nocturno"

Antes de indagar en el lenguaje, quisiéramos terminar el capítulo introductorio
hablando de cómo está hecho Rust, y cómo eso le afecta a usted como
desarrollador de Rust. Nosotros mencionamos en la sección de "Instalación"
que las salidas de este libro fueron generadas en la versión estable
1.21.0 de Rust, pero cualquier ejemplo que compile debería de continuar
compilando en cualquier versión estable de Rust futura. ¡Esta sección está
para explicar cómo aseguramos que eso es cierto!

### Estabilidad Sin Estancamiento

Como lenguaje, a Rust le importa *mucho* la estabilidad de su código. Queremos
que Rust sea una base sólida como roca donde usted pueda construir, y si las
cosas cambian constantemente, eso sería imposible. Al mismo tiempo, si no
podemos experimentar con algunas funcionalidades nuevas, no podríamos encontrar
fallas importantes hasta despues de su lanzamiento, cuando ya no podemos cambiar
nada.

Nuestra solución as este problema es algo que llamamos "estabilidad sin
estancamiento" y es la forma en la que podemos cambiar y mejorar Rust mientras
nos aseguramos de que usar Rust se mantenga como algo bueno, estable y aburrido.

Nuestro principio guía para los lanzamientos de Rust es: nunca deberás temerle
a actualizar a una versión estable de Rust. Cada actualización debería ser sin
dolor. Al mismo tiempo, la actualización debería de traerle nuevas funcionalidades,
menos errores, y mejores tiempos de compilación.

### ¡Choo, Choo! Canales de Lanzamientos y Abordando los Trenes

El desarrollo de Rust opera en una *programación de tren*. Eso significa que
todo desarrollo se hace en la rama `master` del repositorio Rust. Los lanzamientos
siguen un modelo de tren de lanzamiento de software, el cual ha sido usado por
Cisco IOS y otros proyectos de software. Hay tres *canales de lanzamientos*
para Rust:

* Nocturno
* Beta
* Estable

La mayoría de los desarrolladores de Rust principalmente usan el canal estable,
pero aquellos que quieran probar funcionalidades experimentales nuevas pueden
usar el canal nocturno o el beta.

Aquí se muestra un ejemplo de cómo funciona el proceso de desarrollo y lanzamiento:
Asumamos que el equipo Rust está trabajando en el lanzamiento de la versión de Rust
1.5. Ese lanzamiento ocurrió en el Diciembre del 2015, pero nos proveerá con
números de versión reales. Una nueva funcionalidad es añadida a Rust: una nueva
encomienda termina en la rama `master`. Cada noche, una nueva versión nocturna
de Rust es producida. Cada día es un día de lanzamiento de versión, y estos
lanzamientos de versiones son creados por nuestra infraestructura de lanzamientos de
versiones automáticamente. Así que a medida que pasa el tiempo, nuestros lanzamientos
de versiones se ven así, una vez por noche:

```text
nightly: * - - * - - *
```

Cada seis semanas, ¡Es hora de preparar una nueva versión! La rama `beta` del
repositorio de Rust se subdivide de la rama `master` usada por nocturno. Ahora,
hay dos lanzamientos de versiones:

```text
nightly: * - - * - - *
                     |
beta:                *
```

La mayoría de los usuarios de Rust no usan las versiones beta activamente, pero
prueban la beta en sus sistemas CI para ayudar a Rust a encontrar regresiones
posibles. Mientras tanto, aún hay un lanzamiento nocturno cada noche:
```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Digamos que se encontró una regresión. ¡Qué bueno que tuvimos algo de tiempo para
probar la versión beta antes de que la regresión se metiera en la versión estable!


```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Seis semanas despues de que la primera versión beta es creada, ¡Es hora de una
versión estable! la rama `stable` se produce de la rama `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

¡Hurrah! ¡Rust 1.5 está listo! Sin embargo, olvidamos una cosa: debido a que las
seis semanas se han ido, también necesitamos una nueva versión beta para la
*próxima* versión de Rust, 1.6. Así que después de que la rama `stable` resulte de
`beta`, la próxima versión de `beta` resulta de `nightly` otra vez:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Esto es llamado el "modelo de tren" porque cada seis semanas, una versión
"sale de la estación", pero aún tiene que dar un viaje a través del canal beta
antes de llegar como un lanzamiento de versión estable.

Rust hace lanzamiento de versión cada seis semanas, de forma programada. Si
usted sabe la fecha de algún lanzamiento de versión de Rust, podrá saber la
fecha del siguiente lanzamiento de versión: es seis semanas después. Un buen
aspecto de tener lanzamientos de versión programados cada seis semanas es que
el próximo tren viene pronto. Si una funcionalidad no está en una versión en
particular, no habrá necesidad de preocuparse: ¡Otra versión vendrá pronto!
Esto ayuda a reducir la presión de colocar funcionalidades no terminadas justo
antes de la fecha de lanzamiento de la versión.

Gracias a este proceso, siempre podrá revisar la próxima versión de Rust y
verificar por usted mismo que es fácil de actualizar: si una versión beta
no funciona como se esperaba, ¡Puede reportarla al equipo para que la arreglen
antes de que el próximo lanzamiento ocurra! Una fractura es relativamente rara
que suceda en una versión beta, pero `rustc` aún es una pieza de software, y
los errores aún existen.

### Funcionalidades Inestables

Hay otro beneficio con este modelo de lanzamiento de versiones: funcionalidades
inestables. Rust usa una técnica llamada "banderas de funcionalidades" para
determinar qué funcionalidades están presentes en una versión en particular.
Si una nueva funcionalidad está bajo desarrollo activo, terminará en `master`,
y por lo tanto, en nocturno, pero antes de una *bandera de funcionalidad*. Si
usted, como usuario, desea probar la funcionalidad que está en proceso, usted
puede, pero necesitará estar usando una versión nocturna de Rust y anotar su
código fuente con la bandera apropiada para ingresar.

Si usted está usando una versión beta o estable de Rust, no puede usar ninguna
bandera de funcionalidad. Esta es la clave que nos permite tener uso práctico
con nuevas funcionalidades antes de declararlas como estables para siempre.
Aquellos que deseen ingresar a las versiones más nuevas pueden hacerlo, y
aquellos que quieran una experiencia sólida como-roca pueden quedarse con la
versión estable sabiendo que sus códigos jamás fallarán. Estabilidad sin
Estancamiento.

Este libro solo contiene información sobre funcionalidades estables, ya que las
funcionalidades en proceso aún están cambiando, y seguramente serán diferentes
entre el momento en el que se escriba este libro y el momento en el que llegarán
a la versión estable. Usted puede encontrar documentación acerca de sólo
funcionalidades nocturnas en línea.


### Rustup y el Rol de Rust Nocturno

Rustup hace fácil cambiar entre diferentes canales de versiones de Rust,
en un fundamento global o por-proyecto. Por defecto, usted tendrá la versión
de Rust estable instalada. Para instalar la versión nocturna, por ejemplo:

```text
$ rustup install nightly
```

Puede ver todas las *toolchains* (versiones de Rust y componentes asociados)
que tenga instaladas con `rustup` también. Aquí hay un ejemplo de uno de los
computadores de su autor:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Como puede ver, la toolchain estable es la que está por defecto. La mayoría de
los usuarios de Rust usan la versión estable la mayoría del tiempo. Usted
querría usar la versión estable la mayoría del tiempo, pero usar la versión
nocturna en un proyecto en específico, porque le importan las funcionalidades
recién hechas. Para hacerlo, puede usar `rustup override` en ese directorio del
proyecto para establecer la toolchain de la versión nocturna como la que `rustup`
debería de usar cuando esté en ese directorio:

```text
$ cd ~/projects/needs-nightly
$ rustup override add nightly
```

Ahora, cada vez que llame `rustc` o `cargo` dentro de *~/projects/needs-nightly*,
`rustup` se encargará de que usted esté usando la versión nocturna de Rust, en
vez de su versión estable por defecto de Rust. ¡Esto es útil cuando quiere tener
muchos proyectos de Rust!

### El Proceso RFC y Los Equipos

Así que ¿Cómo aprende usted de estas nuevas funcionalidades? El modelo de desarrollo
de Rust sigue un *Request For Comments (RFC)(Proceso de Pedir Por Comentarios)*.
Si quiere una mejora en Rust, puede escribir una propuesta, llamada RFC.

Cualquiera puede escribir RFCs para mejorar Rust, y las propuestas son revisadas
y discutidas por el equipo de Rust, el cual está comprendido de sub-equipos
de muchos temas. Hay una lista completa de los equipos [en Rust’s website]
(https://www.rust-lang.org/en-US/team.html), la cual incluye equipos para cada
area del proyecto: diseño del lenguaje, implementación del compilador,
infraestructura, documentación, y más. Los equipos apropiados leen las propuestas
y los comentarios, escriben algunos comentarios ellos mismo, y eventualmente,
hay un consenso para aceptar o negar la funcionalidad.

Si hay una funcionalidad aceptada, una cuestión se abre en el repositorio de Rust,
y alguien la puede implementar. ¡La persona que la implementa puede fácilmente no
ser la persona que propuso la funcionalidad en primer lugar! Cuando la implementación
está lista, termina en la rama `master` detrás de una entrada de funcionalidades,
como lo discutimos en la sección de "Funcionalidades Inestables".

## Resumen

¡Ya está listo para un buen comienzo en su viaje con Rust! En este capítulo usted ha:

* Aprendido qué hace a Rust único
* Instalado las últimas versiones estables de Rust
* Escrito un programa "Hello, world!" usando `rustc` directamente y usando las
convenciones de `cargo`
* Encontrado sobre el cómo Rust está desarrollado

Este es un buen momento para escribir un programa importante, para acostumbrarse
a leer y escribir códigos Rust. En el próximo capítulo, crearemos un programa de
un juego de adivinanzas. Si prefiere comenzar aprendiendo sobre cómo los conceptos
de programación común funcionan en Rust, vea el Capítulo 3.

