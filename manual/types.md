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

¿A qué propósito sirven los tipos abstractos paramétricos como `Pointy`? Considere si creamos una implementación tipo punto que sólo necesita una coordenada debido a que el punto se encuentra en la diagonal del primer cuadrante (*y = x*):

```jldoctest pointytype
julia> struct DiagPoint{T} <: Pointy{T}
           x::T
       end
```

Ahora tanto `Point{Float64}` como `DiagPoint{Float64}` son implementaciones de la abstracción `Pointy{Float64}` y, similarmente, para cada otra posible elección del tipo `T`. Esto permite programar a un interfaz común compartido por todos los objetos `Pointy`, implementedo tanto por `Point`como por `DiagPoint`. Esto no puede ser totalmente demostrado, sin embargo, hasta que no hayamos introducidos los métodos y el despacho en la siguiente sección, [Methods](@ref).

Hay situaciones donde puede no tener sentido para los parámetros de tipo varíen libremente sobre todos los tipos posibles. En tales situaciones, uno puede restringir el rango de `T` como aquí:

```jldoctest realpointytype
julia> abstract type Pointy{T<:Real} end
```

Con tal declaración, es aceptable usar cualquier tipo que sea un subtipo de [`Real`](@ref) en lugar de `T`, pero no los tipos que no sea subtipos de `Real`:

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

Los parámetros de tipo para tipos compuestos paramétricos pueden ser restringidos de la misma manera:

```julia
struct Point{T<:Real} <: Pointy{T}
    x::T
    y::T
end
```

Para dar un ejemplo del mundo real de cómo toda esta maquinaria de tipos paramétricos puede ser útil, he aquí la definición actual del tipo inmutable `Rational` [`Rational`](@ref) de Julia (omitiendo el constructor por simplicidad), que representa una relacíon exacta de enteros:

```julia
struct Rational{T<:Integer} <: Real
    num::T
    den::T
end
```

Sólo tiene sentido tomar relaciones de valores enteros, por lo que el tipo parametrizado `T` está restringido a ser un subtipo de [`Integer`](@ref), y una razón de enteros representa un valor sobre la línea de los números reales, por lo que cualquier [`Rational`](@ref) es una instancia de la abstracción [`Real`](@ref).

### Tipos tupla

Las tuplas son una abstracción de los argumentos de una función (sin la propia función). Los aspectos salientes de los argumentos de una funcíon son su orden y sus tipos. Por tanto, un tipo tupla es muy similar a un tipo inmutable parametrizado donde cada parámetro es el tupo de un campo. Por ejemplo, un tipo tupla de dos elementos se parece al siguiente tipo inmutable:

```julia
struct Tuple2{A,B}
    a::A
    b::B
end
```

Sin embargo, hay tres diferencias clave:

* Los tipos tupla pueden tener cualquier número de parámetros.
* Los tipos tupla son *covariantes* en sus parámetros: `Tuple{Int}` es un subtipo of `Tuple{Any}`. Por tanto `Tuple{Any}` es considerado un tipo abstracto, y los tipos tupla son solo concretos si sus parámetros lo son.
* Las tuplas no tienen nombres de campo; los campos son sólo accedidos mediante índices.

Los valores tupla son escritos con paréntesis y comas. Cuando se construye una tupla, se genera un tipo tupla apropiado bajo demanda:

```jldoctest
julia> typeof((1,"foo",2.5))
Tuple{Int64,String,Float64}
```

Note las implicaciones de  covarianza:

```jldoctest
julia> Tuple{Int,AbstractString} <: Tuple{Real,Any}
true

julia> Tuple{Int,AbstractString} <: Tuple{Real,Real}
false

julia> Tuple{Int,AbstractString} <: Tuple{Real,}
false
```

Intuitivamente, esto corresponde al tipo de los argumentos de una función, siendo un subtipo de la signatura de la función (cuando la signatura se corresponde).

### Tipos Tupla Vararg

El último parámetro de un tipo tupla puede ser el tipo especial `Vararg`, que denota cualquier número de elementos arrastrados:

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

Notese que `Varags{T}` corresponde a cero o más elementos del tipo `T`. Los tipos tupla vararg se usan para representar los argumentos aceptados por los métodos vararg (ver [Funciones Vararg](@ref)).

