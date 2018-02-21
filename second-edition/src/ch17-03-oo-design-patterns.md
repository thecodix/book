## Implementing an Object-Oriented Design Pattern

The *state pattern* is an object-oriented design pattern. The crux of the
pattern is that a value has some internal state, represented by a set of *state
objects*, and the value’s behavior changes based on the internal state. The
state objects share functionality--in Rust, of course, we use structs and
traits rather than objects and inheritance. Each state object representing the
state is responsible for its own behavior and for governing when it should
change into another state. The value that holds a state object knows nothing
about the different behavior of the states or when to transition between states.

<!-- Below -- requirements for what, for what we need the value for? -->
<!-- I've clarified /Carol -->

Using the state pattern means when the business requirements of the program
change, we won’t need to change the code of the value holding the state or the
code that uses the value. We’ll only need to update the code inside one of the
state objects to change its rules, or perhaps add more state objects. Let’s
look at an example of the state design pattern and how to use it in Rust.

To explore this idea, we’ll implement a blog post workflow in an incremental
way. The blog’s final functionality will look like this:

1. A blog post starts as an empty draft.
2. Once the draft is done, a review of the post is requested.
3. Once the post is approved, it gets published.
4. Only published blog posts return content to print, so unapproved posts can’t
   accidentally be published.

Any other changes attempted on a post should have no effect. For example, if we
try to approve a draft blog post before we’ve requested a review, the post
should stay an unpublished draft.

Listing 17-11 shows this workflow in code form. This is an example usage of the
API we’re going to implement in a library crate named `blog`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate blog;
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());

    post.request_review();
    assert_eq!("", post.content());

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
```

<span class="caption">Listing 17-11: Code that demonstrates the desired
behavior we want our `blog` crate to have</span>

We want to allow the user to create a new draft blog post with `Post::new`.
Then, we want to allow text to be added to the blog post while it’s in the
draft state. If we try to get the post’s content immediately, before
approval, nothing should happen because the post is still a draft. We’ve added
an `assert_eq!` here for demonstration purposes. An excellent unit test for
this would be to assert that a draft blog post returns an empty string from the
`content` method, but we’re not going to write tests for this example.

Next, we want to enable a request for a review of the post, and we want
`content` to return an empty string while waiting for the review. Lastly, when
the post receives approval, it should get published, meaning the text of the
post will be returned when `content` is called.

<!-- Below -- so this is where we'll implement the state pattern? If so, can
you make that explicit, just to be clear! I've added some text to the second
line, not sure if that's accurate though -->
<!-- Yes, the state pattern will be implemented within the `Post` type. I've
tweaked the wording a bit but you've pretty much got it! /Carol-->

Notice that the only type we’re interacting with from the crate is the `Post`
type. This type will use the state pattern and will hold a value that will be
one of three state objects representing the various states a post can be
in---draft, waiting for review, or published. Changing from one state to
another will be managed internally within the `Post` type. The states change in
response to the methods users of our library call on the `Post` instance, but
they don’t have to manage the state changes directly. This also means users
can’t make a mistake with the states, like publishing a post before it is
reviewed.

### Defining `Post` and Creating a New Instance in the Draft State

Let’s get started on the implementation of the library! We know we need a
public `Post` struct that holds some content, so let’s start with the
definition of the struct and an associated public `new` function to create an
instance of `Post`, as shown in Listing 17-12. We’ll also make a private
`State` trait. Then `Post` will hold a trait object of `Box<State>` inside an
`Option` in a private field named `state`. We’ll see why the `Option` is
necessary in a bit.

The `State` trait defines the behavior shared by different post states, and the
`Draft`, `PendingReview`, and `Published` states will all implement the `State`
trait. For now, the trait does not have any methods, and we’re going to start
by defining just the `Draft` state since that’s the state we want a post to
start in:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub struct Post {
    state: Option<Box<State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }
}

trait State {}

struct Draft {}

impl State for Draft {}
```

