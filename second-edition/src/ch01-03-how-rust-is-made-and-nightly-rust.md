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
Si una nueva funcionalidad está bajo desarrollo activo, it lands on
`master`, and therefore, in nightly, but behind a *feature flag*. If you, as a
user, wish to try out the work-in-progress feature, you can, but you must be
using a nightly release of Rust and annotate your source code with the
appropriate flag to opt in.

If you’re using a beta or stable release of Rust, you can’t use any feature
flags. This is the key that allows us to get practical use with new features
before we declare them stable forever. Those who wish to opt into the bleeding
edge can do so, and those who want a rock-solid experience can stick with
stable and know that their code won’t break. Stability without stagnation.

This book only contains information about stable features, as in-progress
features are still changing, and surely they’ll be different between when this
book was written and when they get enabled in stable builds. You can find
documentation for nightly-only features online.


### Rustup and the Role of Rust Nightly

Rustup makes it easy to change between different release channels of Rust, on a
global or per-project basis. By default, you’ll have stable Rust installed. To
install nightly, for example:

```text
$ rustup install nightly
```

You can see all of the *toolchains* (releases of Rust and associated
components) you have installed with `rustup` as well. Here’s an example on one
of your authors’ computers:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

As you can see, the stable toolchain is the default. Most Rust users use stable
most of the time. You might want to use stable most of the time, but use
nightly on a specific project, because you care about a cutting-edge feature.
To do so, you can use `rustup override` in that project’s directory to set the
nightly toolchain as the one `rustup` should use when you’re in that directory:

```text
$ cd ~/projects/needs-nightly
$ rustup override add nightly
```

Now, every time you call `rustc` or `cargo` inside of
*~/projects/needs-nightly*, `rustup` will make sure that you are using nightly
Rust, rather than your default of stable Rust. This comes in handy when you
have a lot of Rust projects!

### The RFC Process and Teams

So how do you learn about these new features? Rust’s development model follows
a *Request For Comments (RFC) process*. If you’d like an improvement in Rust,
you can write up a proposal, called an RFC.

Anyone can write RFCs to improve Rust, and the proposals are reviewed and
discussed by the Rust team, which is comprised of many topic subteams. There’s
a full list of the teams [on Rust’s
website](https://www.rust-lang.org/en-US/team.html), which includes teams for
each area of the project: language design, compiler implementation,
infrastructure, documentation, and more. The appropriate team reads the
proposal and the comments, writes some comments of their own, and eventually,
there’s consensus to accept or reject the feature.

If the feature is accepted, an issue is opened on the Rust repository, and
someone can implement it. The person who implements it very well may not be the
person who proposed the feature in the first place! When the implementation is
ready, it lands on the `master` branch behind a feature gate, as we discussed
in the “Unstable Features” section.

After some time, once Rust developers who use nightly releases have been able
to try out the new feature, team members will discuss the feature, how it’s
worked out on nightly, and decide if it should make it into stable Rust or not.
If the decision is to move forward, the feature gate is removed, and the
feature is now considered stable! It rides the trains into a new stable release
of Rust.

## Summary

You’re already off to a great start on your Rust journey! In this chapter,
you’ve:

* Learned what makes Rust unique
* Installed the latest stable version of Rust
* Written a “Hello, world!” program using both `rustc` directly and using
  the conventions of `cargo`
* Found out about how Rust is developed

This is a great time to build a more substantial program, to get used to
reading and writing Rust code. In the next chapter, we’ll build a guessing game
program. If you’d rather start by learning about how common programming
concepts work in Rust, see Chapter 3.

