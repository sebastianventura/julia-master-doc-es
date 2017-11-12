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

  * [`endof(str)`](@ref) el índice máximo (byte) que se puede utilizar para indexar en `str`.
  * [`length(str)`](@ref) el número de caracteres en `str`.
  * [`i = start(str)`](@ref start) da el primer índice válido en el que se puede encontrar un carácter en `str (típicamente 1).
  * [`c, j = next(str,i)`](@ref next) devuelve el carácter siguiente en o después del índice `i` y el siguiente índice de carácter válido que sigue a éste. Con [`start()`](@ref) y [`endof()`](@ref), se puede utilizar para iterar a través de los caracteres en str`.
  * [`ind2chr(str,i)`](@ref) da el número de caracteres en `str` hasta e incluyendo cualquiera en el índice `i`.
  * [`chr2ind(str,j)`](@ref) da el índice en el cual ocurre el carácter `j`-ésimo en `str`.

## [Literales cadena no estándar](@id non-standard-string-literals)

Hay situaciones en las que se desea construir una cadena o utilizar semántica de cadenas, pero el comportamiento de la construcción de cadena estándar no es lo que se necesita. Para este tipo de situaciones, Julia proporciona [literales cadena no estándar](@ref). Un literal de cadena no estándar es como una cadena literal normal de doble comilla, pero va inmediatamente precedido de un identificador y no se comporta como un literal de cadena normal. El convenio es que los literales no estándar con prefijos en mayúsculas producen objetos cadena reales, mientras que aquellos con prefijos en minúsculas producen objetos que no cadena, como arrays de bytes o expresiones regulares compiladas. Las expresiones regulares, literales arrays de bytes y literales de números de versión, como se describe a continuación, son algunos ejemplos de literales de cadena no estándar. Otros ejemplos se dan en la sección [Metaprogramación](@ref).

## Expresiones Regulares

