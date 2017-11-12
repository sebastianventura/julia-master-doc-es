# [Tipos](@id man-types)

Los sistemas de tipos han caído tradicionalmente en dos categorías muy diferentes: los *sistemas de tipos estáticos*, donde cada expresión del programa debe tener un tipo computable antes de la ejecución del programa, y los *sistemas de tipos dinámicos*, donde nada es sabido sobre los tipos hasta el momento de la ejecución, cuando los valores actuales manipulados por el programa están disponibles. La orientación a objetos permite una flexibilidad en los lenguajes tipados estáticamente dejando que el código sea escrito sin que se conozcan los tipos precisos de los valores en tiempo de compilación. La capacidad de escribir código que pueda operar sobre diferentes tipos se denomina polimorfismo. Todo el código en los lenguajes clásicos tipados dinámicamente es polimórfico: sólo mediante comprobación de equipos explícita o cuando los objetos fallan para soportar las operaciones en tiempo de ejecución, están los tipos de cualquier valor siempre restringidos.

El sistema de tipos de Julia es dinámico, pero tiene algunas de las ventajas de los sistemas de tipos estáticos haciendo posible indicar que ciertos valores son de tipos específicos. Esto puede ser de gran ayuda en la generación de código eficiente, pero incluso más significativamente, permite que el despacho de métodos sobre los tipos de los argumentos a función este profundamente integrado con el lenguaje. El despacho de métodos se explorará en detalle en la sección [Methods](@ref), pero está enraizado en el sistema de equipos presentado en este capítulo..

El comportamiento por defecto en Julia cuando se omiten los tipos es permitir que los valores sean de cualquier tipo. Por tanto, uno puede escribir muchos programa Julia útiles sin siquiera usar explícitamente los tipos. Cuando se necesita una expresividad adicional, sin embargo, es fácil introducir gradualmente anotaciones de tipo explícitas en código previamente no tipado. Hacer eso suele incrementar el rendimiento y la robustez de estos sistemas, y quizás algo contra intuitivo: simplificarlos frecuentemente.

