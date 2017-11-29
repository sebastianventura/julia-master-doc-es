# [Conversión and Promoción](@id conversion-and-promotion)

Julia tiene un sistema para promocionar argumento de operaciones matemáticas a un tipo común, que ha sido mencionado en varias secciones, incluyendo [números enteros y en punto flotante](@ref), [operaciones matemáticas y funciones elmentales](@ref), [tipos](@ref man-types), and [métodos](@ref). En esta sección, explicaremos cómo funciona este sistema de promociones, y también cómo extenderlo a nuevos tipos y aplicarlo a funciones junto a operadores matemáticos predefinidos. Tradicionalmente, los lenguajes de programación caen en dos categorías con respecto a la promoción de los argumentos aritméticos:

  * **Promoción automática para operadores y tipos artiméticos predefinidos**. En la mayoría de los lenguajes, los tipos
    numéricos predefinidos, cuando se usan como operandos de operaciones aritméticas con una sintaxis infija, tal y 
    como `+`, `-`, `*`y `/`, son promocionados automáticamente a un tipo común para producir el resultado esperado. C, 
    Java, Perl y Pytho,, por nombrar unos pocos, calculan todos la suma `1 + 1.5` correctamente como el valor en punto
    flotante `2.5`, incluso aunque uno de los operandos sea un entero. Estos sistemas son convenientes y diseñados
    cuidadosamente de forma que generalmente realizan esta labor de forma invisible al programador: a duras penas, 
    alguien consciente piensa que esta promoción está teniendo lugar cuando escribe una expresión, pero los 
    compiladores e intérpretes deben realizar la conversión antes de la adición debido a que los valores enteros y en 
    punto flotante no pueden sumarse tal cual. Por tanto, reglas complejas para tales conversiones automáticas son 
    una parte inevitable de la especificación e implementación de estos lenguajes.
  * **No promoción automática** Este campo incluy a Ada y ML (lenguajes tipados estáticamente y muy estrictos). En 
    estos lenguajes, cada conversión debe ser especificada por el programador de foma explícita. Por tanto, la expresión 
    de ejemplo `1 + 1.5` daría un error de compilación en ambos lenguajes. En lugar de esta expresión, uno debería 
    escribir `real(1) + 1.5`, convirtiendo explícitamente el 1 enteror a un valores en punto flotante antes de realizar la
    adición. La conversión explícita en todos sitios es tan inconveniente, sin embargo, que incluso Ada tiene algún grado 
    de conversión automática.: los literales enteros son promocionaods al tipo entero esperado automáticamente, y los
    literales en punto flotante son promocionados similarmente a los tipos apropiados en punto flotante.

