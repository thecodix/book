## ¡Hola, Mundo!

Ahora que tiene Rust instalado, comencemos escribiendo su primer programa Rust.
Es una tradición que al aprender un nuevo lenguaje se escriba un pequeño programa
para imprimir el texto "Hello, world!" (¡Hola, mundo!) por pantalla, y en esta
sección, seguiremos esa tradición.

> Nota: Este libro asume familiarización básica con la línea de comandos. Rust
> no hace demandas sobre su edición, herramientas usadas, o en donde usted guarde su
> código, así que si prefiere un IDE en vez de la línea de comandos, siéntase libre de
> usar su IDE favorito. Muchos IDEs ahora tienen cierto grado de soporte para Rust;
> revise la documentación del IDE para más detalles. Permitiendo gran soporte de
> IDE ha sido de gran atención recientemente para el equipo de Rust, y el progreso
> se ha llevado a cabo rápidamente!

### Creando un Directorio de Proyecto

Primero, crée un directorio donde colocar su código Rust. A Rust no le importa
donde usted guarde su código, pero para este libro, sugerimos crear un directorio
*projects* (proyectos) en su directorio hogar y mantener sus proyectos ahí. Abra
un terminal e introduzca los siguientes comandos para crear un directorio y un
directorio dentro de este para el proyecto “Hello, world!”:

Linux y Mac:

```text
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

CMD de Windows:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

PowerShell de Windows:

```powershell
> mkdir $env:USERPROFILE\projects
> cd $env:USERPROFILE\projects
> mkdir hello_world
> cd hello_world
```

### Escribiendo y Ejecutando un Programa de Rust

Luego, cree un nuevo archivo fuente y llámelo *main.rs*. Los archivos Rust siempre
terminan con la extensión *.rs*. Si está usando más de una palabra para el nombre
del archivo, use un piso para separarlas. Por ejemplo, usaría *hello_world.rs* en
vez de *helloworlds.rs*.

Ahora abra el archivo *main.rs* que acaba de crear, y escriba el código mostrado en
Listado 1-1:


<span class="filename">Filename: main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

<span class="caption">Listing 1-1: A program that prints “Hello, world!”</span>

Guarde el archivo, y vuelva a la ventana del terminal. En Linux o en OSX, introduzca
los siguientes comandos:


```text
$ rustc main.rs
$ ./main
Hello, world!
```

En Windows, ejecute `.\main.exe` en vez de `./main`.

```powershell
> rustc main.rs
> .\main.exe
Hello, world!
```

Sin importar su sistema
operativo, verá la cadena `Hello, world!` impresa en el terminal. Si eso ve,
¡Felicidades! Usted ha oficialmente escrito un programa en Rust. ¡Eso lo hace
un programador Rust! ¡Bienvenido!

### Anatomía de un Programa en Rust

Ahora, vamos a revisar lo que acaba de pasar en su programa "Hello, world!"
en detalle. Esta es la primera pieza del rompecabezas:


```rust
fn main() {

}
```

Estas líneas definen una *function* (función) en Rust. La función `main` es
especial: es la primera cosa que se ejecuta en cada programa Rust ejecutable.
La primera línea dice, "Estoy declarando una función llamada `main` que no
tiene parámetros y no retorna nada." Si hay parámetros, sus nombres irían
dentro de los paréntesis, `(` y `)`.

También note que el cuerpo de la función está dentro de llaves, `{` y `}`.
Rust requiere estas alrededor de todos los cuerpos de las funciones. Es
considerado un buen estilo poner la abertura de las llaves en la misma línea
de la declaración de la función, con un espacio de por medio


> Durante la escritura, un formateador automático, `rustfmt`, está en
> desarrollo. Si usted quiere apegarse a un estilo estándar a lo largo
> de los proyectos Rust, `rustfmt` es una herramienta que formateará
> su código con un estilo particular. El plan es eventualmente incluirlo
> con la distribución estándar de Rust, así como `rustc`, entonces dependiendo
> de cuándo usted lea este libro, !Podría ya haberlo instalado! Revise la
> documentación en línea para más detalles.

Dentro de la función `main`:, tenemos este código:

```rust
    println!("Hello, world!");
