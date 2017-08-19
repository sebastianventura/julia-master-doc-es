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

This is fine if a named function effecting the transform already exists to pass as the first argument
to [`map()`](@ref). Often, however, a ready-to-use, named function does not exist. In these
situations, the anonymous function construct allows easy creation of a single-use function object
without needing a name:

```jldoctest
julia> map(x -> x^2 + 2x - 1, [1,3,-1])
3-element Array{Int64,1}:
  2
 14
 -2
```

An anonymous function accepting multiple arguments can be written using the syntax `(x,y,z)->2x+y-z`.
A zero-argument anonymous function is written as `()->3`. The idea of a function with no arguments
may seem strange, but is useful for "delaying" a computation. In this usage, a block of code is
wrapped in a zero-argument function, which is later invoked by calling it as `f()`.

## Multiple Return Values

In Julia, one returns a tuple of values to simulate returning multiple values. However, tuples
can be created and destructured without needing parentheses, thereby providing an illusion that
multiple values are being returned, rather than a single tuple value. For example, the following
function returns a pair of values:

```jldoctest foofunc
julia> function foo(a,b)
           a+b, a*b
       end
foo (generic function with 1 method)
```

If you call it in an interactive session without assigning the return value anywhere, you will
see the tuple returned:

```jldoctest foofunc
julia> foo(2,3)
(5, 6)
```

A typical usage of such a pair of return values, however, extracts each value into a variable.
Julia supports simple tuple "destructuring" that facilitates this:

```jldoctest foofunc
julia> x, y = foo(2,3)
(5, 6)

julia> x
5

julia> y
6
```

You can also return multiple values via an explicit usage of the `return` keyword:

```julia
function foo(a,b)
    return a+b, a*b
end
```

This has the exact same effect as the previous definition of `foo`.

## Varargs Functions

It is often convenient to be able to write functions taking an arbitrary number of arguments.
Such functions are traditionally known as "varargs" functions, which is short for "variable number
of arguments". You can define a varargs function by following the last argument with an ellipsis:

```jldoctest barfunc
julia> bar(a,b,x...) = (a,b,x)
bar (generic function with 1 method)
```

The variables `a` and `b` are bound to the first two argument values as usual, and the variable
`x` is bound to an iterable collection of the zero or more values passed to `bar` after its first
two arguments:

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

In all these cases, `x` is bound to a tuple of the trailing values passed to `bar`.

It is possible to constrain the number of values passed as a variable argument; this will be discussed
later in [Parametrically-constrained Varargs methods](@ref).

On the flip side, it is often handy to "splice" the values contained in an iterable collection
into a function call as individual arguments. To do this, one also uses `...` but in the function
call instead:

```jldoctest barfunc
julia> x = (3, 4)
(3, 4)

julia> bar(1,2,x...)
(1, 2, (3, 4))
```

In this case a tuple of values is spliced into a varargs call precisely where the variable number
of arguments go. This need not be the case, however:

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

Furthermore, the iterable object spliced into a function call need not be a tuple:

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

Also, the function that arguments are spliced into need not be a varargs function (although it
often is):

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

As you can see, if the wrong number of elements are in the spliced container, then the function
call will fail, just as it would if too many arguments were given explicitly.

## Optional Arguments

In many cases, function arguments have sensible default values and therefore might not need to
be passed explicitly in every call. For example, the library function [`parse(T, num, base)`](@ref)
interprets a string as a number in some base. The `base` argument defaults to `10`. This behavior
can be expressed concisely as:

```julia
function parse(T, num, base=10)
    ###
end
```

With this definition, the function can be called with either two or three arguments, and `10`
is automatically passed when a third argument is not specified:

```jldoctest
julia> parse(Int,"12",10)
12

julia> parse(Int,"12",3)
5

julia> parse(Int,"12")
12
```

Optional arguments are actually just a convenient syntax for writing multiple method definitions
with different numbers of arguments (see [Note on Optional and keyword Arguments](@ref)).

## Keyword Arguments

Some functions need a large number of arguments, or have a large number of behaviors. Remembering
how to call such functions can be difficult. Keyword arguments can make these complex interfaces
easier to use and extend by allowing arguments to be identified by name instead of only by position.