En cierto sentido, Julia cae en la categoría "no promoción automática": los operadores automáticos son funciones con sintaxis especial, y los argumentos de funciones no son nunca convertidos automáticamente. Sin embargo, uno puede observar que aplicar operaciones matermáticas a una amplia variedad de tipos de argumentos mixtos es justo un caso extremo del despacho múltiple polimórfico (algo que el despacho de Julia y los sistemas de tipos manejan bastante bien. La promoción "automática" de operandos matermáticos simplemente emerge como una aplicación especial: Julia viene un reglas de despacho "atrapa-todo" predefinidas para los operadores matemáticos, invocadas cuando no existen implementaciones específicas para alguna combinación de tipos de operandos. Estas reglas "atrapa-todo" primero promocionan todos los operandos a un tipo común usando reglas de promoción definibles por el usuario, y luego invoca a una implementación especializada del operador en cuestión para los valores resultantes, ahora del mismo tipo. Los tipos definidos por el usuario pueden participar fácilmente en este sistema de promoción definiendo métodos para la conversión hacia o desde otros tipos, y proporcionar un puñado de reglas de promoción que definan a qué tipos deberían ellos promocionarse cuando se mezclan con otros tipos.

## Conversión

La conversión de valores a varios tipos es llevada a cabo mediante la función `convert`. Esta función suele tomar dos argumentos: el primero es un objeto tipo mientras que el segundo es un valor que hay que convertir a ese tipo; el valor devuelto es el valor convertido a una instancia del tipo dado. La forma más simple de comprender esta función es verla en acción:

```jldoctest
julia> x = 12
12

julia> typeof(x)
Int64

julia> convert(UInt8, x)
0x0c

julia> typeof(ans)
UInt8

julia> convert(AbstractFloat, x)
12.0

julia> typeof(ans)
Float64

julia> a = Any[1 2 3; 4 5 6]
2×3 Array{Any,2}:
 1  2  3
 4  5  6

julia> convert(Array{Float64}, a)
2×3 Array{Float64,2}:
 1.0  2.0  3.0
 4.0  5.0  6.0
```

La conversión no es siempre posible, en cuyo caso se lanza un error no método indicando que `convert` no sabe cómo realizar la conversión solicitada:

```jldoctest
julia> convert(AbstractFloat, "foo")
ERROR: MethodError: Cannot `convert` an object of type String to an object of type AbstractFloat
This may have arisen from a call to the constructor AbstractFloat(...),
since type constructors fall back to convert methods.
```

Algunos lenguajes consideran que el análisis sintáctico de cadenas como número o el formateo de números a cadenas es una conversión (muchos lengaujes dinámicos realizarán esta conversión por ti automáticamente). Sin embargo, Julia no lo hace. Incluso aunque algunas cadenas puedan ser analizadas como números, la mayoría de las cadenas no son representaciones válidas de números, y sólo un subconjunto muy limitado de ellas lo son. Por tanto, en Julia la función  dedicada `parse()` debe ser usada para realizar esta operación, haciéndolo más explícito.

### Definiendo nuevas conversiones

Para definir nuevas conversiones, simpemente propocionaremos un nuevo método para `convert()`. Esto es realmente todo lo que hay que hacer. Por ejemplo, el método para convertir un número real a boolean es:

```julia
convert(::Type{Bool}, x::Real) = x==0 ? false : x==1 ? true : throw(InexactError())
```

El tipo del primer argumento de este método es un [tipo singleton](@ref man-singleton-types), `Type{Bool}`, , la única instancia del cuál es `Bool`. Por tanto, este método es sólo invocado cuando el primer argumento es el valor tipo `Bool`. Nótese la sintaxis usada pora el primer argumento: el nombre del argumento es omitido antes del símbolo `::` y sólo se da el tipo. Esta es la sintaxis de Julia para un argumento a función cuyo tipo está especificado pero su valor nunca se utilizará en el cuerpo de la función. En este ejemplo, como el tipo es un singleton, nunca habría una razón para usar su valor dentro del cuerpo. Cuando se invoca el método determina si un valor numérico es verdadero o falso como un boolean, comparándolo con uno y con cero:

```jldoctest
julia> convert(Bool, 1)
true

julia> convert(Bool, 0)
false

julia> convert(Bool, 1im)
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Bool}, ::Complex{Int64}) at ./complex.jl:31

julia> convert(Bool, 0im)
false
```

Las signaturas de método para métodos de conversión están frecuentemente un poco más implicadas que este ejemplo, especialmente para tipos paramétricos. El ejemplo de antes está pensado para ser pedagógico, y no es el comportamiento actual de Julia. He aquí la implementación actual en Julia:

```julia
convert(::Type{T}, z::Complex) where {T<:Real} =
    (imag(z) == 0 ? convert(T, real(z)) : throw(InexactError()))
```

### [Cas o de estudio: Conversiones de `Rational`](@id man-rational-conversion)

Para continuar con nuestro caso de estudio sobre el tipo [`Rational`](@ref) de Julia, he aquí las conversiones declaradas en [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl), después de la declaración del tipo y sus constructores:

```julia
convert(::Type{Rational{T}}, x::Rational) where {T<:Integer} = Rational(convert(T,x.num),convert(T,x.den))
convert(::Type{Rational{T}}, x::Integer) where {T<:Integer} = Rational(convert(T,x), convert(T,1))

function convert(::Type{Rational{T}}, x::AbstractFloat, tol::Real) where T<:Integer
    if isnan(x); return zero(T)//zero(T); end
    if isinf(x); return sign(x)//zero(T); end
    y = x
    a = d = one(T)
    b = c = zero(T)
    while true
        f = convert(T,round(y)); y -= f
        a, b, c, d = f*a+c, f*b+d, a, b
        if y == 0 || abs(a/b-x) <= tol
            return a//b
        end
        y = 1/y
    end
end
convert(rt::Type{Rational{T}}, x::AbstractFloat) where {T<:Integer} = convert(rt,x,eps(x))

convert(::Type{T}, x::Rational) where {T<:AbstractFloat} = convert(T,x.num)/convert(T,x.den)
convert(::Type{T}, x::Rational) where {T<:Integer} = div(convert(T,x.num),convert(T,x.den))
```

Los cuatro primeros métodos `convert` proporcionan conversión a tipos racionales. El primer método converte el tipo de reacional a otro tipo de racional convirtiendo el nomerador y el denominador al tipo de entero apropiado. El segundo método hace la misma conversión para enteros tomando el denominado para que sea 1. El tercer método implementa un aalgoritmo estándar para aproximar un número de punto flotante por una razón de enteros dentro de un a tolerancia dada, y el cuarto método lo aplica, usando el epsilo de máquina como el valor dado para elumbral. En general, uno debería tener `a//b == convert(Rational{Int64}, a/b)`.

Los dos últimos métodos conversores proporcionan conversiones de tipos racionales a punto flotante y entero. Para convertir a punto flotante, uno simplemente convierte tanto numerador como denominador a punto flotante y luego divide. Para convertir a entero, uno usa el operador `div` para división entera truncada (redondeo hacia cero).

## Promotion

Promotion refers to converting values of mixed types to a single common type. Although it is not
strictly necessary, it is generally implied that the common type to which the values are converted
can faithfully represent all of the original values. In this sense, the term "promotion" is appropriate
since the values are converted to a "greater" type -- i.e. one which can represent all of the
input values in a single common type. It is important, however, not to confuse this with object-oriented
(structural) super-typing, or Julia's notion of abstract super-types: promotion has nothing to
do with the type hierarchy, and everything to do with converting between alternate representations.
For instance, although every [`Int32`](@ref) value can also be represented as a [`Float64`](@ref) value,
`Int32` is not a subtype of `Float64`.

Promotion to a common "greater" type is performed in Julia by the `promote` function, which takes
any number of arguments, and returns a tuple of the same number of values, converted to a common
type, or throws an exception if promotion is not possible. The most common use case for promotion
is to convert numeric arguments to a common type:

```jldoctest
julia> promote(1, 2.5)
(1.0, 2.5)

julia> promote(1, 2.5, 3)
(1.0, 2.5, 3.0)

julia> promote(2, 3//4)
(2//1, 3//4)

julia> promote(1, 2.5, 3, 3//4)
(1.0, 2.5, 3.0, 0.75)

julia> promote(1.5, im)
(1.5 + 0.0im, 0.0 + 1.0im)

julia> promote(1 + 2im, 3//4)
(1//1 + 2//1*im, 3//4 + 0//1*im)
```

Floating-point values are promoted to the largest of the floating-point argument types. Integer
values are promoted to the larger of either the native machine word size or the largest integer
argument type. Mixtures of integers and floating-point values are promoted to a floating-point
type big enough to hold all the values. Integers mixed with rationals are promoted to rationals.
Rationals mixed with floats are promoted to floats. Complex values mixed with real values are
promoted to the appropriate kind of complex value.

That is really all there is to using promotions. The rest is just a matter of clever application,
the most typical "clever" application being the definition of catch-all methods for numeric operations
like the arithmetic operators `+`, `-`, `*` and `/`. Here are some of the catch-all method definitions
given in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl):

