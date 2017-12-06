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

## Evitar elaborar tipos contenedor

Usualmente no es de mucha ayuda construir arrays como los siguientes:

```julia
a = Array{Union{Int,AbstractString,Tuple,Array}}(n)
```

En este caso `Array{Any}(n)` es mejor. Es también de más ayuda para el compilador anotar usos específicos (por ejemplo  `a[i]::Int`) que intentar empaquetar muchas alternativas en un tipo.

## Usar convenciones de nombrado consistentes con el paquete `base/` de Julia

  * Los nombres de los módulos y tipos usan mayúsculas y *came case*: `module SparseArrays`, `struct UnitRange`.
  * Las funciones van en minúscula ([`maximum()`](@ref), [`convert()`](@ref)) y, cuando es legible, con múltiples
    palabras pegadas juntas ([`isequal()`](@ref), [`haskey()`](@ref)). Cuando sea necesario, use guiones bajos
    como separadores de palabra. Los guiones bajos también se usan para indicar una combinacin de conceptos
    ([`remotecall_fetch()`](@ref) como una implementación más eficiente de `fetch(remotecall(...))`) o como 
    modificadores ([`sum_kbn()`](@ref)).
  * Se valora la concisión, pero debe evitarse la abreviatura ([`indexin()`](@ref) en lugar de `indxin()`) ya
    que se vuelve difícil recordar si se abrevian palabras particulares y cómo se han abreviado.

Si el nombre de una función requiere varias palabras, considere si podría representar más de un concepto y podría dividirse mejor en partes.

## No usar demasiado try-catch

Es mejor evitar errores que basarse en atraparlos.

## No meter entre paréntesis las condiciones

Julia no necesita que se rodeen entre paréntesis las condiciones en `if` and `while`. Escriba:

```julia
if a == b
```

en lugar de:

```julia
if (a == b)
```

## No usar demasiado `...`

El uso de `...` en los argumentos de función puede ser adictivo. En lugar de `[a..., b...]`, use `[a; b]`,
que ya concatena arrays. [`collect(a)`](@ref) es mejor que `[a...]`, pero como `a` ya es iterable suele ser 
incluso mejor d3jarlo solo, y no convertirlo en array.

## No usar parámetros estáticos innecesarios

Una signatura de función:

```julia
foo(x::T) where {T<:Real} = ...
```

debería ser escrita como:

```julia
foo(x::Real) = ...
```

especialmente si `T` no se usa en el cuerpo de la función. Incluso si se usa `T`, se puede reemplazar con [`typeof(x)`](@ref) si es conveniente. No hay diferencia de rendimiento. Tenga en cuenta que esto no es una precaución general contra los parámetros estáticos, solo contra uso donde no son necesarios.

Tenga en cuenta también que los tipos de contenedores, específicamente pueden necesitar parámetros de tipo en las llamadas a función. Consulte las Preguntas frecuentes [Evitar campos con contenedores abstractos](@ref) para obtener más información.

## Evitar la confusion sobre si algo es una instancia o un tipo

Conjuntos de definiciones como las siguientes son confusas:

```julia
foo(::Type{MyType}) = ...
foo(::MyType) = foo(MyType)
```

Decida si el concepto en cuestión se escribirá como `MyType` o `MyType()`, y sígalo.

El estilo preferido es usar instancias por defecto, y solo agregue métodos que incluyan `Tipo{MiTipo}` más tarde si se vuelven necesarios para resolver algún problema.

Si un tipo es efectivamente una enumeración, debe definirse como un tipo único (idealmente, `immutable struct` o primitivo), con los valores de enumeración como instancias de este. Los constructores y las conversiones pueden verificar si los valores son válidos. Este diseño es preferible a hacer que la enumeración sea un tipo abstracto, con los "valores" como subtipos.

## No abusar de las macros

Tenga en cuenta cuando una macro realmente podría ser una función en su lugar.

Llamar a [`eval()`](@ref) dentro de una macro es un signo de advertencia particularmente peligroso; significa que la macro solo funcionará cuando se llame al nivel superior. Si tal macro se escribe como una función en su lugar, naturalmente tendrá acceso a los valores en tiempo de ejecución que necesita.

## No exponer operaciones inseguras al nivel de interfaz

Si se tiene un tipo que use un puntero nativo:

```julia
mutable struct NativeType
    p::Ptr{UInt8}
    ...
end
```

