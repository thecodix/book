# NOTICE ABOUT STATUS

#Aviso sobre el estado del proyecto

The second edition of The Rust Programming Language is getting ever closer to being printed!
This means we're not able to make large changes to chapters that are in any column to the
right of, and including, the "Frozen" column [on our Project board][proj]. Issues or pull
requests submitted for frozen chapters are welcome but will be closed until we start work
on a third edition. Thank you!

La segunda edicion de Rust Programming esta mas cerca de ser impresa! Esto significa que no haremos
modificaciones grandes a los capitulos que ya se encuentren finalizados o "congelados", esto se puede
ver en la columna "fronzen" [on our Project board][proj]. Cualquier modicificacion o pull request que
se realice en estos capitulos sera bien recibida pero no sera tomada en cuenta hasta la tercera edicion.
Gracias por su atencion!

[proj]: https://github.com/rust-lang/book/projects/1

# The Rust Programming Language

#EL lenguaje de progracion Rust

[![Build Status](https://travis-ci.org/rust-lang/book.svg?branch=master)](https://travis-ci.org/rust-lang/book)

This repo contains two editions of “The Rust Programming Language”; we
recommend starting with the second edition.

Este repositorio contiene dos ediciones del "Lenguaje de programacion Rust"; 
recomendados empezar por la segunda edicion.

The second edition is a rewrite that will be printed by No Starch Press,
available around May 2018. Check [the No Starch Page][nostarch] for the latest
information on the release date and how to order.

La segunda edicion es una reescritura que sera impresa por No Starh Press, estando
disponible a partir de mayo del 2018. Revise [the No Starch Page][nostarch] para
obtener informacion acerca de la fecha de lanzamiento y como ordenar su copia.

[nostarch]: https://nostarch.com/rust

You can read the book for free online! Please see the book as shipped with the
latest [stable], [beta], or [nightly] Rust releases. Be aware that issues in
those versions may have been fixed in this repository already.

Puede leer el libro online gratis! Por favor revise el libro con las versiones
Rust mas nuevas [stable], [beta], or [nightly]. Tenga en cuenta que los errores
incluidos en estas versiones pueden ya estar resueltos en este repositorio.

[stable]: https://doc.rust-lang.org/stable/book/second-edition/
[beta]: https://doc.rust-lang.org/beta/book/second-edition/
[nightly]: https://doc.rust-lang.org/nightly/book/second-edition/

[The first edition is still available to read online][first].

[first]: https://doc.rust-lang.org/book/


## Requirements

##Requerimientos

Building the book requires [mdBook], ideally the same version that
[rust-lang/rust uses in this file][rust-mdbook]. To get it:

Para compilar el libro necesita [mdBook], idealmente debe ser la misma
version que [rust-lang/rust uses in this file][rust-mdbook]. Para obtenerlo:

[mdBook]: https://github.com/azerupi/mdBook
[rust-mdbook]: https://github.com/rust-lang/rust/blob/master/src/tools/rustbook/Cargo.toml

```bash
$ cargo install mdbook --vers [version-num]
```

## Building

## Compilacion

To build the book, first `cd` into either the `first-edition` or
`second-edition` directory depending on which edition of the book you would
like to build. Then type:

Para compilar el libro, primero use el comando `cd` y cambie entre los
directorios `first-edition` y `second-edition` dependiende de la edicion
que quiera compilar. Luego escriba:

```bash
$ mdbook build
```

The output will be in the `book` subdirectory. To check it out, open it in
your web browser.

La salida estara en el subdirectorio `book`. Para revisar si esta correcto, 
abralo en su navegador.

_Firefox:_
```bash
$ firefox book/index.html                       # Linux
$ open -a "Firefox" book/index.html             # OS X
$ Start-Process "firefox.exe" .\book\index.html # Windows (PowerShell)
$ start firefox.exe .\book\index.html           # Windows (Cmd)
```

_Chrome:_
```bash
$ google-chrome book/index.html                 # Linux
$ open -a "Google Chrome" book/index.html       # OS X
$ Start-Process "chrome.exe" .\book\index.html  # Windows (PowerShell)
$ start chrome.exe .\book\index.html            # Windows (Cmd)
```

Para ejecutar las pruebas:

```bash
$ mdbook test
```

## Contributing

## Contribuciones

We'd love your help! Please see [CONTRIBUTING.md][contrib] to learn about the
kinds of contributions we're looking for.

Apreciaremos su ayudam por favor revise [CONTRIBUTING.md][contrib] para saber 
los tipos de contribucion que buscamos.

[contrib]: https://github.com/rust-lang/book/blob/master/CONTRIBUTING.md

### Translations

### Traducciones

We'd especially love help translating the second edition of the book! See the
[Translations] label to join in efforts that are currently in progress. Open
a new issue to start working on a new language! We're waiting on [mdbook
support] for multiple languages before we merge any in, but feel free to
start! The chapters in [the frozen column] of the project won't see major
changes, so if you start with those, you won't have to redo work :)

Agradeceremos especialmente la ayuda que pueda ofrecer en la traduccion de la
segunda edicion del libro! Revise la etiqueta [Translations] para unirse a los
esfuerzos de los colaboradores. Abra un nuevo issue para empezar a trabajar en
un nuevo lenguaje! estamos esperando por multiples lenguajes en la etiqueta 
[mdbook support] antes de hacer el merge a alguno, pero puede comenzar sin problema!
Los capitulos en la etiqueta [the frozen column] del proyecto no sufriran cambios
grandes, asi que te recomendamos que empieces con ellos y asi no tendras que volver
a hacer el trabajo :)

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/azerupi/mdBook/issues/5
[the frozen column]: https://github.com/rust-lang/book/projects/1