Describir Julia en el lingo de los [sistemas de tipos](https://en.wikipedia.org/wiki/Type_system), es decir que es: *dinámico*, *nominativo* y *paramétrico*. Los tipos genéricos pueden ser parametrizados, y las relaciones jerárquicas entre tipos son [declaradas explítamente](https://en.wikipedia.org/wiki/Nominal_type_system),
en lugar de ser [implicadas mediante una estructura compatible](https://en.wikipedia.org/wiki/Structural_type_system).
Una característica particularmente distintiva del sistema de tipos de Julia es que los tipos concretos no pueden tener subtipos. Todos los tiempos completos son finales y sólo pueden tener tipos abstractos como supertipos. Aunque esto puede parecer excesivamente restrictivo al principio, tiene muchas consecuencias beneficiosas con sorprendentemente pocos inconveniente. Resulta que ser capaz de heredar comportamientos es mucho más importante que seas capaz de heredar estructura, y heredar ambas cosas causa dificultades significativas en los lenguajes orientados a objetos tradicionales. Otros aspectos de alto nivel del sistema del tipo de Julia que Debería ser mencionados son:

  * No hay división entre los valores objetos y no objetos. Todos los valores en Julia son verdaderos objetos que tienen un tipo que pertenece a un solo grafo de tipos totalmente conectado, todos los nodos del cual son de primera clase como tipos.
  * No hay un concepto significativo de tiempo en tiempo de compilación. El único tipo que tiene un valor es su tipo actual cuando el programa está corriendo. Esto se denomina tipo en tiempo de ejecución en los lenguajes orientados a objetos, donde la combinación de complicación estática con polimorfismo hace esta distinción significativa.
  * Sólo los valores, no las variables, tienen tipos. Las variables son siempre nombres enlazados a valores.
  * Tanto los tipos abstractos como concretos pueden ser paralizados por otros tipos. Ellos pueden ser parametrizados mediante símbolos, mediante valores de cualquier tipo para los cuáles [`isbits()`](@ref) devuelve `true` (esencialmente, cosas como números y booleanos que son almacenados como tipos C o estructuras sin punteros a otros objetos), y también mediante tuplas. Los parámetros de tipo pueden ser omitidos cuando no necesitan ser referenciados o restringidos.

El sistema de tipos de Julia está diseñado para ser potente y expresivo, además de claro, intuitivo y no obstructivo. Muchos programadores Julia nunca sentirán la necesidad de escribir código que use los tipos explícitamente. Algunas clases de programación, sin embargo, se vuelven más claras, rápidas y robustas usando tipos declarados.

## Declaraciones de tipo

El operador `::` puede usarse para adjuntar declaraciones de tipo a expresiones y variables en los programas. Hay dos razones principales para esto:

1. Como una aserción para ayudar a confirmar que nuestro programa funciona como se esperaba.
2. Para proporcionar al compilador información extra sobre tipos, que puede mejorar el rendimiento en algunos casos.

Cuando se añade a una expresión que calcula un valor, el operador `::`se lee como "es una instancia de". Puede ser utilizado en cualquier parte para asertar que el valor de la expresión de la izquierda es una instancia del tipo de la derecha. Cuando el tipo de la derecha es concreto, el valor  sobre la izquierda debe tener ese tipo como su implementación (recuerde que todos los tipos concretos son finales, por lo que no hay implementaciones que sean subtipos de otros). Cuando el tipo es abstracto, basta que el valor sea implementado por un tipo concreto que sea un subtipo del tipo abstracto. Si la aserción de tipo no es `true` se lanza una excepción. En caso contrario, se devuelve el valor del lado izquierdo.

```jldoctest
julia> (1+2)::AbstractFloat
ERROR: TypeError: typeassert: expected AbstractFloat, got Int64

julia> (1+2)::Int
3
```

Esto permite que la aserción de tipo sea adjuntada a cualquier expresión *in situ*.

Cuando se añade a una variable sobre el lado izquierdo de una asignación, o como parte de una declaración `local`, el operador `::` significa algo un poco diferente: declara que la variable siempre tendrá el tipo especificado, como una declaración de tipo de los lenguajes tipados estáticamente como C. Cada valor asignado a la variable será convertido al tipo declarado usando [`convert()`](@ref):

```jldoctest
julia> function foo()
           x::Int8 = 100
           x
       end
foo (generic function with 1 method)

julia> foo()
100

julia> typeof(ans)
Int8
```

Esta característica es útil para evitar las trampas de rendimiento que podrían tener lugar si una de las asignaciones a varible cambia su tipo inesperadamente. 

Este comportamiento "declaración" sólo ocurre en contextos específicos:

```julia
local x::Int8  # in a local declaration
x::Int8 = 10   # as the left-hand side of an assignment
```

y se aplica al ámbito actual completo, incluso antes de la declaración. Actualmente, las declaraciones de tipo no pueden ser usadas en un espacio global, como en el REPL, ya que Julia no tiene aún globales de tipo constante.

Las declaraciones pueden también ser enlazadas a las definiciones de función:

```julia
function sinc(x)::Float64
    if x == 0
        return 1
    end
    return sin(pi*x)/(pi*x)
end
```

Retornar de esta función se comporta justo como una asignación a una variable con un tipo declarado: el tipo será siempre convertido a `Float64`.

## Tipos Abstractos

Los tipos abstractos no puede ser instanciados, y sólo sirven como nodos en el grafo de tipos, describiendo de este mundo conjuntos de tipos concretos relacionados: aquellos tipos concretos que son sus descendientes. Comenzamos con tipos abstractos incluso aunque no tienen instanciación debido a que ellos son la espina dorsal del sistema de tipos: ellos forman la jerarquí conceptual qu hace al sistema de tipos de Julia más que una colección de implementaciones de objetos. 

Recuerde que en [Números Enteros y en Punto Flotante](@ref), introdujimos una variedad de tipos de valores numéricos concretos: [`Int8`](@ref), [`UInt8`](@ref), [`Int16`](@ref), [`UInt16`](@ref),
[`Int32`](@ref), [`UInt32`](@ref), [`Int64`](@ref), [`UInt64`](@ref), [`Int128`](@ref),
[`UInt128`](@ref), [`Float16`](@ref), [`Float32`](@ref), and [`Float64`](@ref). Aunque todos ellos tienen diferentes tamaños en representación,  `Int8`, `Int16`, `Int32`, `Int64` and `Int128`
tienen en común que son tipos enteros con signo. Del mismo modo, `UInt8`, `UInt16`, `UInt32`,
`UInt64` and `UInt128` son enteros sin signo, mientras que `Float16`, `Float32` and
`Float64` son tipos en punto flotante. Es común para una pieza de código que ésta tenga sentido, por ejemplo, sólo si sus argumentos son algún tipo de entero, pero no que dependa de un tipo de entero particular. Por ejemplo, el algoritmo del máximo común denominador funciona para todas las clases de enteros, pero no funcionará para los números en punto flotante. Los tipos abstractos permiten la construcción de una jerarquía de tipos, proporcionando un contexto en el cuál los tipos concretos pueden ajustarse. Esto te permite, por ejemplo, programar fácilmente a cualquier tipo que sea un entero, sin restringir el algoritmo a un tipo de entero específico.

Los tipos abstractos se declaran usando la palabra clave `abstract`. Las sintaxis generales para declarar un tipo abstracto son:

```
abstract type «name» end
abstract type «name» <: «supertype» end
```

La palabra clave `abstract type` introduce un nuevo tipo abstracto, cuyo nombre viene dado por `«name»`. Este nombre puede ir seguido opcionalmente de `<:` y un nombre de tipo ya existente, lo cuál indica que este tipo abstracto es un subtipo del ya existente..

Cuando no se proporciona supertipo, el supertipo por defecto es `Any` (un tipo `abstract` predefinido del que todos los objetos son instancias y todos los tipos son subtipos). En teoría de tipos, `Any` suele ser denominado *top* porque es la cúspide del grafo de los tipos. Julia tiene también un tipo abstracto *bottom*, en el punto más bajo del grafo de tipos, que se escribe como `Union{}`. Es el opuesto exacto de `Any`: ningún objeto es instancia de `Union{}` y todos los tipos son sus supertipos.

Consideremos algunos de los tipos abstractos que forman parte de la jerarquía numérica de Julia:

```julia
abstract type Number end
abstract type Real     <: Number end
abstract type AbstractFloat <: Real end
abstract type Integer  <: Real end
abstract type Signed   <: Integer end
abstract type Unsigned <: Integer end
```

El tipo [`Number`](@ref) es un hijo directo de `Any`, y [`Real`](@ref) es su hijo. `Real` tiene dos hijos
(tiene más, pero sólo mostraremos dos aquí): [`Integer`](@ref) y [`AbstractFloat`](@ref), , que dividen 
el mundo entre representaciones de números enteros y reales. Las representaciones de números reales 
incluyen, por supuesto, los tipos en punto flotante, pero también incluyen otros tipos como los racionales. 
Por tanto,  `AbstractFloat` es un subtipo apropiado de `Real` que incluye sólo representaciones en punto 
flotante de los números reales. Los enteros están también divididos en las variedades [`Signed`](@ref) 
y [`Unsigned`](@ref).

El operador `<:` significa, en general, "es un subtipo de" y, usado en declaraciones como esta, declara 
que el tipo de la parte derecha es un supertipo inmediato de tipo que acaba de crearse. También puede 
usarse en expresiones como un obperador de subtipo que devuelve `true` cuando el operando de su izquierda 
es un subtipo del operando de su derecha:

```jldoctest
julia> Integer <: Number
true

julia> Integer <: AbstractFloat
false
```

Un uso importante de los tipos abstractos es proporcionar una implementación por defecto para los tipos concretos. Para dar un ejemplo simple, considere:

```julia
function myplus(x,y)
    x+y
end
```

La primera cosa que hay que notar es que las declaraciones de argumento anteriores son equivalentes a
`x::Any` e `y::Any`.  Cuando se invoca a estas funciones, digamos con `myplus(2,5)`, el despachador 
elige el método más específico cuyo nombres sea `myplus` y que se corresponda con los argumentos dados 
(ver [Métodos](@ref) para más informacíon sobre despacho múltiple).

Asumiendo que no se encuentra método más específico que el anterior, a continuación Julia define y 
compila un método llamado `myplus` específicamente para dos argumentos `Int` basado en la función 
genérica dada anteriormente, es decir, implícitamente define y compila:

```julia
function myplus(x::Int,y::Int)
    x+y
end
```

y, finalmente invoca a este método específico.

Por tanto, los tipos abstractos permiten a los programadores escribir funciones genéricas que puedan
ser usadas después como el método por defecto mediante muchas combinaciones de tipos concretos.
Gracias al despacho múltipel, el programador tiene control total sobre si se usa el método por 
defecto o uno más específico.

Un punto importante que notar es que no hay pérdida en el rendimiento si el programador se baa en 
una función cuyos argumentos sean tipos abstractos, dado que ella es recompilada para cada tupla 
de argumentos de tipos concretos con la cuál sea invocada (sin embargo, puede haber un problema de 
rendimiento en el caso de argumentos función que sean contenedores de tipos abstractos; ver 
[Consejos de Rendimiento](@ref man-performance-tips).)

## Tipos Primitivos

Un tipo primitivo es aquél un tipo concreto cuyos datos consisten en bits normales y corrientes. 
Ejemplos clásicos de tipos bits son los valores enteros y punto flotante. A diferencia de muchos 
enguajes, Julia nos permite declarar nuestros propios tipos bits, en lugar de proporcionar un 
conjunto fijo de tipos bits predefinidos. De hecho, los tupos bits estándar que están definidos 
en el propios lenguaje son:

```julia
primitive type Float16 <: AbstractFloat 16 end
primitive type Float32 <: AbstractFloat 32 end
primitive type Float64 <: AbstractFloat 64 end

primitive type Bool <: Integer 8 end
primitive type Char 32 end

primitive type Int8    <: Signed   8 end
primitive type UInt8   <: Unsigned 8 end
primitive type Int16   <: Signed   16 end
primitive type UInt16  <: Unsigned 16 end
primitive type Int32   <: Signed   32 end
primitive type UInt32  <: Unsigned 32 end
primitive type Int64   <: Signed   64 end
primitive type UInt64  <: Unsigned 64 end
primitive type Int128  <: Signed   128 end
primitive type UInt128 <: Unsigned 128 end
```

Las sintaxis generales para la declaración de un `bitstype` son:

```
primitive type «name» «bits» end
primitive type «name» <: «supertype» «bits» end
```

El `«bits»` indica cuánta memoria requiere el tipo y `«name»` indica el nombre del nuevo 
tipo. Un tipo primitivo puede ser declarado opcionalmente como un subtipo de algún 
supertipo. Si se omite el supertipo, se asigna como supertipo por defecto el tipo `Any`. 
Por tanto, la declaración de [`Bool`](@ref) significa que un valor booleano consume 8 bits 
de almacenamiento, y que [`Integer`](@ref) es su supertipo inmediato. Actualmente sólo se 
soportan tamaños que sea múltiplo de 8 bits. Por tanto, los valores booleanos, aunque sólo 
necesitan un bit, no puden ser declarados como menores de 8 bits.

Los tipos [`Bool`](@ref), [`Int8`](@ref) y [`UInt8`](@ref) tienen representaciones 
idénticas: se trata de bloques de memoria de 8 bits. Como el sistema de tipos de Julia es 
nominativo, sin embargo, ellos no son intercambiables aunque tengan estructura idéntica. 
Otra diferencia fundamental entre ellos es que ellos tienen supertipos diferentes: 
[`Integer`](@ref) es el supertipo directo de [`Bool`](@ref), [`Signed`](@ref) es el de
[`Int8`](@ref), y [`Unsigned`](@ref) es el de  [`UInt8`](@ref). Todas las demás diferencias
entre  [`Bool`](@ref), [`Int8`](@ref), y [`UInt8`](@ref) son cuestiones de comportamiento 
(la forma en la que las funciones son definidas para actuar cuando se pasan como argumentos 
objetos de estos tipos). Esta es la razón por la que es necesario un sistema de tipos 
nominativo: si la estructura determinara el tipo, que a su vez dicta el comportamiento, 
sería imposible hacer que los valores  [`Bool`](@ref) se comportaran de forma diferente a los
[`Int8`](@ref) o los [`UInt8`](@ref).

## Tipos Compuestos

[Los tipos compuestos](https://en.wikipedia.org/wiki/Composite_data_type) se llaman registros, estructuras (*structs*) u objetos en distintos lenguajes. Un tipo compuesto es una colección de campos nombrados, una instancia de los cuales puede ser tratada como un único valor. En muchos lenguajes, los tiempos compuestos sos la única clase de tipos definidos por el usuario, y ellos son de lejos el tiempo definido por el usuario  que se usa mas comúnmente en el lenguaje Julia.

En el mundo de los lenguajes orientados a objetos tales como C++, Java, Python o Rubi, los tiempos compuestos también tienen funciones nombrados asociados con ellos, y la combinación se denomina "objeto". En los lenguajes orientados a objetos puros, tales como Ruby o SmallTalk, todo los valores son objetos sean compuestos uno. En lenguajes orientados objetos menos puros incuyendo C++ y Java, algunos valores tales como los enteros no se consideran objetos, mientras que las instancias de los tipos compuestos definidos por el usuario son verdaderos objetos con métodos asociados. En Julia, todos los valores son objetos, pero las funciones no están ligadas a los objetos sobre los que opera. Esto es necesario ya que Julia exige que método de una función usar mediante despacho múltiple, lo que significa que los tipos de todos los argumentos de una función son considerados cuando se selecciona un método en lugar de sólo el primero (ver la sección [Métodos](@ref) para más información). Por tanto, sería inapropiado para las funciones pertenecer a su primer argumento solo. Organizar métodos en objetos función en lugar de tener bolsas de métodos nombrados "dentro" de cada objeto termina por ser un aspecto muy beneficioso de diseño del lenguaje.

Los tipos compuestos son introducidos por la palabra clave `struct` seguida por un bloque de nombres de campos, opcionalmente anotados con tipos usando el operador `::`

```jldoctest footype
julia> struct Foo
           bar
           baz::Int
           qux::Float64
       end
```

Los campos sin anotación de tipos tiene asignado `Any` como tipo por defecto, y pueden según esto almacenar cualquier tipo de valor.

Para crear nuevos objetos del tipo compuesto `Foo` se crean aplicando el tipo objeto `Foo` como una función con valores para sus campos:

```jldoctest footype
julia> foo = Foo("Hello, world.", 23, 1.5)
Foo("Hello, world.", 23, 1.5)

julia> typeof(foo)
Foo
```

Cuando un tipo es aplicado como una función es llamado *constructor*. Hay dos constructores, denominados *constructores por defecto* que se generan automáticamente al crear el tipo. Uno acepta cualquier argumento y llama a [`convert()`](@ref) para convertirlos a los tipos de los campos, y el otro acepta argumentos que se corresponden exactamente a los tipos de los campos. La razón de que ambos métodos sean generados es que esto hace más sencillo añadir nuevas definiciones sin reemplazar inadvertidamente a un constructor por defecto.

Como el campo `bar` no está restringido en tipo, cualquier valor es válido. Sin embargo, el valor para `baz` debe ser convertible a `Int`:

```jldoctest footype
julia> Foo((), 23.5, 1)
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Int64}, ::Float64) at ./float.jl:680
 [2] Foo(::Tuple{}, ::Float64, ::Int64) at ./none:2
```

La función `fieldnames` devuelve una lista de los nombres de campos de un objeto:

```jldoctest footype
julia> fieldnames(Foo)
3-element Array{Symbol,1}:
 :bar
 :baz
 :qux
```

Para acceder a los valores de los campis de un objeto compuesto puede usarse la notación tradicional `foo.bar`:

```jldoctest footype
julia> foo.bar
"Hello, world."

julia> foo.baz
23

julia> foo.qux
1.5
```

Los tipos compuestos declarados con `struct` son *inmutables*, es decir, no pueden ser modificados después de su construcción. Esto puede parecer raro al principio, pero tiene varias ventajas:

   * Puede ser más eficiciente. Algunos `struct` pueden ser empquetados eficientemente dentro de los arrays, y 
     en algunos casos el compilador es capaz de evitar asignar objetos inmutables completamente.
   * No es posible violar las invariantes proporcionadas por los contructores de tipo.
   * El código con objetos inmutables puede ser más sencillo de interpretar.

Un objeto inmutable puede contener objetos mutables, tales como arrays, como campos. Esos objetos contenidos permanecerán inmutables, sólo los cambos del objeto inmutable en sí no podrán ser cambiados para apuntar a objetos distintos. 

Cuando se requiera, los objetos compuestos mutables podrán ser declarados con la palabra clave `mutable struct`, lo que será discutido en la siguiente sección. 

Los tipos compuestos sin campos son *singletons*, es decir, sólo puede haber una instancia de tales tipos:

```jldoctest
julia> struct NoFields
       end

julia> NoFields() === NoFields()
true
```

La función `===` confirma que las dos instancias construidas de `NoFields` son de hecho una y la misma. Los tipos *singleton* se describirán en más detalle [posteriormente](@ref man-singleton-types).

Hay mucho más que decir sobre cómo se crean las instancias de los tipos compuestos, pero esta discusión depende de los [Tipos Paramétricos](@ref) y de los [Métodos](@ref), y es suficientemente importante para ser tratada en su propia sección: [Constructores](@ref man-constructors).

## Tipos Compuestos Mutables

Si un tipo compuesto es declarado como `mutable struct` en lugar de como `struct`, sus instancias pueden ser modificadas:

```jldoctest bartype
julia> mutable struct Bar
           baz
           qux::Float64
       end

julia> bar = Bar("Hello", 1.5);

julia> bar.qux = 2.0
2.0

julia> bar.baz = 1//2
1//2
```

Para soportar la mutación, tales objetos se alojan generalmente en el montón y tienen direcciones de memoria estables. Un objeto mutable es como un pequeño contenedor que podría almacenar distintos valores en el tiempo, y por tanto sólo puede ser identificado de forma confiable con su dirección. En contraste, una instancia de un tipo inmutable está asociada con valores de campos específicos - los valores de campos solos te dicen todo sobre el objeto. En decidir si hacer un tipo mutable, pregúntat si dos instanciasa con los mismos valores de campos tendrían que ser consideradas idénticas, o si ellas podrían necesitar cambiar independientemente con el tiempo. Si ellas fueran consideradas idénticas, el tipo probablemente debería ser inmutable. 

Para recapitular, dos propiedades esenciales definen la inmutabilidad en Julia:

* Un objeto con un tipo inmutable es pasado (tanto en instrucciones de asignación como en llamadas a función) mediante copia, mientras que un tipo mutable es pasado mediante referencia.
* No está permitido modificar los campos de un tipo compuesto inmutable.

Es instructivo, particularmente para lectores cuyo background es C/C++, considerar por qué estas dos propiedades van juntas. Si fueran separadas, es decir, si los campos de los objetos pasados mediante copia pudieran ser modificados, entonces sería más difícil razonar sobre ciertas instancias de código genérico. Por ejemplo, supongamos que `x` es un argumento de función de un tipo abstracto, y supongamos que la función cambia un campo: `x.isprocessed = true`. Dependiendo de si `x`se pasa mediante copia o mediante referencia, esta instrucción puede alterar o no el argumento actual en la rutina que hace la llamada. Julia evita la posibilidad de crear funciones con efectos desconocidos en este escenario prohibiendo modificación de campos de objetos pasados mediante copia. 

## Tipos declarados

Las tres clases de tipos discutidos en las tres secciones anteriores están muy relacionados. Ellos comparten las mismas propiedades clave:

* Ellos son declarados explícitamente.
* Ellos tienen nombres.
* Ellos tienen supertipos declarados explícitamente.
* Ellos pueden tener parámeros.

Debido a estas propiedades compartidas, estos tipos son representados internamene  como instancias del mismo concepto, `DataType` , que es el tipo de cualquiera de estos tipos:

```jldoctest
julia> typeof(Real)
DataType

julia> typeof(Int)
DataType
```

Un `DataType` puede ser abstracto o concreto. Si es concreto, tiene un tamaño, disposición de almacenamiento y (opcionalmente) nombres de campos especificados. Por tanto, un tipo bits es un `DataType` con tamaño no nulo, pero sin nombres de campos. Un tipo compuesto es un `DataType` que tiene nombres de campos o es acío (tamaño cero).

Cada valor concreto en el sistema es una instancia de algún `DataType`.

## Uniones de Tipo

Una unión de tipo es un tipo abstracto especial que incluye como objetos todas las instancias de alguno de sus tipos argumentos, construidos usando la función especial `Union`:

```jldoctest
julia> IntOrString = Union{Int,AbstractString}
Union{AbstractString, Int64}

julia> 1 :: IntOrString
1

julia> "Hello!" :: IntOrString
"Hello!"

julia> 1.0 :: IntOrString
ERROR: TypeError: typeassert: expected Union{AbstractString, Int64}, got Float64
```

Los compiladores de muchos lenguajes tienen una construcción unión interna para razonar sobre los tipos; Julia simplemente la pone a disposición del programador.

## Tipos Paramétricos

Una característica importante y potente del sistema de tipos de Julia es que es paramétrico: los tipos pueden toomar parámetros, por lo que las declaraciones de tipo introducen de hecho un afamilia completa de nuevos tipos (uno por cada posible combinación de valores de parámetros). Hay muchos lenguajes que soportan alguna versión de la [programación genérica](https://en.wikipedia.org/wiki/Generic_programming), donde las estructuras de datos y algoritmos para manipularlos pueden ser especificadas sin especificar los tipos exactos implicados. Por ejemplo, existe alguna forma de programación genérica en ML, Haskell, Ada, Eiffel, C++, Java, C#, F# y Scala, por nombrar unos pocos. Algunos de estos lenguajes soportan un verdadero polimorfismo paramétrico (Por ej., ML, Haskell, Scala) mientras otros soportan estilos de programación genérica *ad-hoc*, basados en plantillas (Por eje., C++ y Java). Con tantas variedades diferentes de programación genérica y de tipos paramétricos en los distintos lenguajes, no queremos ni siquiera intentar comparar los tipos paramétricos de Julia a otros lenguajes, sino que nos centraremos en explicar el propio sistema de Julia. Notaremos, sin embargo, que como Julia es un lenguaje tipado dinámicamente y no necesita hacer todas las decisiones de tipos en tiempo de compilacíon, muchas dificultades tradicionales encontradas en los sistemas de tipos paramétricos estáticos pueden ser manejadas con relativa facilidad.

Todos los tipos declarados (la variedad `DataType`)  pueden ser parametrizados, con la misma sintaxis en cada caso. Los discutiremos en el siguiente orden: primero tipos compuestos paramétricos, luego tipos abstractos paramétricos y por último tipos bits paramétricos.

### Tipos compuestos paramétricos

Los parámetros de tipo se introducen inmediatamente después del nombre de tipo, rodeado por llaves:

```jldoctest pointtype
julia> struct Point{T}
           x::T
           y::T
       end
```

Esta declaración define un nuevo tipo paramétrico, `Point{T}`, que almacena dos coordenadas de tipo `T`. Uno podría preguntarse, ¿qué es `T`? Bien, este es precisamente el punto de los tipos paramétricos: puede ser cualquier tipo (o un valor de cualquier tipo bits, aunque en esta ocasión usado como un tipo, claramente). `Point{Float64}` es un tipo concreto equivalente al tipo definido reemplazando `T` en la definición de `Point` con [`Float64`](@ref). Por tanto, esta única  declaración declara un número de tipos ilimitado: `Point{Float64}`, `Point{AbstractString}`, `Point{Int64}`, etc. Cada uno de ellos es un tipo concreto usable: 

```jldoctest pointtype
julia> Point{Float64}
Point{Float64}

julia> Point{AbstractString}
Point{AbstractString}
```

El tipo `Point{Float64}` es un punto cuyas coordenadas son valores en punto flotante de 64-bits, mientras que el tipo `Point{AbstractString}` es un “punto” cuyas “coordenadas” son objetos `String` (see [Strings](@ref)).

`Point` es en si mismo un tipo objeto válido también, que contiene todas las instancias `Point{Float64}`, `Point{AbstractString}`, etc. como subtipos:

```jldoctest pointtype
julia> Point{Float64} <: Point
true

julia> Point{AbstractString} <: Point
true
```

Otros tipos, por supuesto, no son subtipos de él:

```jldoctest pointtype
julia> Float64 <: Point
false

julia> AbstractString <: Point
false
```

Tipos `Point` concretos con valores diferentes de `T` no son nunca subtipos uno de otro:

```jldoctest pointtype
julia> Point{Float64} <: Point{Int64}
false

julia> Point{Float64} <: Point{Real}
false
```

!!! warning
    Este último punto es importante: **Incluso aunque `Float64 <: Real` no es cierto que `Point{Float64} <: Point{Real}`**. 

En otras palabras, en términos de teoría de tipos, los parámetros de tipo de Julia son *invariantes*, en lugar de ser [covariantes (o incluso contravariantes)](https://en.wikipedia.org/wiki/Covariance_and_contravariance_%28computer_science%29). Esto es por razones prácticas: aunque alguna instancia de `Point{Float64}` puede ser conceptualmente como una instancia de `Point{Real}`, los dos tipos tienen representaciones en memoria diferentes:

  * Una instancia de `Point{Float64}` puede ser representada compactamente y eficientemente como un par inmediato 
    de valores de 64 bits
  * Una instancia de `Point{Real}` debe ser capaz de alojar cualquier par de instancias de [`Real`](@ref). Como los 
    objetos son instancias de `Real` pueden ser de tamaño y estructura arbitrarios, en la práctica una instancia de
    `Point{Real` debe ser representada como un par de punteros a objetos `Real` asignados individualmente.

La eficiencia ganada por ser capaz de almacenar objetos `Point{Float64}` con valores inmediatos es magnificada enormemente en el caso de arrays: un `Array{Float64}` puede almacenarse como un bloque contiguo de memoria de valores en punto flotante de 64 bits, mientras que un `Array{Real}` debe ser un array de punteros a objetos [`Real`](@ref) asignados individualmente (que puede ser valores en punto flotante de 64 bits [envueltos (*boxed*)](https://en.wikipedia.org/wiki/Object_type_%28object-oriented_programming%29#Boxing), pero que también pueden ser objetos complejos, arbitrariamente grandes, que han sido declarados como implementaciones del tipo abstracto `Real`.

Como `Point{Float64}` no es un subtipo de `Point{Real}`,  el siguiente método no puede ser aplicado a argumentos de tipo `Point{Float64}`:

```julia
function norm(p::Point{Real})
    sqrt(p.x^2 + p.y^2)
end
```

Una forma correcta de definir un método que acepte todos los argumentos de tipo `Point{T}`, donde `T` es un subtipo de [`Real`](@ref) es:

```julia
function norm(p::Point{<:Real})
    sqrt(p.x^2 + p.y^2)
end
```

(Equivalentemente, uno podría definir `function norm{T<:Real}(p::Point{T})` o `function norm(p::Point{T} where T<:Real)`; ver [tipos UnionAll](@ref).)

Más ejemplos se discutirán después en [Métodos](@ref).

¿Cómo construye uno un objeto `Point`? Es posible definir constructores personalizados para tipos compuestos, que serán discutidos en detalle en el capítulo [Constructores](@ref man-constructors), pero en ausencia de ninguna declaración especial de constructor, hay dos formas por defecto de crear nuevos objetos compuestos, uno en el que se dan explícitamente los parámetros de tipo y otro en el que ellos son implicados por los argumentos al objeto constructor.

Como el tipo `Point{Float64}` es un tipo concreto equivalente a `Point` declarado con `Float64` en lugar de `T`, se puede aplicar como un constructor ede acuerdo a ésto:

```jldoctest pointtype
julia> Point{Float64}(1.0, 2.0)
Point{Float64}(1.0, 2.0)

julia> typeof(ans)
Point{Float64}
```

Para el constructor por defecto, debe proporcionarse exactamente un argumento por cada campo: 

```jldoctest pointtype
julia> Point{Float64}(1.0)
ERROR: MethodError: Cannot `convert` an object of type Float64 to an object of type Point{Float64}
This may have arisen from a call to the constructor Point{Float64}(...),
since type constructors fall back to convert methods.
Stacktrace:
 [1] Point{Float64}(::Float64) at ./sysimg.jl:102

julia> Point{Float64}(1.0,2.0,3.0)
ERROR: MethodError: no method matching Point{Float64}(::Float64, ::Float64, ::Float64)
```

Sólo se ha generado un constructor por defecto para tipos paramétricos, ya que sobreescribirlo no es posible. Este constructor acepta cualquier argumento y los convierte a los tipos de los campos.

En muchos casos es redundante proporcionar el tipo del objeto `Point` que uno quiere construir, ya que los tipos de los argumentos en la llamada al constructor ya proporcionan la información de tipos de forma implícita. Por esta razón, también podemos aplicar el propio `Point` como un constructor, dado que el valor implícito del parámetro de tipo `T` no es ambiguo:

```jldoctest pointtype
julia> Point(1.0,2.0)
Point{Float64}(1.0, 2.0)

julia> typeof(ans)
Point{Float64}

julia> Point(1,2)
Point{Int64}(1, 2)

julia> typeof(ans)
Point{Int64}
```

En el caso de `Point` es implicado sin ambigüedad si y sólo si los dos argumentos a `Point` tienen el mismo tipo. Cuando este no es el caso, el constructor fallará con un [`MethodError`](@ref):

```jldoctest pointtype
julia> Point(1,2.5)
ERROR: MethodError: no method matching Point(::Int64, ::Float64)
Closest candidates are:
  Point(::T, !Matched::T) where T at none:2
```

Los método constructores para manejar apropiadamente estos casos mixtos pueden ser definidos, pero esto no será discutido hasta después en [Constructores](@ref man-constructors).

### Tipos Abstractos Paramétricos  

Las declaraciones de tipos abstractos paramétricos declaran una colección de tipos abstractos, de la misma forma:

```jldoctest pointytype
julia> abstract type Pointy{T} end
```

Con esta declaración, `Pointy{T}` es un tipo abstracto distinto para cada tipo o valor entero de `T`. Como con los tipos compuestos paramétricos, cada una de tales instancias es un subtipo de `Pointy`:

```jldoctest pointytype
julia> Pointy{Int64} <: Pointy
true

julia> Pointy{1} <: Pointy
true
```

Los tipos abstractos paramétricos son invariantes, tal como los tipos compuestos paramétricos:

```jldoctest pointytype
julia> Pointy{Float64} <: Pointy{Real}
false

julia> Pointy{Real} <: Pointy{Float64}
false
```

La notación `Pointy{<:Real}` puede usarse para expresar el análogo Julia de un tipo *covariante*, mientras que  `Pointy{>:Int}` es el análogo de un tipo *contravariante*, pero técnicamente estos representan *conjuntos* de tipos (ver [tipos UnionAll](@ref)).

```jldoctest pointytype
julia> Pointy{Float64} <: Pointy{<:Real}
true

julia> Pointy{Real} <: Pointy{>:Int}
true
```

De la misma manera que los tipos abstractos antiguos sirven para crear una jerarquía útil de tipos sobre tipos concretos, los tipos abstractos paramétricos tienen el mismo propósito con respecto a los tipos compuestos paramétricos. Podríamos, por ejemplo, haber declarado `Point {T}` ser un subtipo de `Pointy {T}` de la siguiente manera:

```jldoctest pointytype
julia> struct Point{T} <: Pointy{T}
           x::T
           y::T
       end
```

Dada tal declaración, para cada elección de `T` tenemos `Point{T}` como un subtipo de `Pointy{T}`:

```jldoctest pointytype
julia> Point{Float64} <: Pointy{Float64}
true

julia> Point{Real} <: Pointy{Real}
true

julia> Point{AbstractString} <: Pointy{AbstractString}
true
```

Esta relación es también invariante:

```jldoctest pointytype
julia> Point{Float64} <: Pointy{Real}
false

julia> Point{Float64} <: Pointy{<:Real}
true
```

What purpose do parametric abstract types like `Pointy` serve? Consider if we create a point-like
implementation that only requires a single coordinate because the point is on the diagonal line
*x = y*:

```jldoctest pointytype
julia> struct DiagPoint{T} <: Pointy{T}
           x::T
       end
```

Now both `Point{Float64}` and `DiagPoint{Float64}` are implementations of the `Pointy{Float64}`
abstraction, and similarly for every other possible choice of type `T`. This allows programming
to a common interface shared by all `Pointy` objects, implemented for both `Point` and `DiagPoint`.
This cannot be fully demonstrated, however, until we have introduced methods and dispatch in the
next section, [Methods](@ref).

There are situations where it may not make sense for type parameters to range freely over all
possible types. In such situations, one can constrain the range of `T` like so:

```jldoctest realpointytype
julia> abstract type Pointy{T<:Real} end
```

With such a declaration, it is acceptable to use any type that is a subtype of
[`Real`](@ref) in place of `T`, but not types that are not subtypes of `Real`:

```jldoctest realpointytype
julia> Pointy{Float64}
Pointy{Float64}

julia> Pointy{Real}
Pointy{Real}

julia> Pointy{AbstractString}
ERROR: TypeError: Pointy: in T, expected T<:Real, got Type{AbstractString}

julia> Pointy{1}
ERROR: TypeError: Pointy: in T, expected T<:Real, got Int64
```

Type parameters for parametric composite types can be restricted in the same manner:

```julia
struct Point{T<:Real} <: Pointy{T}
    x::T
    y::T
end
```

To give a real-world example of how all this parametric type machinery can be useful, here is
the actual definition of Julia's [`Rational`](@ref) immutable type (except that we omit the
constructor here for simplicity), representing an exact ratio of integers:

```julia
struct Rational{T<:Integer} <: Real
    num::T
    den::T
end
```

It only makes sense to take ratios of integer values, so the parameter type `T` is restricted
to being a subtype of [`Integer`](@ref), and a ratio of integers represents a value on the
real number line, so any [`Rational`](@ref) is an instance of the [`Real`](@ref) abstraction.

### Tuple Types

Tuples are an abstraction of the arguments of a function -- without the function itself. The salient
aspects of a function's arguments are their order and their types. Therefore a tuple type is similar
to a parameterized immutable type where each parameter is the type of one field. For example,
a 2-element tuple type resembles the following immutable type:

```julia
struct Tuple2{A,B}
    a::A
    b::B
end
```

However, there are three key differences:

  * Tuple types may have any number of parameters.
  * Tuple types are *covariant* in their parameters: `Tuple{Int}` is a subtype of `Tuple{Any}`. Therefore
    `Tuple{Any}` is considered an abstract type, and tuple types are only concrete if their parameters
    are.
  * Tuples do not have field names; fields are only accessed by index.

Tuple values are written with parentheses and commas. When a tuple is constructed, an appropriate
tuple type is generated on demand:

```jldoctest
julia> typeof((1,"foo",2.5))
Tuple{Int64,String,Float64}
```

Note the implications of covariance:

```jldoctest
julia> Tuple{Int,AbstractString} <: Tuple{Real,Any}
true

julia> Tuple{Int,AbstractString} <: Tuple{Real,Real}
false

julia> Tuple{Int,AbstractString} <: Tuple{Real,}
false
```

Intuitively, this corresponds to the type of a function's arguments being a subtype of the function's
signature (when the signature matches).

### Vararg Tuple Types

The last parameter of a tuple type can be the special type `Vararg`, which denotes any number
of trailing elements:

```jldoctest
julia> mytupletype = Tuple{AbstractString,Vararg{Int}}
Tuple{AbstractString,Vararg{Int64,N} where N}

julia> isa(("1",), mytupletype)
true

julia> isa(("1",1), mytupletype)
true

julia> isa(("1",1,2), mytupletype)
true

julia> isa(("1",1,2,3.0), mytupletype)
false
```

Notice that `Vararg{T}` corresponds to zero or more elements of type `T`. Vararg tuple types are
used to represent the arguments accepted by varargs methods (see [Varargs Functions](@ref)).

The type `Vararg{T,N}` corresponds to exactly `N` elements of type `T`.  `NTuple{N,T}` is a convenient
alias for `Tuple{Vararg{T,N}}`, i.e. a tuple type containing exactly `N` elements of type `T`.

#### [Singleton Types](@id man-singleton-types)

There is a special kind of abstract parametric type that must be mentioned here: singleton types.
For each type, `T`, the "singleton type" `Type{T}` is an abstract type whose only instance is
the object `T`. Since the definition is a little difficult to parse, let's look at some examples:

```jldoctest
julia> isa(Float64, Type{Float64})
true

julia> isa(Real, Type{Float64})
false

julia> isa(Real, Type{Real})
true

julia> isa(Float64, Type{Real})
false
```

In other words, [`isa(A,Type{B})`](@ref) is true if and only if `A` and `B` are the same object
and that object is a type. Without the parameter, `Type` is simply an abstract type which has
all type objects as its instances, including, of course, singleton types:

```jldoctest
julia> isa(Type{Float64}, Type)
true

julia> isa(Float64, Type)
true

julia> isa(Real, Type)
true
```

Any object that is not a type is not an instance of `Type`:

```jldoctest
julia> isa(1, Type)
false

julia> isa("foo", Type)
false
```

Until we discuss [Parametric Methods](@ref) and [conversions](@ref conversion-and-promotion), it is difficult to explain
the utility of the singleton type construct, but in short, it allows one to specialize function
behavior on specific type *values*. This is useful for writing methods (especially parametric
ones) whose behavior depends on a type that is given as an explicit argument rather than implied
by the type of one of its arguments.

A few popular languages have singleton types, including Haskell, Scala and Ruby. In general usage,
the term "singleton type" refers to a type whose only instance is a single value. This meaning
applies to Julia's singleton types, but with that caveat that only type objects have singleton
types.

### Parametric Primitive Types

Primitive types can also be declared parametrically. For example, pointers are represented as
primitive types which would be declared in Julia like this:

```julia
# 32-bit system:
primitive type Ptr{T} 32 end

# 64-bit system:
primitive type Ptr{T} 64 end
```

The slightly odd feature of these declarations as compared to typical parametric composite types,
is that the type parameter `T` is not used in the definition of the type itself -- it is just
an abstract tag, essentially defining an entire family of types with identical structure, differentiated
only by their type parameter. Thus, `Ptr{Float64}` and `Ptr{Int64}` are distinct types, even though
they have identical representations. And of course, all specific pointer types are subtypes of
the umbrella `Ptr` type:

```jldoctest
julia> Ptr{Float64} <: Ptr
true

julia> Ptr{Int64} <: Ptr
true
```

## UnionAll Types

We have said that a parametric type like `Ptr` acts as a supertype of all its instances
(`Ptr{Int64}` etc.). How does this work? `Ptr` itself cannot be a normal data type, since without
knowing the type of the referenced data the type clearly cannot be used for memory operations.
The answer is that `Ptr` (or other parametric types like `Array`) is a different kind of type called a
`UnionAll` type. Such a type expresses the *iterated union* of types for all values of some parameter.

`UnionAll` types are usually written using the keyword `where`. For example `Ptr` could be more
accurately written as `Ptr{T} where T`, meaning all values whose type is `Ptr{T}` for some value
of `T`. In this context, the parameter `T` is also often called a "type variable" since it is
like a variable that ranges over types.
Each `where` introduces a single type variable, so these expressions are nested for types with
multiple parameters, for example `Array{T,N} where N where T`.

The type application syntax `A{B,C}` requires `A` to be a `UnionAll` type, and first substitutes `B`
for the outermost type variable in `A`.
The result is expected to be another `UnionAll` type, into which `C` is then substituted.
So `A{B,C}` is equivalent to `A{B}{C}`.
This explains why it is possible to partially instantiate a type, as in `Array{Float64}`: the first
parameter value has been fixed, but the second still ranges over all possible values.
Using explicit `where` syntax, any subset of parameters can be fixed. For example, the type of all
1-dimensional arrays can be written as `Array{T,1} where T`.

Type variables can be restricted with subtype relations.
`Array{T} where T<:Integer` refers to all arrays whose element type is some kind of
[`Integer`](@ref).
The syntax `Array{<:Integer}` is a convenient shorthand for `Array{T} where T<:Integer`.
Type variables can have both lower and upper bounds.
`Array{T} where Int<:T<:Number` refers to all arrays of [`Number`](@ref)s that are able to
contain `Int`s (since `T` must be at least as big as `Int`).
The syntax `where T>:Int` also works to specify only the lower bound of a type variable,
and `Array{>:Int}` is equivalent to `Array{T} where T>:Int`.

Since `where` expressions nest, type variable bounds can refer to outer type variables.
For example `Tuple{T,Array{S}} where S<:AbstractArray{T} where T<:Real` refers to 2-tuples
whose first element is some [`Real`](@ref), and whose second element is an `Array` of any
kind of array whose element type contains the type of the first tuple element.

The `where` keyword itself can be nested inside a more complex declaration. For example,
consider the two types created by the following declarations:

```jldoctest
julia> const T1 = Array{Array{T,1} where T, 1}
Array{Array{T,1} where T,1}

julia> const T2 = Array{Array{T,1}, 1} where T
Array{Array{T,1},1} where T
```

Type `T1` defines a 1-dimensional array of 1-dimensional arrays; each
of the inner arrays consists of objects of the same type, but this type may vary from one inner array to the next.
On the other hand, type `T2` defines a 1-dimensional array of 1-dimensional arrays all of whose inner arrays must have the
same type.  Note that `T2` is an abstract type, e.g., `Array{Array{Int,1},1} <: T2`, whereas `T1` is a concrete type. As a consequence, `T1` can be constructed with a zero-argument constructor `a=T1()` but `T2` cannot.

There is a convenient syntax for naming such types, similar to the short form of function
definition syntax:

```julia
Vector{T} = Array{T,1}
```

This is equivalent to `const Vector = Array{T,1} where T`.
Writing `Vector{Float64}` is equivalent to writing `Array{Float64,1}`, and the umbrella type
`Vector` has as instances all `Array` objects where the second parameter -- the number of array
dimensions -- is 1, regardless of what the element type is. In languages where parametric types
must always be specified in full, this is not especially helpful, but in Julia, this allows one
to write just `Vector` for the abstract type including all one-dimensional dense arrays of any
element type.

## Type Aliases

Sometimes it is convenient to introduce a new name for an already expressible type.
This can be done with a simple assignment statement.
For example, `UInt` is aliased to either [`UInt32`](@ref) or [`UInt64`](@ref) as is
appropriate for the size of pointers on the system:

```julia-repl
# 32-bit system:
julia> UInt
UInt32

# 64-bit system:
julia> UInt
UInt64
```

This is accomplished via the following code in `base/boot.jl`:

```julia
if Int === Int64
    const UInt = UInt64
else
    const UInt = UInt32
end
```

Of course, this depends on what `Int` is aliased to -- but that is predefined to be the correct
type -- either [`Int32`](@ref) or [`Int64`](@ref).

(Note that unlike `Int`, `Float` does not exist as a type alias for a specific sized
[`AbstractFloat`](@ref). Unlike with integer registers, the floating point register sizes
are specified by the IEEE-754 standard. Whereas the size of `Int` reflects the size of a
native pointer on that machine.)

## Operations on Types

Since types in Julia are themselves objects, ordinary functions can operate on them. Some functions
that are particularly useful for working with or exploring types have already been introduced,
such as the `<:` operator, which indicates whether its left hand operand is a subtype of its right
hand operand.

The [`isa`](@ref) function tests if an object is of a given type and returns true or false:

```jldoctest
julia> isa(1, Int)
true

julia> isa(1, AbstractFloat)
false
```

The [`typeof()`](@ref) function, already used throughout the manual in examples, returns the type
of its argument. Since, as noted above, types are objects, they also have types, and we can ask
what their types are:

```jldoctest
julia> typeof(Rational{Int})
DataType

julia> typeof(Union{Real,Float64,Rational})
DataType

julia> typeof(Union{Real,String})
Union
```

What if we repeat the process? What is the type of a type of a type? As it happens, types are
all composite values and thus all have a type of `DataType`:

```jldoctest
julia> typeof(DataType)
DataType

julia> typeof(Union)
DataType
```

`DataType` is its own type.

Another operation that applies to some types is [`supertype()`](@ref), which reveals a type's
supertype. Only declared types (`DataType`) have unambiguous supertypes:

```jldoctest
julia> supertype(Float64)
AbstractFloat

julia> supertype(Number)
Any

julia> supertype(AbstractString)
Any

julia> supertype(Any)
Any
```

If you apply [`supertype()`](@ref) to other type objects (or non-type objects), a [`MethodError`](@ref)
is raised:

```jldoctest
julia> supertype(Union{Float64,Int64})
ERROR: MethodError: no method matching supertype(::Type{Union{Float64, Int64}})
Closest candidates are:
  supertype(!Matched::DataType) at operators.jl:41
  supertype(!Matched::UnionAll) at operators.jl:46
```

## Custom pretty-printing

Often, one wants to customize how instances of a type are displayed.  This is accomplished by
overloading the [`show()`](@ref) function.  For example, suppose we define a type to represent
complex numbers in polar form:

```jldoctest polartype
julia> struct Polar{T<:Real} <: Number
           r::T
           Θ::T
       end

julia> Polar(r::Real,Θ::Real) = Polar(promote(r,Θ)...)
Polar
```

Here, we've added a custom constructor function so that it can take arguments of different
[`Real`](@ref) types and promote them to a common type (see [Constructors](@ref man-constructors)
and [Conversion and Promotion](@ref conversion-and-promotion)).
(Of course, we would have to define lots of other methods, too, to make it act like a
[`Number`](@ref), e.g. `+`, `*`, `one`, `zero`, promotion rules and so on.) By default,
instances of this type display rather simply, with information about the type name and
the field values, as e.g. `Polar{Float64}(3.0,4.0)`.

If we want it to display instead as `3.0 * exp(4.0im)`, we would define the following method to
print the object to a given output object `io` (representing a file, terminal, buffer, etcetera;
see [Networking and Streams](@ref)):

```jldoctest polartype
julia> Base.show(io::IO, z::Polar) = print(io, z.r, " * exp(", z.Θ, "im)")
```

More fine-grained control over display of `Polar` objects is possible. In particular, sometimes
one wants both a verbose multi-line printing format, used for displaying a single object in the
REPL and other interactive environments, and also a more compact single-line format used for
[`print()`](@ref) or for displaying the object as part of another object (e.g. in an array). Although
by default the `show(io, z)` function is called in both cases, you can define a *different* multi-line
format for displaying an object by overloading a three-argument form of `show` that takes the
`text/plain` MIME type as its second argument (see [Multimedia I/O](@ref)), for example:

```jldoctest polartype
julia> Base.show(io::IO, ::MIME"text/plain", z::Polar{T}) where{T} =
           print(io, "Polar{$T} complex number:\n   ", z)
```

(Note that `print(..., z)` here will call the 2-argument `show(io, z)` method.) This results in:

```jldoctest polartype
julia> Polar(3, 4.0)
Polar{Float64} complex number:
   3.0 * exp(4.0im)

julia> [Polar(3, 4.0), Polar(4.0,5.3)]
2-element Array{Polar{Float64},1}:
 3.0 * exp(4.0im)
 4.0 * exp(5.3im)
```

where the single-line `show(io, z)` form is still used for an array of `Polar` values.   Technically,
the REPL calls `display(z)` to display the result of executing a line, which defaults to `show(STDOUT, MIME("text/plain"), z)`,
which in turn defaults to `show(STDOUT, z)`, but you should *not* define new [`display()`](@ref)
methods unless you are defining a new multimedia display handler (see [Multimedia I/O](@ref)).

Moreover, you can also define `show` methods for other MIME types in order to enable richer display
(HTML, images, etcetera) of objects in environments that support this (e.g. IJulia).   For example,
we can define formatted HTML display of `Polar` objects, with superscripts and italics, via:

```jldoctest polartype
julia> Base.show(io::IO, ::MIME"text/html", z::Polar{T}) where {T} =
           println(io, "<code>Polar{$T}</code> complex number: ",
                   z.r, " <i>e</i><sup>", z.Θ, " <i>i</i></sup>")
```

A `Polar` object will then display automatically using HTML in an environment that supports HTML
display, but you can call `show` manually to get HTML output if you want:

```jldoctest polartype
julia> show(STDOUT, "text/html", Polar(3.0,4.0))
<code>Polar{Float64}</code> complex number: 3.0 <i>e</i><sup>4.0 <i>i</i></sup>
```

```@raw html
<p>An HTML renderer would display this as: <code>Polar{Float64}</code> complex number: 3.0 <i>e</i><sup>4.0 <i>i</i></sup></p>
```

As a rule of thumb, the single-line `show` method should print a valid Julia expression for creating
the shown object.  When this `show` method contains infix operators, such as the multiplication
operator (`*`) in our single-line `show` method for `Polar` above, it may not parse correctly when
printed as part of another object.  To see this, consider the expression object (see [Program
representation](@ref)) which takes the square of a specific instance of our `Polar` type:

```jldoctest polartype
julia> a = Polar(3, 4.0)
Polar{Float64} complex number:
   3.0 * exp(4.0im)

julia> print(:($a^2))
3.0 * exp(4.0im) ^ 2
```

Because the operator `^` has higher precedence than `*` (see [Operator Precedence](@ref)), this
output does not faithfully represent the expression `a ^ 2` which should be equal to `(3.0 *
exp(4.0im)) ^ 2`.  To solve this issue, we must make a custom method for `Base.show_unquoted(io::IO,
z::Polar, indent::Int, precedence::Int)`, which is called internally by the expression object when
printing:

```jldoctest polartype
julia> function Base.show_unquoted(io::IO, z::Polar, ::Int, precedence::Int)
           if Base.operator_precedence(:*) <= precedence
               print(io, "(")
               show(io, z)
               print(io, ")")
           else
               show(io, z)
           end
       end

julia> :($a^2)
:((3.0 * exp(4.0im)) ^ 2)
```

The method defined above adds parentheses around the call to `show` when the precedence of the
calling operator is higher than or equal to the precedence of multiplication.  This check allows
expressions which parse correctly without the parentheses (such as `:($a + 2)` and `:($a == 2)`) to
omit them when printing:

```jldoctest polartype
julia> :($a + 2)
:(3.0 * exp(4.0im) + 2)

julia> :($a == 2)
:(3.0 * exp(4.0im) == 2)
```

## "Value types"

In Julia, you can't dispatch on a *value* such as `true` or `false`. However, you can dispatch
on parametric types, and Julia allows you to include "plain bits" values (Types, Symbols, Integers,
floating-point numbers, tuples, etc.) as type parameters.  A common example is the dimensionality
parameter in `Array{T,N}`, where `T` is a type (e.g., [`Float64`](@ref)) but `N` is just an `Int`.

You can create your own custom types that take values as parameters, and use them to control dispatch
of custom types. By way of illustration of this idea, let's introduce a parametric type, `Val{x}`,
and a constructor `Val(x) = Val{x}()`, which serves as a customary way to exploit this technique
for cases where you don't need a more elaborate hierarchy.

`Val` is defined as:

```jldoctest valtype
julia> struct Val{x}
       end
Base.@pure Val(x) = Val{x}()
```

There is no more to the implementation of `Val` than this.  Some functions in Julia's standard
library accept `Val` instances as arguments, and you can also use it to write your own functions.
 For example:

```jldoctest valtype
julia> firstlast(::Val{true}) = "First"
firstlast (generic function with 1 method)

julia> firstlast(::Val{false}) = "Last"
firstlast (generic function with 2 methods)

julia> firstlast(Val(true))
"First"

julia> firstlast(Val(false))
"Last"
```

For consistency across Julia, the call site should always pass a `Val`*instance* rather than using
a *type*, i.e., use `foo(Val(:bar))` rather than `foo(Val{:bar})`.

It's worth noting that it's extremely easy to mis-use parametric "value" types, including `Val`;
in unfavorable cases, you can easily end up making the performance of your code much *worse*.
 In particular, you would never want to write actual code as illustrated above.  For more information
about the proper (and improper) uses of `Val`, please read the more extensive discussion in [the performance tips](@ref man-performance-tips).

## [Nullable Types: Representing Missing Values](@id man-nullable-types)

In many settings, you need to interact with a value of type `T` that may or may not exist. To
handle these settings, Julia provides a parametric type called [`Nullable{T}`](@ref), which can be thought
of as a specialized container type that can contain either zero or one values. `Nullable{T}` provides
a minimal interface designed to ensure that interactions with missing values are safe. At present,
the interface consists of several possible interactions:

  * Construct a `Nullable` object.
  * Check if a `Nullable` object has a missing value.
  * Access the value of a `Nullable` object with a guarantee that a [`NullException`](@ref)
    will be thrown if the object's value is missing.
  * Access the value of a `Nullable` object with a guarantee that a default value of type
    `T` will be returned if the object's value is missing.
  * Perform an operation on the value (if it exists) of a `Nullable` object, getting a
    `Nullable` result. The result will be missing if the original value was missing.
  * Performing a test on the value (if it exists) of a `Nullable`
    object, getting a result that is missing if either the `Nullable`
    itself was missing, or the test failed.
  * Perform general operations on single `Nullable` objects, propagating the missing data.

### Constructing [`Nullable`](@ref) objects

To construct an object representing a missing value of type `T`, use the `Nullable{T}()` function:

```jldoctest
julia> x1 = Nullable{Int64}()
Nullable{Int64}()

julia> x2 = Nullable{Float64}()
Nullable{Float64}()

julia> x3 = Nullable{Vector{Int64}}()
Nullable{Array{Int64,1}}()
```

To construct an object representing a non-missing value of type `T`, use the `Nullable(x::T)`
function:

```jldoctest
julia> x1 = Nullable(1)
Nullable{Int64}(1)

julia> x2 = Nullable(1.0)
Nullable{Float64}(1.0)

julia> x3 = Nullable([1, 2, 3])
Nullable{Array{Int64,1}}([1, 2, 3])
```

Note the core distinction between these two ways of constructing a `Nullable` object:
in one style, you provide a type, `T`, as a function parameter; in the other style, you provide
a single value of type `T` as an argument.

### Checking if a `Nullable` object has a value

You can check if a `Nullable` object has any value using [`isnull()`](@ref):

```jldoctest
julia> isnull(Nullable{Float64}())
true

julia> isnull(Nullable(0.0))
false
```

### Safely accessing the value of a `Nullable` object

You can safely access the value of a `Nullable` object using [`get()`](@ref):

```jldoctest
julia> get(Nullable{Float64}())
ERROR: NullException()
Stacktrace:
 [1] get(::Nullable{Float64}) at ./nullable.jl:92

julia> get(Nullable(1.0))
1.0
```

If the value is not present, as it would be for `Nullable{Float64}`, a [`NullException`](@ref)
error will be thrown. The error-throwing nature of the `get()` function ensures that any
attempt to access a missing value immediately fails.

In cases for which a reasonable default value exists that could be used when a `Nullable`
object's value turns out to be missing, you can provide this default value as a second argument
to `get()`:

```jldoctest
julia> get(Nullable{Float64}(), 0.0)
0.0

julia> get(Nullable(1.0), 0.0)
1.0
```

!!! tip
    Make sure the type of the default value passed to `get()` and that of the `Nullable`
    object match to avoid type instability, which could hurt performance. Use [`convert()`](@ref)
    manually if needed.

### Performing operations on `Nullable` objects

`Nullable` objects represent values that are possibly missing, and it
is possible to write all code using these objects by first testing to see if
the value is missing with [`isnull()`](@ref), and then doing an appropriate
action. However, there are some common use cases where the code could be more
concise or clear by using a higher-order function.

The [`map`](@ref) function takes as arguments a function `f` and a `Nullable` value
`x`. It produces a `Nullable`:

 - If `x` is a missing value, then it produces a missing value;
 - If `x` has a value, then it produces a `Nullable` containing
   `f(get(x))` as value.

This is useful for performing simple operations on values that might be missing
if the desired behaviour is to simply propagate the missing values forward.

The [`filter`](@ref) function takes as arguments a predicate function `p`
(that is, a function returning a boolean) and a `Nullable` value `x`.
It produces a `Nullable` value:

 - If `x` is a missing value, then it produces a missing value;
 - If `p(get(x))` is true, then it produces the original value `x`;
 - If `p(get(x))` is false, then it produces a missing value.

In this way, `filter` can be thought of as selecting only allowable
values, and converting non-allowable values to missing values.

While `map` and `filter` are useful in specific cases, by far the most useful
higher-order function is [`broadcast`](@ref), which can handle a wide variety of cases,
including making existing operations work and propagate `Nullable`s. An example
will motivate the need for `broadcast`. Suppose we have a function that computes the
greater of two real roots of a quadratic equation, using the quadratic formula:

```jldoctest nullableroot
julia> root(a::Real, b::Real, c::Real) = (-b + √(b^2 - 4a*c)) / 2a
root (generic function with 1 method)
```

We may verify that the result of `root(1, -9, 20)` is `5.0`, as we expect,
since `5.0` is the greater of two real roots of the quadratic equation.

Suppose now that we want to find the greatest real root of a quadratic
equations where the coefficients might be missing values. Having missing values
in datasets is a common occurrence in real-world data, and so it is important
to be able to deal with them. But we cannot find the roots of an equation if we
do not know all the coefficients. The best solution to this will depend on the
particular use case; perhaps we should throw an error. However, for this
example, we will assume that the best solution is to propagate the missing
values forward; that is, if any input is missing, we simply produce a missing
output.

The `broadcast()` function makes this task easy; we can simply pass the
`root` function we wrote to `broadcast`:

```jldoctest nullableroot
julia> broadcast(root, Nullable(1), Nullable(-9), Nullable(20))
Nullable{Float64}(5.0)

julia> broadcast(root, Nullable(1), Nullable{Int}(), Nullable{Int}())
Nullable{Float64}()

julia> broadcast(root, Nullable{Int}(), Nullable(-9), Nullable(20))
Nullable{Float64}()
```

If one or more of the inputs is missing, then the output of
`broadcast()` will be missing.

There exists special syntactic sugar for the `broadcast()` function
using a dot notation:

```jldoctest nullableroot
julia> root.(Nullable(1), Nullable(-9), Nullable(20))
Nullable{Float64}(5.0)
```

In particular, the regular arithmetic operators can be `broadcast()`
conveniently using `.`-prefixed operators:

```jldoctest
julia> Nullable(2) ./ Nullable(3) .+ Nullable(1.0)
Nullable{Float64}(1.66667)
```