```julia
+(x::Number, y::Number) = +(promote(x,y)...)
-(x::Number, y::Number) = -(promote(x,y)...)
*(x::Number, y::Number) = *(promote(x,y)...)
/(x::Number, y::Number) = /(promote(x,y)...)
```

These method definitions say that in the absence of more specific rules for adding, subtracting,
multiplying and dividing pairs of numeric values, promote the values to a common type and then
try again. That's all there is to it: nowhere else does one ever need to worry about promotion
to a common numeric type for arithmetic operations -- it just happens automatically. There are
definitions of catch-all promotion methods for a number of other arithmetic and mathematical functions
in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl), but beyond
that, there are hardly any calls to `promote` required in the Julia standard library. The most
common usages of `promote` occur in outer constructors methods, provided for convenience, to allow
constructor calls with mixed types to delegate to an inner type with fields promoted to an appropriate
common type. For example, recall that [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl)
provides the following outer constructor method:

```julia
Rational(n::Integer, d::Integer) = Rational(promote(n,d)...)
```

This allows calls like the following to work:

```jldoctest
julia> Rational(Int8(15),Int32(-5))
-3//1

julia> typeof(ans)
Rational{Int32}
```

For most user-defined types, it is better practice to require programmers to supply the expected
types to constructor functions explicitly, but sometimes, especially for numeric problems, it
can be convenient to do promotion automatically.