```

Esta línea hace todo el trabajo en este pequeño programa: ella imprime
por pantalla. Hay algunos detalles a notar aquí. El primero es que el
estilo de Rust es para indentar con cuatro espacios, no con un tab
(tabulador).

La segunda parte importante es `println!`. Esto llama a un *macro* de Rust,
que es cómo se hace la metaprogramación en Rust. Si llamara a una función,
se vería así: `println` (sin el `!`). Discutiremos los macros de Rust con
más detalle en Appendix E, pero por ahora sólo debe saber que cuando vea
un `!` significa que usted está llamando a un macro en vez de a una función
normal.


> ### Porqué `println!` es un Macro
>
> Hay múltiples razones por las cuales `println!` es un macro en vez de
> una función, y aún no hemos explicado Rust, así que no es exactamente
> obvio. Aquí están las razones:
>
> * La cadena pasada a `println!` puede tener especificadores de formateo
>   en ella, y esos se revisan en tiempo de compilación.
> * Las funciones Rust sólo pueden tener un número predeterminado de argumentos
>   pero `println!` (y los macros en general) pueden tomar un número variable.
> * Los especificadores de formateo pueden tener argumentos nombrados, a
>   diferencia de la funciones de Rust que no pueden tenerlos.
> * Toma implícitamente sus argumentos por referencia incluso cuando son pasados
>   por valor.
>
> Si nada de esto tiene sentido, no se preocupe. Luego cubriremos estos conceptos con
> más detalle.

Luego viene `"Hello, world!"` el cual es un *string* (cadena). Pasamos esta cadena
como argumento a `println!`, el cual imprime la cadena por pantalla. ¡Bastante
facil!

La línea termina con un punto y coma (`;`). El `;` indica que esta expresión
acabó, y que la próxima está lista para comenzar. La mayoría de las líneas del
código de Rust terminan con `;`.

### Compilar y Ejecutar Son Pasos Diferentes

En "Escribiendo y Ejecutando un Programa de Rust", mostramos cómo ejecutar
un programa recién creado. Ahora vamos a descomponer este proceso para examinar
cada paso.

Antes de ejecutar el programa de Rust, usted tiene que compilarlo. Puede usar
el compilador de Rust introduciendo el comando `rustc` y pasando el nombre de
su archivo fuente, así:


```text
$ rustc main.rs
```

Si usted viene de usar C o C++, notará que esto es similar a `gcc` o `clang`.
Luego de compilar con éxito, Rust debería de producir un ejecutable en binario.


En Linux, Mac, y PowerShell en Windows, puede ver el ejecutable introduciendo
el comando `ls` en su shell así:

```text
$ ls
main  main.rs
```

En CMD de Windows, usted introduciría:

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

Esto muestra que tenemos dos archivos: el código fuente, con la extensión *.rs*,
y el ejecutable (*main.exe* en Windows, *main* en los demás). Lo que queda por
hacer aquí es ejectuar el archivo *main* o *main.exe*, así:


```text
$ ./main  # or .\main.exe on Windows
```

Si su programa "Hello, wordl!" fuese *main.rs*, imprimiría `Hello, world!` en
su terminal.

Si usted viene de lenguajes dinámicos como Ruby, Python, o JavaScript, podría
no estar acostumbrado a que compilar y ejecutar sean dos pasos diferentes. Rust
es un lenguaje *compilado desde-antes*, lo que significa que usted puede compilar
un programa, dárselo a alguien más, y esa persona puede ejecutarlo aunque no
tenga Rust instalado. Si usted le da un archivo `.rb`, `.py`, o `.js` a otra
persona, por otro lado, necesitará tener una implementación de Ruby, Python, o
JavaScript instalada (respectivamente), pero usted sólo necesita un comando para
compilar y ejecutar su programa. En el diseño de lenguajes, todo es un balance.

Sólo con compilar con `rustc` está bien para programas simples, pero a medida que
su programa crece, usted querrá poder manejar todas las opciones que su proyecto
tenga y tener facilidad de compartir su código con otras personas y proyectos.
A continuación, le presentaremos una herramienta llamada Cargo, la cual le ayudará
a escribir programas en Rust del mundo real.

## ¡Hola, Cargo!


Cargo es el sistema de construcción de Rust y administrador de paquetes, y los Rustaceanos
usan Cargo para administrar sus proyectos de Rust ya que hace muchas tareas más
fáciles. Por ejemplo, Cargo se encarga de construir su código, descargar las librerías
de las cuales depende su código, y de construir esas librerías. Llamamos *dependencies*
(dependencias) a las librerías que su código necesita.

Los programas en Rust más simples, como el que hemos escrito hasta ahora, no tiene
ninguna dependencia, así que ahora, usted sólo usaría la parte de Cargo que se
encarga de construir su código. A medida que escribe códigos más complejos en
Rust, usted querrá añadir dependencias, y si empieza usando Cargo, eso será mucho
más fácil.

Como la gran, gran mayoría de proyectos en Rust usan Cargo, asumiremos que usted lo
estará usando por el resto del libro. Cargo viene instalado con Rust, si usted usó
los instaladores oficiales que fueron cubiertos en el capítulo de Instalación.
Si instaló Rust de alguna otra forma, puede comprobar si tiene Cargo instalado
escribiendo lo siguiente en su terminal:


```text
$ cargo --version
```

Si ve un número de versión, ¡Entonces perfecto! Si ve un error como `command not found`
(comando no encontrado), entonces tendrá que revisar la documentación para su
método de instalación para determinar cómo instalar Cargo por separado.

### Creando un Proyecto con Cargo

Vamos a crear un nuevo proyecto usando Cargo y ver cómo difiere con nuestro proyecto
en `hello_world`. Vuelva a su directorio de proyectos (o donde sea que decidió
almacenar su código):


Linux, Mac, y PowerShell:

```text
$ cd ~/projects
```

CMD para Windows:

```cmd
> cd \d "%USERPROFILE%\projects"
```

Y luego en cualquier sistema operativo ejecute:

```text
$ cargo new hello_cargo --bin
$ cd hello_cargo
```

Hemos pasado el argumento `--bin` a `cargo new` porque nuesto objetivo es hacer
una aplicación ejecutable, a diferencia de una librería. Los ejecutables son
archivos ejecutables binarios generalmente llamados solo *binarios*. Le hemos
dado el nombre `hello_cargo` a nuestro proyecto, y Cargo crea sus archivos
en un directorio con el mismo nombre en el cual podemos entrar.

Si observamos los archivos en el directorio *hello_cargo*, podemos ver que
Cargo ha generado dos archivos y un directorio para nosotros: un *Cargo.toml*
y un directorio *src* con un archivo *main.rs* adentro. También ha iniciado
un nuevo repositorio git en el directorio *hello_cargo* para nosotros, junto
con un archivo *.gitignore*. Git es un sistema de control de versiones común.
Usted puede cambiar `cargo new` para usar un sistema de control de versiones
diferentes, o no usar ningún sistema de control de versiones, usando la bandera
`--vcs`. Ejecute `cargo new --help` para ver las opciones disponibles.

Abra *Cargo.toml* en su editor de texto de preferencia. Debería verse igual
al código en Listado 1-2:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
```