El tipo `Vararg{T,N}` se corresponde a exactamente `N` elementos de tipo `T`. `NTuple{N,T}` es un alias conveniente para `Tuple{Vararg{T,N}}`, es decir, un tipo tupla conteniendo exactamente `N` elementos de tipo `T`.

#### [Tipos Singleton](@id man-singleton-types)

Hay una clase especial de tipo paramétrico abstracto que hay que mencionar aquí: los tipos singleton. Para cada tipo `T` el tipo singleton `Type{T}` es un tipo abstracto cuya única instancia es el objeto `T`.  Como la definición es un poco difícil de analizar, echemos un vistazo a los siguientes ejemplos:

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

En otras palabras, [`isa(A,Type{B})`](@ref) es `true` si y solo si `A` y `B` son el mismo objeto y este objeto es un tipo. Sin el parámetro, `Type` es simplemente un tipo abstracto que tiene como instancias todos los objetos tipo, incluyendo, por supuesto, los tipos singleton:

```jldoctest
julia> isa(Type{Float64}, Type)
true

julia> isa(Float64, Type)
true

julia> isa(Real, Type)
true
```

Cualquier objeto que no es un tipo no es una instancia de `Type`:

```jldoctest
julia> isa(1, Type)
false

julia> isa("foo", Type)
false
```

Hasta que discutamos loa [métodos paramétricos](@ref) y las [conversiones](@ref conversion-and-promotion), , es difícil explicar la utilidad de la construcción tipo singleton, pero abreviando, permite a uno especializar el comportamiento de una función sobre *valores* de un tipo específico. Esto es útil para escribir métodos (especialmente paramétricos) cuy ocomportamiento dependa de un tipo que es dado como un argumento explícito en lugar de implicado por el tipo de un o de sus argumentos.

Unos pocos lenguajes de programación tienen tipos singleton, incluyendo Haskell, Scala y Ruby. En uso general, el término "tipo singleton" se refiere a un tipo cuya única instancia es un solo valor. Este significado se aplicaa a los tipos singleton de Julia, pero con la advertencia de que sólo los objetos tipo tienen tipos singleton.

### Tipos primitivos paramétricos

Los tipos bits pueden ser declarados paramétricamente. Por ejemplo, los punteros son reprentados como tipos bits encajados que serían declarados en Julia de esta forma:

```julia
# 32-bit system:
primitive type Ptr{T} 32 end

# 64-bit system:
primitive type Ptr{T} 64 end
```