no escriba definiciones como la siguiente:

```julia
getindex(x::NativeType, i) = unsafe_load(x.p, i)
```

El problema es que los usuarios de este tipo pueden escribir `x[i]` sin darse cuenta de que la operación no es segura y, luego, ser susceptibles a errores de memoria.

Dicha función debería verificar la operación para asegurarse de que sea segura, o incluir `unsafe` en alguna parte de su nombre para alertar a las personas que la invocan.

## No sobrecargar métodos de tipos de contenedores base

Es posible escribir definiciones como la siguiente:

```julia
show(io::IO, v::Vector{MyType}) = ...
```

Esto proporcionaría una muestra personalizada de vectores con un nuevo tipo de elemento específico. Aunque es tentador, es algo que debe evitarse. El problema es que los usuarios esperarán que un tipo conocido como `Vector()` se comporte de cierta manera, y la personalización excesiva de su comportamiento puede dificultar el trabajo.

## Evitar la piratería de tipos

La "Piratería de Tipos" se refiere a la práctica de extender o redefinir métodos en Base u otros paquetes en tipos que no han definido. En algunos casos, puede la piratería de tipo va a tener un efecto poco negativo. Sin embargo, en casos extremos, incluso puede bloquear Julia (por ejemplo, si la extensión o redefinición de su método hace que se pase una entrada inválida a `ccall`). La piratería de tipos puede complicar el razonamiento sobre el código y puede introducir incompatibilidades que son difíciles de predecir y diagnosticar.

Como ejemplo, suponga que quiere definir la multiplicación en símbolos en un módulo: 

```julia
module A
import Base.*
*(x::Symbol, y::Symbol) = Symbol(x,y)
end
```

El problema es que ahora cualquier otro módulo que use `Base.*` También verá esta definición. Dado que `Symbol` se define en Base y es utilizado por otros módulos, esto puede cambiar el comportamiento del código no relacionado de forma inesperada. Aquí hay varias alternativas, incluido el uso de un nombre de función diferente o el ajuste de `Symbol`s en otro tipo que defina.

Algunas veces, los paquetes acoplados pueden involucrarse en la piratería de tipos para separar las características de las definiciones, especialmente cuando los paquetes fueron diseñados por autores colaboradores, y cuando las definiciones son reutilizables. Por ejemplo, un paquete puede proporcionar algunos tipos útiles para trabajar con colores; otro paquete podría definir métodos para aquellos tipos que permiten conversiones entre espacios de color. Otro ejemplo podría ser un paquete que actúa como un envoltorio delgado para algún código C, que otro paquete podría piratear para implementar una API de nivel superior compatible con Julia.

## Ser cuidadoso con la igualdad de tipos

Por lo general, uno desea utilizar [`isa()`](@ref) y [`<:`](@ref) para los tipos de prueba, no `==`. La comprobación de los tipos para la igualdad exacta normalmente solo tiene sentido cuando se compara con un tipo concreto conocido (por ejemplo, `T == Float64`), o si *realmente* uno sabe lo que está haciendo.

## No escribir `x->f(x)`

Como las funciones de orden superior a menudo se llaman con funciones anónimas, es fácil concluir que esto es deseable o incluso necesario. Pero cualquier función se puede pasar directamente, sin estar "envuelta" en una función anónima. En lugar de escribir `map (x-> f(x), a)`, escriba [`map(f, a)`](@ref).

## Evitar usar floats para literales numericos en codigo generico cuando sea posible

Si escribe código genérico que maneja números, y que se puede esperar que se ejecute con muchos tipos de argumentos numéricos diferentes, intente utilizar literales de un tipo numérico que afectarán los argumentos lo menos posible mediante la promoción.

Por ejemplo,

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

mientras que

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

Como puede ver, la segunda versión, donde usamos un literal `Int`, conserva el tipo de argumento de entrada, mientras que la primera no. Esto se debe a, por ejemplo, `promote_type(Int, Float64) == Float64`, y la promoción ocurre con la multiplicación. De manera similar, los literales [`Rational`](@ref) son menos disruptivos que los literales [`Float64`](@ref), pero son más perjudiciales que los `Int`s:

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

Por tanto, use literales `Int` cuando sea posible, con `Rational{Int}` para literales numéricos no enteros, en orden a hacer nuestro código ms fácil de usar.
