# Métodos

Recordemos de la sección [Funciones](@ref man-functions) que una función es un objeto que establece 
una correspondencia entre una tupla de argumentos y un valor de retorno o lanza una excepción si 
no puede devolverse el valor apropiado. Para la misma función conceptual es común soy implementada 
de una forma muy diferente para tipos de argumentos diferentes: sumar dos enteros es distinto de 
sumar dos valores en punto flotante y ambos son distintos de sumar un entero y 1 punto flotante. 
A pesar de las diferencias de implementación, éstas operaciones caen todas bajo el concepto general 
de "suma". En consecuencia, en Julia, estos comportamientos pertenecen todos a un solo objeto: 
la función `+`.

Para facilitar el uso de muchas implementaciones distintas del mismo concepto suavemente, las 
funciones necesitan no ser definidas de una vez, sino poder ser definidas a trozos proporcionando 
comportamientos distintos para ciertas combinaciones de tipos de argumentos y cuentas. Llamamos 
*método* a la definición de un posible comportamiento para una función. Hasta ahora sólo se han 
presentado ejemplos de funciones definidas como sólo método, aplicables a todo tipo de 
argumentos. Sin embargo, las asignaturas de las definiciones de los métodos pueden anotarse 
para indicar los tipos de los argumentos además de su número, y puede proporcionarse más de 
una sola definición de método. Cuando una función se aplica a una dupla de argumentos particular, 
se aplica el método más específico y aplicable a esos argumentos. Por tanto, el comportamiento 
global de una función es un collage de los comportamientos de sus distintas definiciones de 
métodos. Si el collage está bien diseñado, incluso aunque las implementaciones de los métodos 
puedan ser bastante diferentes, el comportamiento exterior de la función pareciera contínuo y 
consistente.