<span class="caption">Listing 1-2: Contents of *Cargo.toml* generated by `cargo
new`</span>

Este archivo está en el formato [*TOML*][toml]<!-- ignore --> (Tom’s Obvious,
Minimal Language)(Formato Minimalista y obvio de Tom). TOML es usado como el
formato de configuración de Cargo.

[toml]: https://github.com/toml-lang/toml

La primera línea, `[package]`, es una cabecera de sección que indica que las
siguiente declaraciones están configurando un paquete. A medida que añadimos
más información a este archivo, añadiremos otras secciones.

Las tres líneas siguientes establecen los tres pedazos de configuración que Cargo
necesita ver para saber que debería de compilar su programa: su nombre, cuál
versión es, y quién lo escribió. Cargo obtiene su nombre y su información de
email de su ambiente. Si no es correcto, arréglelo y guarde el archivo.

La última línea, `[dependencies]`, es el comienzo de una sección para usted
para listar *crates* (cajas) (así llamamos a los paquetes de código de Rust) de
los cuales su proyecto dependerá para que Cargo sepa para descargarlos y
compilarlos también. No necesitaremos ningún otro crate en este proyecto, pero
si necesitaremos otros en el tutorial del juego de adivinanzas en el Capítulo 2.

Ahora observemos al *src/main.rs*

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

¡Cargo ha generado un "Hello World!" para usted, así como el que escribimos
en el Listado 1-1! Así que esa parte es igual. Las diferencias entre nuestro
proyecto anterior y el proyecto generado por Cargo que hemos visto hata ahora
son:

- Nuestro código va en el directorio *src*
- El nivel tope contiene un archivo de configuración *Cargo.toml*

Cargo espera que sus archivos fuente estén dentro del directorio *src* para
que el directorio del proyecto de nivel-tope sea solo para READMEs, información
de licencias, archivos de configuración, y cualquier otra cosa que no esté
relacionada con su código. De esta manera, usar Cargo le ayuda a mantener sus
proyectos limpios y arreglados. Hay un lugar para cada cosa, y todo está en su
lugar.

Si usted comenzó un proyecto que no usa Cargo, como hicimos con nuestro proyecto
en el directorio "hello_wordl*, puede convertirlo a un proyecto que usa Cargo
moviendo su código al directorio *src* y creando un *Cargo.toml* apropiado.

### Construyendo y Ejecutando un Proyecto Cargo

