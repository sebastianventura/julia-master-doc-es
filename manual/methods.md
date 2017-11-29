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

En este ejemplo, observe que la nueva definición para `newfun` ha sido creada, pero no puede ser llamada inmediatamente.
El nuevo global es inmediatamente visible para la función `tryeval`, para que pueda escribir `return newfun` (sin paréntesis).
¡Pero ni usted ni ninguna de las personas que llama, ni las funciones a las que llama, etc. puede llamar a esta nueva definición de método!

Pero hay una excepción: las llamadas futuras a `newfun` *del REPL* funcionan como se esperaba, pudiendo tanto ver como invocar la nueva definición de` newfun`. Sin embargo, las futuras llamadas a `tryeval` continuarán viendo la definición de` newfun` tal como era *en la instrucción anterior en REPL*, y por lo tanto antes de esa llamada a `tryeval`.

Es posible que desee probar esto para ver cómo funciona.

La implementación de este comportamiento es un "contador de edad mundial". Este valor monótonamente creciente rastrea cada operación de definición de método. Esto permite describir "el conjunto de definiciones de métodos visibles para un entorno de tiempo de ejecución dado" como un solo número, o "edad mundial". También permite comparar los métodos disponibles en dos mundos simplemente comparando su valor ordinal. En el ejemplo anterior, vemos que el "mundo actual" (en el que existe el método `newfun ()`) es uno mayor que el "mundo de tiempo de ejecución" local de la tarea que se corrigió cuando se inició la ejecución de `tryeval`.

A veces es necesario evitar esto (por ejemplo, si está implementando el REPL anterior). Afortunadamente, hay una solución fácil: llamar a la función usando [`Base.invokelatest`] (@ ref):

```jldoctest
julia> function tryeval2()
           @eval newfun2() = 2
           Base.invokelatest(newfun2)
       end
tryeval2 (generic function with 1 method)

julia> tryeval2()
2
```
Por último, echemos un vistazo a algunos ejemplos más complejos donde esta regñla se pone en funcionamiento. Definamos una función `f(x)`, que inicialmente tiene un método:

```jldoctest redefinemethod
julia> f(x) = "original definition"
f (generic function with 1 method)
```

Iniciamos algunas operaciones que usan `f(x)`:

```jldoctest redefinemethod
julia> g(x) = f(x)
g (generic function with 1 method)

julia> t = @async f(wait()); yield();
```

Ahora añadimos algunos métodos nuevos a `f(x)`:

```jldoctest redefinemethod
julia> f(x::Int) = "definition for Int"
f (generic function with 2 methods)

julia> f(x::Type{Int}) = "definition for Type{Int}"
f (generic function with 3 methods)
```

Compare cómo difieren estos resultados:

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

Los parámetros de función pueden también ser usados para restringir el número de argumentos que pueden ser proporcionados a una función "varags" (ver [Funciones Vararg](@ref)).  La notación `Vararg{T,N}` se usa para indicar tal restricción. Por ejemplo:

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

Más útil aún, es posible restringir métodos varargs mediante un parámetro. Por ejemplo:

```julia
function getindex(A::AbstractArray{T,N}, indexes::Vararg{Number,N}) where {T,N}
```

sería llamado sólo cuando el número de `indexes` se correspondiera con la dimensionalidad del array.

Cuando sólo el tipo de los argumentos propoprcionados tenga que ser restrincido , `Vararg{T}` puede escribirse de forma equivalente como `T...`. Por ejemplo `f(x::Int...) = x` es una abreviación de `f(x::Vararg{Int}) = x`.

## Note on Optional and keyword Arguments

Como se menciona brevemente en [Funciones](@ref man-functions), los argumentos opcionales se implementan como sintaxis para múltiples definiciones de métodos. Por ejemplo, esta definición:

```julia
f (a = 1, b = 2) = a + 2b
```

se traduce a los siguientes tres métodos:

```julia
f (a, b) = a + 2b
f (a) = f (a, 2)
f () = f (1,2)
```

Esto significa que llamar a `f()` es equivalente a llamar a `f(1,2)`. En este caso, el resultado es `5`, porque` f (1,2) `invoca el primer método de `f` anterior. Sin embargo, este no siempre es el caso. Si define un cuarto método que es más especializado para enteros:

```julia
f (a :: Int, b :: Int) = a-2b
```

entonces el resultado de ambos `f()` y `f(1,2)` es `-3`. En otras palabras, los argumentos opcionales están vinculados a una función, no a ningún método específico de esa función. Depende de los tipos de argumentos opcionales qué método se invoca. Cuando los argumentos opcionales se definen en términos de una variable global, el tipo de argumento opcional puede incluso cambiar en tiempo de ejecución.