<span class="caption">Listing 17-12: Definition of a `Post` struct and a `new`
function that creates a new `Post` instance, a `State` trait, and a `Draft`
struct</span>

When we create a new `Post`, we set its `state` field to a `Some` value that
holds a `Box`. This `Box` points to a new instance of the `Draft` struct. This
ensures whenever we create a new instance of `Post`, it’ll start out as a
draft. Because the `state` field of `Post` is private, there’s no way to create
a `Post` in any other state!

### Storing the Text of the Post Content

In the `Post::new` function, we set the `content` field to a new, empty
`String`. Listing 17-11 showed that we want to be able to call a method named
`add_text` and pass it a `&str` that’s then added to the text content of the
blog post. We implement this as a method rather than exposing the `content`
field as `pub`. This means we can implement a method later that will control
how the `content` field’s data is read. The `add_text` method is pretty
straightforward, so let’s add the implementation in Listing 17-13 to the `impl
Post` block:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct Post {
#     content: String,
# }
#
impl Post {
    // --snip--
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

<span class="caption">Listing 17-13: Implementing the `add_text` method to add
text to a post’s `content`</span>

`add_text` takes a mutable reference to `self`, since we’re changing the `Post`
instance that we’re calling `add_text` on. We then call `push_str` on the
`String` in `content` and pass the `text` argument to add to the saved
`content`. This behavior doesn’t depend on the state the post is in so it’s not
part of the state pattern. The `add_text` method doesn’t interact with the
`state` field at all, but it is part of the behavior we want to support.

### Ensuring the Content of a Draft Post is Empty

Even after we’ve called `add_text` and added some content to our post, we still
want the `content` method to return an empty string slice since the post is
still in the draft state, as shown on line 8 of Listing 17-11. For now, let’s
implement the `content` method with the simplest thing that will fulfill this
requirement: always returning an empty string slice. We’re going to change this
later once we implement the ability to change a post’s state so it can be
published. So far, though, posts can only be in the draft state, so the post
content should always be empty. Listing 17-14 shows this placeholder
implementation:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct Post {
#     content: String,
# }
#
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        ""
    }
}
```

<span class="caption">Listado 17-14: Añadiendo una implementación de marcador para
el método `content` en `Post` que siempre devuelva un pedazo de hilo vacío</span>

Con este método `content` añadido, todo en el listado 17-11 hasta la línea 8
trabaja como lo hemos previsto.

### Solicitar una revisión la publicación cambia su estado

A continuación necesitamos añadir funcionalidad para solicitar una revisión de un estado, lo que
debería cambiar su estado de `Draft` a `PendingReview`. Queremos darle a `Post`
un método público llamado `request_review` que llevará referencias mutables a
`self`. Entonces vamos a llamar un método interno `request_review`  en el
estado actual de `Post`, y este segundo método `request_review` consumirá
el estado actual y devolverá un nuevo estado. El listado 17-15 muestra este código:

<!-- NOTE TO DE/AU: We might want to move this explanation to after the code if
you want to add wingdings, we can see once we transfer it to Word -->
<!-- I decided to move some of this explanation after the code for this reason
and because we got some questions about this example that I wanted to expand
upon /Carol -->

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
# pub struct Post {
#     state: Option<Box<State>>,
#     content: String,
# }
#
impl Post {
    // --snip--
    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<State>;
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<State> {
        Box::new(PendingReview {})
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<State> {
        self
    }
}
```

<span class="caption">Listado 17-15: Implementando métodos `request_review` en
los rasgos `Post` y `State`</span>

Hemos añadido el método `request_review` al rasgo `State`; todos los tipos
que implementen el rasgo ahora necesitarán implementar el método`request_review`.
Nota que en vez de tener a `self`, `&self`, o `&mut self` como el primer
parámetro del método, tenemos a `self: Box<Self>`. Ésta sintáxis significa que
método sólo es válido cuando se le llama en un `Box` que albergue el tipo. Ésta sintáxis toma 
posesión de `Box<Self>`, invalidando el estado viejo para que así el valor de estado de
`Post` pueda transformarse él mismo en un nuevo estado.

<!-- Above -- so Post can transform, or so Draft can transform? -->
<!-- Technically it's so the Draft value can transform into another value,
which changes the state of Post-- I've tried to clarify. /Carol -->

Para consumir el viejo estado, el método `request_review` necesita tomar posesión
del valor de estado. Aquí es donde el `Option` en el campo `state` de `Post`
entra: llamamos el método `take` para tomar el valor `Some` del campo `state`
y dejar un `None` en su lugar, ya que Rust no nos permite tener
campos sin poblar en estructura. Esto nos deja mover el valor `state` del
`Post` en vez de tomarlo prestado. Entonces asignaremos el valor `state` de la publicación al
resultado de esta operación.

Necesitamos asignar `state` a `None` temporalmente, en vez de usar un código como `self.state
= self.state.request_review();` que asignaría el campo `state` directamente, para
tomar posesión del valor `state`. Ésto asegura que `Post` no pueda usar el viejo
valor `state` después de que lo hayamos transformado en un nuevo estado.

El método `request_review` en `Draft` necesita devolver una nueva instancia en una caja de
una nueva estructura `PendingReview`, lo que representa el estado en el que una publicación está esperando
por una revisión. La estructura `PendingReview` también implementa el método `request_review`,
pero no hace ninguna transformación. En cambio, se devuelve a sí mismo, ya que
cuando solicitamos una revisión en una publicación que ya está en el estado `PendingReview`,
se debería quedar en el estado `PendingReview`.

Ahora podemos empezar a ver las ventajas del patrón de estado: el
método `request_review` en `Post` es el mismo sin importar su valor `state`. Cada
estado es responsable de sus propias reglas.

Vamos a dejar el método `content` en `Post` como está, devolviendo un 
pedazo de hilo vacío. Ahora podemos tener un `Post` en el estado `PendingReview` así como
en el estado `Draft`, pero queremos el mismo comportamiento en el estado `PendingReview`.
¡El listado 17-11 ahora funciona hasta la linea 11! 

### Añadiendo el método `approve` que cambia el comportamiento de `content`

El método `approve` será similar al método `request_review`: se
ajustará `state` al valor que el estado actual diga que debe tener cuando ese
estado sea aprobado, como se muestra en el listado 17-16.

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
# pub struct Post {
#     state: Option<Box<State>>,
#     content: String,
# }
#
impl Post {
    // --snip--
    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<State>;
    fn approve(self: Box<Self>) -> Box<State>;
}

struct Draft {}

impl State for Draft {
#     fn request_review(self: Box<Self>) -> Box<State> {
#         Box::new(PendingReview {})
#     }
#
    // --snip--
    fn approve(self: Box<Self>) -> Box<State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
#     fn request_review(self: Box<Self>) -> Box<State> {
#         self
#     }
#
    // --snip--
    fn approve(self: Box<Self>) -> Box<State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<State> {
        self
    }
}
```

<span class="caption">Listado 17-16: Implementando el método `approve` en
los rasgos `Post` y `State`</span>

Añadimos el método `approve` al rasgo `State`, y añadimos una nueva estructura que
implemente `State`, el estado `Published`.

Similar a `request_review`, si llamamos el método `approve` en `Draft`, no
tendrá efecto ya que devolverá `self`. Cuando llamamos `approve` en
`PendingReview`, devuelve una nueva instancia en caja de la estructura `Published`.
La estructura `Published` implementará el rasgo `State`, y para ambos el 
método `request_review` y el método `approve`, se devuelve a sí mismo, ya que la
publicación debe permanecer en el estado `Published` en esos casos.

Ahora para actualizar el método `content` en `Post`: Si el estado es `Published` vamos a
querer devolver el valor en el campo `content` de la publicación; de otra manera vamos 
a querer que devuelva un pedazo de hilo vacío, como se muestra en el listado 17-17:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
# trait State {
#     fn content<'a>(&self, post: &'a Post) -> &'a str;
# }
# pub struct Post {
#     state: Option<Box<State>>,
#     content: String,
# }
#
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        self.state.as_ref().unwrap().content(&self)
    }
    // --snip--
}
```

<span class="caption">Listado 17-17: Actualizando el método `content` en `Post` para
delegarlo a un método `content` en `State`</span>

Ya que la meta es manetener todas estas reglas dentro de las estructuras que implementen
`State`, llamamos un método `content` sobre el valor en `state` y pasamos la instancia de 
la publicación (que es, `self`) como un argumento. Entonces regresamos el valor que es
retornado al usar el método `content` en el valor `state`.

Llamamos el método  `as_ref` en `Option` porque queremos una referencia al
valor dentro de `Option` en vez de tener posesión del mismo. Ya que `state` es un
`Option<Box<State>>`, el llamar `as_ref` devuelve un `Option<&Box<State>>`. Si 
no hubieramos llamado `as_ref`, obtendríamos un error porque no podemos mover `state` fuera 
del prestado `&self` del parámetro de la función.

Entonces estamos llamado el método `unwrap`, el que sabemos que nunca entrará en pánico, ya que
sabemos que los métodos en `Post` aseguran que `state` siempre contendrá un valor `Some`
cuando esos métodos terminen. Este es uno de los casos de los que hablamos en el
capítulo 12 cuando sabemos que un valor `None` nunca es posible, incluso si el
compilador es incapaz de entender eso.

Así que tenemos un `&Box<State>`, y entonces llamamos al `content` en el, la coerción
deref tendrá efecto en el `&` y el `Box` y así el método `content`
será llamado en el tipo que implemente el rasgo `State`.

Eso significa que necesitamos añadir `content` a la definición del rasgo `State` y allí es
donde pondremos la lógica para decidir qué contenido devuelva dependiendo de qué estado
tengamos, como lo muestra el listado 17-18:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
# pub struct Post {
#     content: String
# }
trait State {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        ""
    }
}