For example, consider a function `plot` that plots a line. This function might have many options,
for controlling line style, width, color, and so on. If it accepts keyword arguments, a possible
call might look like `plot(x, y, width=2)`, where we have chosen to specify only line width. Notice
that this serves two purposes. The call is easier to read, since we can label an argument with
its meaning. It also becomes possible to pass any subset of a large number of arguments, in any
order.

Functions with keyword arguments are defined using a semicolon in the signature:

```julia
function plot(x, y; style="solid", width=1, color="black")
    ###
end
```

When the function is called, the semicolon is optional: one can either call `plot(x, y, width=2)`
or `plot(x, y; width=2)`, but the former style is more common. An explicit semicolon is required
only for passing varargs or computed keywords as described below.

Keyword argument default values are evaluated only when necessary (when a corresponding keyword
argument is not passed), and in left-to-right order. Therefore default expressions may refer to
prior keyword arguments.

The types of keyword arguments can be made explicit as follows:

```julia
function f(;x::Int=1)
    ###
end
```

Extra keyword arguments can be collected using `...`, as in varargs functions:

```julia
function f(x; y=0, kwargs...)
    ###
end
```

Inside `f`, `kwargs` will be a collection of `(key,value)` tuples, where each `key` is a symbol.
Such collections can be passed as keyword arguments using a semicolon in a call, e.g. `f(x, z=1; kwargs...)`.
Dictionaries can also be used for this purpose.

One can also pass `(key,value)` tuples, or any iterable expression (such as a `=>` pair) that
can be assigned to such a tuple, explicitly after a semicolon. For example, `plot(x, y; (:width,2))`
and `plot(x, y; :width => 2)` are equivalent to `plot(x, y, width=2)`. This is useful in situations
where the keyword name is computed at runtime.

The nature of keyword arguments makes it possible to specify the same argument more than once.
For example, in the call `plot(x, y; options..., width=2)` it is possible that the `options` structure
also contains a value for `width`. In such a case the rightmost occurrence takes precedence; in
this example, `width` is certain to have the value `2`.

## Evaluation Scope of Default Values

When optional and keyword argument default expressions are evaluated, only *previous* arguments are in
scope.
For example, given this definition:

```julia
function f(x, a=b, b=1)
    ###
end
```

the `b` in `a=b` refers to a `b` in an outer scope, not the subsequent argument `b`.

## Do-Block Syntax for Function Arguments

Passing functions as arguments to other functions is a powerful technique, but the syntax for
it is not always convenient. Such calls are especially awkward to write when the function argument
requires multiple lines. As an example, consider calling [`map()`](@ref) on a function with several
cases:

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

Julia provides a reserved word `do` for rewriting this code more clearly:

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

The `do x` syntax creates an anonymous function with argument `x` and passes it as the first argument
to [`map()`](@ref). Similarly, `do a,b` would create a two-argument anonymous function, and a
plain `do` would declare that what follows is an anonymous function of the form `() -> ...`.

How these arguments are initialized depends on the "outer" function; here, [`map()`](@ref) will
sequentially set `x` to `A`, `B`, `C`, calling the anonymous function on each, just as would happen
in the syntax `map(func, [A, B, C])`.

This syntax makes it easier to use functions to effectively extend the language, since calls look
like normal code blocks. There are many possible uses quite different from [`map()`](@ref), such
as managing system state. For example, there is a version of [`open()`](@ref) that runs code ensuring
that the opened file is eventually closed:

```julia
open("outfile", "w") do io
    write(io, data)
end
```

This is accomplished by the following definition:

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

Here, [`open()`](@ref) first opens the file for writing and then passes the resulting output stream
to the anonymous function you defined in the `do ... end` block. After your function exits, [`open()`](@ref)
will make sure that the stream is properly closed, regardless of whether your function exited
normally or threw an exception. (The `try/finally` construct will be described in [Control Flow](@ref).)

With the `do` block syntax, it helps to check the documentation or implementation to know how
the arguments of the user function are initialized.

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

We should mention here that this is far from a complete picture of defining functions. Julia has
a sophisticated type system and allows multiple dispatch on argument types. None of the examples
given here provide any type annotations on their arguments, meaning that they are applicable to
all types of arguments. The type system is described in [Types](@ref man-types) and defining a function
in terms of methods chosen by multiple dispatch on run-time argument types is described in [Methods](@ref).