La elección de qué método ejecutar cuando se aplica una función se llama *despacho*. Julia 
permite al proceso de despacho elegir qué método de una función llamar basándose en el número 
de argumentos y en los tipos de todos los argumentos dados a la función. Este mecanismo es 
diferente al que ocurre en los lenguajes orientados al objeto tradicionales, donde el despacho 
se basa solo en el primer argumento, que frecuentemente tiene una sintaxis especial, y que es 
muchas veces implicado el lugar de ser escrito explícitamente como argumento. [^1] Usar todos 
los argumentos de la función para elegir qué método debería ser invocado es conocido como 
[multiple dispatch](https://en.wikipedia.org/wiki/Multiple_dispatch).. El despacho múltiple es 
particularmente útil para código matemático, donde  tiene poco sentido considerar que las 
operaciones pertenecen a un argumento más que los demás. Más allá de las operaciones 
matemáticas, sin embargo, el despacho múltiple ha resultado ser un paradigma potente y 
conveniente para estructurar y organizar los programas.

[^1]:
    En C++ o Java, por ejemplo, en una llamada a un método como `obj.meth(arg1,arg2)`, el objeto
    obj "recibe" la llamada al método y es pasado implícitamente vía la palabra clave `this`, 
    en lugar de con un argumento de método explícito. Cuando el objeto `this` actual es el 
    receptor de una llamada a método él puede ser omitido, escribiendo justo `meth(arg1,arg2)`, 
    con `this` implicito como objeto receptor.

## Definir Métodos

En los ejemplos estudiados hasta ahora, sólo se han definido funciones con un único método que tienen argumentos con los tipos no restringidos. Estas funciones se comporta como las que hay en lenguajes con tipos dinámicos tradicionales. Sin embargo, también se han usado despacho múltiple y métodos sin ser consciente de ello: todas las funciones estándar y operadores de Julia, tal como función `+`, tiene muchos métodos que definen su comportamiento sobre varias combinaciones posibles número y tipo de argumentos.

Cuando se define una función, uno puede opcionalmente restringir los tipos de los parámetros sobre los que se aplica usando el operador de la selección de tipos `::`, introducido en la sección [Tipos compuestos](@ref):

```jldoctest fofxy
julia> f(x::Float64, y::Float64) = 2x + y
f (generic function with 1 method)
```

Esta definición de función se aplica sólo a llamadas en las que `x` e `y` sean ambos valores del tipo [`Float64`](@ref):

```jldoctest fofxy
julia> f(2.0, 3.0)
7.0
```

Aplicar esta definición a otros tipos de argumentos dará como resultado un [`MethodError`](@ref):

```jldoctest fofxy
julia> f(2.0, 3)
ERROR: MethodError: no method matching f(::Float64, ::Int64)
Closest candidates are:
  f(::Float64, !Matched::Float64) at none:1

julia> f(Float32(2.0), 3.0)
ERROR: MethodError: no method matching f(::Float32, ::Float64)
Closest candidates are:
  f(!Matched::Float64, ::Float64) at none:1

julia> f(2.0, "3.0")
ERROR: MethodError: no method matching f(::Float64, ::String)
Closest candidates are:
  f(::Float64, !Matched::Float64) at none:1

julia> f("2.0", "3.0")
ERROR: MethodError: no method matching f(::String, ::String)
```

Como puede comprobarse, los argumentos tienen que ser exactamente del tipo [`Float64`](@ref). Otros tipos numéricos tales como `Float32` o `Int` no serán convertidos automáticamente en `Float64`. Algo parecido sucede con los datos `String`. Como el tipo `Float64` es un tipo concreto y los tipos concretos no pueden tener subclases en Julia, esta definición sólo puede aplicarse a argumentos que sean exactamente del tipo `Float64`. Esto puede ser útil en ocasiones, sin embargo, para escribir métodos más generales se hace uso de parámetros cuyos tipos sean abstractos:

```jldoctest fofxy
julia> f(x::Number, y::Number) = 2x - y
f (generic function with 2 methods)

julia> f(2.0, 3)
1.0
```

Esta definición de método se aplica a cualquier par de argumentos que sean instancias de  [`Number`](@ref).
Ellas no tienen que ser del mismo tipo, mientras que ambas sean valores numéricos. El problema de manejar tipos numéricos dispares se delega a las operaciones aritméticas en la expresión `2x - y`.

Para definir una función con múltiples métodos, uno simplemente define la función varías veces, con diferentes números de argumentos y tipos. La primera definición de método para la función crea el objeto función y las definiciones de métodos subsecuentes añaden nuevos métodos al objeto función existente. La definición de método más específica que case con el número los tipos de argumentos será la ejecutada cuando se aplique la función. Por tanto, las dos definiciones de métodos anteriores, considerados juntas, define el comportamiento de la función `f` sobre todos los padres de instancias del tipo abstracto `Number` (pero con un comportamiento específico para pares de valores [`Float64`](@ref). Si uno de los argumentos es un valor en punto flotante de 64 bits, pero el otro no lo es, entonces el método `f(Float64,Float64)` no puede ser invocado y se utilizará el método más general `f(Number,Number)`:

```jldoctest fofxy
julia> f(2.0, 3.0)
7.0

julia> f(2, 3.0)
1.0

julia> f(2.0, 3)
1.0

julia> f(2, 3)
1
```

La definición `2x+y` sólo se usa en el primer caso, mientras que la definición `2x-y` se usa en los demás. Nunca se realiza conversión automática en los otros: todas las conversiones son no mágicas y completamente exlícitas. En la sección [Conversión y promoción](@ref conversion-and-promotion), sin embargo, se muestra cómo las aplicaciones inteligentes de tecnología suficientemente avanzada pueden ser indistinguibles de la magic [^Clarke61]

Para valores no numéricos, ,y para menores de dos argumentos, la función `f` permanece indefinida, y aplicándola se obtendrá como resultado un [`MethodError`](@ref):

```jldoctest fofxy
julia> f("foo", 3)
ERROR: MethodError: no method matching f(::String, ::Int64)
Closest candidates are:
  f(!Matched::Number, ::Number) at none:1

julia> f()
ERROR: MethodError: no method matching f()
Closest candidates are:
  f(!Matched::Float64, !Matched::Float64) at none:1
  f(!Matched::Number, !Matched::Number) at none:1
```

Puedes ver fácilmente que métodos existen para una función entrando el propio nombre del objeto en una sesión interactiva:

```jldoctest fofxy
julia> f
f (generic function with 2 methods)
```

La salida nos dice que `f` es un objeto función con dos métodos. Para encontrar cuáles son las signaturas de esos métodos, utilizaremos la función [`methods()`](@ref):

```julia-repl
julia> methods(f)
# 2 methods for generic function "f":
[1] f(x::Float64, y::Float64) in Main at none:1
[2] f(x::Number, y::Number) in Main at none:1
```

que muestra que `f` tiene dos métodos: uno que toma dos argumentos `Float64` y una que toma dos argumentos de tipo `Number`. También indica el fichero y el número de línea donde los métodos fueron definidos aunque, si los métodos fueron definidos en el REPL, se obtendrá `none:1`.

En ausencia de una declaración de tipo con  `::` el tipo de un parámetro de un método es `Any` por defecto, lo que significa que está sin restricciones ya que todos los valores en Julia son instancias del tipo abstracto `Any`. Por tanto, podemos definir un método atrapatodo para `f` tal como:

```jldoctest fofxy
julia> f(x,y) = println("Whoa there, Nelly.")
f (generic function with 3 methods)

julia> f("foo", 1)
Whoa there, Nelly.
```

Este atrapatodo es menos específico que cualquier otra posible definición de método para un par de valores de parámetros, por lo que sólo será llamada sobre pares de argumentos a los cuales no pueda aplicarse otra definición de método.

Aunque parece un concepto simple, el despacho múltiple sobre los tipos de valores es quizás la característica más potente y central del lenguaje Julia. Las operaciones del núcleo tienen típicamente docenas de metodos:

```julia-repl
julia> methods(+)
# 180 methods for generic function "+":
[1] +(x::Bool, z::Complex{Bool}) in Base at complex.jl:227
[2] +(x::Bool, y::Bool) in Base at bool.jl:89
[3] +(x::Bool) in Base at bool.jl:86
[4] +(x::Bool, y::T) where T<:AbstractFloat in Base at bool.jl:96
[5] +(x::Bool, z::Complex) in Base at complex.jl:234
[6] +(a::Float16, b::Float16) in Base at float.jl:373
[7] +(x::Float32, y::Float32) in Base at float.jl:375
[8] +(x::Float64, y::Float64) in Base at float.jl:376
[9] +(z::Complex{Bool}, x::Bool) in Base at complex.jl:228
[10] +(z::Complex{Bool}, x::Real) in Base at complex.jl:242
[11] +(x::Char, y::Integer) in Base at char.jl:40
[12] +(c::BigInt, x::BigFloat) in Base.MPFR at mpfr.jl:307
[13] +(a::BigInt, b::BigInt, c::BigInt, d::BigInt, e::BigInt) in Base.GMP at gmp.jl:392
[14] +(a::BigInt, b::BigInt, c::BigInt, d::BigInt) in Base.GMP at gmp.jl:391
[15] +(a::BigInt, b::BigInt, c::BigInt) in Base.GMP at gmp.jl:390
[16] +(x::BigInt, y::BigInt) in Base.GMP at gmp.jl:361
[17] +(x::BigInt, c::Union{UInt16, UInt32, UInt64, UInt8}) in Base.GMP at gmp.jl:398
...
[180] +(a, b, c, xs...) in Base at operators.jl:424
```

El despacho múltiple junto con el sistema de tipos paramétrico flexible dan a Julia su capacidad para expresar de forma abstract algoritmos de alto nivel desacoplados de los detalles de implementación, generando aún código eficiente y especializado para manejar cada caso en tiempo de ejecución.

## [Ambigüedades de Métodos](@id man-ambiguities)

Es posible definir un conjunto de métodos de función tales que no haya un método único más específico aplicable a alguna combinación de argumentos:

```jldoctest gofxy
julia> g(x::Float64, y) = 2x + y
g (generic function with 1 method)

julia> g(x, y::Float64) = x + 2y
g (generic function with 2 methods)

julia> g(2.0, 3)
7.0

julia> g(2, 3.0)
8.0

julia> g(2.0, 3.0)
ERROR: MethodError: g(::Float64, ::Float64) is ambiguous.
[...]
```

Aquí, la llamada `g(2.0, 3.0)` podría ser manejada por los métodos `g(Float64, Any)` o `g(Any, Float64)` y ninguno es más específico que el otro. En tales casos, Julia lanza un [`MethodError`](@ref) en lugar de elegir uno de los métodos arbitrariamente. Podemos obviar las ambigüedades de los métodos especificando un método apropiado para el caso intersección:

```jldoctest gofxy
julia> g(x::Float64, y::Float64) = 2x + 2y
g (generic function with 3 methods)

julia> g(2.0, 3)
7.0

julia> g(2, 3.0)
8.0

julia> g(2.0, 3.0)
10.0
```

Se recomienda que el método que suprime la ambigüedad sea definido primero, ya que en caso contrario la ambigüedad existe, transitoriamente, hasta que el método más especifico sea definido.

En casos ms complejos, resolver ambigüedades de métodos implica un cierto elemento de diseño; este tema se explorará [posteriormente](@ref man-method-design-ambiguities).

## Métodos paramétricos

Las definiciones de métodos pueden tener, opcionalmente, parámetros de tipo cualificando la signatura:

```jldoctest same_typefunc
julia> same_type(x::T, y::T) where {T} = true
same_type (generic function with 1 method)

julia> same_type(x,y) = false
same_type (generic function with 2 methods)
```

El primer método se aplica cuando ambos argumentos son del mismo tipo concreto, independientemente del tipo que sea, mientras que el segundo actúa como un atrapatodo, cubriendo todos los demás casos. Por tanto, en conjunto, esto define una función booleana que comprueba si dos argumentos son del mismo tipo:

```jldoctest same_typefunc
julia> same_type(1, 2)
true

julia> same_type(1, 2.0)
false

julia> same_type(1.0, 2.0)
true

julia> same_type("foo", 2.0)
false

julia> same_type("foo", "bar")
true

julia> same_type(Int32(1), Int64(2))
false
```

Tales definiciones corresponden a métodos cuyas signaturas de tipo son tipos `UnionAll` (ver [tipos UnionAll](@ref).

Esta clase de definición del comportamiento de una función mediante despacho es bastante común (incluso idiomático) en Julia. Los métodos con parámetros de tipo no están restringidos a ser usados como los tipos de los parámetros: ellos pueden ser usados en cualquier parte donde un palo estaría en la signatura de la función o cuerpo de la función. He aquí un eemplo donde el parámetro de tipo del método `T` se sa como el parámetro de tipo al tipo paramétrico `Vector{T}` en la signatura del método:

```jldoctest
julia> myappend(v::Vector{T}, x::T) where {T} = [v..., x]
myappend (generic function with 1 method)

julia> myappend([1,2,3],4)
4-element Array{Int64,1}:
 1
 2
 3
 4

julia> myappend([1,2,3],2.5)
ERROR: MethodError: no method matching myappend(::Array{Int64,1}, ::Float64)
Closest candidates are:
  myappend(::Array{T,1}, !Matched::T) where T at none:1

julia> myappend([1.0,2.0,3.0],4.0)
4-element Array{Float64,1}:
 1.0
 2.0
 3.0
 4.0

julia> myappend([1.0,2.0,3.0],4)
ERROR: MethodError: no method matching myappend(::Array{Float64,1}, ::Int64)
Closest candidates are:
  myappend(::Array{T,1}, !Matched::T) where T at none:1
```

Como puedes ver, el tipo del elemento añadido tiene que corresponderse con el tipo de elemento del vector al que se está añadiendo, o se lanzará un [`MethodError`](@ref).  En el siguiente ejemplo, el parámetro de tipo del método `T` se usa como valor de retorno:

```jldoctest
julia> mytypeof(x::T) where {T} = T
mytypeof (generic function with 1 method)

julia> mytypeof(1)
Int64

julia> mytypeof(1.0)
Float64
```

Así como puedes poner restricciones de subtipo para los parámetros de tipo en declaraciones de tipo (ver [Tipos Paramétricos](@ref)) también puedes restringir los parámetros de tipo de los métodos:

```jldoctest
julia> same_type_numeric(x::T, y::T) where {T<:Number} = true
same_type_numeric (generic function with 1 method)

julia> same_type_numeric(x::Number, y::Number) = false
same_type_numeric (generic function with 2 methods)

julia> same_type_numeric(1, 2)
true

julia> same_type_numeric(1, 2.0)
false

julia> same_type_numeric(1.0, 2.0)
true

julia> same_type_numeric("foo", 2.0)
ERROR: MethodError: no method matching same_type_numeric(::String, ::Float64)
Closest candidates are:
  same_type_numeric(!Matched::T<:Number, ::T<:Number) where T<:Number at none:1
  same_type_numeric(!Matched::Number, ::Number) at none:1

julia> same_type_numeric("foo", "bar")
ERROR: MethodError: no method matching same_type_numeric(::String, ::String)

julia> same_type_numeric(Int32(1), Int64(2))
false
```

La función `same_type_numeric` se comporta como la función `same_type` descrita antes, pero sólo está definida para pares de números.

Los métodos paramétricos permiten la misma sintaxis que las expresiones `where` usadas para escribir tipos (ver [tipos UnionAll](@ref)). 

Si hay un único parámetro, las llaves que lo encierran (en `where {T}`) pueden ser omitidas, aunque suele ser preferible mantenerlas por claridad.

Los parámetros múltiples pueden ser separados con comas, por ejemplo `where {T, S<:Real}`, o escritos usando `where` anidados, por ejemplo, `where S<:Real where T`.

Redefiniendo Métodos
--------------------

Cuando se redefine un método o se añaden nuevos métodos, es importante comprender que estos cambios no tienen efecto inmediatamente. Esto es clave para la capacidad de Julia para inferir estáticamente y compilar código para ejecutar rápido, sin los trucos de JIT y sobrecargas usuales. De hecho, cualquier nueva definición de método no será visible al entorno de ejecución actual, incluyendo tareas e hilos (y cualquier otra función definida con `@generated`). Comencemos con un ejemplo para ver qué significa esto:

```julia-repl
julia> function tryeval()
           @eval newfun() = 1
           newfun()
       end
tryeval (generic function with 1 method)

julia> tryeval()
ERROR: MethodError: no method matching newfun()
The applicable method may be too new: running in world age xxxx1, while current world is xxxx2.
Closest candidates are:
  newfun() at none:1 (method too new to be called from this world context.)
 in tryeval() at none:1
 ...

julia> newfun()
1
```

In this example, observe that the new definition for `newfun` has been created,
but can't be immediately called.
The new global is immediately visible to the `tryeval` function,
so you could write `return newfun` (without parentheses).
But neither you, nor any of your callers, nor the functions they call, or etc.
can call this new method definition!

But there's an exception: future calls to `newfun` *from the REPL* work as expected,
being able to both see and call the new definition of `newfun`.

However, future calls to `tryeval` will continue to see the definition of `newfun` as it was
*at the previous statement at the REPL*, and thus before that call to `tryeval`.

You may want to try this for yourself to see how it works.

The implementation of this behavior is a "world age counter".
This monotonically increasing value tracks each method definition operation.
This allows describing "the set of method definitions visible to a given runtime environment"
as a single number, or "world age".
It also allows comparing the methods available in two worlds just by comparing their ordinal value.
In the example above, we see that the "current world" (in which the method `newfun()` exists),
is one greater than the task-local "runtime world" that was fixed when the execution of `tryeval` started.

Sometimes it is necessary to get around this (for example, if you are implementing the above REPL).
Fortunately, there is an easy solution: call the function using [`Base.invokelatest`](@ref):

```jldoctest
julia> function tryeval2()
           @eval newfun2() = 2
           Base.invokelatest(newfun2)
       end
tryeval2 (generic function with 1 method)

julia> tryeval2()
2
```

Finally, let's take a look at some more complex examples where this rule comes into play.
Define a function `f(x)`, which initially has one method:

```jldoctest redefinemethod
julia> f(x) = "original definition"
f (generic function with 1 method)
```

Start some other operations that use `f(x)`:

```jldoctest redefinemethod
julia> g(x) = f(x)
g (generic function with 1 method)

julia> t = @async f(wait()); yield();
```

Now we add some new methods to `f(x)`:

```jldoctest redefinemethod
julia> f(x::Int) = "definition for Int"
f (generic function with 2 methods)

julia> f(x::Type{Int}) = "definition for Type{Int}"
f (generic function with 3 methods)
```

Compare how these results differ:

```jldoctest redefinemethod
julia> f(1)
"definition for Int"

julia> g(1)
"definition for Int"

julia> wait(schedule(t, 1))
"original definition"

julia> t = @async f(wait()); yield();

julia> wait(schedule(t, 1))
"definition for Int"
```

## Parametrically-constrained Varargs methods

Function parameters can also be used to constrain the number of arguments that may be supplied
to a "varargs" function ([Varargs Functions](@ref)).  The notation `Vararg{T,N}` is used to indicate
such a constraint.  For example:

```jldoctest
julia> bar(a,b,x::Vararg{Any,2}) = (a,b,x)
bar (generic function with 1 method)

julia> bar(1,2,3)
ERROR: MethodError: no method matching bar(::Int64, ::Int64, ::Int64)
Closest candidates are:
  bar(::Any, ::Any, ::Any, !Matched::Any) at none:1

julia> bar(1,2,3,4)
(1, 2, (3, 4))

julia> bar(1,2,3,4,5)
ERROR: MethodError: no method matching bar(::Int64, ::Int64, ::Int64, ::Int64, ::Int64)
Closest candidates are:
  bar(::Any, ::Any, ::Any, ::Any) at none:1
```

More usefully, it is possible to constrain varargs methods by a parameter. For example:

```julia
function getindex(A::AbstractArray{T,N}, indexes::Vararg{Number,N}) where {T,N}
```

would be called only when the number of `indexes` matches the dimensionality of the array.

When only the type of supplied arguments needs to be constrained `Vararg{T}` can be equivalently
written as `T...`. For instance `f(x::Int...) = x` is a shorthand for `f(x::Vararg{Int}) = x`.

## Note on Optional and keyword Arguments

As mentioned briefly in [Functions](@ref man-functions), optional arguments are implemented as syntax for multiple
method definitions. For example, this definition:

```julia
f(a=1,b=2) = a+2b
```

translates to the following three methods:

```julia
f(a,b) = a+2b
f(a) = f(a,2)
f() = f(1,2)
```

This means that calling `f()` is equivalent to calling `f(1,2)`. In this case the result is `5`,
because `f(1,2)` invokes the first method of `f` above. However, this need not always be the case.
If you define a fourth method that is more specialized for integers:

```julia
f(a::Int,b::Int) = a-2b
```

then the result of both `f()` and `f(1,2)` is `-3`. In other words, optional arguments are tied
to a function, not to any specific method of that function. It depends on the types of the optional
arguments which method is invoked. When optional arguments are defined in terms of a global variable,
the type of the optional argument may even change at run-time.

Keyword arguments behave quite differently from ordinary positional arguments. In particular,
they do not participate in method dispatch. Methods are dispatched based only on positional arguments,
with keyword arguments processed after the matching method is identified.

## Function-like objects

Methods are associated with types, so it is possible to make any arbitrary Julia object "callable"
by adding methods to its type. (Such "callable" objects are sometimes called "functors.")

For example, you can define a type that stores the coefficients of a polynomial, but behaves like
a function evaluating the polynomial:

```jldoctest polynomial
julia> struct Polynomial{R}
           coeffs::Vector{R}
       end

julia> function (p::Polynomial)(x)
           v = p.coeffs[end]
           for i = (length(p.coeffs)-1):-1:1
               v = v*x + p.coeffs[i]
           end
           return v
       end
```

Notice that the function is specified by type instead of by name. In the function body, `p` will
refer to the object that was called. A `Polynomial` can be used as follows:

```jldoctest polynomial
julia> p = Polynomial([1,10,100])
Polynomial{Int64}([1, 10, 100])

julia> p(3)
931
```

This mechanism is also the key to how type constructors and closures (inner functions that refer
to their surrounding environment) work in Julia, discussed [later in the manual](@ref constructors-and-conversion).

## Empty generic functions

Occasionally it is useful to introduce a generic function without yet adding methods. This can
be used to separate interface definitions from implementations. It might also be done for the
purpose of documentation or code readability. The syntax for this is an empty `function` block
without a tuple of arguments:

```julia
function emptyfunc
end
```

## [Method design and the avoidance of ambiguities](@id man-method-design-ambiguities)

Julia's method polymorphism is one of its most powerful features, yet
exploiting this power can pose design challenges.  In particular, in
more complex method hierarchies it is not uncommon for
[ambiguities](@ref man-ambiguities) to arise.

Above, it was pointed out that one can resolve ambiguities like

```julia
f(x, y::Int) = 1
f(x::Int, y) = 2
```

by defining a method

```julia
f(x::Int, y::Int) = 3
```

This is often the right strategy; however, there are circumstances
where following this advice blindly can be counterproductive. In
particular, the more methods a generic function has, the more
possibilities there are for ambiguities. When your method hierarchies
get more complicated than this simple example, it can be worth your
while to think carefully about alternative strategies.

Below we discuss particular challenges and some alternative ways to resolve such issues.

### Tuple and NTuple arguments

`Tuple` (and `NTuple`) arguments present special challenges. For example,

```julia
f(x::NTuple{N,Int}) where {N} = 1
f(x::NTuple{N,Float64}) where {N} = 2
```

are ambiguous because of the possibility that `N == 0`: there are no
elements to determine whether the `Int` or `Float64` variant should be
called. To resolve the ambiguity, one approach is define a method for
the empty tuple:

```julia
f(x::Tuple{}) = 3
```

Alternatively, for all methods but one you can insist that there is at
least one element in the tuple:

```julia
f(x::NTuple{N,Int}) where {N} = 1           # this is the fallback
f(x::Tuple{Float64, Vararg{Float64}}) = 2   # this requires at least one Float64
```

### [Orthogonalize your design](@id man-methods-orthogonalize)

When you might be tempted to dispatch on two or more arguments,
consider whether a "wrapper" function might make for a simpler
design. For example, instead of writing multiple variants:

```julia
f(x::A, y::A) = ...
f(x::A, y::B) = ...
f(x::B, y::A) = ...
f(x::B, y::B) = ...
```

you might consider defining

```julia
f(x::A, y::A) = ...
f(x, y) = f(g(x), g(y))
```

where `g` converts the argument to type `A`. This is a very specific
example of the more general principle of
[orthogonal design](https://en.wikipedia.org/wiki/Orthogonality_(programming)),
in which separate concepts are assigned to separate methods. Here, `g`
will most likely need a fallback definition

```julia
g(x::A) = x
```

A related strategy exploits `promote` to bring `x` and `y` to a common
type:

```julia
f(x::T, y::T) where {T} = ...
f(x, y) = f(promote(x, y)...)
```

One risk with this design is the possibility that if there is no
suitable promotion method converting `x` and `y` to the same type, the
second method will recurse on itself infinitely and trigger a stack
overflow. The non-exported function `Base.promote_noncircular` can be
used as an alternative; when promotion fails it will still throw an
error, but one that fails faster with a more specific error message.

### Dispatch on one argument at a time

If you need to dispatch on multiple arguments, and there are many
fallbacks with too many combinations to make it practical to define
all possible variants, then consider introducing a "name cascade"
where (for example) you dispatch on the first argument and then call
an internal method:

```julia
f(x::A, y) = _fA(x, y)
f(x::B, y) = _fB(x, y)
```

Then the internal methods `_fA` and `_fB` can dispatch on `y` without
concern about ambiguities with each other with respect to `x`.

Be aware that this strategy has at least one major disadvantage: in
many cases, it is not possible for users to further customize the
behavior of `f` by defining further specializations of your exported
function `f`. Instead, they have to define specializations for your
internal methods `_fA` and `_fB`, and this blurs the lines between
exported and internal methods.

### Abstract containers and element types

Where possible, try to avoid defining methods that dispatch on
specific element types of abstract containers. For example,

```julia
-(A::AbstractArray{T}, b::Date) where {T<:Date}
```

generates ambiguities for anyone who defines a method

```julia
-(A::MyArrayType{T}, b::T) where {T}
```

The best approach is to avoid defining *either* of these methods:
instead, rely on a generic method `-(A::AbstractArray, b)` and make
sure this method is implemented with generic calls (like `similar` and
`-`) that do the right thing for each container type and element type
*separately*. This is just a more complex variant of the advice to
[orthogonalize](@ref man-methods-orthogonalize) your methods.

When this approach is not possible, it may be worth starting a
discussion with other developers about resolving the ambiguity; just
because one method was defined first does not necessarily mean that it
can't be modified or eliminated.  As a last resort, one developer can
define the "band-aid" method

```julia
-(A::MyArrayType{T}, b::Date) where {T<:Date} = ...
```

that resolves the ambiguity by brute force.

### Complex method "cascades" with default arguments

If you are defining a method "cascade" that supplies defaults, be
careful about dropping any arguments that correspond to potential
defaults. For example, suppose you're writing a digital filtering
algorithm and you have a method that handles the edges of the signal
by applying padding:

```julia
function myfilter(A, kernel, ::Replicate)
    Apadded = replicate_edges(A, size(kernel))
    myfilter(Apadded, kernel)  # now perform the "real" computation
end
```

This will run afoul of a method that supplies default padding:

```julia
myfilter(A, kernel) = myfilter(A, kernel, Replicate()) # replicate the edge by default
```

Together, these two methods generate an infinite recursion with `A` constantly growing bigger.

The better design would be to define your call hierarchy like this:

```julia
struct NoPad end  # indicate that no padding is desired, or that it's already applied

myfilter(A, kernel) = myfilter(A, kernel, Replicate())  # default boundary conditions

function myfilter(A, kernel, ::Replicate)
    Apadded = replicate_edges(A, size(kernel))
    myfilter(Apadded, kernel, NoPad())  # indicate the new boundary conditions
end

# other padding methods go here

function myfilter(A, kernel, ::NoPad)
    # Here's the "real" implementation of the core computation
end
```

`NoPad` is supplied in the same argument position as any other kind of
padding, so it keeps the dispatch hierarchy well organized and with
reduced likelihood of ambiguities. Moreover, it extends the "public"
`myfilter` interface: a user who wants to control the padding
explicitly can call the `NoPad` variant directly.

[^Clarke61]: Arthur C. Clarke, *Profiles of the Future* (1961): Clarke's Third Law.