### Defining Promotion Rules

Although one could, in principle, define methods for the `promote` function directly, this would
require many redundant definitions for all possible permutations of argument types. Instead, the
behavior of `promote` is defined in terms of an auxiliary function called `promote_rule`, which
one can provide methods for. The `promote_rule` function takes a pair of type objects and returns
another type object, such that instances of the argument types will be promoted to the returned
type. Thus, by defining the rule:

```julia
promote_rule(::Type{Float64}, ::Type{Float32}) = Float64
```

one declares that when 64-bit and 32-bit floating-point values are promoted together, they should
be promoted to 64-bit floating-point. The promotion type does not need to be one of the argument
types, however; the following promotion rules both occur in Julia's standard library:

```julia
promote_rule(::Type{UInt8}, ::Type{Int8}) = Int
promote_rule(::Type{BigInt}, ::Type{Int8}) = BigInt
```

In the latter case, the result type is [`BigInt`](@ref) since `BigInt` is the only type
large enough to hold integers for arbitrary-precision integer arithmetic. Also note that
one does not need to define both `promote_rule(::Type{A}, ::Type{B})` and
`promote_rule(::Type{B}, ::Type{A})` -- the symmetry is implied by the way `promote_rule`
is used in the promotion process.

The `promote_rule` function is used as a building block to define a second function called `promote_type`,
which, given any number of type objects, returns the common type to which those values, as arguments
to `promote` should be promoted. Thus, if one wants to know, in absence of actual values, what
type a collection of values of certain types would promote to, one can use `promote_type`:

```jldoctest
julia> promote_type(Int8, UInt16)
Int64
```

Internally, `promote_type` is used inside of `promote` to determine what type argument values
should be converted to for promotion. It can, however, be useful in its own right. The curious
reader can read the code in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl),
which defines the complete promotion mechanism in about 35 lines.

### Case Study: Rational Promotions

Finally, we finish off our ongoing case study of Julia's rational number type, which makes relatively
sophisticated use of the promotion mechanism with the following promotion rules:

```julia
promote_rule(::Type{Rational{T}}, ::Type{S}) where {T<:Integer,S<:Integer} = Rational{promote_type(T,S)}
promote_rule(::Type{Rational{T}}, ::Type{Rational{S}}) where {T<:Integer,S<:Integer} = Rational{promote_type(T,S)}
promote_rule(::Type{Rational{T}}, ::Type{S}) where {T<:Integer,S<:AbstractFloat} = promote_type(T,S)
```

The first rule says that promoting a rational number with any other integer type promotes to a
rational type whose numerator/denominator type is the result of promotion of its numerator/denominator
type with the other integer type. The second rule applies the same logic to two different types
of rational numbers, resulting in a rational of the promotion of their respective numerator/denominator
types. The third and final rule dictates that promoting a rational with a float results in the
same type as promoting the numerator/denominator type with the float.

This small handful of promotion rules, together with the [conversion methods discussed above](@ref man-rational-conversion),
are sufficient to make rational numbers interoperate completely naturally with all of Julia's
other numeric types -- integers, floating-point numbers, and complex numbers. By providing appropriate
conversion methods and promotion rules in the same manner, any user-defined numeric type can interoperate
just as naturally with Julia's predefined numerics.
