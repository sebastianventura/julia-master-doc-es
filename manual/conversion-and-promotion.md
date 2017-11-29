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

### [Caso de estudio: Conversiones de `Rational`](@id man-rational-conversion)

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

## Promoción

La promoción se refiere a convetir valores de tipos mezclados a un solo tipo común. Aunque esto no es estrictamente necesario, se supone generalmente que el tipo común al cuál los valores son convertidos puede representar de forma fidedigna todos los valores. En este sentido, el término "promoción" es apropiado ya que los valores son convertidos a un tipo "mayor" (es decir, uno que pueda reprsentar todos los valores de entrada en un solo tipo común). Es importante, sin embargo, no confundir esto con los super-tipos orientados a objetos (estructurales), o l nocíon de super-tipos abstractos de Julia: la promoción no tiene nada que ver con la jerarquía de tipos, y todo que ver con convertir entre representaciones alternas. Por ejemplo, aunque cad valor [`Int32`](@ref) puede también ser representado como un valor [`Float64`](@ref), `Int32` no es un subtipo de `Float64`.

La promoción a un tipo mayor común es realizada por Julia mediante la función `promote`, que toma cualquier número de argumentos, y devuelve un atupla con el mismo número de valores, convertidos a un tipo común, o lanza una excepción si no es posible la promoción. El caso de uso más común para la promoción es convertir argumentos numéricos a un tipo común:

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

Los valores en punto flotante son promocionados al mayor de los tipos de los argumento en punto flotante. Los valores enteros son promocionados al mayor de el tamaño de palabra de máquina nativo o dl mayor tipo dde argumento entero. Las mezclas de valores enteros y en punto flotante son promocionados a tipos en punto flotante bastent grandes como para almacenar todos los valores. Los enteros mezclados con racionales promocionan a racionales. Los racionales mezclados con valores en punto flotante son promocionados a valores en punto flotante. Los valores complejos mezclados con valores relaes se promocionan al tipo apropiado de valor complejo.

Esto es realmente todo lo que es usar promociones. El resto es sólo cuestión de una aplicación inteligente, siendo la definición de métodos "atrapa-todo" para operaciones numéricas tales como las aritméticas `+`, `-`, `*` y `/` las más típicas aplicaciones inteligentes. He aquí algunas de las definiciones de métodos "atrapa-todo" dados en [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl):

```julia
+(x::Number, y::Number) = +(promote(x,y)...)
-(x::Number, y::Number) = -(promote(x,y)...)
*(x::Number, y::Number) = *(promote(x,y)...)
/(x::Number, y::Number) = /(promote(x,y)...)
```

Estas definiciones de métodos dicen que en la ausencia de reglas más específicas para sumar, restar, multiplicar y dividir pares de valores numéricos, promocionemos los valores a un tipo común y entonces intentemos de nuevo. Este es todo aquí: en ninguna otra parte hay que preocuparse por la promoción a un tipo numérico común para las operaciones aritméticas (ello sucede automáticamente). Hay definiciones de métodos de promoción "atrapa-todo" para un número de otras funciones arimtéticas y matemáticas en [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl), pero más allá de eso, difícilmente encontremos ninguna llamada a `promote` necesaria en la librería estándar de Julia. Los usos más comunes de `promote` ocurren en los métodos de construcción externos, proporcionados por conveniencia, para permitir llamadas a constructor con tipos mezclados para delegar a un tipo interno con campos promocionados a un  tipo común aproximado. Por ejemplo, recordemos que [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl) proporciona el siguiente método constructor externo:

```julia
Rational(n::Integer, d::Integer) = Rational(promote(n,d)...)
```

Esto permite que llamadas como la siguiente funcionen:

```jldoctest
julia> Rational(Int8(15),Int32(-5))
-3//1

julia> typeof(ans)
Rational{Int32}
```

Para la mayoría de los tipos definidos por usuario, es mejor práctica requerir que los programadores proporcionen los tipos esperados para las funciones constructor explícitamente, pero algunas veces, especialmente para problemas numéricos, puede ser conveniente realizar la conversión de forma automática.