Julia tiene expresiones regulares compatibles con Perl (expresiones regulares), tal y como las proporciona la biblioteca [PCRE](http://www.pcre.org/). Las expresiones regulares se relacionan con las cadenas de dos maneras: la conexión obvia es que las expresiones regulares se utilizan para encontrar patrones regulares en cadenas; La otra conexión es que las expresiones regulares se introducen ellas mismas como cadenas, que se analizan en una máquina de estado que puede utilizarse para buscar patrones en cadenas de forma eficiente. En Julia, las expresiones regulares se introducen usando literales de cadena no estándar prefijados con varios identificadores comenzando por `r`. El literal de expresión regular más básico sin ninguna opción activada sólo utiliza `r"..."`:

```jldoctest
julia> r"^\s*(?:#|$)"
r"^\s*(?:#|$)"

julia> typeof(ans)
Regex
```

Para comprobar si una *regex* se corresponde con una cadena, se utiliza  [`ismatch()`](@ref):

```jldoctest
julia> ismatch(r"^\s*(?:#|$)", "not a comment")
false

julia> ismatch(r"^\s*(?:#|$)", "# a comment")
true
```

Como puede verse aquí, [`ismatch()`](@ref) simplemente devuelve `true` o `false`, indicando si la *regex* dada coincide o no con la cadena. Es común, sin embargo, que uno quiera saber no sólo si una cadena coincide, sino también *cómo* coincide. Para capturar esta información sobre una coincidencia, se utiliza la función [`match()`](@ref):

```jldoctest
julia> match(r"^\s*(?:#|$)", "not a comment")

julia> match(r"^\s*(?:#|$)", "# a comment")
RegexMatch("#")
```

Si la expresión regular no coincide con la cadena dada, [`match()`](@ref)  no devuelve nada` -- un valor especial que no imprime nada en el indicador interactivo. Aparte de no imprimir, es un valor completamente normal, como podemos comprobar en el siguiente código:

```julia
m = match(r"^\s*(?:#|$)", line)
if m === nothing
    println("not a comment")
else
    println("blank or comment")
end
```

Si la expresión regular coincide, el valor devuelto por [`match()`](@ref)  es un objeto `RegexMatch`. Estos objetos registran cómo coincide la expresión, incluyendo la subcadena que coincide con el patrón y cualquier subcadena capturada, si la hay. Este ejemplo sólo captura la parte de la subcadena que coincide, pero tal vez quisiéramos capturar cualquier texto no en blanco después del carácter de comentario. Podríamos hacer lo siguiente:

```jldoctest
julia> m = match(r"^\s*(?:#\s*(.*?)\s*$|$)", "# a comment ")
RegexMatch("# a comment ", 1="a comment")
```

Al invocar a [`match()`](@ref), tenemos la opción de especificar un índice en el que iniciar la búsqueda. Por ejemplo:

```jldoctest
julia> m = match(r"[0-9]","aaaa1aaaa2aaaa3",1)
RegexMatch("1")

julia> m = match(r"[0-9]","aaaa1aaaa2aaaa3",6)
RegexMatch("2")

julia> m = match(r"[0-9]","aaaa1aaaa2aaaa3",11)
RegexMatch("3")
```

Puede extraer la siguiente información de un objeto `RegexMatch`:

* La totalidad de la subcadena emparejada: `m.match`
* Las subcadenas capturadas como una matriz de cadenas: `m.captures`
* El desplazamiento en el que comienza la coincidencia del patrón: `m.offset`
* Los desplazamientos de las subcadenas capturadas como un vector: `m.offsets`

Para cuando una captura no coincide, en lugar de una subcadena, `m.captures` no contiene nada en esa posición, y `m.offsets` tiene un desplazamiento de cero (recuerde que los índices en Julia son *1-based*, por lo que un desplazamiento de cero en una cadena es inválido). Aquí hay un par de ejemplos algo artificiales:

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

Es conveniente que las capturas sean retornadas como un array para que uno pueda usar la sintaxis de desestructurante para enlazarlas a variables locales: 

```jldoctest acdmatch
julia> first, second, third = m.captures; first
"a"
```

Las capturas también está accesibles indexando el objeto `RegexMatch` con el número o nombre del grupo captura:

```jldoctest
julia> m=match(r"(?<hour>\d+):(?<minute>\d+)","12:45")
RegexMatch("12:45", hour="12", minute="45")

julia> m[:minute]
"45"

julia> m[2]
"45"
```

Las capturas pueden referenciarse en una cadena de sustitución cuando se utiliza [`replace()`](@ref) utilizando `\n` para referirse al grupo de captura `n`-ésimo y prefijando la cadena de subsitución con `s`. El grupo de captura `0` se refiere a todo el objeto de coincidencia. Los grupos de captura nombrados se pueden hacer referencia en la sustitución con g<groupname>. Por ejemplo:
 
```jldoctest
julia> replace("first second", r"(\w+) (?<agroup>\w+)", s"\g<agroup> \1")
"second first"
```

Los grupos de captura numerados pueden también ser referenciados como `\g<n>` para evitar ambigüedad, como en:
```jldoctest
julia> replace("a", r".", s"\g<0>1")
"a1"
```

Puedes modificar el comportamiento de las expresiones regulares mediante una combinación de los flags `i`, `m`, `s` y `x` después de la marca de comillas dobles de cierre. Estas banderas tienen el mismo significado que en Perl, tal y como se describe en este fragmento de la [página de manual del referencia de Perl(http://perldoc.perl.org/perlre.html#Modifiers):

```
i   Hace coincidencia de patrón insensible a mayúsculas y minúsculas.

    Si las reglas de concordancia de configuración local están en vigor, se toma el mapa de casos desde la ubicación 
    actual para puntos de código inferiores a 255, y desde reglas Unicode para puntos de código mayores. Sin embargo, 
    los partidos que cruzaría el límite de reglas Unicode / reglas no Unicode (Ords 255/256) no tendrá éxito.

m   Trate la cadena como varias líneas.  Es decir, cambiar "^" y "$" de coincidir con el inicio o el final de la cadena 
    con la inicio o fin de cualquier línea en cualquier parte de la cadena.

s   Trate la cadena como una sola línea.  Es decir, cambiar "." Para igualar cualquier carácter, incluso una nueva 
    línea, que normalmente sería No coinciden. 

    Utilizados juntos, como r "" ms, dejan que el "." Coincidir con cualquier personaje sea cual sea, mientras todavía 
    permite que "^" y "$" coincidan, respectivamente, justo después y justo antes de las nuevas líneas dentro de la cadena.

x   Indica al analizador de expresiones regulares que ignore la mayoría de los espacios en blanco. Que no está ni 
    retrocedido ni dentro de una clase de caracteres. Tú puede utilizar esto para romper su expresión regular en 
    (ligeramente) más legibles. El carácter '#' también es como un metacaracter introduciendo un comentario, como en 
    código ordinario.
```

Por ejemplo, la siguiente regex tiene activados los tres *flags*:

```jldoctest
julia> r"a+.*b+.*?d$"ism
r"a+.*b+.*?d$"ims

julia> match(r"a+.*b+.*?d$"ism, "Goodbye,\nOh, angry,\nBad world\n")
RegexMatch("angry,\nBad world")
```

Las cadenas *regex* con triples comillas, de la forma `r"""..."""` están también soportadas (y puede ser conveniente para expresiones regulares que contengan comillas o caracteres de salto de línea).

## [Byte Array Literals](@id man-byte-array-literals)

Otro literal de cadena no estándar útil es el literal de cadena de bytes: `b "..."`. Esta forma nos permite usar la notación de cadena para expresar arrays de bytes literales, es decir, arrays de valores [`UInt8`](@ref). Las reglas para los literales de arrays de bytes son las siguientes:

* Los caracteres ASCII y los escapes ASCII producen un solo byte.
* `\x` y las secuencias de escape octales producen el *byte* correspondiente al valor de escape.
* Las secuencias de escape Unicode producen una secuencia de bytes que codifican ese punto de código en UTF-8.

Hay una cierta superposición entre estas reglas ya que el comportamiento de `\x` y escapes octales menores de `0x80` (128) están cubiertos por las dos primeras reglas, pero aquí estas reglas están de acuerdo. Juntas, estas reglas permiten usar fácilmente caracteres ASCII, valores arbitrarios de bytes y secuencias UTF-8 para producir matrices de bytes. Aquí hay un ejemplo usando los tres:

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

La secuencia ASCII "DATA" corresponde a los bytes 68, 65, 84, 65. `\xff` produce el byte simple 255. El escape Unicode `\u2200` está codificado en UTF-8 como los tres bytes 226, 136, 128. Nótese que la matriz de bytes resultante no corresponde a una cadena UTF-8 válida - si intenta utilizar esto como una cadena literal normal, obtendrá un error de sintaxis:

```julia-repl
julia> "DATA\xff\u2200"
ERROR: syntax: invalid UTF-8 sequence
```

Observe también la distinción significativa entre `\xff` y `\uff`: la secuencia de escape anterior codifica el *byte 255*, mientras que la última secuencia de escape representa el *punto de código 255*, que se codifica como dos bytes en UTF-8:

```jldoctest
julia> b"\xff"
1-element Array{UInt8,1}:
 0xff

julia> b"\uff"
2-element Array{UInt8,1}:
 0xc3
 0xbf
```

En los literales de caracteres, esta distinción se pasa por alto y `\xff` está autorizado a  representar el punto de código 255, porque los caracteres *siempre* representan puntos de código. En las cadenas, sin embargo, los escapes `\x` siempre representan bytes, no puntos de código, mientras que los escapes `\u` y `\U` siempre representan puntos de código, que están codificados en uno o más bytes. Para los puntos de código inferiores a `\u80`, ocurre que la codificación UTF-8 de cada punto de código es sólo el byte producido por el escape `\x` correspondiente, por lo que la distinción puede ignorarse con seguridad. Sin embargo, para los escapes `\x80` a `\xff` en comparación con `\u80` a `\uff`, existe una diferencia importante: el primero escapa a todos los bytes sencillos de codificación, los cuales -a menos que sean seguidos por bytes de continuación muy específicos- no forman UTF-8 válido, mientras que los últimos escapes representan puntos de código Unicode con codificaciones de dos bytes.

Si todo esto es muy confuso, intente leer ["The Absolute Minimum Every
Software Developer Absolutely, Positively Must Know About Unicode and Character 
Sets"](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/). Es una excelente introducción a Unicode y UTF-8, y puede ayudar a aliviar cierta confusión sobre el asunto.

## [Literales Número de Versión](@id man-version-number-literals)

Los números de versión se pueden expresar fácilmente con literales de cadena no estándar del forma `v"..."`. Los literales de número de versión crean objetos `VersionNumber` que siguen las especificaciones del [control de versiones semánticas](http://semver.org/) y, por lo tanto, se componen de valores numéricos mayor, menor y de parche, seguidos de anotaciones alfanuméricas de pre-liberación y construcción. Por ejemplo, `v "0.2.1-rc1 + win64"` se divide en versión principal `0`, versión secundaria `2`, versión de revisión `1`, `rc1` de pre-lanzamiento y construcción `win64`. Al introducir una versión literal, todo excepto el número de versión principal es opcional, por ejemplo, `v"0.2"` es equivalente a `v"0.2.0"` (con anotaciones previas / de compilación vacías), `v"2"` equivale a `v"2.0.0"`, y así sucesivamente.

Los objetos `VersionNumber` son en su mayoría útiles para comparar fácilmente y correctamente dos (o más) versiones. Por ejemplo, la constante `VERSION` contiene el número de versión de Julia como un objeto `VersionNumber` y, por lo tanto, se puede definir algún comportamiento específico de la versión utilizando declaraciones simples como:

```julia
if v"0.2" <= VERSION < v"0.3-"
    # do something specific to 0.2 release series
end
```

Obsérvese que en el ejemplo anterior se utiliza el número de versión no estándar `v"0.3-"`, con un guión `-` en cola: esta notación es una extensión Julia del estándar, y se usa para indicar una versión que es más baja que cualquier versión `0.3`, Incluyendo todas sus pre-lanzamientos. Por lo tanto, en el ejemplo anterior, el código sólo se ejecuta con versiones estable `0.2` y excluye las versiones `v"0.3.0-rc1"`. Para permitir también versiones `0.2` inestables (es decir, pre-liberación), la verificación del límite inferior debería modificarse de la siguiente manera: `v"0.2-" <= VERSION`.

Otra extensión de especificación de versión no estándar permite usar un `+` de cola para expresar un límite superior en las versiones de compilación, por ej. `VERSIÓN> v "0.2-rc1 +"` se puede utilizar para significar cualquier versión por encima de `0.2-rc1` y cualquiera de sus compilaciones: devolverá `false` para la versión `v"0.2-rc1+win64"` y `true` para `v"0.2-rc2"`.

Es una buena práctica utilizar estas versiones especiales en comparaciones (en particular, el valor `-`de cola siempre debe utilizarse en los límites superiores a menos que haya una buena razón para no hacerlo), pero no deben utilizarse como el número de versión real de nada, ya que son inválidos en el esquema de versiones semánticas.

Además de ser utilizados por la constante [`VERSION`](@ref), los objetos `VersionNumber` son ampliamente utilizados en el módulo `Pkg`, para especificar las versiones de paquetes y sus dependencias.

## [Literales *Raw String*](@id man-raw-string-literals)

Las cadenas en bruto (*raw*) sin interpolación o *unescaping* pueden ser expresadas con literales cadena no estándar de la forma `raw"..."`. Los literales cadena en bruto crean objetos `String` ordinarios que contienen los contenidos encerrados exactamente como entrados sin interpolación ni separación. Esto es útil para cadenas que contiene código o marcado en otros idiomas que usan `$` o `\` como caracteres especiales. La excepción son las comillas que aún deben ser escapadas, por ejemplo,  `raw" \ "" `es equivalente a` "\" "`.