La característica ligeramente extraña de estas declaraciones comparadas con los tipos compuestos paramétricos típicos es que el parámetro de tipo `T` no se usa en la definición del propio tipo (es justo un *tag* abstracto, esencialmente definiendo una familia entera de tipos con idéntica estructura, diferenciada sólo por su parámetro de tipo. Por tanto `Ptr{Float}` y `Ptr{Int64}` son tipos distintos, incluso auntque ellos tengan representaciones idénticas. Y, por supuesto, todos los tipos puntero específicos son subtipo del tipo sombrilla `Ptr`:

```jldoctest
julia> Ptr{Float64} <: Ptr
true

julia> Ptr{Int64} <: Ptr
true
```

## Tipos UnionAll

Hemos dicho que un tipo paramétrico como `Ptr` actúa como un supertipo de todas sus instancias (Ptr{Int64} etc.). ¿Cómo funciona esto? `Ptr` en si mismo no puede ser un tipo normal, ya que sin saber el tipo de los datos referenciados el tipo claramente no puede ser usado para operaciones en memoria. La respuesta es que `Ptr` (u otros tipos paramétricos como `Array`) es una clase diferente de tipo llamado `UnionAll`. Tal tipo expresa la unión iterada de tipos para todos los valores de algún parámetro.

Los tipos `UnionAll`suelen ser escritos usando la palabra clave `where`. Por ejemplo, `Ptr` podría ser escrito de forma más exacta como `Ptr{T} where T`, lo que significa que todos los valores cuyo tipo es `Ptr{T}` para algún valor de `T`. En este contexto, el parámetro `T` suele llamarse también una "variable tipo", ya que es como una variable que se extiende sobre los tipos. Cada `where` introduce una sola variable tipo, por lo que estas expresiones están anidadas para tipos con múltiples parámetros, por ejemplo `Array{T,N} where N where T`.

La sintaxis de la aplicación de tipo `A{B, C}` requiere que `A` sea un tipo `UnionAll` y primero sustituye `B`  por la variable de tipo más externa en `A`. Se espera que el resultado sea otro tipo `UnionAll` en el cuál `C` será sustituido. Por tanto, `A{B,C}` es equivalente a  `A{B}{C}`. Esto explica por qué es posible instanciar parcialmente un tipo, como en `Array{Float64}`: El primer valor de parámetro ha sido fijado, pero el segundo aún se extiende sobre todos os posibles valores. Usando explícitamente la sintaxis `where`, cualquier subconjunto de parámetros puede ser fijado. Por ejemplo, el tipo de todos los arrays unidimensionales puede ser escrito como `Array{T,1} where T`.

Las variables de tipo pueden ser restringidas con relaciones de subtipos. `Array{T} where T<:Integer` se refiere a todos los arrays cuyo elemento de tipo es alguna clase de [`Integer`](@ref). La sintaxis `Array{<:Integer}` es una abreviatura conveniente para `Array{T} where T<:Integer`. Las variables tipo pueden tener tanto límites superiores como inferiores. `Array{T} where Int<:T<:Number` se refiere a todos los arrays de [`Number`](@ref)s que son capaces de contener `Int`s (dado que T debe ser al menos tan grande como `Int`). La sintaxis `where T>:Int` también funciona para especificar sólo el límite inferior de una variable de tipo, y `Array{>:Int}` es  equivalente a `Array{T} where T>:Int`.

En el caso de expresiones `where` anidadas, los límites de la variable de tipo  pueden referirse a las variables de tipo más externas. Por ejemplo, `Tuple{T,Array{S}} where S<:AbstractArray{T} where T<:Real` se refiere a dos tuplas cuyo primer elemento es algo [`Real`](@ref) y cuyo segundo elemento es un `Array` de cualquier clase cuyo tipo de elemento contenga el tipo del primero elmento de la tupla.

La palabra clave `where` en si misma puede ser anidada dentro de una declaración más compleja. Por ejemplo, considere los dos tipos creados por las siguientes declaraciones:

```jldoctest
julia> const T1 = Array{Array{T,1} where T, 1}
Array{Array{T,1} where T,1}

julia> const T2 = Array{Array{T,1}, 1} where T
Array{Array{T,1},1} where T
```

El tipo `T1` define un array unidimensional de arrays unidimensionales; cada uno de los arrays internos consta de objetos del mismo tipo, pero este tipo puede variar de un array interno al siguiente. Por otra parte, el tipo `T2` define un array unidimensional de arrays unidimensionales de manera que los arrays internos tienen todos el mismo tipo. Notese que `T2` es un tipo abstracto, es decir, `Array{Array{Int,1},1} <: T2`, mientras que `T1` es un tipo concreto. Como consecuencia, `T1` puede ser construido con un constructor de cero argumentos `a=T1()` pero `T2` no puede.

Hay una sintaxis conveniente para nombrar tales tipos, similar a la forma corta de la sintaxis de definición de función:

```julia
Vector{T} = Array{T,1}
```

Esto es equivalente a const Vector = Array{T,1} where T. Escribir Vector{Float64} es equivalente a escribir Array{Float64,1}, y el tipo paraguas Vector tiene como instancias todos los objetos Array donde el segundo parámetro (el número de dimensiones del array) es uno, sin importar cuál es el tipo del elemento. En lenguajes donde los tipos paramétricos deben siempre ser especificados por completo, esto no suele ser de ayuda, pero en Julia, esto permite a uno escribir justo Vector para el tipo abstract incluyendo arrays densos unidimensionales de cualquier tipo de elementos.

## Aliases de Tipos

Algunas veces es conveniente introducir un nuevo nombre para un tipo ya expresable. Para tales ocasiones, Julia proporciona el mecanismo `typealias`. Por ejemplo, `UInt` es un alias de [`UInt32`](@ref) o [`UInt64`](@ref) dependiendo de los punteros de tamaño del sistema:

```julia-repl
# 32-bit system:
julia> UInt
UInt32

# 64-bit system:
julia> UInt
UInt64
```

Esto se consigue via el siguiente código en `base/boot.jl`:

```julia
if Int === Int64
    const UInt = UInt64
else
    const UInt = UInt32
end
```

Por supuesto, esto depende de a qué representa el alias `Int` si a [`Int32`](@ref) o a [`Int64`](@ref).

(Note que, a diferencia de `Int`, `Float` no existe como un alias de tipo para un tamaño específico de [`AbstractFloat`](@ref). A diferencia de con los registros enteros, los tamaños de los registros en punto flotante están especificados por el estándar IEEE-754 standard. Mientras que el tamaño de `Int` refleja el tamaño de un puntero nativo de esta máquina.)

## Operaciones sobre tipos

Como los tipos en Julia son objetos en sí mismos, las funcoines ordinarias pueden operar sobre ellos. Algunas funciones que son particularmente útiles para trabajar con o explorar tipos han sido ya introducidas. Por ejemplo, el operador `<:` que indica si el operando a su izquierda es un subtipo del operando a su derecha.

La función [`isa`](@ref) comprueba si un objeto es de un tipo dado y devuelve `true` o `false`:

```jldoctest
julia> isa(1, Int)
true

julia> isa(1, AbstractFloat)
false
```

La función [`typeof()`](@ref) , ya usada a través del manual en ejemplos, devuelve el tupo de su argumento. Como, con se notó anteriormente, los tipos son objetos, ellos también tienen tipos, y podemos preguntar cuáles son sus tipos:

```jldoctest
julia> typeof(Rational{Int})
DataType

julia> typeof(Union{Real,Float64,Rational})
DataType

julia> typeof(Union{Real,String})
Union
```

¿Qué pasa si repetimos el proceso? ¿Cuál es el tipo de un tipo de un tipo? Como sucede, los tipos son todos valores compuestos y por tanto todos tendrán un tipo de `DataType`:

```jldoctest
julia> typeof(DataType)
DataType

julia> typeof(Union)
DataType
```

`DataType` is its own type.

Otra operación que se aplica a algunos tipos es  [`supertype()`](@ref), que revela el supertipo de un tipo. Sólo los tipos declarados (`DataType`) tienen supertipos no ambiguos:

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

Si aplicamos [`supertype()`](@ref) a otros objetos tipo (u objetos no tipo) se lanzará un [`MethodError`](@ref):

```jldoctest
julia> supertype(Union{Float64,Int64})
ERROR: MethodError: no method matching supertype(::Type{Union{Float64, Int64}})
Closest candidates are:
  supertype(!Matched::DataType) at operators.jl:41
  supertype(!Matched::UnionAll) at operators.jl:46
```

## Custom pretty-printing

Frecuentemente, uno quiere personalizar cómo se mostrarán las instancias de un tipo. Esto se consigue sobrecargando la función [`show()`](@ref). Por ejemplo, supongamos que definimos un tipo para representar número complejos en forma polar:

```jldoctest polartype
julia> struct Polar{T<:Real} <: Number
           r::T
           Θ::T
       end

julia> Polar(r::Real,Θ::Real) = Polar(promote(r,Θ)...)
Polar
```

Aquí, hemos añadido una función constructor personalizado para que pueda tomar argumentos de distinto tipos [`Real`](@ref) y los promocione a un tipo común (ver [Constructores](@ref man-constructors) y [Conversión y Promoción](@ref conversion-and-promotion)). (Por supuesto, tendríamos que definir montones de otros métodos también, para hacer que actíe como un [`Number`](@ref), por ejemplo, `+`, `*`, `one`, `zero`, reglas de promoción y otras cosas). Por defecto, las instancias de este tipo se muestran de forma bastante simple, con información sobre el nombre del tipo y los valores de los campos, como por ejemplo en  `Polar{Float64}(3.0,4.0)`.

Si en lugar de usar este modo de presentacin preferimos que su presentación sea `3.0 * exp(4.0im)`, hay que definir el siguiente método para que imprima el objeto a un objeto de salida `io`dado (que representa un fichero, terminal, buffer, etc.; ver [Networking and Streams](@ref)):

```jldoctest polartype
julia> Base.show(io::IO, z::Polar) = print(io, z.r, " * exp(", z.Θ, "im)")
```

Es posible un control de grano más fino sobre la visualizacin de los objetos `Polar`. En particular, algunas veces uno desea 
un formato de impresión detallado multilínea, utilizado para mostrar un solo objeto en REPL y otros entornos interactivos, y también un formato de línea única más compacto utilizado para [`print ()`] @ref) o para mostrar el objeto como parte de otro objeto (por ejemplo, en una matriz). Aunque de forma predeterminada se llama a la función `show (io, z)` en ambos casos, puede definir un formato multilínea *diferente* para mostrar un objeto sobrecargando una forma de tres argumentos de `show` que toma el tipo MIME `text/plain` como su segundo argumento (consulte [E/S multimedia](@ref)), por ejemplo:

