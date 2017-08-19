# [Funciones](@id man-functions)

En Julia, una función es un objeto que hace corresponde una tupla de valores argumentos en
un valor de retorno. Las funciones de Julia no son funciones matemáticas puras, en el sentido 
de que pueden alterar y ser afectadas por el estado global del programa. La sintaxis básica 
a definir funciones en Julia es:

```jldoctest
julia> function f(x,y)
           x + y
       end
f (generic function with 1 method)
```

Hay una segunda sintaxis, más concisa, para definir una función en Julia. La declaración de 
función tradicional mostrada anteriormente es equivalente a la denominada "forma de asignación". 
Por ejemplo:

```jldoctest fofxy
julia> f(x,y) = x + y
f (generic function with 1 method)
```

En esta segunda forma, el cuerpo de la función debe ser una sola expresión, aunque puede 
tratarse de una expresión compuesta (see [Expresiones Compuestas](@ref man-compound-expressions)). 
Estas definiciones de función cortas y simples son comunes en Julia. La sintaxis de funciones
cortas es, por tanto, bastante idiomática, reduciendo considerablemente tanto la escritura como 
el ruido visual.

Para invocar una función se usa la sintaxis tradicional basada en el uso del paréntesis:

```jldoctest fofxy
julia> f(2,3)
5
```

Sin usar paréntesis, la expresión `f` se refiere al objeto función, y puede ser tratada como 
cualquier otro valor:

```jldoctest fofxy
julia> g = f;

julia> g(2,3)
5
```

Y, como en el caso de las variables, podemos usar Unicode en el caso de los nombres de función:

```jldoctest
julia> ∑(x,y) = x + y
∑ (generic function with 1 method)

julia> ∑(2, 3)
5
```

## Comportamiento del Paso de Argumentos

Los argumentos de función en Julia siguen un convenio denominado a veces "paso por compartición", 
que significa que los valores no son copiados cuando se pasan a las funciones. Los argumentos de 
las funciones actúan ellos mismos como nuevos enlaces a variable (nuevas localizaciones que pueden 
referirse a valores) pero los valores a los que se refieren son idénticos a los valores pasados. 
Las modificaciones a valores mutables (tales como los Arrays) hechos dentro de la función serán 
visibles desde fuera de ésta. Este es el mismo comportamiento que presenta Scheme, la mayoría de 
versiones de Lisp, Python, Ruby y Perl, entre otros lenguajes dinámicos.

## La palabra clave `return`

El valor devuelto por una función es el valor de la última expresión evaluada, el cual, por 
defecto, es la última expresión en el cuerpo de definición de la función. En la función `f`, 
mostrada en la sección anterior, el valor devuelto sería la suma `x + y`. Como en C y la mayoría 
de los demás lenguajes imperativos o funcionales, la palabra clave `return` causa que la función 
retorne inmediatamente, proporcionando una función cuyo valor es devuelto:

```julia
function g(x,y)
    return x * y
    x + y
end
```

Como las definiciones a función pueden ser introducidas en una sesión interactiva, es muy sencillo 
comparar estas definiciones:

```jldoctest
julia> f(x,y) = x + y
f (generic function with 1 method)

julia> function g(x,y)
           return x * y
           x + y
       end
g (generic function with 1 method)

julia> f(2,3)
5

julia> g(2,3)
6
```

Por supuesto, en una función con cuerpo puramente lineal como `g`, el uso de `return` es irrelevante 
ya que la expresión `x+y` nunca va a ser evaluada, por lo que podríamos hacer que `x*y` fuese la última 
línea de la función y omitir el `return`. Sin embargo, cuando hacemos uso de esta instrucción junto 
con otras de control de flujo, el resultado puede ser muy interesante. Aquí, por ejemplo, hay una 
función que calcula la longitud de la hipotenusa de un triángulo equilátero correcto con catetos de
longitudes `x` e `y`, evitando un desbordamiento:

```jldoctest
julia> function hypot(x,y)
           x = abs(x)
           y = abs(y)
           if x > y
               r = y/x
               return x*sqrt(1+r*r)
           end
           if y == 0
               return zero(x)
           end
           r = x/y
           return y*sqrt(1+r*r)
       end
hypot (generic function with 1 method)

julia> hypot(3, 4)
5.0
```

