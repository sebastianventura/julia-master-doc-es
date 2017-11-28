# [Constructores](@id man-constructors)

Los constructores [^1] son funciones que crean nuevos objetos (específicamente instancias de tipos compuestos). En Julia, los objetos también sirven como funciones constructor: ellos crean instancias de sí mismos cuando se aplican a una dupla de argumentos como una función. Esto se mencionó brevemente cuando se habló de tipos compuestos. Por ejemplo:

```jldoctest footype
julia> struct Foo
           bar
           baz
       end

julia> foo = Foo(1, 2)
Foo(1, 2)

julia> foo.bar
1

julia> foo.baz
2
```

Para muchos tipos, formar nuevos objetos enlazando valores de campo juntos es todo se necesita para crear instancias. Hay, sin embargo, casos donde se requiere más funcionalidad cuando se crean objetos compuestos. Algunas invariantes deben ser forzadas, bien chequeando argumentos o transformándolos. Las [estructuras de datos recursivas](https://en.wikipedia.org/wiki/Recursion_%28computer_science%29#Recursive_data_structures_.28structural_recursion.29),
especialmente aquellas que pueden ser auto referenciadas frecuentemente, no pueden construirse limpiamente sin que primero sean creadas en un estado incompleto y después sean alteradas programáticamente para ser completadas, como un paso separado de la creación del objeto. Algunas veces, es conveniente ser capaz de construir objetos con menos o diferentes tipos de parámetros que el número de campos que tiene. El sistema Julia para construcción de objetos cubre estos casos y más.

[^1]:
    Nomenclature: while the term "constructor" generally refers to the entire function which constructs
    objects of a type, it is common to abuse terminology slightly and refer to specific constructor
    methods as "constructors". In such situations, it is generally clear from context that the term
    is used to mean "constructor method" rather than "constructor function", especially as it is often
    used in the sense of singling out a particular method of the constructor from all of the others.

## Métodos constructores externos

Un constructor es como cualquier otro función en Julia en que es su comportamiento global está definido por el comportamiento combinado de sus métodos. Según esto, se puede añadir funcionalidad a ún constructor simplemente definiendo nuevos métodos. Por ejemplo, supóngase que se desea añadir un método constructor para objetos `Foo` que tomar un argumento y usa el valor dado para los dos campos que presentan `baz` y `bar`. Esto es sencillo::

```jldoctest footype
julia> Foo(x) = Foo(x,x)
Foo

julia> Foo(1)
Foo(1, 1)
```

Podría también añadirse un constructor `Foo` sin argumentos que proporciona valores por defecto para los campos `bar` y `baz`:

```jldoctest footype
julia> Foo() = Foo(0)
Foo

julia> Foo()
Foo(0, 0)
```

Aquí, el método constructor sin argumentos llama al método constructor con un argumento, que a su vez llama al método constructor de dos argumentos proporcionado automáticamente. Por razones que se aclararán pronto, los metodos constructor adicionales declarados como métodos formales como éstos se denominan *métodos constructores externos*. Los métodos constructores externos sólo puede crear una nueva instancia llamando a otro método constructor, tal como los  proporcionados automáticamente por defecto.

## Métodos Constructores Internos

Aunque los constructores externos resuelven con éxito el problema de proporcionar métodos adicionales para construir objetos, ellos fallan en los otros dos casos de uso mencionados en la introducción este capítulo: forzar invariantes y permitir la construcción de objetos autorreferenciales. Para estos problemas, se necesitan los *métodos constructores internos*. Un método constructor interno es parecido a uno externo, con dos diferencias:

1. Se declara dentro del bloque de la declaración del tipo, el lugar donde fuera como los métodos normales.
2. Tiene acceso a una función especial, existente totalmente, llamada `new` que crea objetos del tipo del bloque.

Poner ejemplo, supóngase que uno quiere declarar un tipo que almacene un par de elementos reales, sujetos a la restricción de que el primer número no es mayor que el segundo. Uno podría declararlo así:

```jldoctest pairtype
julia> struct OrderedPair
           x::Real
           y::Real
           OrderedPair(x,y) = x > y ? error("out of order") : new(x,y)
       end

```

Ahora sólo pueden construirse objetos `OrderedPair` tales que `x <= y`:

```jldoctest pairtype
julia> OrderedPair(1, 2)
OrderedPair(1, 2)

julia> OrderedPair(2,1)
ERROR: out of order
Stacktrace:
 [1] OrderedPair(::Int64, ::Int64) at ./none:4
```

Si el tipo se declara `mutable`, se puede acceder y cambiar directamente los valores de campo para violar esta invariante, pero se considera deficiente la interacción con las partes internas de un objeto sin invitación. Usted (u otra persona) también puede proporcionar más métodos constructores externos en cualquier momento posterior, pero una vez que se declara un tipo, no hay forma de agregar más métodos internos de construcción. Como los métodos constructores externos solo pueden crear objetos llamando a otros métodos de construcción, en última instancia, se debe llamar a algún constructor interno para crear un objeto. Esto garantiza que todos los objetos del tipo declarado deben existir mediante una llamada a uno de los métodos de constructor internos proporcionados con el tipo, dando así cierto grado de cumplimiento de las invariantes de un tipo.

Si se define cualquier método de constructor interno, no se proporciona ningún método constructor predeterminado: se supone que se ha provisto de todos los constructores internos que necesita. El constructor predeterminado es equivalente a escribir su propio método constructor interno que toma todos los campos del objeto como parámetros (restringidos para ser del tipo correcto, si el campo correspondiente tiene un tipo), y los pasa a `new`, devolviendo el objeto resultante: 

```jldoctest
julia> struct Foo
           bar
           baz
           Foo(bar,baz) = new(bar,baz)
       end

```

Esta declaración tiene el mismo efecto que la definición anterior del tipo `Foo` sin un método constructor interno específico. Los siguientes dos tipos son equivalentes (uno con un constructor por defecto y el otro con un constructor explícito):

```jldoctest
julia> struct T1
           x::Int64
       end

julia> struct T2
           x::Int64
           T2(x) = new(x)
       end

julia> T1(1)
T1(1)

julia> T2(1)
T2(1)

julia> T1(1.0)
T1(1)

julia> T2(1.0)
T2(1)
```
Se considera una buena práctica proporcionar tan pocos constructores internos como sea posible: sólo aquellos que tomen todos los argumentos explícitamente y fuercen la comprobación de errores y las transformaciones esenciales. Los demás constructores proporcionados, que proporcionan valores por defecto o transformaciones auxiliares, deberían proporcionarse como constructores externos que llaman a los internos para hacer el trabajo pesado. Esta situación suele ser bastante natural.

## Inicialización incompleta

El problema final que aún no se ha resuelto es la construcción de objetos autorreferenciales o, más generalmente, estructuras de datos recursivas. Como la dificultad fundamental puede no ser obvia de inmediato, se explicará brevemente. Considere la siguiente declararción de tipo recursivo:

```jldoctest selfrefer
julia> mutable struct SelfReferential
           obj::SelfReferential
       end

```

Este tipo puede parecer bastante inócuo a menos que uno considere cómo construir una instancia de él. Si `a` es una instancia de `SelfReferential`, entonces una segunda instancia `b` podría crearse mediante la llamada:

```julia-repl
julia> b = SelfReferential(a)
```

¿Pero cómo se construye la primera instancia cuando no existe ninguna otra instancia para propocionar un valor válido para el campo `obj`? La única solución es permitir la creación de una instancia de `SelfReferential` que no esté inicializada por completo, con el campo `obj` no asignado, y usar esta instancia incompleta como un valor válido que se podcría asignar al campo `obj` de otra instancia, o incluso al de ella misma.

Para permitir la creación de objetos inicializados de forma incompleta, Julia permite que la  función `new` sea llamada con menos argumentos del número de campos que el objeto tiene, devolviendo un objeto con los campos no especificados sin inicializar. El método constructor interno pues entonces usar el método incompleto, finalizando su inicialización antes de devolverlo. Aquí por ejemplo, se intenta definir el tipo `SelfReferental` con un constructor interno con cero argumentos que devuelve instancias con sus campos `obj` apuntando a ellos mismos:

```jldoctest selfrefer2
julia> mutable struct SelfReferential
           obj::SelfReferential
           SelfReferential() = (x = new(); x.obj = x)
       end

```

Podemos verificar que este constructor funciona y construye objetos que son, de hecho, autorreferenciados:

```jldoctest selfrefer2
julia> x = SelfReferential();

julia> x === x
true

julia> x === x.obj
true

julia> x === x.obj.obj
true
```

Aunque se permite crear objetos con campos no inicializados, cualquier objeto a una referencia no inicializada es un eror inmediato:

```jldoctest incomplete
julia> mutable struct Incomplete
           xx
           Incomplete() = new()
       end

julia> z = Incomplete();
```

Aunque se permite crear objetos con campos no inicializados, cualquier objeto a una referencia no inicializada es un eror inmediato:

```jldoctest incomplete
julia> z.xx
ERROR: UndefRefError: access to undefined reference
```

Esto evita la necesidad de estar comprobando datos `null` continuamente. Sin embargo, no todos los campos de objetos son referencias. Julia considera algunos tipos Como "datos planos", Lo que significa que todos sus datos son auto contenidos y que no referencian otros objetos. Los tipos de datos planos son los tipos primitivos (es decir `Int`) y las estructuras inmutables de otros tipos de datos planos. Los contenidos iniciales de un tipo de datos planos son indefinidos:

```julia-repl
julia> struct HasPlain
           n::Int
           HasPlain() = new()
       end

julia> HasPlain()
HasPlain(438103441441)
```

Los arrays de tipos de datos planos exhiben el mismo comportamiento.

Uno puede pasar objetos incompletos a otras funciones desde los constructores internos para  delegar su terminación:

```jldoctest
julia> mutable struct Lazy
           xx
           Lazy(v) = complete_me(new(), v)
       end
```

Como sucede con los objetos incompletos devueltos desde los constructores, si `complete_me` o alguno de los métodos que lo llaman intenta acceder al campo `xx` del objeto `Lazy` antes de que éste sea inicializado, se lanzará un error de inmediato.

## Constructores paramétricos

Los tipos paramétricos añaden algunas complicaciones al tema de los constructores. Recuérdese la sección [Tipos Paramétricos](@ref) que por defecto pueden construirse instancias de estos tipos dando  explícitamente los parámetros de tipo o con parámetros de tipo implicados por los tipos de los argumentos dados al constructor. He aquí algunos ejemplos:

```jldoctest parametric
julia> struct Point{T<:Real}
           x::T
           y::T
       end

julia> Point(1,2) ## implicit T ##
Point{Int64}(1, 2)

julia> Point(1.0,2.5) ## implicit T ##
Point{Float64}(1.0, 2.5)

julia> Point(1,2.5) ## implicit T ##
ERROR: MethodError: no method matching Point(::Int64, ::Float64)
Closest candidates are:
  Point(::T<:Real, !Matched::T<:Real) where T<:Real at none:2

julia> Point{Int64}(1, 2) ## explicit T ##
Point{Int64}(1, 2)

julia> Point{Int64}(1.0,2.5) ## explicit T ##
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Int64}, ::Float64) at ./float.jl:680
 [2] Point{Int64}(::Float64, ::Float64) at ./none:2

julia> Point{Float64}(1.0, 2.5) ## explicit T ##
Point{Float64}(1.0, 2.5)

julia> Point{Float64}(1,2) ## explicit T ##
Point{Float64}(1.0, 2.0)
```

Como podemos ver, para las llamadas a constructor con parámetros de tipo explícito, los argumentos se convierten a los tipos implícitos de los campos: `Point{Int64}(1,2)` funciona, pero `Point{Int64}(1.0,2.5)` lanza un [`InexactError`](@ref) cuando `2.5` se convierte a [`Int64`](@ref). Cuando el tipo es implicado por los argumentos de la llamada al constructor, como en `Point(1,2)`, entonces los tipos de los argumentos deben concordar para que se pueda determinar `T`, pero da igual cuáles sean los tipos mientras ambos sean iguales y, además, subclases de `Real`.

Lo que está pasando aquí realmente es que `Point`, `Point{Float64}` y `Point{Int64}` son funcioens constructores diferentes. De hecho, `Point{T}` es una función constructor distinto para cada tipo `T`. Sin ningún constructor interno propocionado explícitamente, la declaración del tipo compuesto `Point{T<:Real}` proporciona automáticamente un constructor interno `Point{T}` para cada posible ipo `T<:Real` que se comporta justo como lo hacen los constructores internos no paramétricos por defecto. Ella también proporciona un solo constructor general externo que toma pares de argumentos reales, que deben ser del mismo tipo. Esta provisión automática de constructores es equivalente a la siguiente declaración explícita:

```jldoctest parametric2
julia> struct Point{T<:Real}
           x::T
           y::T
           Point{T}(x,y) where {T<:Real} = new(x,y)
       end

julia> Point(x::T, y::T) where {T<:Real} = Point{T}(x,y);
```

Observe que cada definición se parece a la forma de llamada de constructor que maneja. La llamada `Point{Int64}(1,2)` invocará la definición `Point{T}(x, y)` dentro del
bloque `type`.

La declaración de constructor externo, por otro lado, define un método para el constructor general de `Point` que sólo se aplica a pares de valores del mismo tipo real. Esta declaración hace que las llamadas al constructor sin parámetros de tipo explícitos, como `Punto(1,2)` y `Punto(1.0,2.5)`, funcionen. Dado que la declaración del método restringe los argumentos para que sean del mismo tipo, las llamadas como `Point(1,2.5)`, con argumentos de diferentes tipos, dan como resultado errores "no method".

Supongamos que queremos hacer la llamada a constructor `Point(1, 2.5)` funcione promocionando el valor entero `1` a punto flotante `1.0`. La forma más sencilla de conseguir eso es definir el siguiente método constructor adicional:

```jldoctest parametric2
julia> Point(x::Int64, y::Float64) = Point(convert(Float64,x),y);
```

Este método usa la función [`convert()`](@ref) para convertir explícitamente `x` a [`Float64`](@ref) y entonces delegar la construcción al constructor general para el caso de que ambos argumentos sean [`Float64`](@ref). Con esta definición de método, lo  que previamente producía un  [`MethodError`](@ref) ahora crea con éxito un punto de tipo `Point{Float64}`:

```jldoctest parametric2
julia> Point(1,2.5)
Point{Float64}(1.0, 2.5)

julia> typeof(ans)
Point{Float64}
```

Sin embargo, otras llamadas similares siguen sin funcionar:

```jldoctest parametric2
julia> Point(1.5,2)
ERROR: MethodError: no method matching Point(::Float64, ::Int64)
Closest candidates are:
  Point(::T<:Real, !Matched::T<:Real) where T<:Real at none:1
```

Para una forma mucho más general de hacer que todas estas llamadas funcionen sensiblemente, ver [Conversión y promoción](@ref conversion-and-promotion). A riesgo de estropear el suspense, podemos revelar aquí que todo lo toma el siguiente método externo para hacer que todas las llamadas al constructor general `Point` trabajen como uno debería esperar:

```jldoctest parametric2
julia> Point(x::Real, y::Real) = Point(promote(x,y)...);
```

La función `promote` convierte todos sus argumentos a un tipo común (en este caso, `Float64`). Con esta definición de método el constructor `Point` promociona sus argumentos de la misma forma que lo hacen los operadores aritméticos como [`+`](@ref) y funciona para todos los tipos de números reales:

```jldoctest parametric2
julia> Point(1.5,2)
Point{Float64}(1.5, 2.0)

julia> Point(1,1//2)
Point{Rational{Int64}}(1//1, 1//2)

julia> Point(1.0,1//2)
Point{Float64}(1.0, 0.5)
```

Por tanto, mientras los constructores con parámetros de tipo implícitos proporcionados por defecto en Julia son muy estrictos, es posible hacer que se comporten de una forma más relajada pero sensible con bastante facilidad. Además, como los constructores pueden sacar ventaja de toda la potencia del sistema de tipos, métodos y despacho múltiple, definir comportamientos sofisticados suele ser bastante simple.

## Case Study: Rational

Quizás la mejor forma de unir todas las piezas es presentar un ejemplo del mundo real de un tipo compuesto paramétrico y sus métodos constructores. Para este fin, he aquí una parte de  [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl), que implementa los [Números Racionales](@ref) en Julia:

```jldoctest rational
julia> struct OurRational{T<:Integer} <: Real
           num::T
           den::T
           function OurRational{T}(num::T, den::T) where T<:Integer
               if num == 0 && den == 0
                    error("invalid rational: 0//0")
               end
               g = gcd(den, num)
               num = div(num, g)
               den = div(den, g)
               new(num, den)
           end
       end

julia> OurRational(n::T, d::T) where {T<:Integer} = OurRational{T}(n,d)
OurRational

julia> OurRational(n::Integer, d::Integer) = OurRational(promote(n,d)...)
OurRational

julia> OurRational(n::Integer) = OurRational(n,one(n))
OurRational

julia> //(n::Integer, d::Integer) = OurRational(n,d)
// (generic function with 1 method)

julia> //(x::OurRational, y::Integer) = x.num // (x.den*y)
// (generic function with 2 methods)

julia> //(x::Integer, y::OurRational) = (x*y.den) // y.num
// (generic function with 3 methods)

julia> //(x::Complex, y::Real) = complex(real(x)//y, imag(x)//y)
// (generic function with 4 methods)

julia> //(x::Real, y::Complex) = x*y'//real(y*y')
// (generic function with 5 methods)

julia> function //(x::Complex, y::Complex)
           xy = x*y'
           yy = real(y*y')
           complex(real(xy)//yy, imag(xy)//yy)
       end
// (generic function with 6 methods)
```

La primera línea -- `struct OurRational{T<:Integer} <: Real` -- declara que `OurRational` toma un parámetro de un subtipo de `Integer`, aunque él en si mismo es un tipo `Real`. Las declaraciones de campo `num::T` y `den::T` indican que los datos almacenados en un objeto `OurRational{T}` será un par de enteros de tipo `T`, uno que representará el numerador y otro el denominador. 

Ahora las cosas se ponen interesantes. `OurRational` tiene un solo constructor interno que comprueba que tanto `num` como `den` no son cero, y aegura que cada número racional se construye en sus términos mínimos con un denominador no negativo. Esto se consigue dividiendo los valores de numerador y denominador por su máximo común divisor, el cuál se calcula a través de la función `gcd`. Por último, y como `gcd` asigna el signo del primer argumento (en este caso `den`) se garantiza que el denominador ya no sea negativo. Como este es el único constructor interno de `Rational`, podemos estar seguros de que los objetos de este tipo siempre se construyen en forma normalizada.

`Rational` también proporciona varios métodos constructores externos por conveniencia. El primero es el constructor general "esándar", que infiere el tupo del parámetro `T` a partir del tipo del numerado y denominador que tienen que ser del mismo tipo. El segundo se aplica cuando numerador y denominador tiene tipos distintos: los promociona a un tipo común y entonces delega la construcción al otro constructor externo con argumentos del mismo tipo. En tercer constructor externo convierte valores enteros en racionales proporcionando un denominador de valor 1.

Siguiendo las definiciones de constructores externos, tenemos una serie de métodos para el operador [`//`](@ref) que proporcionan una sintaxis para escribir racionales. Antes de estas definiciones,  [`//`](@ref)es un operador completamente indefinido con sólo sintaxis y sin significado. Después, se comporta tal y com se describe en [Números Racionales](@ref) -- 
(su comportamiento completo está descrito en estas pocas líneas). La primera y más básica definición hace `a//b` construya un  `OurRational` aplicando el constructor de este tipo sobre `a` y `b` cuando ambos son enteros. Cuando uno de los operandos de  [`//`](@ref) ya es un número racional se construye un nuevo número racional para la razón resultante con una leve diferencia: este comportamiento es igual a la división de un racional entre un entero. Por último, aplicar [`//`](@ref) a valores complejos enteros crea una instancia de `Complex{Rational}`, que es un complejo cuyas partes real el imaginaria son racionales:

```jldoctest rational
julia> ans = (1 + 2im)//(1 - 2im);

julia> typeof(ans)
Complex{OurRational{Int64}}

julia> ans <: Complex{OurRational}
false
```

Por tanto, aunque el operador [`//`](@ref) suela devolver una instancia de `OurRational`, si uno de sus rgumentos es un complejo entero, devolverá una instancia de `Complex{OurRational}`. El lector interesado debería considerar la lectura del resto de [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl): es corto, autocontenido e implmeenta un tipo básico de Julia al completo.

## [Constructores and Conversión](@id constructors-and-conversion)

Los constructores `T(args)` se implementan como otros objetos invocables: los métodos se añaden a sus tipos. El tipo de un tipo es `Type` por lo que los métodos constructores se almacenan en la tabla de métodos para el tipo `Type`. Esto significa que se pueden declarar constructores más flexibles, es decir, constructores para tipos abstractos, mediante la definición explícita de métodos para los tipos apropiados.

Sin embargo, en algunos casos, uno debería considerar añadir métodos a `Base.convert` en ougar de definir un constructor, dado que Julia retrocede para llamar a [`convert()`](@ref) si no se encuentra un constructor que coincida. Por ejemplo, si no existe para constructor para `T(args...) =` se llamará a `Base.convert(::Type{T}, args...)=...`

`convert` se usa extensivamente a través de Julia cuando un tipo tenga que ser convertido en otro (por ejemplo, en asignación, [`ccall`](@ref), etcetera), y sólo debería ser definido (o exitoso) si la conversión se realiza sin pérdidas. Por ejemplo, `convert(Int, 3.0)` produce `3`, pero `convert(Int,3.2)` lanza un `InexactError`. Si desea construirse un constructor para una conversión sin pérdidas de un tipo a otro, probablemente sería mejor definir un método `convert`.

Por otra parte, si el constructro no representa una conversión sin pérdida, o no represnta ningua conversion es mejor dejarlo como constructor en lugar de como un método `convert`. Por ejemplo, el constructor `Array{Int}` crea un array cero-dimensional del tipo `Int` pero no es realmente una conversión de `Int` a `Array`.

## Constructores sólo exteriores

Como se ha visto, un tipo paramétrico típico tiene constructroes internos que son invocados cuano se conocen los tupos de los parámetros, por ejemplo, se aplican a `Point{Int}`paro no a `Point`. Opcionalmente, los constructores externos que determinan los parámetros de tipo  pueden ser añadidos automáticamente, por ejemplo, construir un `Point{Int}` a partir de la llamada `Point(1,2)`. Los constructores externos llaman a los constructores internos para que hagan el trabajo básico de hacer una instancia.  Sin embargo, en algunos caos, uno podría en lugar de eso no proporcionar contructores internos para que los parámetros específicos no puedan ser solicitados manualmente. 

Por ejemplo, suponga que se define un tipo que almacena un vector con una representaciócn exacta de su suma:

```jldoctest
julia> struct SummedArray{T<:Number,S<:Number}
           data::Vector{T}
           sum::S
       end

julia> SummedArray(Int32[1; 2; 3], Int32(6))
SummedArray{Int32,Int32}(Int32[1, 2, 3], 6)
```

El problema es que nosotros queremos que `S` sea un tipo más grande que `T`, por lo que podemos sumar muchos elementos con menos pérdida de información. Por ejemplo, cuando `T` es [`Int32`](@ref), querríamos que `S` fiuera [`Int64`](@ref). Por tano queremos evitar un interfaz que permita al usuario construir instanca del tipo `SummedArray{Int32,Int32}`. Una forma de hacer esto es proporcionar sólo un constructor más exterior para `SummedArray`.  Esto puede hacerse usando a definicíon de métdo por tipo pero dentro del bloque de definición `type` para suprimir la generación de bucles por defecto:

```jldoctest
julia> struct SummedArray{T<:Number,S<:Number}
           data::Vector{T}
           sum::S
           function SummedArray(a::Vector{T}) where T
               S = widen(T)
               new{T,S}(a, sum(S, a))
           end
       end

julia> SummedArray(Int32[1; 2; 3], Int32(6))
ERROR: MethodError: no method matching SummedArray(::Array{Int32,1}, ::Int32)
Closest candidates are:
  SummedArray(::Array{T,1}) where T at none:5
```

El construcror será invocado por la sintaxis `SummedArray(a)`. La sintaxis `new{T,S}` permite especificar parámetrospara el tipo que se va a construir, es decir, esta llamada devolvera un `SummedArray{T, s]`.
`new{T,S}` puede usarse en cualquier definicin de constructor, pero por conveniencia los parámtros a `new{}` se derivan automáticamente del tipo que se está construyendo cuando sea posible.