Los argumentos de palabra clave se comportan de manera bastante diferente de los argumentos posicionales ordinarios. En particular, no participan en el envío del método. Los métodos se envían basados *únicamente en argumentos posicionales, con argumentos de palabra clave procesados* después de que se identifica el método de coincidencia.

## Funciones como objetos

Los métodos están asociados con tipos, por lo que es posible hacer algún objeto Julia arbitrario "invocable" añadiendo métoos a este tipo (tales objetos "invocables" son denominados en ocasiones "functores"). 

Por ejemplo, podemos definir un tipo que almacena los coeficientes de un polinomio, pero que se comporta como una función que evalúa el polinomio:

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

Note que la función es especificada por el tipo en lugar de por el nombre. En el cuerpo de la función, `p ` se referira´al objeto que fue llamado. Un `Polynomial` puede ser usado como sigue:

```jldoctest polynomial
julia> p = Polynomial([1,10,100])
Polynomial{Int64}([1, 10, 100])

julia> p(3)
931
```

Este mecanismo es también la clave de cómo los constructores de tipo y cierres (funciones internas que se refieren al entorno que las rodea) funcionan en Julia, lo cuñal se discute [después en el manual](@ref constructors-and-conversion).

## Empty generic functions

Ocasionalmente, es útil introducir una función genérica sin agregar métodos. Esto se puede usar para separar las definiciones de interfaz de las implementaciones. También se puede hacer con el fin de la documentación o la legibilidad del código. La sintaxis para esto es un bloque de `function` vacío sin una tupla de argumentos:

```julia
function emptyfunc
end
```

## [Diseño de métodos y evitación de ambigüedades](@id man-method-design-ambiguities)

El polimorfismo de los métodos de Julia es una de sus características más poderosas, pero explotar este poder puede plantear desafíos de diseño. En particular, en jerarquías de métodos más complejos no es raro que surjan [ambigüedades](@ref man-ambiguities).

Arriba se indicó que uno puede resolver ambigüedades como

```julia
f(x, y::Int) = 1
f(x::Int, y) = 2
```

definiendo un método

```julia
f(x::Int, y::Int) = 3
```

Esta es a menudo la estrategia correcta; sin embargo, hay circunstancias en las que seguir este consejo a ciegas puede ser contraproducente. En particular, cuantos más métodos tenga una función genérica, más posibilidades habrá de ambigüedades. Cuando sus jerarquías de métodos se vuelven más complicadas que este simple ejemplo, puede valer la pena pensar cuidadosamente sobre estrategias alternativas.

A continuación, discutimos los desafíos particulares y algunas formas alternativas de resolver dichos problemas.

### Tuple and NTuple arguments

Los argumentos `Tuple` (y `NTuple`) presentan retos especiales. Por ejemplo,

```julia
f(x::NTuple{N,Int}) where {N} = 1
f(x::NTuple{N,Float64}) where {N} = 2
```

son ambiguos debido a la posibilidad de que `N == 0`: no hay elementos para determinar si se debe invocar a la variante` Int` o `Float64`. Para resolver la ambigüedad, un enfoque es definir un método para la tupla vacía:

```julia
f(x::Tuple{}) = 3
```

Alternativamente, para todos los métodos excepto uno podemos insistir en que hay al menos un elemento en la tupla:

```julia
f(x::NTuple{N,Int}) where {N} = 1           # this is the fallback
f(x::Tuple{Float64, Vararg{Float64}}) = 2   # this requires at least one Float64
```

### [Orthogonalice su diseño](@id man-methods-orthogonalize)

Cuando tenga la tentación de despachar en dos o más argumentos, considere si una función de "envoltura" podría hacer un diseño más simple. Por ejemplo, en lugar de escribir múltiples variantes:

```julia
f (x :: A, y :: A) = ...
f (x :: A, y :: B) = ...
f (x :: B, y :: A) = ...
f (x :: B, y :: B) = ...
```

podría considerar definir

```julia
f (x :: A, y :: A) = ...
f (x, y) = f (g (x), g (y))
```