```jldoctest polartype
julia> Base.show(io::IO, ::MIME"text/plain", z::Polar{T}) where{T} =
           print(io, "Polar{$T} complex number:\n   ", z)
```

(Note que `print(..., z)` aquí invocará al método con dos argumentos `show(io, z)`). Esto dará como resultado:

```jldoctest polartype
julia> Polar(3, 4.0)
Polar{Float64} complex number:
   3.0 * exp(4.0im)

julia> [Polar(3, 4.0), Polar(4.0,5.3)]
2-element Array{Polar{Float64},1}:
 3.0 * exp(4.0im)
 4.0 * exp(5.3im)
```

donde se sigue utilizando la forma de línea `show(io, z)` para un array de valores  `Polar`. Técnicamente, el REPL llama a  `display(z)` para mostrar el resultado de ejecutar una línea que por defecto es `show (STDOUT, MIME (" text / plain "), z)`,
 que a su vez por defecto es `show (STDOUT, z) `, pero debe *no* definir nuevos métodos [` display () `] (@ ref) a menos que esté definiendo un nuevo controlador de pantalla multimedia (consulte [E/S multimedia] (@ref)).

Además, también puede definir métodos `show` para otros tipos MIME para permitir una visualización más rica (HTML, imágenes, etc.) de los objetos en entornos que lo admitan (por ejemplo, IJulia). Por ejemplo, podemos definir la visualización HTML formateada de objetos `Polar`, con superíndices y cursiva, a través de:

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

