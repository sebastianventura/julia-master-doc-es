# Guía de Estilo

Las siguientes secciones Explica unos cuantos aspectos del estilo de codificación idiomático de 
Julia. Ninguna de estas reglas son absolutas; sólo son sugerencias para ayudar a familiarizarte 
con el lenguaje y ayudarte a elegir entre diseños alternativos.

## Escribe funciones, no sólo *scripts*

Escribir código como una serie de pasos a nivel superior Es una forma rápida de empezar a 
resolver un problema, pero uno debería intentar dividir un programa en funciones tan pronto como 
sea posible. La función son más reusabes y testables, y clarifican qué pasos se están dando y 
cuáles son sus entradas y sus salidas. Además, el código dentro de las funciones tiende a ejecutar 
mucho más rápido que el código de nivel superior debido a cómo funciona el compilador de Julia.

También merece la pena señalar que las funciones deberían tomar argumentos, en lugar de operar
directamente sobre las variables globales (aparte de constantes como [`pi`](@ref)).

## Evita escribir tipos demasiado específicos

El código debería ser tan genérico como sea posible. En lugar de escribir:

```julia
convert(Complex{Float64}, x)
```

es mejor usar funciones genéricas disponibles:

```julia
complex(float(x))
```

La segunda versión convertirá `x` a un tipo apropiado, en lugrar de siempre al mismo tipo. 