Hay tres posibles puntos de retorno en esta función, devolviendo los valores de tres expresiones 
diferentes, dependiendo de los valores de `x` e `y`. El `return` de la última línea podría ser 
omitido ya que es la última expresión.

## Los Operadores son Funciones

En Julia, la mayoría de los operadores son funciones con soporte para una sintaxis especial (la 
excepción a esta regla son las operaciones con una semática de evaluación especial, tales como 
`&&` y `||`. Estos operadores no pueden ser funciones porque la [Evaluacin en Cortocircuito]
(@ref) requiere que sus operandos no sean evaluados antes de la evaluación
del operador). De acuerdo con ésto, podemos usar listas de argumentos entre paréntesis, tal como 
en cualquier otra función:

```jldoctest
julia> 1 + 2 + 3
6

julia> +(1,2,3)
6
```

La forma infija es equivalente a la forma de aplicación función. De hecho, la primera es transformada 
para producir la llamada a función internamente. Esto también significa que puedes asignar y pasar 
operadores tales como [`+()`](@ref) y [`*()`](@ref) , tal y como se hace con otros valores función:

```jldoctest
julia> f = +;

julia> f(1,2,3)
6
```

Sin embargo, cuando se usa el formato de función, como `f`, no se puede usar notación infija.

## Operators With Special Names

A few special expressions correspond to calls to functions with non-obvious names. These are:

| Expresión         | Llamada                |
|:----------------- |:---------------------- |
| `[A B C ...]`     | [`hcat()`](@ref)       |
| `[A; B; C; ...]`  | [`vcat()`](@ref)       |
| `[A B; C D; ...]` | [`hvcat()`](@ref)      |
| `A'`              | [`ctranspose()`](@ref) |
| `A.'`             | [`transpose()`](@ref)  |
| `1:n`             | [`colon()`](@ref)      |
| `A[i]`            | [`getindex()`](@ref)   |
| `A[i]=x`          | [`setindex!()`](@ref)  |

## [Funciones Anónimas](@id man-anonymous-functions)