## "Valores tipo"

En Julia uno no puede despachar sobre un *valor*  tal como `true` o `false`. Sin embargo, se se puede despachar sobre tipos paramétricos, y Julia  permite incluir valores "plain bits" (tipos, símbolos, enteros, números en punto flotante, tuplas, etc.) como parámetros de tipo. Un ejemplo común es el parámetro de dimensionalidad en `Array{T,N}`, donde `T` es un tipo (por ejemplo,  [`Float64`](@ref)) pero `N` es un `Int`.

Podemos crear nuestros propio tipos personalizados que tomen valores como parámetros, y usarlos para controlar el despacho de los tipos personalizados. A modo de ilustración de esta idea, introduzcamos un tipo paramétrico, `Val{T}` que sirve como una forma tradicional de explotar esta técnica para casos donde tu no necesitas una jerarquía más elaborada.

`Val` es definida como:

```jldoctest valtype
julia> struct Val{x}
       end
Base.@pure Val(x) = Val{x}()
```

No hay más implementaciones de `Val` que esta. Algunas funciones en la librería estándar de Julia aceptan los tipos `Val` como argumentos, y uno también puede usarlos para escribir sus propias funciones. Por ejemplo:

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

Por consistencia con Julia, el sitio de llamada debería siempre pasar un tipo `Val` en lugar de crear una instancia, por ejemplo, usar `foo(Val{:bar})` en lugar de `foo(Val{:bar}())`.

