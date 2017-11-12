# [Cadenas](@id man-strings)

Las cadenas son secuencias finitas de caracteres. Por supuesto, el verdadero problema viene cuando 
uno se pregunta qué es un carácter. Los caracteres con los que están familiarizados con los hablantes 
de inglés son las letras `A`, `B`, `C`, etc., junto con los números y los símbolos de puntuación 
comunes. Estos caracteres se estandarizan junto con una correspondencia a valores enteros entre 0 y 
127 a través del estándar ASCII. Hay, por supuesto, muchos otros caracteres utilizados en lenguas no 
inglesas, incluyendo variantes de los caracteres ASCII con acentos y otras modificaciones, escrituras 
relacionadas como cirílico y griego, y escrituras no relacionadas en abosluto con ASCII o inglés, entre 
los que se incluyen árabe, chino, Hebreo, hindi, japonés y coreano. El estándar [Unicode](https://en.wikipedia.org/wiki/Unicode) aborda las complejidades de lo que es exactamente un carácter, 
y es generalmente aceptado como el estándar definitivo que aborda este problema. Dependiendo de tus 
necesidades, puedes ignorar estas complejidades por completo y fingir que sólo existen caracteres ASCII, 
o puedes escribir código que pueda manejar cualquiera de los caracteres o codificaciones que se pueden 
encontrar al manejar texto no ASCII. Julia hace que el manejo de texto ASCII sencillo sea simple y 
eficiente, y el manejo de Unicode tan simple y eficiente como sea posible. En particular, puedes 
escribir código de cadenas con estilo C para procesar cadenas ASCII y funcionarán como se esperaba, 
tanto en términos de rendimiento como de semántica. Si dicho código encuentra texto no ASCII, fallará 
graciosamente con un mensaje de error claro, en lugar de introducir en silencio resultados corruptos. 
Cuando esto sucede, modificar el código para manejar datos no ASCII es sencillo.

Hay algunas características destacadas de alto nivel sobre las cadenas de caracteres en Julia:

  * El tipo de concreto incorporado utilizado para cadenas (y literales de cadena) en Julia es [`String`](@ref).
    Esto soporta el rango completo de caracteres  [Unicode](https://en.wikipedia.org/wiki/Unicode) a través de 
    la codificación [UTF-8](https://en.wikipedia.org/wiki/UTF-8). (se proporciona una función [`transcode()`](@ref)  
    para convertir a/desde otras codificaciones Unicode).
  * Todos los tipos de cadenas son subtipos del tipo abstracto `AbstractString` y los paquetes 
    externos definen subtipos `AbstractString` adicionales (por ejemplo, para otras codificaciones). 
    Si define una función que espera un argumento de cadena, debe declarar el tipo como 
    `AbstractString` para aceptar cualquier tipo de cadena.
  * Como C y Java, pero a diferencia de la mayoría de los lenguajes dinámicos, Julia tiene un tipo 
    de primera clase que representa un solo carácter, llamado `Char`. Esto es sólo un tipo especial 
    de bits de 32 bits cuyo valor numérico representa un punto de código Unicode.
  * Como en Java, las cadenas son inmutables: el valor de un objeto `AbstractString` no se puede 
    cambiar. Para construir un valor de cadena diferente, se construye una nueva cadena de partes 
    de otras cadenas.
  * Conceptualmente, una cadena es una *función parcial* de índices a caracteres: para algunos 
    valores de índice, no se devuelve ningún valor de carácter y, en su lugar, se genera una 
    excepción. Esto permite una indexación eficiente en cadenas por el índice de bytes de una 
    representación codificada en lugar de por un índice de caracteres, que no se puede implementar 
    de manera eficiente y sencilla para encodificaciones de anchura variable de cadenas Unicode.

## [Caracteres](@id man-characters)

Un valor Char representa un solo carácter: es sólo un bitstype de 32 bits con una representación literal especial y comportamientos aritméticos apropiados, cuyo valor numérico se interpreta como un [punto de código Unicode](https://en.wikipedia.org/wiki/Code_point). Aquí se muestra cómo se introducen y se muestran los valores `Char`:

```jldoctest
julia> 'x'
'x': ASCII/Unicode U+0078 (category Ll: Letter, lowercase)

julia> typeof(ans)
Char
```

Podemos convertir un `Char` a su valor entero (su punto de código) fácilmente:

```jldoctest
julia> Int('x')
120

julia> typeof(ans)
Int64
```

En arquitecturas de 32 bits,  [`typeof(ans)`](@ref) será [`Int32`](@ref). Puede convertir un valor entero de nuevo a un `Char` fácilmente:

```jldoctest
julia> Char(120)
'x': ASCII/Unicode U+0078 (category Ll: Letter, lowercase)
```

No todos los valores enteros son puntos de código Unicode válidos, pero por una cuestión de  rendimiento, la conversión `Char()` no comprueba que cada valor de carácter sea válido. Si desea comprobar que cada valor convertido es un punto de código válido, utilice la función [`isvalid()`](@ref):

```jldoctest
julia> Char(0x110000)
'\U110000': Unicode U+110000 (category Cn: Other, not assigned)

julia> isvalid(Char, 0x110000)
false
```

A partir de este momento, los puntos de código Unicode válidos son `U+00` a `U+d7ff` y `U+e000` a `U+10ffff`. A estos no se les han asignado todavía significados inteligibles, ni son necesariamente interpretables por las aplicaciones, pero todos ellos se consideran caracteres Unicode válidos.

Puede introducir cualquier carácter Unicode entre comillas simples utilizando `\u` seguido de hasta cuatro dígitos hexadecimales o `\U` seguido de hasta ocho dígitos hexadecimales (el valor válido más largo sólo requiere seis):

```jldoctest
julia> '\u0'
'\0': ASCII/Unicode U+0000 (category Cc: Other, control)

julia> '\u78'
'x': ASCII/Unicode U+0078 (category Ll: Letter, lowercase)

julia> '\u2200'
'∀': Unicode U+2200 (category Sm: Symbol, math)

julia> '\U10ffff'
'\U10ffff': Unicode U+10ffff (category Cn: Other, not assigned)
```

Julia utiliza la configuración regional y de idioma de tu sistema para determinar qué caracteres se pueden imprimir tal cual y cuáles se deben imprimir utilizando las formas de entrada genéricas, escapadas con `\u` o `\U`. Además de estas formas de escape de Unicode, también se pueden usar todas las [formas de entrada de escape tradicionales de C](https://en.wikipedia.org/wiki/C_syntax#Backslash_escapes):

```jldoctest
julia> Int('\0')
0

julia> Int('\t')
9

julia> Int('\n')
10

julia> Int('\e')
27

julia> Int('\x7f')
127

julia> Int('\177')
127

julia> Int('\xff')
255
```

Puedes hacer comparaciones y una cantidad limitada de aritmética con los valores `Char`:

```jldoctest
julia> 'A' < 'a'
true

julia> 'A' <= 'a' <= 'Z'
false

julia> 'A' <= 'X' <= 'Z'
true

julia> 'x' - 'a'
23

julia> 'A' + 1
'B': ASCII/Unicode U+0042 (category Lu: Letter, uppercase)
```

## Fundamentos de Cadenas

Los literales de cadenas están delimitados por comillas dobles o comillas dobles triples:

```jldoctest helloworldstring
julia> str = "Hello, world.\n"
"Hello, world.\n"

julia> """Contains "quote" characters"""
"Contains \"quote\" characters"
```

Si desea extraer un carácter de una cadena, indéxelo:

```jldoctest helloworldstring
julia> str[1]
'H': ASCII/Unicode U+0048 (category Lu: Letter, uppercase)

julia> str[6]
',': ASCII/Unicode U+002c (category Po: Punctuation, other)

julia> str[end]
'\n': ASCII/Unicode U+000a (category Cc: Other, control)
```

Toda la indexación en Julia está basada en 1: el primer elemento de cualquier objeto indexado medidante enteros se encuentra en el índice `1`, y el último elemento se encuentra en el índice `n`, cuando la cadena tiene una longitud de `n`.

En cualquier expresión de indexación, puede usarse la palabra clave `end` como una abreviatura para el último índice (calculado mediante [`endof(str)`](@ref)). Puede realizar operaciones aritméticas y otras con `end`, como si de un valor normal se tratara:

```jldoctest helloworldstring
julia> str[end-1]
'.': ASCII/Unicode U+002e (category Po: Punctuation, other)

julia> str[end÷2]
' ': ASCII/Unicode U+0020 (category Zs: Separator, space)
```

Usar un índice menor que 1 o mayor que `end` lanza un error:

```jldoctest helloworldstring
julia> str[0]
ERROR: BoundsError: attempt to access "Hello, world.\n"
  at index [0]
[...]

julia> str[end+1]
ERROR: BoundsError: attempt to access "Hello, world.\n"
  at index [15]
[...]
```

También puedes extraer una subcadena usando indexación mediante un rango:

```jldoctest helloworldstring
julia> str[4:9]
"lo, wo"
```

Nótese que las expresiones `str[k]` y `str[k:k]` no dan el mismo resultado:

```jldoctest helloworldstring
julia> str[6]
',': ASCII/Unicode U+002c (category Po: Punctuation, other)

julia> str[6:6]
","
```

La primera es un valor carácter de tipo `Char`, mientras que la segunda es un valor cadena que tiene un único carácter. En Julia se trata de cosas muy diferentes.happens to contain only a single character. In Julia these are very different things.

## Unicode y UTF-8

Julia soporta totalmente caracteres y cadenas Unicode. Como se ha [comentado anteriormente](@ref man-characters), en literales de caracteres, los puntos de código Unicode se pueden representar usando las secuencias de escape Unicode `\u` y `\U`, así como todas las secuencias de escape C estándar. Éstos también se pueden utilizar para escribir literales de cadena:
```jldoctest unicodestring

julia> s = "\u2200 x \u2203 y"
"∀ x ∃ y"
```

Si estos caracteres Unicode se muestran como escapes o se muestran como caracteres especiales depende de la configuración regional de tu terminal y su compatibilidad con Unicode. Los literales de cadena se codifican utilizando la codificación UTF-8. UTF-8 es una codificación de ancho variable, lo que significa que no todos los caracteres están codificados en el mismo número de bytes. En UTF-8, los caracteres ASCII -es decir, aquellos con puntos de código inferiores a 0x80 (128) - están codificados como lo están en ASCII, usando un solo byte, mientras que los puntos de código 0x80 y superiores se codifican utilizando múltiples bytes (hasta cuatro por carácter). Esto significa que no todos los índices de bytes en una cadena UTF-8 es necesariamente un índice válido para un carácter. Si indexas una cadena en un índice de bytes no válido, se genera un error:

```jldoctest unicodestring
julia> s[1]
'∀': Unicode U+2200 (category Sm: Symbol, math)

julia> s[2]
ERROR: UnicodeError: invalid character index
[...]

julia> s[3]
ERROR: UnicodeError: invalid character index
[...]

julia> s[4]
' ': ASCII/Unicode U+0020 (category Zs: Separator, space)
```

En este caso, el carácter ∀ es un carácter de tres bytes, por lo que los índices 2 y 3 no son válidos y el índice del siguiente carácter es 4; este siguiente índice válido puede ser calculado con [`nextind(s,1)`](@ref), y el siguiente índice después de éste con `nextind(s,4)` y así sucesivamente.

Debido a las codificaciones de longitud variable, el número de caracteres de una cadena (dada por [`length(s)`](@ref)) no siempre lo mismo que el último índice. Si se itera a través de los índices 1 hasta [`endof(s)`](@ref) y se indexa en `s`, la secuencia de caracteres devueltos cuando no se lanzan errores es la secuencia de caracteres que contiene la cadena `s`. Por tanto, tenemos la identidad de que `length(s) <= endof(s)`, ya que cada carácter en una cadena debe tener su propio índice. La siguiente es una forma ineficaz y verbosa de iterar a través de los caracteres de s:

```jldoctest unicodestring
julia> for i = 1:endof(s)
           try
               println(s[i])
           catch
               # ignore the index error
           end
       end
∀

x

∃

y
```

Las líneas en blanco en realidad tienen espacios en ellos. Afortunadamente, el idioma anterior incómodo es innecesario para iterar a través de los caracteres de una cadena, ya que se puede utilizar la cadena como un objeto iterable, sin que se requiera el manejo de excepciones:

```jldoctest unicodestring
julia> for c in s
           println(c)
       end
∀

x

∃

y
```

Julia utiliza la codificación UTF-8 de forma predeterminada y el soporte para nuevas codificaciones puede agregarse mediante paquetes. Por ejemplo, el paquete [LegacyStrings.jl](https://github.com/JuliaArchive/LegacyStrings.jl) implementa los tipos `UTF16String` y `UTF32String`. Una mayor discusión sobre otras codificaciones y cómo implementar el soporte para ellas está más allá del alcance de este documento por el momento. Para más información sobre los problemas de codificación UTF-8, consulte la sección siguiente sobre [literales byte array](@ref man-byte-array-literals). La función [`transcode()`](@ref) se proporciona para convertir datos entre las distintas codificaciones UTF-xx, principalmente para trabajar con datos y bibliotecas externas.

## Concatenation

Una de las operaciones de cadena más comunes y útiles es la concatenación:

```jldoctest stringconcat
julia> greet = "Hello"
"Hello"

julia> whom = "world"
"world"

julia> string(greet, ", ", whom, ".\n")
"Hello, world.\n"
```

Julia también proporciona el operador `*` para concatenar cadenas:

```jldoctest stringconcat
julia> greet * ", " * whom * ".\n"
"Hello, world.\n"
```

Aunque `*` puede parecer una opción sorprendente a los usuarios de lenguajes que proporcionan `+` para concatenación de cadenas, este uso de `*`tiene precedentes en matemáticas, particularmente en álgebra abstracta.

En matemáticas, `+` suele denotar una operación conmutativa, donde el orden de los operandos no importan. Un ejemplo de eesto es la suma de matrices, donde`A + B == B + A` para dos matrices cualesquiera `A` y `B` que tengan la misma forma. En contraste, `*` suele denotar una opración no conmutativa, donde el orden de los operandos importa. Un ejemplo de esto es la multiplicación de matrices donde, en general, `A * B != B * A`. Como con la multiplicación de matrices, la concatenación es no conmutativa: `greet * whom != whom * greet`. Por tanto, `*` es una elección más natural para el operador infijo de concatenación, consistente con el uso matemático común.

Más precisamente, el conjunto de todas las cadenas *S* de longitud finita junto con el operador de concatenación `*` forman un [monoide libre](https://en.wikipedia.org/wiki/Free_monoid) (S, `*`). El elemento identidad de este conjunto es la cadena vacía "". Siempre que un monoide libre es no conmutativo, la operación suele ser representada por `\cdot`, `*`, o un símbolo similar, en luga de con `+` que implica conmutatividad.

## [Interpolación](@id string-interpolation)

Construir cadenas mediante concatenación puede llegar a ser un poco engorroso, sin embargo. Para reducir la necesidad de estas llamadas verbosas a [`string()`](@ref) o multiplicaciones repetidas, Julia permite la interpolación en literales de cadena usando `$`, como en Perl:

```jldoctest stringconcat
julia> "$greet, $whom.\n"
"Hello, world.\n"
```

Esto es más legible y conveniente, y equivalente a la concatenación de cadena anterior - el sistema rescribe este aparente literal de cadena única en una concatenación de literales de cadena con variables.

La expresión completa más corta después de `$` se toma como la expresión cuyo valor debe ser interpolado en la cadena. Por lo tanto, puede interpolar cualquier expresión en una cadena usando paréntesis:

```jldoctest
julia> "1 + 2 = $(1 + 2)"
"1 + 2 = 3"
```

Tanto la concatenación como la interpolación de cadena llaman a [`string()`](@ref) para convertir objetos al formato de cadena. La mayoría de los objetos que no son `AbstractString` se convierten en cadenas que se corresponden estrechamente con la forma en que se introducen como expresiones literales:

```jldoctest
julia> v = [1,2,3]
3-element Array{Int64,1}:
 1
 2
 3

julia> "v: $v"
"v: [1, 2, 3]"
```

[`string()`](@ref) es la identidad para los valores `AbstractString` y `Char` values, por lo que estos se interpolan en cadenas como ellos mismos, sin entrecomilla y sin escapar:

```jldoctest
julia> c = 'x'
'x': ASCII/Unicode U+0078 (category Ll: Letter, lowercase)

julia> "hi, $c"
"hi, x"
```

Para incluir un literal `$` en una cadena, lo escaparemos con un backslash:

```jldoctest
julia> print("I have \$100 in my account.\n")
I have $100 in my account.
```

## Literales cadena con triples comillas

Cuando las cadenas se crean utilizando comillas triples (`"""..."""`) tienen un comportamiento especial que puede ser útil para crear bloques de texto más largos. En primer lugar, si la apertura """ es seguida por una nueva línea, la nueva línea se quita de la cadena resultante:

```julia
"""hello"""
```

es equivalente a

```julia
"""
hello"""
```

pero

```julia
"""

hello"""
```

contendrá un literal *new line* al principio. Los espacios en blanco no se modifican. Pueden contener símbolos `"` sin escapar. Las cadenas de triple comilla también se dedican al nivel de la línea menos indentada. Esto es útil para definir cadenas dentro del código que está sangrado Por ejemplo:   

```jldoctest
julia> str = """
           Hello,
           world.
         """
"  Hello,\n  world.\n"
```

En este caso la línea final (vacía) antes del cierre `"""` establece el nivel de indentación.

Tenga en cuenta que las saltos de línea en cadenas literales, sean de una sola o triple comilla, resultan en un carácter de línea nueva (LF) `\n` en la cadena, incluso si su editor usa una combinación de retorno de carro (CR) o CRLF para finalizar líneas. Para incluir un CR en una cadena, utilice un escape explícito `\r`; Por ejemplo, puede introducir la cadena literal `"una línea CRLF que termina \r \n"`.

## Operaciones comunes

Los operadores de comparación estándar permiten comparar cadenas lexicográficamente:

```jldoctest
julia> "abracadabra" < "xylophone"
true

julia> "abracadabra" == "xylophone"
false

julia> "Hello, world." != "Goodbye, world."
true

julia> "1 + 2 = 3" == "1 + 2 = $(1 + 2)"
true
```

La función [`search()`](@ref) permite buscar el índice de una carácter en una cadena:

```jldoctest
julia> search("xylophone", 'x')
1

julia> search("xylophone", 'p')
5

julia> search("xylophone", 'z')
0
```

Y se puede arrancar la búsqueda de un carácter a partir de un desplazamiento proporcionado por un tercer argumento:

```jldoctest
julia> search("xylophone", 'o')
4

julia> search("xylophone", 'o', 5)
7

julia> search("xylophone", 'o', 8)
0
```

La función [`contains()`](@ref) se usa para comprobar si una subcadena está contenida en una cadena:

```jldoctest
julia> contains("Hello, world.", "world")
true

julia> contains("Xylophon", "o")
true

julia> contains("Xylophon", "a")
false

julia> contains("Xylophon", 'o')
ERROR: MethodError: no method matching contains(::String, ::Char)
Closest candidates are:
  contains(!Matched::Function, ::Any, !Matched::Any) at reduce.jl:664
  contains(::AbstractString, !Matched::AbstractString) at strings/search.jl:378
```

Este último error es debido a que 'o'  es un literal carácter, y [`contains()`](@ref) es una función genérica que busca subsecuencias. Para buscar un elemento en una secuencia, debemos usar la función [`in()`](@ref) en lugra de la anterior.

instead.
[`repeat()`](@ref) y  [`join()`](@ref) son otras dos funciones de cadena muy útiles:

```jldoctest
julia> repeat(".:Z:.", 10)
".:Z:..:Z:..:Z:..:Z:..:Z:..:Z:..:Z:..:Z:..:Z:..:Z:."

julia> join(["apples", "bananas", "pineapples"], ", ", " and ")
"apples, bananas and pineapples"
```

Algunas otras funciones útiles son:

  * [`endof(str)`](@ref) gives the maximal (byte) index that can be used to index into `str`.
  * [`length(str)`](@ref) the number of characters in `str`.
  * [`i = start(str)`](@ref start) gives the first valid index at which a character can be found in `str`
    (typically 1).
  * [`c, j = next(str,i)`](@ref next) returns next character at or after the index `i` and the next valid
    character index following that. With [`start()`](@ref) and [`endof()`](@ref), can be used to iterate
    through the characters in `str`.
  * [`ind2chr(str,i)`](@ref) gives the number of characters in `str` up to and including any at index
    `i`.
  * [`chr2ind(str,j)`](@ref) gives the index at which the `j`th character in `str` occurs.

## [Non-Standard String Literals](@id non-standard-string-literals)

There are situations when you want to construct a string or use string semantics, but the behavior
of the standard string construct is not quite what is needed. For these kinds of situations, Julia
provides [non-standard string literals](@ref). A non-standard string literal looks like a regular
double-quoted string literal, but is immediately prefixed by an identifier, and doesn't behave
quite like a normal string literal.  Regular expressions, byte array literals and version number
literals, as described below, are some examples of non-standard string literals. Other examples
are given in the [Metaprogramming](@ref) section.

## Regular Expressions

Julia has Perl-compatible regular expressions (regexes), as provided by the [PCRE](http://www.pcre.org/)
library. Regular expressions are related to strings in two ways: the obvious connection is that
regular expressions are used to find regular patterns in strings; the other connection is that
regular expressions are themselves input as strings, which are parsed into a state machine that
can be used to efficiently search for patterns in strings. In Julia, regular expressions are input
using non-standard string literals prefixed with various identifiers beginning with `r`. The most
basic regular expression literal without any options turned on just uses `r"..."`:

```jldoctest
julia> r"^\s*(?:#|$)"
r"^\s*(?:#|$)"

julia> typeof(ans)
Regex
```

To check if a regex matches a string, use [`ismatch()`](@ref):

```jldoctest
julia> ismatch(r"^\s*(?:#|$)", "not a comment")
false

julia> ismatch(r"^\s*(?:#|$)", "# a comment")
true
```

As one can see here, [`ismatch()`](@ref) simply returns true or false, indicating whether the
given regex matches the string or not. Commonly, however, one wants to know not just whether a
string matched, but also *how* it matched. To capture this information about a match, use the
[`match()`](@ref) function instead:

```jldoctest
julia> match(r"^\s*(?:#|$)", "not a comment")

julia> match(r"^\s*(?:#|$)", "# a comment")
RegexMatch("#")
```

If the regular expression does not match the given string, [`match()`](@ref) returns `nothing`
-- a special value that does not print anything at the interactive prompt. Other than not printing,
it is a completely normal value and you can test for it programmatically:

```julia
m = match(r"^\s*(?:#|$)", line)
if m === nothing
    println("not a comment")
else
    println("blank or comment")
end
```

If a regular expression does match, the value returned by [`match()`](@ref) is a `RegexMatch`
object. These objects record how the expression matches, including the substring that the pattern
matches and any captured substrings, if there are any. This example only captures the portion
of the substring that matches, but perhaps we want to capture any non-blank text after the comment
character. We could do the following:

```jldoctest
julia> m = match(r"^\s*(?:#\s*(.*?)\s*$|$)", "# a comment ")
RegexMatch("# a comment ", 1="a comment")
```

When calling [`match()`](@ref), you have the option to specify an index at which to start the
search. For example:

```jldoctest
julia> m = match(r"[0-9]","aaaa1aaaa2aaaa3",1)
RegexMatch("1")

julia> m = match(r"[0-9]","aaaa1aaaa2aaaa3",6)
RegexMatch("2")

julia> m = match(r"[0-9]","aaaa1aaaa2aaaa3",11)
RegexMatch("3")
```

You can extract the following info from a `RegexMatch` object:

  * the entire substring matched: `m.match`
  * the captured substrings as an array of strings: `m.captures`
  * the offset at which the whole match begins: `m.offset`
  * the offsets of the captured substrings as a vector: `m.offsets`

For when a capture doesn't match, instead of a substring, `m.captures` contains `nothing` in that
position, and `m.offsets` has a zero offset (recall that indices in Julia are 1-based, so a zero
offset into a string is invalid). Here is a pair of somewhat contrived examples:

```jldoctest acdmatch
julia> m = match(r"(a|b)(c)?(d)", "acd")
RegexMatch("acd", 1="a", 2="c", 3="d")

julia> m.match
"acd"

julia> m.captures
3-element Array{Union{SubString{String}, Void},1}:
 "a"
 "c"
 "d"

julia> m.offset
1

julia> m.offsets
3-element Array{Int64,1}:
 1
 2
 3

julia> m = match(r"(a|b)(c)?(d)", "ad")
RegexMatch("ad", 1="a", 2=nothing, 3="d")

julia> m.match
"ad"

julia> m.captures
3-element Array{Union{SubString{String}, Void},1}:
 "a"
 nothing
 "d"

julia> m.offset
1

julia> m.offsets
3-element Array{Int64,1}:
 1
 0
 2
```

It is convenient to have captures returned as an array so that one can use destructuring syntax
to bind them to local variables:

```jldoctest acdmatch
julia> first, second, third = m.captures; first
"a"
```

Captures can also be accessed by indexing the `RegexMatch` object with the number or name of the
capture group:

```jldoctest
julia> m=match(r"(?<hour>\d+):(?<minute>\d+)","12:45")
RegexMatch("12:45", hour="12", minute="45")

julia> m[:minute]
"45"

julia> m[2]
"45"
```

Captures can be referenced in a substitution string when using [`replace()`](@ref) by using `\n`
to refer to the nth capture group and prefixing the subsitution string with `s`. Capture group
0 refers to the entire match object. Named capture groups can be referenced in the substitution
with `g<groupname>`. For example:

```jldoctest
julia> replace("first second", r"(\w+) (?<agroup>\w+)", s"\g<agroup> \1")
"second first"
```

Numbered capture groups can also be referenced as `\g<n>` for disambiguation, as in:

```jldoctest
julia> replace("a", r".", s"\g<0>1")
"a1"
```

You can modify the behavior of regular expressions by some combination of the flags `i`, `m`,
`s`, and `x` after the closing double quote mark. These flags have the same meaning as they do
in Perl, as explained in this excerpt from the [perlre manpage](http://perldoc.perl.org/perlre.html#Modifiers):

```
i   Do case-insensitive pattern matching.

    If locale matching rules are in effect, the case map is taken
    from the current locale for code points less than 255, and
    from Unicode rules for larger code points. However, matches
    that would cross the Unicode rules/non-Unicode rules boundary
    (ords 255/256) will not succeed.

m   Treat string as multiple lines.  That is, change "^" and "$"
    from matching the start or end of the string to matching the
    start or end of any line anywhere within the string.

s   Treat string as single line.  That is, change "." to match any
    character whatsoever, even a newline, which normally it would
    not match.

    Used together, as r""ms, they let the "." match any character
    whatsoever, while still allowing "^" and "$" to match,
    respectively, just after and just before newlines within the
    string.

x   Tells the regular expression parser to ignore most whitespace
    that is neither backslashed nor within a character class. You
    can use this to break up your regular expression into
    (slightly) more readable parts. The '#' character is also
    treated as a metacharacter introducing a comment, just as in
    ordinary code.
```

For example, the following regex has all three flags turned on:

```jldoctest
julia> r"a+.*b+.*?d$"ism
r"a+.*b+.*?d$"ims

julia> match(r"a+.*b+.*?d$"ism, "Goodbye,\nOh, angry,\nBad world\n")
RegexMatch("angry,\nBad world")
```

Triple-quoted regex strings, of the form `r"""..."""`, are also supported (and may be convenient
for regular expressions containing quotation marks or newlines).

## [Byte Array Literals](@id man-byte-array-literals)

Another useful non-standard string literal is the byte-array string literal: `b"..."`. This form
lets you use string notation to express literal byte arrays -- i.e. arrays of
[`UInt8`](@ref) values. The rules for byte array literals are the following:

  * ASCII characters and ASCII escapes produce a single byte.
  * `\x` and octal escape sequences produce the *byte* corresponding to the escape value.
  * Unicode escape sequences produce a sequence of bytes encoding that code point in UTF-8.

There is some overlap between these rules since the behavior of `\x` and octal escapes less than
0x80 (128) are covered by both of the first two rules, but here these rules agree. Together, these
rules allow one to easily use ASCII characters, arbitrary byte values, and UTF-8 sequences to
produce arrays of bytes. Here is an example using all three:

```jldoctest
julia> b"DATA\xff\u2200"
8-element Array{UInt8,1}:
 0x44
 0x41
 0x54
 0x41
 0xff
 0xe2
 0x88
 0x80
```

The ASCII string "DATA" corresponds to the bytes 68, 65, 84, 65. `\xff` produces the single byte 255.
The Unicode escape `\u2200` is encoded in UTF-8 as the three bytes 226, 136, 128. Note that the
resulting byte array does not correspond to a valid UTF-8 string -- if you try to use this as
a regular string literal, you will get a syntax error:

```julia-repl
julia> "DATA\xff\u2200"
ERROR: syntax: invalid UTF-8 sequence
```

Also observe the significant distinction between `\xff` and `\uff`: the former escape sequence
encodes the *byte 255*, whereas the latter escape sequence represents the *code point 255*, which
is encoded as two bytes in UTF-8:

```jldoctest
julia> b"\xff"
1-element Array{UInt8,1}:
 0xff

julia> b"\uff"
2-element Array{UInt8,1}:
 0xc3
 0xbf
```

In character literals, this distinction is glossed over and `\xff` is allowed to represent the
code point 255, because characters *always* represent code points. In strings, however, `\x` escapes
always represent bytes, not code points, whereas `\u` and `\U` escapes always represent code points,
which are encoded in one or more bytes. For code points less than `\u80`, it happens that the
UTF-8 encoding of each code point is just the single byte produced by the corresponding `\x` escape,
so the distinction can safely be ignored. For the escapes `\x80` through `\xff` as compared to
`\u80` through `\uff`, however, there is a major difference: the former escapes all encode single
bytes, which -- unless followed by very specific continuation bytes -- do not form valid UTF-8
data, whereas the latter escapes all represent Unicode code points with two-byte encodings.

If this is all extremely confusing, try reading ["The Absolute Minimum Every
Software Developer Absolutely, Positively Must Know About Unicode and Character
Sets"](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/).
It's an excellent introduction to Unicode and UTF-8, and may help alleviate
some confusion regarding the matter.

## [Version Number Literals](@id man-version-number-literals)

Version numbers can easily be expressed with non-standard string literals of the form `v"..."`.
Version number literals create `VersionNumber` objects which follow the specifications of [semantic versioning](http://semver.org),
and therefore are composed of major, minor and patch numeric values, followed by pre-release and
build alpha-numeric annotations. For example, `v"0.2.1-rc1+win64"` is broken into major version
`0`, minor version `2`, patch version `1`, pre-release `rc1` and build `win64`. When entering
a version literal, everything except the major version number is optional, therefore e.g.  `v"0.2"`
is equivalent to `v"0.2.0"` (with empty pre-release/build annotations), `v"2"` is equivalent to
`v"2.0.0"`, and so on.

`VersionNumber` objects are mostly useful to easily and correctly compare two (or more) versions.
For example, the constant `VERSION` holds Julia version number as a `VersionNumber` object, and
therefore one can define some version-specific behavior using simple statements as:

```julia
if v"0.2" <= VERSION < v"0.3-"
    # do something specific to 0.2 release series
end
```

Note that in the above example the non-standard version number `v"0.3-"` is used, with a trailing
`-`: this notation is a Julia extension of the standard, and it's used to indicate a version which
is lower than any `0.3` release, including all of its pre-releases. So in the above example the
code would only run with stable `0.2` versions, and exclude such versions as `v"0.3.0-rc1"`. In
order to also allow for unstable (i.e. pre-release) `0.2` versions, the lower bound check should
be modified like this: `v"0.2-" <= VERSION`.

Another non-standard version specification extension allows one to use a trailing `+` to express
an upper limit on build versions, e.g.  `VERSION > v"0.2-rc1+"` can be used to mean any version
above `0.2-rc1` and any of its builds: it will return `false` for version `v"0.2-rc1+win64"` and
`true` for `v"0.2-rc2"`.

It is good practice to use such special versions in comparisons (particularly, the trailing `-`
should always be used on upper bounds unless there's a good reason not to), but they must not
be used as the actual version number of anything, as they are invalid in the semantic versioning
scheme.

Besides being used for the [`VERSION`](@ref) constant, `VersionNumber` objects are widely used
in the `Pkg` module, to specify packages versions and their dependencies.

## [Raw String Literals](@id man-raw-string-literals)

Raw strings without interpolation or unescaping can be expressed with
non-standard string literals of the form `raw"..."`. Raw string literals create
ordinary `String` objects which contain the enclosed contents exactly as
entered with no interpolation or unescaping. This is useful for strings which
contain code or markup in other languages which use `$` or `\` as special
characters. The exception is quotation marks that still must be
escaped, e.g. `raw"\""` is equivalent to `"\""`.