Las funciones en Julia son [objetos de primera clase](https://en.wikipedia.org/wiki/First-class_citizen):
ellas pueden ser asignadas a variables y ser invocadas usando la sintaxis estándar de llamadas 
a función desde la variable a la que han sido asignadas. Ellas pueden ser usadas como argumentos 
y ser devueltas como valores. Ellas pueden también ser usadas de forma anónima sin dárseles un 
nombre, usando alguna de estas sintaxis:

```jldoctest
julia> x -> x^2 + 2x - 1
(::#1) (generic function with 1 method)

julia> function (x)
           x^2 + 2x - 1
       end
(::#3) (generic function with 1 method)
```

Esto crea una función que toma un argumento `x` y devuelve el valor del polinomio `x^2 + 2x - 1`. 
Nótese que el resultado es una función genérica, pero con un nombre generado por el compilador 
basado en una numeración consecutiva.

El uso primario de las funciones anónimas es pasarlas a funciones que toman otras funciones como 
argumentos. Un ejemplo clásico es [`map()`](@ref), , que aplica una función a cada valor de un 
array y devuelve un nuevo array que contienen los valores resultantes:

```jldoctest
julia> map(round, [1.2,3.5,1.7])
3-element Array{Float64,1}:
 1.0
 4.0
 2.0
```

Esto está bien si ya exite una función que efectúa la transformación que uno desea para pasarla 
como primer argumento de [`map()`](@ref). Sin embargo, no es frecuente que exista este tipo de 
función. En estas situaciones, el constructor de la función anónima permite una fácil creación 
de un objeto función de un solo uso sin necesidad de asignarle un nombre:

```jldoctest
julia> map(x -> x^2 + 2x - 1, [1,3,-1])
3-element Array{Int64,1}:
  2
 14
 -2
```

Para escribir funciones anónimas que aceptan múltiples argumentos puede utilizarse la sintaxis 
`(x,y,z) -> 2x + y +z`. Una función anónima con cero argumentos se escribe como `() -> 3`.  La 
idea de una funci´n sin argumentos puede parecer extraña, pero es útil para demorar un cálculo. 
En este uso, un bloque de código es envuelto en una función con cero argumentos, el cual es 
después invocado mediante una llamada como `f()`.

## Multiple Return Values

En Julia, uno devuelve una tupla para simular el retorno de múltiples valores. Sin embargo, como
las tuplas puede salteadas y destruidas sin necesitar paréntesis, podemos proporcionar una ilusión 
de que se están devolviendo múltiples valores. Por ejemplo, la siguiente función devuelve un par 
de valores:

```jldoctest foofunc
julia> function foo(a,b)
           a+b, a*b
       end
foo (generic function with 1 method)
```

Si invocamos esta función en una sesión interactiva sin asignar los valores en ningún sitio, 
comprobaremos que la función devuelve una tupla:

```jldoctest foofunc
julia> foo(2,3)
(5, 6)
```

Un uso típico de tal par de valores devueltos es extraer cada valor en una variable. Julia 
soporta la "desestructuración" simple de una tupla que facilita esto:

```jldoctest foofunc
julia> x, y = foo(2,3)
(5, 6)

julia> x
5

julia> y
6
```

Y también podemos devolver múltiples valores mediante el uso explícito de la palabra clave
`return`:

```julia
function foo(a,b)
    return a+b, a*b
end
```

Esto tiene exactamente el mismo efecto que la definicin anterior de `foo`.

## Funciones con argumentos variables (varargs)

Suele ser muy conveniente ser capaz de escribir funciones que toman un número arbitrario de 
argumentos. Estas funciones se conocen como *funciones vararg*. Podemos definir funciones de tal 
tipo poniendo puntos suspensivos `…` después del último argumento.

```jldoctest barfunc
julia> bar(a,b,x...) = (a,b,x)
bar (generic function with 1 method)
```

Las variables `a` y `b` están asociadas a los dos primeros argumentos como es natural, y la variable 
`x` se asocia a una colección, iterable de cero o más valores pasados a la función `bar` después de 
estos dos argumentos:

```jldoctest barfunc
julia> bar(1,2)
(1, 2, ())

julia> bar(1,2,3)
(1, 2, (3,))

julia> bar(1, 2, 3, 4)
(1, 2, (3, 4))

julia> bar(1,2,3,4,5,6)
(1, 2, (3, 4, 5, 6))
```

En todos los casos, `x` es asociada a una tupla con el resto de valores pasados a la función.

Es posible restringir el número de argumentos pasados como argumento variable. Esto se discutirá 
más adelante en la sección [métodos *vararg* restringidos paramétricamente](@ref).

Como contraposición, es frecuente manejar la división de los valores contenidos en una colección 
iterable en una llamada a función como argumentos individuales. Para hacer eso, se utilizará la 
notación de puntos suspensivos, pero esta vez en la llamada a función.

```jldoctest barfunc
julia> x = (3, 4)
(3, 4)

julia> bar(1,2,x...)
(1, 2, (3, 4))
```

En este caso hay una tupla que se divide en una llamada *vararg* precisamente donde está el 
número de argumentos variable. Esa necesidad no tiene por qué ser el caso:

```jldoctest barfunc
julia> x = (2, 3, 4)
(2, 3, 4)

julia> bar(1,x...)
(1, 2, (3, 4))

julia> x = (1, 2, 3, 4)
(1, 2, 3, 4)

julia> bar(x...)
(1, 2, (3, 4))
```

Además, el objeto iterable dividido durante la llamada a función no tiene que ser una tupla:

```jldoctest barfunc
julia> x = [3,4]
2-element Array{Int64,1}:
 3
 4

julia> bar(1,2,x...)
(1, 2, (3, 4))

julia> x = [1,2,3,4]
4-element Array{Int64,1}:
 1
 2
 3
 4

julia> bar(x...)
(1, 2, (3, 4))
```

También, la función cuyos argumentos son divididos no tiene por qué ser una función *vararg* 
(aunque frecuentemente lo sea):

```jldoctest
julia> baz(a,b) = a + b;

julia> args = [1,2]
2-element Array{Int64,1}:
 1
 2

julia> baz(args...)
3

julia> args = [1,2,3]
3-element Array{Int64,1}:
 1
 2
 3

julia> baz(args...)
ERROR: MethodError: no method matching baz(::Int64, ::Int64, ::Int64)
Closest candidates are:
  baz(::Any, ::Any) at none:1
```

Como puede comprobarse, si el número de elementos que se van a sacar del contenedor es inapropiado 
para pasar a la función como argumentos, se generará un error, igual que si hubiéramos realizado la 
llamada a función con un número de argumentos inapropiado.

## Argumentos Opcionales

En muchos casos, los argumentos de función tienen valores por defecto sensibles y, por tanto, puede 
no ser necesario que se pasen explícitamente en cada llamada. Por ejemplo, la función de librería 
[`parse(type, num, base`](@ref) interpreta una cadena como un número en cierta base. El argumento 
`base` tiene un valor por defecto de `10`. Este comportamiento puede expresarse de forma concisa 
como:

```julia
function parse(T, num, base=10)
    ###
end
```

Con esta definición, la función puede ser llamada con dos o tres argumentos y, cuando no se pase 
el tercer argumento, la función asignara el valor por defecto de `10` al parámetro `base`.

```jldoctest
julia> parse(Int,"12",10)
12

julia> parse(Int,"12",3)
5

julia> parse(Int,"12")
12
```

Los argumentos opcionales son una sintaxis conveniente para escribir múltiples definiciones de 
métodos con diferentes números de argumentos (ver [Nota sobre Argumentos opcionales y *keyword*](@ref)).

## Argumentos *keyword*

Algunas funciones necesitan un número de argumentos grande o tienen un gran número de comportamientos. 
Recordar como llamar a tales funciones puede ser difícil. Los argumentos *keyword* pueden hacer que 
estas interfaces complejas sean más fáciles de usar y extender permitiendo que los argumentos sean 
identificados por su nombre en lugar del por su posición.

Por ejemplo, considere una función `plot` que traza una línea. Esta función puede tener muchas 
opciones para controlar el estilo de línea, su ancho, su color, etc. Si la función  aceptara argumentos 
*keyword*, una posible llamada al método seria `plot(x,y, width=2)`, donde hemos elegido especificar 
sólo el ancho de línea. Nótese que esto sirve a dos propósitos: La llamada es más sencillo leer, ya 
que podemos etiquetar un argumento con su significado. También se vuelve posible pasar cualquier 
subconjunto de un gran número de argumentos, en cualquier orden.

Las funciones con argumentos *keyword* se definen usando un punto y coma en la signatura:

```julia
function plot(x, y; style="solid", width=1, color="black")
    ###
end
```

Cuando la función es invocada, el punto y coma es opcional: uno puede hacer la llamada como 
`plot(x, y, width=2)`o como `plot(x, y; width=2)`, aunque el primero es más común. Se 
requiere un punto y coma explícito sólo en el caso de pasar *vargars* o palabras clave 
calculadas como se describe abajo.

Los valores por defecto de los argumentos *keyword* son evaluados sólo cuando sea necesario 
(cuando no se pasa el correspondiente argumento *keyword*) y en orden izquierda a derecha. 
Por tanto, las expresiones por defecto pueden referirse a argumentos *keyword* previos.

Los tipos de argumentos *keyword*  pueden hacerse explícitos de la siguiente forma:

```julia
function f(;x::Int=1)
    ###
end
```

Los argumentos *keyword* extra pueden ser recolectados usando `...` como en las funciones *vararg*:

```julia
function f(x; y=0, kwargs...)
    ###
end
```

Dentro de `f`, `kwargs` será una colección de tuplas `(clave,valor)`, donde cada `clave` es un 
símbolo. Tales colecciones pueden ser pasadas como argumentos *keyword* usando un punto y coma en 
la llamada. Por ejemplo: `f(x, z=1; kwargs...)`. Los diccionarios pueden ser también usados para 
este propósito.

Uno puede también pasar tuplas `(clave, valor)` o cualquier expresión iterable (tal como un par `=>`) 
que puede ser asignado a una tupla, explícitamente después de un punto y coma. Por ejemplo, 
`plot(x, y; (:width,2))` y `plot(x, y; :width => 2)` son equivalentes a `plot(x, y, width=2)`. 
Esto es útil en situaciones donde el nombre de la palabra clave se calcula en tiempo de ejecución.

La naturaleza de los argumentos *keyword*  le hace posible especificar el mismo argumento más de una vez.
Por ejemplo, en la llamada `plot(x, y; options..., width=2)` es posible que la estructura `options` 
contenga también un valor para `width`. En tal caso la ocurrencia más a la derecha toma precedencia; en
este ejemplo `width` tendrá el valor `2`.

## Ámbito de evaluación de Valores por defecto

Cuando se evalúan expresiones en argumentos opcionales y *keyword* sólo los argumentos *previos* están
en el ámbito. Por ejemplo, dada esta definición:

```julia
function f(x, a=b, b=1)
    ###
end
```

la `b` en `a=b` se refiere a la `b` de un ámbito más externo, no el siguiente argumento `b`.

## Sintaxis Bloque Do para Argumentos Function

Pasar funciones como argumentos a otras funciones es una técnica muy potente, pero su sintaxs no es 
siempre conveniente. Estas llamadas son especialmente incómodas de escribir cuando la función 
argumento necesita varias líneas. Por ejemplo, consideremos llamar a  [`map()`](@ref) sobre una 
función con varios casos:

```julia
map(x->begin
           if x < 0 && iseven(x)
               return 0
           elseif x == 0
               return 1
           else
               return x
           end
       end,
    [A, B, C])
```

Julia proporciona la palabra reservada `do` para reescribir este código de forma más clara:

```julia
map([A, B, C]) do x
    if x < 0 && iseven(x)
        return 0
    elseif x == 0
        return 1
    else
        return x
    end
end
```

La sintaxis `do x` crea una función anónima con argumento `x` y la pasa como primer argumento a 
[`map()`](@ref). Similarmente, `do a,b` crearía una función anónima de dos argumentos, y un `do` 
solo sería una función anónima de la forma `() -> ...`.

Cómo se inicializan estos argumentos depende de la función más externa; aquí `map()` fijará 
secuencialmente `x` a `A,B,C` llamando a la función anónima sobre cada uno de ellos, tal y 
como pasa en la sintaxis `map(func, [A,B,C])`.

Esta sintaxis hace más fácil usar funciones para extender el lenguaje de forma efectiva, ya que 
las llamadas tiene el aspecto de códigos de bloque normales. Hay muchos usos posibles diferentes 
al de [`map()`](@ref), tal como la gestión del estado del sistema. Por ejemplo, hay una versión 
de  [`open()`](@ref) que ejecuta código asegurando que el fichero abierto es cerrado eventualmente:

```julia
open("outfile", "w") do io
    write(io, data)
end
```

Esto se consigue mediante la siguiente definición:

```julia
function open(f::Function, args...)
    io = open(args...)
    try
        f(io)
    finally
        close(io)
    end
end
```

Here, [`open()`](@ref) primero abre el fichero para escritura y luego pasa el flujo de salida 
resultante a la función anónima que se define en el bloque `do...end`.  Después de que la 
función exista, [`open()`](@ref) asegurará que el flujo ha sido cerrado apropiadamente, sin 
preocuparse de si la función salió normalmente o lanzó una excepción (la construcción 
`try/finally` será descrita en  [Control de Flujo](@ref).)

Con la sintaxis de bloque `do` se ayuda a chequear la documentación o implementaciones para 
saber cómo se inicializan los argumentos de la función de usuario.

## [Dot Syntax for Vectorizing Functions](@id man-vectorized)

In technical-computing languages, it is common to have "vectorized" versions of functions, which
simply apply a given function `f(x)` to each element of an array `A` to yield a new array via
`f(A)`. This kind of syntax is convenient for data processing, but in other languages vectorization
is also often required for performance: if loops are slow, the "vectorized" version of a function
can call fast library code written in a low-level language. In Julia, vectorized functions are
*not* required for performance, and indeed it is often beneficial to write your own loops (see
[Performance Tips](@ref man-performance-tips)), but they can still be convenient. Therefore, *any* Julia function
`f` can be applied elementwise to any array (or other collection) with the syntax `f.(A)`.
For example `sin` can be applied to all elements in the vector `A`, like so:

```jldoctest
julia> A = [1.0, 2.0, 3.0]
3-element Array{Float64,1}:
 1.0
 2.0
 3.0

julia> sin.(A)
3-element Array{Float64,1}:
 0.841471
 0.909297
 0.14112
```

Of course, you can omit the dot if you write a specialized "vector" method of `f`, e.g. via `f(A::AbstractArray) = map(f, A)`,
and this is just as efficient as `f.(A)`. But that approach requires you to decide in advance
which functions you want to vectorize.

More generally, `f.(args...)` is actually equivalent to `broadcast(f, args...)`, which allows
you to operate on multiple arrays (even of different shapes), or a mix of arrays and scalars (see
[Broadcasting](@ref)). For example, if you have `f(x,y) = 3x + 4y`, then `f.(pi,A)` will return
a new array consisting of `f(pi,a)` for each `a` in `A`, and `f.(vector1,vector2)` will return
a new vector consisting of `f(vector1[i],vector2[i])` for each index `i` (throwing an exception
if the vectors have different length).

```jldoctest
julia> f(x,y) = 3x + 4y;

julia> A = [1.0, 2.0, 3.0];

julia> B = [4.0, 5.0, 6.0];

julia> f.(pi, A)
3-element Array{Float64,1}:
 13.4248
 17.4248
 21.4248

julia> f.(A, B)
3-element Array{Float64,1}:
 19.0
 26.0
 33.0
```

Moreover, *nested* `f.(args...)` calls are *fused* into a single `broadcast` loop. For example,
`sin.(cos.(X))` is equivalent to `broadcast(x -> sin(cos(x)), X)`, similar to `[sin(cos(x)) for x in X]`:
there is only a single loop over `X`, and a single array is allocated for the result. [In contrast,
`sin(cos(X))` in a typical "vectorized" language would first allocate one temporary array for
`tmp=cos(X)`, and then compute `sin(tmp)` in a separate loop, allocating a second array.] This
loop fusion is not a compiler optimization that may or may not occur, it is a *syntactic guarantee*
whenever nested `f.(args...)` calls are encountered. Technically, the fusion stops as soon as
a "non-dot" function call is encountered; for example, in `sin.(sort(cos.(X)))` the `sin` and `cos`
loops cannot be merged because of the intervening `sort` function.

Finally, the maximum efficiency is typically achieved when the output array of a vectorized operation
is *pre-allocated*, so that repeated calls do not allocate new arrays over and over again for
the results (see [Pre-allocating outputs](@ref)). A convenient syntax for this is `X .= ...`, which
is equivalent to `broadcast!(identity, X, ...)` except that, as above, the `broadcast!` loop is
fused with any nested "dot" calls. For example, `X .= sin.(Y)` is equivalent to `broadcast!(sin, X, Y)`,
overwriting `X` with `sin.(Y)` in-place. If the left-hand side is an array-indexing expression,
e.g. `X[2:end] .= sin.(Y)`, then it translates to `broadcast!` on a `view`, e.g. `broadcast!(sin, view(X, 2:endof(X)), Y)`,
so that the left-hand side is updated in-place.

Since adding dots to many operations and function calls in an expression
can be tedious and lead to code that is difficult to read, the macro
[`@.`](@ref @__dot__) is provided to convert *every* function call,
operation, and assignment in an expression into the "dotted" version.

```jldoctest
julia> Y = [1.0, 2.0, 3.0, 4.0];

julia> X = similar(Y); # pre-allocate output array

julia> @. X = sin(cos(Y)) # equivalent to X .= sin.(cos.(Y))
4-element Array{Float64,1}:
  0.514395
 -0.404239
 -0.836022
 -0.608083
```

Binary (or unary) operators like `.+` are handled with the same mechanism:
they are equivalent to `broadcast` calls and are fused with other nested "dot" calls.
 `X .+= Y` etcetera is equivalent to `X .= X .+ Y` and results in a fused in-place assignment;
 see also [dot operators](@ref man-dot-operators).

## Further Reading

Deberíamos mencionar que esto está lejos de ser una visión completa de las definiciones de 
función. Julia tiene un sistema de tipos sofisticado y permite despacho múltiple sobre los 
tipos de argumento. Ninguno de los ejemplos dados aquí proporciona anotaciones de tipo sobre 
sus argmentos, lo que significa que son aplicables a cualquier tipo de argumento. El sistema 
de tipos es descrito en [Tipos](@ref man-types) definir una función en términos de métodos 
elegidos mediante despacho múltiple sobre los tpos de argumento en tiempo de ejecucoión 
se describe en el capítulo [Methods](@ref).