Vale la pena señalar que es extremadamente sencillo usar mal los tipos "valor" paramétricos, incluyendo `Val`; en caso desfavorables, tu puedes fácilmente acabar haciendo el rendimiento de tu código mucho *peor*. En particular, tu nunca tendrás que querer escribir código actual como ilustramos anteriormente. Para má información sobre el uso apropiado de `Val` consulte por fabor la discusión más extensa en los [consejos de rendimiento](@ref man-performance-tips).

## [Tipos `Nullable`: representando valores perdidos](@id man-nullable-types)

En muchas situaciones, uno necesita interactuar con un valor de tipo `T` que puede o no existir. Para manejar estas situaciones, Julia proporciona un tipo paramétrico denominado [`Nullable{T}`](@ref) que puede ser pensado como un tipo contenedor especializado que puede contener cero o un valores. `Nullable{T}` proporciona una interfaz mínima diseñada para asegurar qué interacciones con valores perdidos son seguras. En la actualidad, la interfaz consiste en varias posibles interacciones:

  * Construir un objeto `Nullable`.
  * Comprobar si un objeto `Nullable` tiene un valor perdido.
  * Acceder al valor de un objeto `Nullable` con la garantía de que se lanzará una [`NullException`](@ref) si el valor
    del objeto está perdido.
  * Acceder al valor de un objeto `Nullable` con una garantía de que el valor por defecto del tipo `T` será devuelto 
    si el valor del objeto se pierde.
  * Realizar una operación sobre el valor (si existe) de un objeto `Nullable`, obteniendo un resultado` Nullable`. 
    El resultado faltará si falta el valor original.
  * Realizar una prueba sobre el valor (si existe) de un objeto `Nullable`, obteniendo un resultado que será perdido
    si faltaba el` Nullable` o si la prueba falla.
  * Realizar operaciones generales en objetos individuales `Nullable`, propagando los datos faltantes.
   
### Construyendo objetos [`Nullable`](@ref)

Para construir un objeto representando un valor perdido de tipo `T`, use la siguiente función `Nullable{T}():

```jldoctest
julia> x1 = Nullable{Int64}()
Nullable{Int64}()

julia> x2 = Nullable{Float64}()
Nullable{Float64}()

julia> x3 = Nullable{Vector{Int64}}()
Nullable{Array{Int64,1}}()
```

Para construir un objeto representando un valor no perdido de tipo `T`, use la función `Nullable(x::T)`:

```jldoctest
julia> x1 = Nullable(1)
Nullable{Int64}(1)

julia> x2 = Nullable(1.0)
Nullable{Float64}(1.0)

julia> x3 = Nullable([1, 2, 3])
Nullable{Array{Int64,1}}([1, 2, 3])
```

Note que la distinción clave entre estas dos formas de construir un objeto `Nullable`: en un estilo, tu proporcionas un tipo, `T` como un parámetro a función; en el otro estilo, tu proporcionas un solo valor de tipo `T` como argumento.

###  Comprobar si un objeto `Nullable` tiene un valor

Puedes comprobbar si un objeto `Nullable` tiene algún valor usando [`isnull()`](@ref):

```jldoctest
julia> isnull(Nullable{Float64}())
true

julia> isnull(Nullable(0.0))
false
```

### Acceder de forma segura al valor de un objeto `Nullable`

Puedes acceder al valor de un objeto `Nullable` usando  [`get()`](@ref):

```jldoctest
julia> get(Nullable{Float64}())
ERROR: NullException()
Stacktrace:
 [1] get(::Nullable{Float64}) at ./nullable.jl:92

julia> get(Nullable(1.0))
1.0
```

Si el valor no está presente, como podría ser para un `Nullable{Float64}` se lanzará un error  [`NullException`](@ref). La naturaleza del error lanzado de la función `get()` asegur aque cualquier intento de acceder al valor perdido falle inmediatamente.

En los casos para los cuales existe un valor por defecto razonable que podría ser usando cuando el valor de los objetos `Nullable` se volviera perdido, uno puede proporcionar este valos por defecto como un segundo argumento a `get()`:

```jldoctest
julia> get(Nullable{Float64}(), 0.0)
0.0

