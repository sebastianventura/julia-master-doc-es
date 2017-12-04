# Documentación

Julia permite a los desarrolladores y usuarios de paquetes documentar funciones, tipos y otros objetos fácilmente a través de un sistema de documentación incorporado desde Julia 0.4.

La sintaxis básica es muy simple: cualquier cadena que aparezca en el nivel superior justo antes de que un objeto (función, macro, tipo o instancia) se interpretará como documentación (se llaman *docstrings*). He aquí un ejemplo muy simple:

```julia
"Tell whether there are too foo items in the array."
foo(xs::Array) = ...
```

La documentación se interpreta como [Markdown](https://en.wikipedia.org/wiki/Markdown), por lo que puede usar indentación y vallas de código para delimitar ejemplos de código del texto. Técnicamente, cualquier objeto puede asociarse con cualquier otro como metadatos; Markdown es el formato predeterminado, pero uno puede construir otras macros de cadena y pasarlas a la macro `@doc` también.

Aquí hay un ejemplo más complejo, aún usando Markdown:

````julia
"""
    bar(x[, y])

Compute the Bar index between `x` and `y`. If `y` is missing, compute
the Bar index between all pairs of columns of `x`.

# Examples
```julia-repl
julia> bar([1, 2], [1, 2])
1
```
"""
function bar(x, y) ...
````

Como en el ejemplo anterior, recomendamos seguir algunas convenciones simples al escribir documentación:

1. Siempre muestre la firma de una función en la parte superior de la documentación, con una sangría de cuatro espacios
   para que se imprima como código Julia.
   
   Esto puede ser idéntico a la firma presente en el código Julia (como `mean (x :: AbstractArray)`),
   o una forma simplificada. Los argumentos opcionales deben representarse con sus valores predeterminados (es decir
   `f(x,y = 1)`) cuando sea posible, siguiendo la sintaxis real de Julia. Los argumentos opcionales que no tengan 
   un valor predeterminado deben ponerse entre corchetes (es decir, `f(x[,y])` y `f (x [,y[,z]])`). Una solución
   alternativa es usar varias líneas: una sin argumentos opcionales y la otra(s) con ellos. Esta
   solución también se puede usar para documentar varios métodos relacionados de una función dada. Cuando una función
   acepta muchos argumentos de palabras clave, solo incluye un marcador de posición `<keyword arguments>` en la firma
   (es decir, `f (x; <keyword arguments>)`), y proporcione la lista completa bajo una sección `#Argumentos`
   (ver el punto 4 a continuación).

2. Incluya una oración de una sola línea que describa qué hace la función o qué representa el objeto después del 
   bloque de firma simplificado. Si es necesario, brinde más detalles en un segundo párrafo, después de una línea 
   en blanco.
   
   La frase de una línea debe usar la forma imperativa ("Hacer esto", "Devolver eso") en lugar de la tercera 
   persona (no escribir "Devuelve la longitud ...") al documentar funciones. Debería terminar con un punto. Si el 
   significado de una función no se puede resumir fácilmente, dividirla en partes compostables separadas podría 
   ser beneficioso (sin embargo, esto no debería tomarse como un requisito absoluto para cada caso).
   
3. No te repitas.

   Como el nombre de la función viene dado por la firma, no es necesario iniciar la documentación con 
   "La función` bar` ... ": vaya directamente al punto. De forma similar, si la firma especifica los tipos de 
   argumentos, mencionarlos en la descripción es redundante.

4. Solo proporcione una lista de argumentos cuando sea realmente necesario.

   Para funciones simples, a menudo es más claro mencionar el papel de los argumentos directamente en la 
   descripción del propósito de la función. Una lista de argumentos solo repetiría la información ya provista en otro 
   lugar. Sin embargo, proporcionar una lista de argumentos puede ser una buena idea para funciones complejas con muchos
   argumentos (en particular argumentos de palabras clave). En ese caso, insértelo después de la descripción general de la 
   función, bajo un encabezado `# Arguments`, con una viñeta` -` para cada argumento. La lista debe mencionar los tipos y
   valores predeterminados (si los hay) de los argumentos:

   ```julia
   """
   ...
   # Arguments
   - `n::Integer`: the number of elements to compute.
   - `dim::Integer=1`: the dimensions along which to perform the computation.
   ...
   """
   ```
5. Incluya cualquier ejemplo de código en la sección `#Examples`.

   Los ejemplos deben, siempre que sea posible, escribirse como *doctests*. A *doctest* es un bloque de código vallado (ver [Bloques de código](@ref)) que comienza con `` `` `` `` jldoctest````` y contiene cualquier cantidad de instrucciones `julia>` junto con las entradas y resultados esperados que imitan a Julia REPL.

    Por ejemplo, en la siguiente docstring se define una variable `a` y el resultado esperado, como se imprime en un Julia REPL, aparece después:

   Examples should, whenever possible, be written as *doctests*. A *doctest* is a fenced code block
   (see [Code blocks](@ref)) starting with ````` ```jldoctest````` and contains any number of `julia>`
   prompts together with inputs and expected outputs that mimic the Julia REPL.

   For example in the following docstring a variable `a` is defined and the expected result, as printed
   in a Julia REPL, appears afterwards:

   ````julia
   """
   Some nice documentation here.

   # Examples

   ```jldoctest
   julia> a = [1 2; 3 4]
   2×2 Array{Int64,2}:
    1  2
    3  4
   ```
   """
   ````

   !!! warning
   
       Invocar a `rand` y otras funciones relacionadas con RNG deben evitarse en doctests ya que no producirán 
       salidas consistentes durante diferentes sesiones de Julia. Si desea mostrar alguna funcionalidad relacionada 
       con la generación de números aleatorios, una opción es construir explícitamente y sembrar su propio 
       [`MersenneTwister`](@ref) (u otro generador de números pseudoaleatorios) y pasarlo a las funciones que está 
       confirmando.

        El tamaño de palabra del sistema operativo ([`Int32`] (@ ref) o [` Int64`] (@ ref)) así como las diferencias 
        del separador de ruta (`/` o `\`) también afectarán la reproducibilidad de algunos documentos.

        ¡Tenga en cuenta que el espacio en blanco en su doctest es significativo! El doctest fallará si desalinea 
        la salida de impresión bonita de una matriz, por ejemplo.
       
A continuación, puede ejecutar `make -C doc doctest` para ejecutar todos los doctests en el Manual de Julia, lo que garantizará que su ejemplo funcione.

    Los ejemplos que no se pueden verificar deben escribirse dentro de bloques de código delimitados que comiencen con `` `` `` `` julia````` para que se destaquen correctamente en la documentación generada.   
    
   !!! Tip
       Siempre que sea posible, los ejemplos deben ser ** autónomos ** y ** ejecutables ** para que los lectores 
       puedan probarlos sin tener que incluir ninguna dependencia.
       
6. Usa los backticks para identificar el código y las ecuaciones.

    Los identificadores de Julia y los extractos del código siempre deben aparecer entre los backticks
    `` `` `` `para habilitar el resaltado. Las ecuaciones en la sintaxis de LaTeX se pueden insertar entre los 
    backticks dobles `` `` `` ``.
    Use caracteres Unicode en lugar de su secuencia de escape LaTeX, es decir `` `` `α = 1``` `` en lugar de 
    `` `` `\\ alpha = 1``` ``.
7. Coloque los caracteres iniciales y finales `` `` `en líneas por sí mismos.

   Esto es, escriba:

   ```julia
   """
   ...

   ...
   """
   f(x, y) = ...
   ```

   en lugar de:

   ```julia
   """...

   ..."""
   f(x, y) = ...
   ```

Esto deja más claro dónde comienzan y finalizan las cadenas de documentos.
8. Respete el límite de longitud de línea utilizado en el código circundante.

    Las *Docstrings* se editan usando las mismas herramientas que el código. Por lo tanto, las mismas convenciones deberían aplicarse.
    Se aconseja agregar saltos de línea después de 92 caracteres.

## Acceder a documentación

Documentation can be accessed at the REPL or in [IJulia](https://github.com/JuliaLang/IJulia.jl)
by typing `?` followed by the name of a function or macro, and pressing `Enter`. For example,

```julia
?cos
?@time
?r""
```

will bring up docs for the relevant function, macro or string macro respectively. In [Juno](http://junolab.org)
using `Ctrl-J, Ctrl-D` will bring up documentation for the object under the cursor.

## Functions & Methods

Functions in Julia may have multiple implementations, known as methods. While it's good practice
for generic functions to have a single purpose, Julia allows methods to be documented individually
if necessary. In general, only the most generic method should be documented, or even the function
itself (i.e. the object created without any methods by `function bar end`). Specific methods should
only be documented if their behaviour differs from the more generic ones. In any case, they should
not repeat the information provided elsewhere. For example:

```julia
"""
    *(x, y, z...)

Multiplication operator. `x * y * z *...` calls this function with multiple
arguments, i.e. `*(x, y, z...)`.
"""
function *(x, y, z...)
    # ... [implementation sold separately] ...
end

"""
    *(x::AbstractString, y::AbstractString, z::AbstractString...)

When applied to strings, concatenates them.
"""
function *(x::AbstractString, y::AbstractString, z::AbstractString...)
    # ... [insert secret sauce here] ...
end

help?> *
search: * .*

  *(x, y, z...)

  Multiplication operator. x * y * z *... calls this function with multiple
  arguments, i.e. *(x,y,z...).

  *(x::AbstractString, y::AbstractString, z::AbstractString...)

  When applied to strings, concatenates them.
```

When retrieving documentation for a generic function, the metadata for each method is concatenated
with the `catdoc` function, which can of course be overridden for custom types.

## Advanced Usage

The `@doc` macro associates its first argument with its second in a per-module dictionary called
`META`. By default, documentation is expected to be written in Markdown, and the `doc""` string
macro simply creates an object representing the Markdown content. In the future it is likely to
do more advanced things such as allowing for relative image or link paths.

When used for retrieving documentation, the `@doc` macro (or equally, the `doc` function) will
search all `META` dictionaries for metadata relevant to the given object and return it. The returned
object (some Markdown content, for example) will by default display itself intelligently. This
design also makes it easy to use the doc system in a programmatic way; for example, to re-use
documentation between different versions of a function:

```julia
@doc "..." foo!
@doc (@doc foo!) foo
```

Or for use with Julia's metaprogramming functionality:

```julia
for (f, op) in ((:add, :+), (:subtract, :-), (:multiply, :*), (:divide, :/))
    @eval begin
        $f(a,b) = $op(a,b)
    end
end
@doc "`add(a,b)` adds `a` and `b` together" add
@doc "`subtract(a,b)` subtracts `b` from `a`" subtract
```

Documentation written in non-toplevel blocks, such as `begin`, `if`, `for`, and `let`, is
added to the documentation system as blocks are evaluated. For example:

```julia
if VERSION > v"0.5"
    "..."
    f(x) = x
end
```

will add documentation to `f(x)` when the condition is `true`. Note that even if `f(x)` goes
out of scope at the end of the block, its documentation will remain.

### Dynamic documentation

Sometimes the appropriate documentation for an instance of a type depends on the field values of that
instance, rather than just on the type itself. In these cases, you can add a method to `Docs.getdoc`
for your custom type that returns the documentation on a per-instance basis. For instance,

```julia
struct MyType
    value::String
end

Docs.getdoc(t::MyType) = "Documentation for MyType with value $(t.value)"

x = MyType("x")
y = MyType("y")
```

`?x` will display "Documentation for MyType with value x" while `?y` will display
"Documentation for MyType with value y".

## Syntax Guide

A comprehensive overview of all documentable Julia syntax.

In the following examples `"..."` is used to illustrate an arbitrary docstring which may be one
of the follow four variants and contain arbitrary text:

```julia
"..."

doc"..."

"""
...
"""

doc"""
...
"""
```

`@doc_str` should only be used when the docstring contains `$` or `\` characters that should not
be parsed by Julia such as LaTeX syntax or Julia source code examples containing interpolation.

### Functions and Methods

```julia
"..."
function f end

"..."
f
```

Adds docstring `"..."` to `Function``f`. The first version is the preferred syntax, however both
are equivalent.

```julia
"..."
f(x) = x

"..."
function f(x)
    x
end

"..."
f(x)
```

Adds docstring `"..."` to `Method``f(::Any)`.

```julia
"..."
f(x, y = 1) = x + y
```

Adds docstring `"..."` to two `Method`s, namely `f(::Any)` and `f(::Any, ::Any)`.

### Macros

```julia
"..."
macro m(x) end
```

Adds docstring `"..."` to the `@m(::Any)` macro definition.

```julia
"..."
:(@m)
```

Adds docstring `"..."` to the macro named `@m`.

### Types

```
"..."
abstract type T1 end

"..."
mutable struct T2
    ...
end

"..."
struct T3
    ...
end
```

Adds the docstring `"..."` to types `T1`, `T2`, and `T3`.

```julia
"..."
struct T
    "x"
    x
    "y"
    y
end
```

Adds docstring `"..."` to type `T`, `"x"` to field `T.x` and `"y"` to field `T.y`. Also applicable
to `mutable struct` types.

### Modules

```julia
"..."
module M end

module M

"..."
M

end
```

Adds docstring `"..."` to the `Module``M`. Adding the docstring above the `Module` is the preferred
syntax, however both are equivalent.

```julia
"..."
baremodule M
# ...
end

baremodule M

import Base: @doc

"..."
f(x) = x

end
```

Documenting a `baremodule` by placing a docstring above the expression automatically imports
`@doc` into the module. These imports must be done manually when the module expression is not
documented. Empty `baremodule`s cannot be documented.

### Global Variables

```julia
"..."
const a = 1

"..."
b = 2

"..."
global c = 3
```

Adds docstring `"..."` to the `Binding`s `a`, `b`, and `c`.

`Binding`s are used to store a reference to a particular `Symbol` in a `Module` without storing
the referenced value itself.

!!! note
    When a `const` definition is only used to define an alias of another definition, such as is the
    case with the function `div` and its alias `÷` in `Base`, do not document the alias and instead
    document the actual function.

    If the alias is documented and not the real definition then the docsystem (`?` mode) will not
    return the docstring attached to the alias when the real definition is searched for.

    For example you should write

    ```julia
    "..."
    f(x) = x + 1
    const alias = f
    ```

    rather than

    ```julia
    f(x) = x + 1
    "..."
    const alias = f
    ```

```julia
"..."
sym
```

Adds docstring `"..."` to the value associated with `sym`. Users should prefer documenting `sym`
at it's definition.

### Multiple Objects

```julia
"..."
a, b
```

Adds docstring `"..."` to `a` and `b` each of which should be a documentable expression. This
syntax is equivalent to

```julia
"..."
a

"..."
b
```

Any number of expressions many be documented together in this way. This syntax can be useful when
two functions are related, such as non-mutating and mutating versions `f` and `f!`.

### Macro-generated code

```julia
"..."
@m expression
```

Adds docstring `"..."` to expression generated by expanding `@m expression`. This allows for expressions
decorated with `@inline`, `@noinline`, `@generated`, or any other macro to be documented in the
same way as undecorated expressions.

Macro authors should take note that only macros that generate a single expression will automatically
support docstrings. If a macro returns a block containing multiple subexpressions then the subexpression
that should be documented must be marked using the [`@__doc__`](@ref Core.@__doc__) macro.

The `@enum` macro makes use of `@__doc__` to allow for documenting `Enum`s. Examining it's definition
should serve as an example of how to use `@__doc__` correctly.

```@docs
Core.@__doc__
```

## Markdown syntax

The following markdown syntax is supported in Julia.

### Inline elements

Here "inline" refers to elements that can be found within blocks of text, i.e. paragraphs. These
include the following elements.

#### Bold

Surround words with two asterisks, `**`, to display the enclosed text in boldface.

```
A paragraph containing a **bold** word.
```

#### Italics

Surround words with one asterisk, `*`, to display the enclosed text in italics.

```
A paragraph containing an *emphasised* word.
```

#### Literals

Surround text that should be displayed exactly as written with single backticks, ``` ` ``` .

```
A paragraph containing a `literal` word.
```

Literals should be used when writing text that refers to names of variables, functions, or other
parts of a Julia program.

!!! tip
    To include a backtick character within literal text use three backticks rather than one to enclose
    the text.

    ```
    A paragraph containing a ``` `backtick` character ```.
    ```

    By extension any odd number of backticks may be used to enclose a lesser number of backticks.

#### ``\LaTeX``

Surround text that should be displayed as mathematics using ``\LaTeX`` syntax with double backticks,
``` `` ``` .

```
A paragraph containing some ``\LaTeX`` markup.
```

!!! tip
    As with literals in the previous section, if literal backticks need to be written within double
    backticks use an even number greater than two. Note that if a single literal backtick needs to
    be included within ``\LaTeX`` markup then two enclosing backticks is sufficient.

#### Links

Links to either external or internal addresses can be written using the following syntax, where
the text enclosed in square brackets, `[ ]`, is the name of the link and the text enclosed in
parentheses, `( )`, is the URL.

```
A paragraph containing a link to [Julia](http://www.julialang.org).
```

It's also possible to add cross-references to other documented functions/methods/variables within
the Julia documentation itself. For example:

```julia
"""
    eigvals!(A,[irange,][vl,][vu]) -> values

Same as [`eigvals`](@ref), but saves space by overwriting the input `A`, instead of creating a copy.
"""
```

This will create a link in the generated docs to the `eigvals` documentation
(which has more information about what this function actually does). It's good to include
cross references to mutating/non-mutating versions of a function, or to highlight a difference
between two similar-seeming functions.

!!! note
    The above cross referencing is *not* a Markdown feature, and relies on
    [Documenter.jl](https://github.com/JuliaDocs/Documenter.jl), which is
    used to build base Julia's documentation.

#### Footnote references

Named and numbered footnote references can be written using the following syntax. A footnote name
must be a single alphanumeric word containing no punctuation.

```
A paragraph containing a numbered footnote [^1] and a named one [^named].
```

!!! note
    The text associated with a footnote can be written anywhere within the same page as the footnote
    reference. The syntax used to define the footnote text is discussed in the [Footnotes](@ref) section
    below.

### Toplevel elements

The following elements can be written either at the "toplevel" of a document or within another
"toplevel" element.

#### Paragraphs

A paragraph is a block of plain text, possibly containing any number of inline elements defined
in the [Inline elements](@ref) section above, with one or more blank lines above and below it.

```
This is a paragraph.

And this is *another* one containing some emphasised text.
A new line, but still part of the same paragraph.
```

#### Headers

A document can be split up into different sections using headers. Headers use the following syntax:

```julia
# Level One
## Level Two
### Level Three
#### Level Four
##### Level Five
###### Level Six
```

A header line can contain any inline syntax in the same way as a paragraph can.

!!! tip
    Try to avoid using too many levels of header within a single document. A heavily nested document
    may be indicative of a need to restructure it or split it into several pages covering separate
    topics.

#### Code blocks

Source code can be displayed as a literal block using an indent of four spaces as shown in the
following example.

```
This is a paragraph.

    function func(x)
        # ...
    end

Another paragraph.
```

Additionally, code blocks can be enclosed using triple backticks with an optional "language" to
specify how a block of code should be highlighted.

````
A code block without a "language":

```
function func(x)
    # ...
end
```

and another one with the "language" specified as `julia`:

```julia
function func(x)
    # ...
end
```
````

!!! note
    "Fenced" code blocks, as shown in the last example, should be prefered over indented code blocks
    since there is no way to specify what language an indented code block is written in.

#### Block quotes

Text from external sources, such as quotations from books or websites, can be quoted using `>`
characters prepended to each line of the quote as follows.

```
Here's a quote:

> Julia is a high-level, high-performance dynamic programming language for
> technical computing, with syntax that is familiar to users of other
> technical computing environments.
```

Note that a single space must appear after the `>` character on each line. Quoted blocks may themselves
contain other toplevel or inline elements.

#### Images

The syntax for images is similar to the link syntax mentioned above. Prepending a `!` character
to a link will display an image from the specified URL rather than a link to it.

```julia
![alternative text](link/to/image.png)
```

#### Lists

Unordered lists can be written by prepending each item in a list with either `*`, `+`, or `-`.

```
A list of items:

  * item one
  * item two
  * item three
```

Note the two spaces before each `*` and the single space after each one.

Lists can contain other nested toplevel elements such as lists, code blocks, or quoteblocks. A
blank line should be left between each list item when including any toplevel elements within a
list.

```
Another list:

  * item one

  * item two

    ```
    f(x) = x
    ```

  * And a sublist:

      + sub-item one
      + sub-item two
```

!!! note
    The contents of each item in the list must line up with the first line of the item. In the above
    example the fenced code block must be indented by four spaces to align with the `i` in `item two`.

Ordered lists are written by replacing the "bullet" character, either `*`, `+`, or `-`, with a
positive integer followed by either `.` or `)`.

```
Two ordered lists:

 1. item one
 2. item two
 3. item three

 5) item five
 6) item six
 7) item seven
```

An ordered list may start from a number other than one, as in the second list of the above example,
where it is numbered from five. As with unordered lists, ordered lists can contain nested toplevel
elements.

#### Display equations

Large ``\LaTeX`` equations that do not fit inline within a paragraph may be written as display
equations using a fenced code block with the "language" `math` as in the example below.

````julia
```math
f(a) = \frac{1}{2\pi}\int_{0}^{2\pi} (\alpha+R\cos(\theta))d\theta
```
````

#### Footnotes

This syntax is paired with the inline syntax for [Footnote references](@ref). Make sure to read
that section as well.

Footnote text is defined using the following syntax, which is similar to footnote reference syntax,
aside from the `:` character that is appended to the footnote label.

```
[^1]: Numbered footnote text.

[^note]:

    Named footnote text containing several toplevel elements.

      * item one
      * item two
      * item three

    ```julia
    function func(x)
        # ...
    end
    ```
```

!!! note
    No checks are done during parsing to make sure that all footnote references have matching footnotes.

#### Reglas horizontales

El equivalente de un tag HTML  `<hr>` puede escribirse usando la siguiente sintaxis:

```
Text above the line.

---

And text below the line.
```

#### Tablas

Pueden escribirse tablas básicas usando la sintaxis descrita debajo. Nótese que las tablas markdown tienen características limitadas y no pueden contener elementosde nivel superior anidados a diferencia de otros elementos discutidos antes – sólo se permiten elementos en línea. Las tablas deber siempre contener una fila cabecera con nombres de columnas. Las celdas no se pueden extender múltiples filas o columnas de la tabla.

```
| Column One | Column Two | Column Three |
|:---------- | ---------- |:------------:|
| Row `1`    | Column `2` |              |
| *Row* 2    | **Row** 2  | Column ``3`` |
```

!!! note
    As illustrated in the above example each column of `|` characters must be aligned vertically.

    A `:` character on either end of a column's header separator (the row containing `-` characters)
    specifies whether the row is left-aligned, right-aligned, or (when `:` appears on both ends) center-aligned.
    Providing no `:` characters will default to right-aligning the column.

#### Admoniciones

Los bloques especialmente formateados con títulos como "Notas", "Advertencia" o "Consejos" se conocen como advertencias y se utilizan cuando alguna parte de un documento necesita atención especial. Se pueden definir con la siguiente sintaxis `!!!`:

```
!!! note

    This is the content of the note.

!!! warning "Beware!"

    And this is another one.

    This warning admonition has a custom title: `"Beware!"`.
```

Las advertencias, como la mayoría de los otros elementos de nivel, pueden contener otros elementos de alto nivel. Cuando no se incluye texto de título, especificado después del tipo de advertencia entre comillas dobles, el título utilizado será el tipo del bloque, es decir, `" Nota "` en el caso de la advertencia de "nota".

## Extensiones de sintaxis Markdown

El Markdown de Julia admite la interpolación de una manera muy similar a los literales de cadena básicos, con la diferencia de que almacenará el objeto en el árbol Markdown (en lugar de convertirlo en una cadena). Cuando se procesa el contenido de Markdown, se invocarán los métodos `show` habituales, que pueden anularse como siempre. Este diseño permite que Markdown pueda ser extendido con características arbitrariamente complejas (como referencias) sin saturar la sintaxis básica.

En principio, el analizador de Markdown también se puede extender arbitrariamente por paquetes, o se puede usar un estilo de Markdown totalmente personalizado, pero esto generalmente no debería ser necesario.