## No Starch

## No starch

As the second edition of the book will be published by No Starch, we first
iterate here, then ship the text off to No Starch. Then they do editing, and we
fold it back in.

Como la segunda edicion sera publicada por No Starch, pero queremos distibuirlo
aqui, luego enviarlo a No Starch. Luego ellos lo editaran y nosotros lo volvere-
mos a montar.

As such, there’s a directory, *nostarch*, which corresponds to the text in No
Starch’s system.

Pro esto, existe un directorio llamado *nostarch* que contiene el texto en el
sistema No Starch.

When we've started working with No Starch in a word doc, we will also check
those into the repo in the *nostarch/odt* directory. To extract the text from
the word doc as markdown in order to backport changes to the online book:

Cuando comenzamos a trabajar con No Starch en un documento word, tmabien revi-
samos dentro del repositorio en el directorio *nostarch/odt*. Para extraer el
texto del documento word y guardar los cambios realizados en el libro online:

1. Open the doc file in LibreOffice
1. Abra el documento en LibreOffice
1. Accept all tracked changes
1. Acepte todos los cambios
1. Save as Microsoft Word 2007-2013 XML (.docx) in the *tmp* directory
1. Guardelo como un documento Word 2007-2013 XML (.docx) en el directorio *tmp*
1. Run `./doc-to-md.sh`
1. Ejecute `./doc-to-md.sh`
1. Inspect changes made to the markdown file in the *nostarch* directory and
   copy the changes to the *src* directory as appropriate.
1. Revise los cambios realzidos al archivo en el directorio *nostarch* y copie 
los cambbios al directorio *src*.

## Graphviz dot

This is mostly for Carol's reference because she keeps having to look it up.

Esto es para Carol, porque ella debe seguir revisandolo.

We're using [Graphviz](http://graphviz.org/) for some of the diagrams in the
book. The source for those files live in the `dot` directory. To turn a `dot`
file, for example, `dot/trpl04-01.dot` into an `svg`, run:

Estamos usando  [Graphviz](http://graphviz.org/) para algunos diagramas en el
libro. La fuente de estos archivos se encuentra en el directorio `dot`. Para 
cabiar un archivo `dot` como por ejemplo, `dot/trpl04-01.dot` a `svg`, ejecute: 

```bash
$ dot dot/trpl04-01.dot -Tsvg > src/img/trpl04-01.svg
```

In the generated SVG, remove the width and the height attributes from the `svg`
element and set the `viewBox` attribute to `0.00 0.00 1000.00 1000.00` or other
values that don't cut off the image.

En el archivo SVG moderado, remueva los atributos de ancho y largo del elemento
`svg` y configure el atributo de `viewBox` a `0.00 0.00 1000.00 1000.00` u otro
valor que no corte la imagen.

## Spellchecking

## Correccion de ortografia

To scan source files for spelling errors, you can use the `spellcheck.sh`
script. It needs a dictionary of valid words, which is provided in
`dictionary.txt`. If the script produces a false positive (say, you used word
`BTreeMap` which the script considers invalid), you need to add this word to
`dictionary.txt` (keep the sorted order for consistency).

Para escanear un archivo en busqueda de errores ortograficos, puede usar el
script `spellcheck.sh`, que necesita un diccionario con las palabras correc-
tas, provistas en el archivo `dictionary.txt`. Si el script produce un falso
positivo (por ejemplo, usa la palabra `BTreeMap` que el diccionario no consi-
dera valida), debe agregar esta palabra al archivo `dictionary.txt` (Al agregar
mantenge el orden alfabetico para tener consistencia en el archivo)