Este punto de estilo es especialmente relevante para los argumentos de función. Por ejemplo, no declare que un argumento sea de tipo `Int` o [`Int32`](@ref) si realmente pudiera ser cualquier número entero, expresado con el tipo abstracto [`Integer`](@ref). De hecho, en muchos casos puede omitir el tipo de argumento por completo, a menos que sea necesario para eliminar la ambigüedad de otras definiciones de método, ya que se lanzará [`MethodError`](@ref) de todos modos si se pasa un tipo que no admite ninguna de las operaciones requeridas. (Esto se conoce como [*duck typing*] (https://en.wikipedia.org/wiki/Duck_typing).)

Por ejemplo, considere las siguientes definiciones de una función `addone` que devuelve uno más su argumento:


```julia
addone(x::Int) = x + 1                 # works only for Int
addone(x::Integer) = x + oneunit(x)    # any integer type
addone(x::Number) = x + oneunit(x)     # any numeric type
addone(x) = x + oneunit(x)             # any type supporting + and oneunit
```

La última definición de `addone` maneja cualquier tipo que soporte [`oneunit`](@ref) (que devuelve 1 en el mismo tipo que `x`, lo que evita la promoción de tipos no deseados) y  la función [`+`](@ref) con esos argumentos. La clave para darse cuenta es que no hay *ninguna penalización de rendimiento* para definir *solo* el general `addone (x) = x + oneunit (x)`, porque Julia compilará automáticamente versiones especializadas según sea necesario. Por ejemplo, la primera vez que llame a `addone(12)`, Julia compilará automáticamente una función especializada `addone` para argumentos `x :: Int`, reemplazando la llamada a `oneunit` por su valor` 1`. Por lo tanto, las primeras tres definiciones de 'addone' anteriores son completamente redundantes con la cuarta definición.

## Manejar el exceso de diversidad de argumentos en el "código llamador"

En lugar de:

```julia
function foo(x, y)
    x = Int(x); y = Int(y)
    ...
end
foo(x, y)
```

use:

```julia
function foo(x::Int, y::Int)
    ...
end
foo(Int(x), Int(y))
```

Este es mucho mejor estilo debido a que `foo` no acepta realmente números de todos los tipos, sino que necesita `Int` s.

Un problema aquí es que si una función requiere números enteros intrínsecamente, podría ser mejor forzar al autor de la llamada a decidir cómo deberían convertirse los no enteros (por ejemplo, redondeando por abajo o por arriba). Otro problema es que la declaración de tipos más específicos deja más "espacio" para las futuras definiciones de métodos.

## Añadir `!` para los nombres de funciones que modifican sus argumentos

En lugar de:

```julia
function double(a::AbstractArray{<:Number})
    for i = 1:endof(a)
        a[i] *= 2
    end
    return a
end
```

use:

```julia
function double!(a::AbstractArray{<:Number})
    for i = 1:endof(a)
        a[i] *= 2
    end
    return a
end
```

La biblioteca estándar de Julia usa esta convención y contiene ejemplos de funciones con formas tanto de copiado como de modificación (por ejemplo, [`sort()`](@ref) y [`sort!()`](@ref)), y otras que simplemente están modificando (por ejemplo, [`push!()`](@ref), [`pop!()`](@ref), [`splice!()`](@ref)). Es típico para tales funciones devolver también la matriz modificada por conveniencia.

## Evitar tipos `Union` extraños

Tipos tales como `Union{Function,AbstractString}` son frecuentemente un signo de que hay que limpiar algo en el diseño.

## Evitar las Uniones de tipos en campos

Cuando se crea un tipo tal como :

```julia
mutable struct MyType
    ...
    x::Union{Void,T}
end
```

pregunte si la opción de `x` para ser `nada` (de tipo `Void`) es realmente necesaria. Aquí hay algunas alternativas a considerar:

  * Encuentre un valor predeterminado seguro con el que inicializar `x`
  * Introduce otro tipo del que carece `x`
  * Si hay muchos campos como `x`, guárdelos en un diccionario
  * Determine si hay una regla simple para cuando `x` es` nada`. Por ejemplo, a menudo el campo comenzará 
    como `nada` pero se inicializará en algún punto bien definido. En ese caso, considere dejarlo indefinido 
    al principio.
  * Si `x` realmente no necesita contener ningún valor en algún momento, defínalo como `::Nullable{T}` en 
    su lugar, ya que esto garantiza estabilidad de tipo en el código que accede a este campo 
    (ver [TiposNullable](@ref man-nullable-types)).

## Avoid elaborate container types

It is usually not much help to construct arrays like the following:

```julia
a = Array{Union{Int,AbstractString,Tuple,Array}}(n)
```

In this case `Array{Any}(n)` is better. It is also more helpful to the compiler to annotate specific
uses (e.g. `a[i]::Int`) than to try to pack many alternatives into one type.

## Use naming conventions consistent with Julia's `base/`

  * modules and type names use capitalization and camel case: `module SparseArrays`, `struct UnitRange`.
  * functions are lowercase ([`maximum()`](@ref), [`convert()`](@ref)) and, when readable, with multiple
    words squashed together ([`isequal()`](@ref), [`haskey()`](@ref)). When necessary, use underscores
    as word separators. Underscores are also used to indicate a combination of concepts ([`remotecall_fetch()`](@ref)
    as a more efficient implementation of `fetch(remotecall(...))`) or as modifiers ([`sum_kbn()`](@ref)).
  * conciseness is valued, but avoid abbreviation ([`indexin()`](@ref) rather than `indxin()`) as
    it becomes difficult to remember whether and how particular words are abbreviated.

If a function name requires multiple words, consider whether it might represent more than one
concept and might be better split into pieces.

## Don't overuse try-catch

It is better to avoid errors than to rely on catching them.

## Don't parenthesize conditions

Julia doesn't require parens around conditions in `if` and `while`. Write:

```julia
if a == b
```

instead of:

```julia
if (a == b)
```

## Don't overuse `...`

Splicing function arguments can be addictive. Instead of `[a..., b...]`, use simply `[a; b]`,
which already concatenates arrays. [`collect(a)`](@ref) is better than `[a...]`, but since `a`
is already iterable it is often even better to leave it alone, and not convert it to an array.

## Don't use unnecessary static parameters

A function signature:

```julia
foo(x::T) where {T<:Real} = ...
```

should be written as:

```julia
foo(x::Real) = ...
```

instead, especially if `T` is not used in the function body. Even if `T` is used, it can be replaced
with [`typeof(x)`](@ref) if convenient. There is no performance difference. Note that this is
not a general caution against static parameters, just against uses where they are not needed.

Note also that container types, specifically may need type parameters in function calls. See the
FAQ [Avoid fields with abstract containers](@ref) for more information.

## Avoid confusion about whether something is an instance or a type

Sets of definitions like the following are confusing:

```julia
foo(::Type{MyType}) = ...
foo(::MyType) = foo(MyType)
```

Decide whether the concept in question will be written as `MyType` or `MyType()`, and stick to
it.

The preferred style is to use instances by default, and only add methods involving `Type{MyType}`
later if they become necessary to solve some problem.

If a type is effectively an enumeration, it should be defined as a single (ideally immutable struct or primitive)
type, with the enumeration values being instances of it. Constructors and conversions can check
whether values are valid. This design is preferred over making the enumeration an abstract type,
with the "values" as subtypes.

## Don't overuse macros

Be aware of when a macro could really be a function instead.

Calling [`eval()`](@ref) inside a macro is a particularly dangerous warning sign; it means the
macro will only work when called at the top level. If such a macro is written as a function instead,
it will naturally have access to the run-time values it needs.

## Don't expose unsafe operations at the interface level

If you have a type that uses a native pointer:

```julia
mutable struct NativeType
    p::Ptr{UInt8}
    ...
end
```

don't write definitions like the following:

```julia
getindex(x::NativeType, i) = unsafe_load(x.p, i)
```

The problem is that users of this type can write `x[i]` without realizing that the operation is
unsafe, and then be susceptible to memory bugs.

Such a function should either check the operation to ensure it is safe, or have `unsafe` somewhere
in its name to alert callers.

## Don't overload methods of base container types

It is possible to write definitions like the following:

```julia
show(io::IO, v::Vector{MyType}) = ...
```

This would provide custom showing of vectors with a specific new element type. While tempting,
this should be avoided. The trouble is that users will expect a well-known type like `Vector()`
to behave in a certain way, and overly customizing its behavior can make it harder to work with.

## Avoid type piracy

"Type piracy" refers to the practice of extending or redefining methods in Base
or other packages on types that you have not defined. In some cases, you can get away with
type piracy with little ill effect. In extreme cases, however, you can even crash Julia
(e.g. if your method extension or redefinition causes invalid input to be passed to a
`ccall`). Type piracy can complicate reasoning about code, and may introduce
incompatibilities that are hard to predict and diagnose.

As an example, suppose you wanted to define multiplication on symbols in a module:

```julia
module A
import Base.*
*(x::Symbol, y::Symbol) = Symbol(x,y)
end
```

The problem is that now any other module that uses `Base.*` will also see this definition.
Since `Symbol` is defined in Base and is used by other modules, this can change the
behavior of unrelated code unexpectedly. There are several alternatives here, including
using a different function name, or wrapping the `Symbol`s in another type that you define.

Sometimes, coupled packages may engage in type piracy to separate features from definitions,
especially when the packages were designed by collaborating authors, and when the
definitions are reusable. For example, one package might provide some types useful for
working with colors; another package could define methods for those types that enable
conversions between color spaces. Another example might be a package that acts as a thin
wrapper for some C code, which another package might then pirate to implement a
higher-level, Julia-friendly API.

## Be careful with type equality

You generally want to use [`isa()`](@ref) and [`<:`](@ref) for testing types,
not `==`. Checking types for exact equality typically only makes sense when comparing to a known
concrete type (e.g. `T == Float64`), or if you *really, really* know what you're doing.

## Do not write `x->f(x)`

Since higher-order functions are often called with anonymous functions, it is easy to conclude
that this is desirable or even necessary. But any function can be passed directly, without being
"wrapped" in an anonymous function. Instead of writing `map(x->f(x), a)`, write [`map(f, a)`](@ref).

## Avoid using floats for numeric literals in generic code when possible

If you write generic code which handles numbers, and which can be expected to run with many different
numeric type arguments, try using literals of a numeric type that will affect the arguments as
little as possible through promotion.

For example,

```jldoctest
julia> f(x) = 2.0 * x
f (generic function with 1 method)

julia> f(1//2)
1.0

julia> f(1/2)
1.0

julia> f(1)
2.0
```

while

```jldoctest
julia> g(x) = 2 * x
g (generic function with 1 method)

julia> g(1//2)
1//1

julia> g(1/2)
1.0

julia> g(1)
2
```

As you can see, the second version, where we used an `Int` literal, preserved the type of the
input argument, while the first didn't. This is because e.g. `promote_type(Int, Float64) == Float64`,
and promotion happens with the multiplication. Similarly, [`Rational`](@ref) literals are less type disruptive
than [`Float64`](@ref) literals, but more disruptive than `Int`s:

```jldoctest
julia> h(x) = 2//1 * x
h (generic function with 1 method)

julia> h(1//2)
1//1

julia> h(1/2)
1.0

julia> h(1)
2//1
```

Thus, use `Int` literals when possible, with `Rational{Int}` for literal non-integer numbers,
in order to make it easier to use your code.