donde `g` convierte el argumento para escribir `A`. Esto es un ejemplo muy específico del principio más general de [diseño ortogonal](https://en.wikipedia.org/wiki/Orthogonality_ (programación)), en el que los conceptos separados se alinean a métodos separados. Aquí, `g` muy probablemente necesitará una definición de repliegue

```julia
g (x :: A) = x
```

Una estrategia relacionada explota `promote` para llevar` x` y `y` a un tipo común:

```julia
f (x :: T, y :: T) donde {T} = ...
f (x, y) = f (promover (x, y) ...)
```

Un riesgo de este diseño es la posibilidad de que si no hay un método de promoción adecuado para convertir `x` y` y` al mismo tipo, el segundo método se repetirá en sí mismo infinitamente y desencadenará un desbordamiento de la pila. La función no exportada `Base.promote_noncircular` se puede usar como alternativa; cuando la promoción falla, aún arrojará un error, pero uno que falla más rápido con un mensaje de error más específico.

### Despacho en un argumento a la vez

Si necesita despachar en múltiples argumentos, y hay muchos retrocesos con demasiadas combinaciones para que sea práctico definir todas las variantes posibles, entonces considere introducir una "cascada de nombres" donde (por ejemplo) despache en el primer argumento y luego llame un método interno:

```julia
f (x :: A, y) = _fA (x, y)
f (x :: B, y) = _fB (x, y)
```

Entonces los métodos internos `_fA` y` _fB` pueden enviarse en `y` sin preocuparse por las ambigüedades entre sí con respecto a `x`.

Tenga en cuenta que esta estrategia tiene al menos una desventaja importante: en muchos casos, no es posible para los usuarios personalizar aún más el comportamiento de `f` definiendo más especializaciones de su función` f` exportada. En su lugar, tienen que definir especializaciones para sus métodos internos `_fA` y` _fB`, y esto borra las líneas entre los métodos exportados e internos.

### Contenedores abstractos y tipos de elementos

Donde sea posible, trate de evitar definir los métodos que se despachan en tipos de elementos específicos de contenedores abstractos. Por ejemplo,

```julia
- (A :: AbstractArray {T}, b :: Date) donde {T <: Date}
```

genera ambigüedades para cualquiera que defina un método

```julia
- (A :: MyArrayType {T}, b :: T) donde {T}
```

El mejor enfoque es evitar definir *cualquiera* de estos métodos: en su lugar, confíe en un método genérico `-(A::AbstractArray, b)` y haga Asegúrese de que este método se implemente con llamadas genéricas (como `similar` y
`-`) que hacen lo correcto para cada tipo de contenedor y tipo de elemento *por separado*. Esta es solo una variante más compleja de los consejos para [ortogonalize](@ ref man-methods-ortogonalize) sus métodos.

Cuando este enfoque no es posible, puede valer la pena comenzar un discusión con otros desarrolladores sobre la resolución de la ambigüedad; sólo porque un método se definió primero no necesariamente significa que no puede ser modificado o eliminado Como último recurso, un desarrollador puede definir el método de "curita"

```julia
- (A :: MyArrayType {T}, b :: Date) donde {T <: Date} = ...
```

eso resuelve la ambigüedad por la fuerza bruta.

### Método complejo "cascadas" con argumentos predeterminados

Si está definiendo un método "cascada" que suministra valores predeterminados, sea cuidado al eliminar cualquier argumento que corresponda al potencial por defecto. Por ejemplo, supongamos que estás escribiendo un filtro digital algoritmo y usted tiene un método que maneja los bordes de la señal mediante la aplicación de relleno:

```julia
function myfilter (A, kernel, :: Replicate)
    Apadded = replicate_edges (A, tamaño (kernel))
    myfilter (Apadded, kernel) # ahora realiza el cálculo "real"
fin
```

Esto entrará en conflicto con un método que proporciona relleno predeterminado:

```julia
myfilter (A, kernel) = myfilter (A, kernel, Replicate ()) # replica el borde por defecto
```

Juntos, estos dos métodos generan una recursión infinita con 'A' creciendo cada vez más.

El mejor diseño sería definir su jerarquía de llamadas de esta manera:

```julia
struct NoPad end # indica que no se desea relleno, o que ya está aplicado

myfilter (A, kernel) = myfilter (A, kernel, Replicate ()) # condiciones de contorno predeterminadas

function myfilter (A, kernel, :: Replicate)
    Apadded = replicate_edges (A, tamaño (kernel))
    myfilter (Apadded, kernel, NoPad ()) # indican las nuevas condiciones de contorno
fin

# otros métodos de relleno van aquí

function myfilter (A, kernel, :: NoPad)
    # Aquí está la implementación "real" de la computación central
fin
```

`NoPad` se proporciona en la misma posición de argumento que cualquier otro tipo de relleno, por lo que mantiene la jerarquía de despacho bien organizada y con menor probabilidad de ambigüedades. Además, amplía lo "público"interfaz `myfilter`: un usuario que quiere controlar el relleno explícitamente puede llamar a la variante `NoPad` directamente.

[^Clarke61]: Arthur C. Clarke, *Profiles of the Future* (1961): Clarke's Third Law.