julia> get(Nullable(1.0), 0.0)
1.0
```

!!! tip
    Asegúrese de que el tipo de valor predeterminado pasado a `get ()` y el del objeto `Nullable` coincidan para evitar 
    la inestabilidad de tipo, lo que podría perjudicar el rendimiento. Utilice [`convert ()`] (@ ref) manualmente si 
    es necesario.

### Realizando operaciones sobre objetos `Nullable`

Los objetos `Nullable` representan valores que están posiblemente perdidos, y es posible escribir todo el código usando estos objetos primero comprobando para ver si el valor está perdido con [`isnull()`](@ref), y luego realizar la accin apropiada. Sin embargo, hay algunos casos de uso comunes donde el código podría ser ms conciso o claro usando una función de orden superior.

La función [`map`](@ref) toma como argumentos una función `f` y un valor `x` de tipo `Nullable`. Ella produce un `Nullable`:

 - Si `x` es un valor perdido, entonces produce un valor perdido;
 - Si `x` tiene un valor, entonces produce un objeto `Nullable` que contiene `f(get(x))` como valor.

Esto es útil para realizar operaciones simples sobre valores que podrían estar perdidos si el comportamiento deseado es simplemente propagar hacia adelante los valores perdidos.

La función [`filter`](@ref) toma como argumentos una función predicado `p` (es decir, una función que devuelve un boolean) y un valor `x` de tipo `Nullable`. Ella produce un `Nullable`:

 - Si `x` es un valor perdido, entonces produce un valor perdido;
 - Si `p(get(x))` es true, entoces produce el valor original `x`;
 - Si `p(get(x))` es false, entonces produce un valor perdido.

De esta forma, `filter` puede ser considerado como seleccionar sólo valores permisibles, y convertir valores no permisibles en valores perdidos.

Mientras que `map` y `filter` son útiles para casos específicos, la función de orden superior más útil es, con diferencia, [`broadcast`](@ref), que puede manejar una amplia variedad de casos, incluyendo hacer operaciones existentes funcionen y propaguen `Nullable`s. El siguiente ejemplo motivará la necesidad de `broadcast`. Supongamos que tenemos una funcion que calcula la mayor de las dos raices reales de una ecuacion cuadratica, usando la formula cuadratica:

```jldoctest nullableroot
julia> root(a::Real, b::Real, c::Real) = (-b + √(b^2 - 4a*c)) / 2a
root (generic function with 1 method)
```

Podemos verificar que el resultado de `root (1, -9, 20)` es `5.0` como esperamos,
ya que `5.0` es la mayor de dos raíces reales de la ecuación cuadrática.

Supongamos ahora que queremos encontrar la mayor raíz real de una ecuación cuadrática donde los coeficientes pueden ser valores perdidos. Tener valores perdidos en los conjuntos de datos es una ocurrencia común en los datos del mundo real, por lo que es importante poder tratar con ellos. Pero no podemos encontrar las raíces de una ecuación si no conocemos todos los coeficientes. La mejor solución para esto dependerá del caso de uso particular; quizás deberíamos arrojar un error. Sin embargo, para este ejemplo, asumiremos que la mejor solución es propagar los valores perdidos; es decir, si falta alguna entrada, simplemente producimos una salida faltante.

La función `broadcast ()` facilita esta tarea; simplemente podemos pasar la función `root` que escribimos a` broadcast`:

```jldoctest nullableroot
julia> broadcast(root, Nullable(1), Nullable(-9), Nullable(20))
Nullable{Float64}(5.0)

julia> broadcast(root, Nullable(1), Nullable{Int}(), Nullable{Int}())
Nullable{Float64}()

julia> broadcast(root, Nullable{Int}(), Nullable(-9), Nullable(20))
Nullable{Float64}()
```

Si faltan una o más de las entradas, faltará la salida de `broadcast ()`.

Existe un convenio sintáctico especial para la función `broadcast()` usando la notación punto:

```jldoctest nullableroot
julia> root.(Nullable(1), Nullable(-9), Nullable(20))
Nullable{Float64}(5.0)
```

En particular, los operadores aritméticos regulares pueden ser `broadcast ()` convenientemente usando operadores `.`-prefijo:

```jldoctest
julia> Nullable(2) ./ Nullable(3) .+ Nullable(1.0)
Nullable{Float64}(1.66667)
```