// --snip--
struct Published {}

impl State for Published {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}
```

<span class="caption">Listado 17-18: Añadiendo el método `content` al rasgo `State`
</span>

Añadimos una implementación por defecto para el método `content` que devuelva un pedazo
vacío de hilo. Eso significa que no necesitamos implementar `content` en las estructuras `Draft`
y `PendingReview`. La estructura `Published` entonces anulará el método `content`
y devolverá el valor en `post.content`.

Nota que necesitamos anotaciones de tiempo de vida en este método, como discutimos en el 
Capítulo 10. Estamos tomando una referencia a un `post` como un argumento, y devolviendo 
una referencia para partir desde ese `post`, para que así el tiempo de vida de la referencia devuelta
esté relacionado con el tiempo de vida del argumento`post`.

<!-- Is this it finished, without the touch up we make to get rid of the empty
string? That's pretty awesome coding, maybe give it some ceremony here. Does
all of 17-11 now work? -->
<!-- Yep! Good point, so added! /Carol -->

Y hemos terminado-- ¡Todo el listado 17-11 funciona! Hemos implementado el patrón
de estado con las reglas del ritmo de trabajo de la publicación en el blog. La lógica que gira alrededor de las reglas
vive en los objetos de estados en vez de estar dispersas por todo el `Post`.

### Tradeoffs of the State Pattern

We’ve shown that Rust is capable of implementing the object-oriented state
pattern to encapsulate the different kinds of behavior a post should have in
each state. The methods on `Post` know nothing about the different kinds of
behavior. The way this code is organized, we only have to look in one place to
know the different ways a published post can behave: the implementation of the
`State` trait on the `Published` struct.

If we were to create an alternative implementation that didn’t use the state
pattern we might use `match` statements in the methods on `Post`, or even in
the `main` code that checks the state of the post and changes behavior in those
places instead. That would mean we’d have to look in a lot of places to
understand all the implications of a post being in the published state! This
would only increase the more states we added: each of those `match` statements
would need another arm.

With the state pattern, the `Post` methods and the places we use `Post` don’t
need `match` statements, and to add a new state we would only need to add a new
`struct` and implement the trait methods on that one struct.

This implementation is easy to extend to add more functionality. To see the
simplicity of maintaining code that uses this patterns, try out a few of these
suggestions:

- Allow users to add text content only when a post is in the `Draft` state
- Add a `reject` method that changes the post’s state from `PendingReview` back
  to `Draft`
- Require two calls to `approve` before the state can be changed to `Published`

One downside of the state pattern is that, because the states implement the
transitions between states, some of the states are coupled to each other. If we
add another state between `PendingReview` and `Published`, such as `Scheduled`,
we would have to change the code in `PendingReview` to transition to
`Scheduled` instead. It would be less work if `PendingReview` wouldn’t need to
change with the addition of a new state, but that would mean switching to
another design pattern.

Another downside is that we find ourselves with a few bits of duplicated logic.
To eliminate this, we might try to make default implementations for the
`request_review` and `approve` methods on the `State` trait that return `self`,
but this would violate object safety, since the trait doesn’t know what the
concrete `self` will be exactly. We want to be able to use `State` as a trait
object, so we need its methods to be object safe.

The other duplication is the similar implementations of the `request_review`
and `approve` methods on `Post`. Both methods delegate to the implementation of
the same method on the value in the `state` field of `Option`, and set the new
value of the `state` field to the result. If we had a lot of methods on `Post`
that followed this pattern, we might consider defining a macro to eliminate the
repetition (see Appendix D on macros).

By implementing this pattern exactly as it’s defined for object-oriented
languages, we’re not taking full advantage of Rust’s strengths as much as we
could. Let’s take a look at some changes we can make to this code that can make
invalid states and transitions into compile time errors.

#### Encoding States and Behavior as Types

We’re going to show how to rethink the state pattern to get a different set of
tradeoffs. Rather than encapsulating the states and transitions completely so
that outside code has no knowledge of them, we’re going to encode the states
into different types. Like this, Rust’s type checking system will make attempts
to use draft posts where only published posts are allowed into a compiler error.

Let’s consider the first part of `main` from Listing 17-11:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());
}
```