### Definiendo reglas de promoción


Aunque uno podría, en principio, definir métodos para la función `promote` directamente, esto requeriría muchas defniciones redundantes para todas las posibles permutaciones de tipos de argumentos. En lugar de ello, el comportamiento de `promote` es definido en términos de una función auxiliar denominada `promote_rule`, para la que uno puede proporcionar métodos. La funci´no `promote_rule` toma un par de objetos tipo y devuelve otro objeto tipo, tal que instancias de los tipos de argumentos sean promocionadas al tipo retornado. De este modo, definiendo la regla:

```julia
promote_rule(::Type{Float64}, ::Type{Float32}) = Float64
```

uno declara que cuando se promocionan juntos valores en punto flotante de 32 y de 64 bits, ellos deberían ser promocionados a punto flotante de 64 bits. El tipo de promoción no tiene que ser uno de los tipos de los argumentos. He aquí un par de ejemplos de reglas de promoción que aparecen en la librería estándar de Julia: 

```julia
promote_rule(::Type{UInt8}, ::Type{Int8}) = Int
promote_rule(::Type{BigInt}, ::Type{Int8}) = BigInt
```

En el último caso, el tipo de resultado es  [`BigInt`](@ref) since `BigInt` ya que éste es el único tipo lo bastante grande como para alojar enteros para aritmética entera de precisión arbitraria. Nótese también que uno no necesita definir dos reglas simétricas `promote_rule(::Type{A}, ::Type{B})` y `promote_rule(::Type{B}, ::Type{A})` (esta simetría es supuesta por la forma en que `promote_rule` es utilizada en el proceso de promoción.

La función `promote_rule` se usa com un bloque constructivo para definir una segunda función llamada `promote_type` la cuál, dado cualquier número de objetos tupo, devuelve el tipo común al cuál esos valores, como argumentos a `promote` deberían ser promocionados. Por tanto, si uno quiere saber, en ausencia de valores actuales, a qué tipo promocionaría una colección de valores de cierto tipo, uno podría usar `promote_type`:

```jldoctest
julia> promote_type(Int8, UInt16)
Int64
```

Internamente, `promote_type` se usa dentro de `promote` para determinar a qué valores argumento tipo deberían ser convertidos tras una promoción. Ell puede, sin embargo, ser útil en sí misma. el lector curioso puede leer el código en [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl), , que define el mecanismo de promoción completo en aproximadamente 35 líneas.

### Caso de estudio: promociones Rational

Finalmente, finalizaremos nuestro caso de estudio el tipo de los números racionales en Julia, que hace un uso relativamente sofistidcado del mecanismo de promoción con las siguientes reglas de promoción:

```julia
promote_rule(::Type{Rational{T}}, ::Type{S}) where {T<:Integer,S<:Integer} = Rational{promote_type(T,S)}
promote_rule(::Type{Rational{T}}, ::Type{Rational{S}}) where {T<:Integer,S<:Integer} = Rational{promote_type(T,S)}
promote_rule(::Type{Rational{T}}, ::Type{S}) where {T<:Integer,S<:AbstractFloat} = promote_type(T,S)
```

La primera regla dice que promocionar un número racional con algún otro entero promociona a un tipo racional cuyo tipo numerados/denominador es el resultado de la promoción de sus tipos numerador/denoinador con el otro tipo entero. La segunda regla aplica la misma lógic a dos tipos diferentes de números racionales, dando como resultado un racional de la promoción de sus respectivos tipos numerador/denominador. Las reglas tercera y última dictan que promocionar un racional con un punto flotante da como resultado el mismo tipo que promocionar el tipo de numerador/denominador con el float.

Este pequeño puñado de reglas de promoción, junto con los [métodos de conversión discutidos antes](@ref man-rational-conversion), son suficiente para hacer que los números racionales interoperen completamente y de forma natural con todos los demás tipos numéricos de Julia (enteros, números en punto flotante y números complehjos). Proporcionando métodos de converión apropiadosy reglas de promoción en la misma manera, cualquier tipo numérico definido por el usuario puede interoperar así de naturalmente con los numéricos predefinidos en Julia.