¡Ahora veamos qué es diferente en cuanto a construir y ejecutar su programa Hello
World a través de Cargo! Para hacerlo, introduzca los siguientes comandos:

```text
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Esto crea un archivo ejecutable en *target/debug/hello_cargo* (o
*target\\debug\\hello_cargo.exe* en Windows), el cual usted puede ejecutar con
este comando:

```text
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

¡Bam! Si todo sale bien, `Hello, Wolrd`debería de imprimirse en el terminal
una vez más.

Ejecutar por primera vez `cargo build` también causa que Cargo cree un nuevo
archivo en el nivel tope llamado *Cargo.lock*. Cargo usa *Cargo.lock* para
mantener el rastro de las versiones exactas de las dependencias usadas para
construir su proyecto. Este proyecto no tiene dependencias, así que el archivo
es un poco escaso. Nunca necesitará tocar este archivo por usted mismo, Cargo
manejará sus contenidos por usted.

Acabamos de construir un proyecto con `cargo build` y lo ejecutamos con
`./target/debug/hello_cargo`, pero también podemos usar `cargo run` para
compilar y luego ejecutar:

```text
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Note que esta vez, no vimos una salida diciéndonos que cargo ha compilado
`hello_cargo`. Cargo entendió que los archivos no han cambiado, así que
sólo ejecutó el binario. Si usted hubiese modificado su código fuente,
Cargo hubiese reconstruido el proyecto antes de ejecutarlo, y usted hubiese
visto una salida así:

```text
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Finalmente, está `cargo check`. Este revisará rápidamente su código para
asegurarse de que compilará, pero no se molestará en producir un ejecutable:

```text
$ cargo check
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

¿Por qué no querría usted un ejecutable? `cargo check` muchas veces es más
rápido que `cargo build`, ya que Cargo puede saltarse todo el paso de producir
un ejecutable. Si estamos revisando nuestro trabajo a través del proceso de
escribir el código, ¡Esto acelerará las cosas! Como tal, muchos Rustaceanos
ejecutan `cargo check` mientras que escriben sus programas para asegurarse
de que compilarán, y luego ejecutan `cargo build` una vez que estén listos para
probarlos ellos mismos.

Algunas otras diferencias que hemos visto hasta ahora:

- En vez de usar `rustc`, construya un proyecto usando `cargo build` o `cargo
check` (o constrúyalo y ejecútelo en un sólo paso con `cargo run`).
- En vez de que el resultado de la construcción sea colocado en el mismo
directorio que nuestro código, Cargo lo colocará en el directorio *target/debug*.

La otra ventaja de usar Cargo es que los comandos son los mismo sin importar
cuál sistema operativo esté usando, así que hasta este punto no tiene que proveer
instrucciones específicas para Linux y Mac en comparación con Windows.

### Construcción para Lanzamiento

Cuando su proyecto esté finalmente listo para el lanzamiento, usted puede usar
`cargo build --release` para compilar su proyecto con optimizaciones. Esto creará
un ejecutable en *target/release* en vez de *target/debug*. Estas optimizaciones
hacen que su código de Rust se ejecute mucho más rápido, pero activarlas hará que
la compilación de su programa tome mucho más tiempo. Por esto es que hay dos perfiles
diferentes: uno para el desarrollo cuando usted quiera ser capaz de reconstruir
rápido y muy seguido, y uno para construir el programa final que usted le dará a
un usuario que no reconstruirá y que queremos que se ejecute lo más rápido posible.
Si usted está evaluando el tiempo de ejecución de su código, asegúrese de ejecutar
`cargo build --releas` y evaluar con el ejecutable en *target/release*

### Cargo como Convención

Con proyectos simples, Cargo no provee mucho valor en comparación a sólo usar
`rustc`, pero proveerá su valor a medida que usted continúe. Con proyectos complejos
compuestos con crates múltiples, es mucho más fácil dejar a Cargo coordinar la
construcción. Con Cargo, usted puede tan solo ejecutar `cargo build`, y funcionará
de la forma adecuada.

Incluso cuando el proyecto `hello_cargo` es simple, ahora usa muchas de las
herramientas reales que usted usará por el resto de su carrera con Rust. De hecho,
usted puede comenzar con virtualmente todos los proyectos de Rust en los que quiera
trabajar con el siguiente comando para revisar el código usando Git, cambiar al
directorio del proyecto, y construir:

```text
$ git clone someurl.com/someproject
$ cd someproject
$ cargo build
```

Si quiere saber más sobre los detalles de Cargo, revise [su documentación], la cual
cubre todas sus funcionalidades.

[su documentación]: https://doc.rust-lang.org/cargo/