We still enable the creation of new posts in the draft state using `Post::new`,
and the ability to add text to the post’s content. But instead of having a
`content` method on a draft post that returns an empty string, we’ll make it so
that draft posts don’t have the `content` method at all. That way, if we try to
get a draft post’s content, we’ll get a compiler error telling us the method
doesn’t exist. This will make it impossible for us to accidentally display
draft post content in production, since that code won’t even compile. Listing
17-19 shows the definition of a `Post` struct, a `DraftPost` struct, and
methods on each:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub struct Post {
    content: String,
}

pub struct DraftPost {
    content: String,
}

impl Post {
    pub fn new() -> DraftPost {
        DraftPost {
            content: String::new(),
        }
    }

    pub fn content(&self) -> &str {
       &self.content
    }
}

impl DraftPost {
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

<span class="caption">Listing 17-19: A `Post` with a `content` method and a
`DraftPost` without a `content` method</span>

Both the `Post` and `DraftPost` structs have a private `content` field that
stores the blog post text. The structs no longer have the `state` field since
we’re moving the encoding of the state to the types of the structs. `Post` will
represent a published post, and it has a `content` method that returns the
`content`.

We still have a `Post::new` function, but instead of returning an instance of
`Post`, it returns an instance of `DraftPost`. Because `content` is private,
and there aren’t any functions that return `Post`, it’s not possible to create
an instance of `Post` right now.

`DraftPost` has an `add_text` method so we can add text to `content` as before,
but note that `DraftPost` does not have a `content` method defined! So now the
program ensures all posts start as draft posts, and draft posts don’t have
their content available for display. Any attempt to get around these
constraints will result in a compiler error.

#### Implementing Transitions as Transformations into Different Types

So how do we get a published post then? We want to enforce the rule that a
draft post has to be reviewed and approved before it can be published. A post
in the pending review state should still not display any content. Let’s
implement these constraints by adding another struct, `PendingReviewPost`,
defining the `request_review` method on `DraftPost` to return a
`PendingReviewPost`, and defining an `approve` method on `PendingReviewPost` to
return a `Post` as shown in Listing 17-20:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct Post {
#     content: String,
# }
#
# pub struct DraftPost {
#     content: String,
# }
#
impl DraftPost {
    // --snip--

    pub fn request_review(self) -> PendingReviewPost {
        PendingReviewPost {
            content: self.content,
        }
    }
}

pub struct PendingReviewPost {
    content: String,
}

impl PendingReviewPost {
    pub fn approve(self) -> Post {
        Post {
            content: self.content,
        }
    }
}
```

<span class="caption">Listing 17-20: A `PendingReviewPost` that gets created by
calling `request_review` on `DraftPost`, and an `approve` method that turns a
`PendingReviewPost` into a published `Post`</span>

The `request_review` and `approve` methods take ownership of `self`, thus
consuming the `DraftPost` and `PendingReviewPost` instances and transforming
them into a `PendingReviewPost` and a published `Post`, respectively. This way,
we won’t have any `DraftPost` instances lingering around after we’ve called
`request_review` on them, and so forth. `PendingReviewPost` doesn’t have a
`content` method defined on it, so attempting to read its content results in a
compiler error, as with `DraftPost`. Because the only way to get a published
`Post` instance that does have a `content` method defined is to call the
`approve` method on a `PendingReviewPost`, and the only way to get a
`PendingReviewPost` is to call the `request_review` method on a `DraftPost`,
we’ve now encoded the blog post workflow into the type system.

This does mean we have to make some small changes to `main`. The
`request_review` and `approve` methods return new instances rather than
modifying the struct they’re called on, so we need to add more `let post = `
shadowing assignments to save the returned instances. We also can’t have the
assertions about the draft and pending review post’s contents being empty
strings, nor do we need them: we can’t compile code that tries to use the
content of posts in those states any longer. The updated code in `main` is
shown in Listing 17-21:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate blog;
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");

    let post = post.request_review();

    let post = post.approve();

    assert_eq!("I ate a salad for lunch today", post.content());
}
```

<span class="caption">Listing 17-21: Modifications to `main` to use the new
implementation of the blog post workflow</span>

These changes we need to make to `main` to reassign `post` means this
implementation doesn’t quite follow the object-oriented state pattern anymore:
the transformations between the states are no longer encapsulated entirely
within the `Post` implementation. However, our gain is that invalid states are
now impossible because of the type system and the type checking that happens at
compile time! This ensures that certain bugs, such as the content of an
unpublished post being displayed, will be discovered before they make it to
production.

Try the tasks suggested for additional requirements that we mentioned at the
start of this section on this code, to see how working with this version of the
code feels.

We’ve seen that even though Rust is capable of implementing object-oriented
design patterns, other patterns like encoding state into the type system are
also available in Rust. These patterns have different tradeoffs. While you may
be very familiar with object-oriented patterns, rethinking the problem in order
to take advantage of Rust’s features can provide benefits like preventing some
bugs at compile-time. Object-oriented patterns won’t always be the best
solution in Rust, because of the features like ownership that object-oriented
languages don’t have.

## Summary

No matter whether you think Rust is an object-oriented language or not after
reading this chapter, you’ve now seen that trait objects are a way to get some
object-oriented features in Rust. Dynamic dispatch can give your code some
flexibility in exchange for a bit of runtime performance. This flexibility can
be used to implement object-oriented patterns that can help with the
maintainability of your code. Rust also has other different features, like
ownership, that object-oriented languages don’t have. An object-oriented
pattern won’t always be the best way to take advantage of Rust’s strengths, but
is an available option.

Next, let’s look at another feature of Rust that enables lots of flexibility:
patterns. We’ve looked at them briefly throughout the book, but haven’t seen
everything they’re capable of yet. Let’s go!
