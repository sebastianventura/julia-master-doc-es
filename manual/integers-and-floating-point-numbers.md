# [Números enteros y en punto flotante](@id integers-and-floating-point-numbers)

Los valores enteros y punto flotante son los bloques constructivos básicos de la aritmética y la computación. Las representaciones construidas para estos valores son denominadas *tipos primitivos*, mientras que las reprentacionesde números enteros y en punto flotante como valores inmediatos en código se conocen como *literales numéricos*. Por ejemplo, `1` es un literal entero, mientras que `1.0` es un literal en punto flotante; sus representaciones binarias en memoria como objetos son los tipos primitivos.

Julia proporciona un amplio rango de tipos primitivos numéricos, y un complemento complemento de operadores aritmétidos y de bits así como funciones matemáticas estándar definidas sobre ellos. Los operadores establecen una correspondencia enre los tipos numéricos y las operaciones que son soportadas de forma nativa sobre los ordenadores modernos, permitiendo a Julia sacar plena ventaja de los recursos computacionales. Además, Julia proporciona soporte software para *aritmética de precisión arbitraria* que puede manejar operaciones sobre valores numéricos que no puede ser representada de forma efectiva en representaciones hardware nativas, pero al coste de un rendimiento relativamente menor.

Los tipos primitivos de Julia son los siguientes:

  * **Integer types:**

| Type              | Signed? | Number of bits | Smallest value | Largest value |
|:----------------- |:------- |:-------------- |:-------------- |:------------- |
| [`Int8`](@ref)    | ✓       | 8              | -2^7           | 2^7 - 1       |
| [`UInt8`](@ref)   |         | 8              | 0              | 2^8 - 1       |
| [`Int16`](@ref)   | ✓       | 16             | -2^15          | 2^15 - 1      |
| [`UInt16`](@ref)  |         | 16             | 0              | 2^16 - 1      |
| [`Int32`](@ref)   | ✓       | 32             | -2^31          | 2^31 - 1      |
| [`UInt32`](@ref)  |         | 32             | 0              | 2^32 - 1      |
| [`Int64`](@ref)   | ✓       | 64             | -2^63          | 2^63 - 1      |
| [`UInt64`](@ref)  |         | 64             | 0              | 2^64 - 1      |
| [`Int128`](@ref)  | ✓       | 128            | -2^127         | 2^127 - 1     |
| [`UInt128`](@ref) |         | 128            | 0              | 2^128 - 1     |
| [`Bool`](@ref)    | N/A     | 8              | `false` (0)    | `true` (1)    |

  * **Floating-point types:**

| Type              | Precision                                                                      | Number of bits |
|:----------------- |:------------------------------------------------------------------------------ |:-------------- |
| [`Float16`](@ref) | [media](https://en.wikipedia.org/wiki/Half-precision_floating-point_format)     | 16             |
| [`Float32`](@ref) | [sencilla](https://en.wikipedia.org/wiki/Single_precision_floating-point_format) | 32             |
| [`Float64`](@ref) | [double](https://en.wikipedia.org/wiki/Double_precision_floating-point_format) | 64             |

Adicionalmente, se ha construído un soporte completo para [Números Complejos y Racionales](@ref) encima de estos tipos primitivos. Todos los tipos primitivos interoperan de forma natural sin tener que realizar conversiones específicas, gracias a un [sistema de promoción de tipos](@ref conversion-and-promotion) flexible y extensible por el usuario..

## Enteros

Los literales enteros se representan del modo estándar:

```jldoctest
julia> 1
1

julia> 1234
1234
```

El tipo por defecto para un literal entero depende de su el sistema objetivo tiene una aquitectura de 
32 o de 64 bits:

```julia-repl
# 32-bit system:
julia> typeof(1)
Int32

# 64-bit system:
julia> typeof(1)
Int64
```

La variable interna de Julia [`Sys.WORD_SIZE`](@ref) indica si el sistema objetivo es de 32 bits 
o de 64 bits:

```julia-repl
# 32-bit system:
julia> Sys.WORD_SIZE
32

# 64-bit system:
julia> Sys.WORD_SIZE
64
```

Julia también define los tipos `Int` y `UInt`, que son aliases para los tipos enteros nativos del sistema con y sin signo:

```julia-repl
# 32-bit system:
julia> Int
Int32
julia> UInt
UInt32

# 64-bit system:
julia> Int
Int64
julia> UInt
UInt64
```

Los enteros mayores que no pueden ser representados usando sólo 32 bits pero pueden ser representados en 64 bits se crean como enteros de 64 bits, independientemente del tipo que tenga el sistema por defecto:w

```jldoctest
# 32-bit or 64-bit system:
julia> typeof(3000000000)
Int64
```

Los enteros sin signo son introducidos y mostrados usando el prefijo `0x` y los dígitos hexadecimales `0-9a-f` (los dígitos capitalizados `A-F` también funcionan para la entrada). El tamaño de un valor sin signo está determinado por el número de dígitos hexadecimales usados:

```jldoctest
julia> 0x1
0x01

julia> typeof(ans)
UInt8

julia> 0x123
0x0123

julia> typeof(ans)
UInt16

julia> 0x1234567
0x01234567

julia> typeof(ans)
UInt32

julia> 0x123456789abcdef
0x0123456789abcdef

julia> typeof(ans)
UInt64
```

Este comportamiento está basado en la observación de que cuando uno usa literales hexadecimales sin signo para valores enteros, se los suele utilizar para representar una secuencia de bytes numéricos fijos en lugar de un valor entero.

Recuerde que la variable [`ans`](@ref) se establece en el valor de la última expresión evaluada en una sesión interactiva. Esto no ocurre cuando el código Julia se ejecuta de otras maneras.

Los literales binarios y octales también están soportados:

```jldoctest
julia> 0b10
0x02

julia> typeof(ans)
UInt8

julia> 0o10
0x08

julia> typeof(ans)
UInt8
```

Los valores máximo y mínimo de tipos primitivos numéricos representables como enteros vienen dados por las funciones  [`typemin()`](@ref) y [`typemax()`](@ref):

```jldoctest
julia> (typemin(Int32), typemax(Int32))
(-2147483648, 2147483647)

julia> for T in [Int8,Int16,Int32,Int64,Int128,UInt8,UInt16,UInt32,UInt64,UInt128]
           println("$(lpad(T,7)): [$(typemin(T)),$(typemax(T))]")
       end
   Int8: [-128,127]
  Int16: [-32768,32767]
  Int32: [-2147483648,2147483647]
  Int64: [-9223372036854775808,9223372036854775807]
 Int128: [-170141183460469231731687303715884105728,170141183460469231731687303715884105727]
  UInt8: [0,255]
 UInt16: [0,65535]
 UInt32: [0,4294967295]
 UInt64: [0,18446744073709551615]
UInt128: [0,340282366920938463463374607431768211455]
```

Los valores devueltos por [`typemin()`](@ref) y [`typemax()`](@ref) siempre son del tipo de argumento dado. (La expresión anterior utiliza varias características que todavía tenemos que introducir, incluyendo [blucles for](@ref man-loops),
[Cadenas](@ref man-strings), e [Interpolación](@ref), pero debería ser lo suficientemente fácil de entender para los usuarios con cierta experiencia en programación).)

### Comportamiento ante el Desbordamiento

En Julia, superar el valor máximo representable de un tipo dado da como resultado un comportamiento envolvente:

```jldoctest
julia> x = typemax(Int64)
9223372036854775807

julia> x + 1
-9223372036854775808

julia> x + 1 == typemin(Int64)
true
```

Así, la aritmética con enteros de Julia es en realidad una forma de [aritmética modular](https://en.wikipedia.org/wiki/Modular_arithmetic).
Esto refleja las características de la aritmética subyacente de números enteros tal como se implementa en las computadoras modernas. En aplicaciones donde es posible el desbordamiento, es esencial comprobar explícitamente el envolvente producido por el desbordamiento. De lo contrario, se recomienda el tipo [`BigInt`](@ref) en [Aritmética de Precisión Arbitraria](@ref).

### Errores de división

La división entera (la función `div`) tiene dos casos excepcionales: dividir por cero, y dividir 
el número  negativo más bajo  ([`typemin()`](@ref)) por -1. Ambos casos lanzan un 
[`DivideError`](@ref).
El resto y las funciones de módulo (`rem` y `mod`) lanzan un  [`DivideError`](@ref) cuando su 
segundo argumento es cero.

## Floating-Point Numbers

Los literales de números en punto flotante son representados en las formas estándar:

```jldoctest
julia> 1.0
1.0

julia> 1.
1.0

julia> 0.5
0.5

julia> .5
0.5

julia> -1.23
-1.23

julia> 1e10
1.0e10

julia> 2.5e-4
0.00025
```

Los resultados anteriores son todos valores [`Float64`](@ref). Los valores literales [`Float32`](@ref) pueden
introducirse escribiendo `f` en lugar de `e`:

```jldoctest
julia> 0.5f0
0.5f0

julia> typeof(ans)
Float32

julia> 2.5f-4
0.00025f0
```

Los valores pueden ser convertidos a `Float32` fácilmente:

```jldoctest
julia> Float32(-1.5)
-1.5f0

julia> typeof(ans)
Float32
```

También son válidos los literales de punto flotante en formato hexadecimal, pero sólo sobre valores [`Float64`](@ref):

```jldoctest
julia> 0x1p0
1.0

julia> 0x1.8p3
12.0

julia> 0x.4p-1
0.125

julia> typeof(ans)
Float64
```

También esta soportados los números en punto flotante de media precisión ([`Float16`](@ref)), pero sólo 
como un formato de almacenamiento. En los cálculos son convertidos a [`Float32`](@ref).

```jldoctest
julia> sizeof(Float16(4.))
2

julia> 2*Float16(4.)
Float16(8.0)
```

El guión bajo (*underscore*) puede usarse como separador de dígitos:

```jldoctest
julia> 10_000, 0.000_000_005, 0xdead_beef, 0b1011_0010
(10000, 5.0e-9, 0xdeadbeef, 0xb2)
```

### Cero en punto flotante

Los números en punto flotante tienen [dos ceros](https://en.wikipedia.org/wiki/Signed_zero), 
positivo y negativo. Ellos son iguales entre sí, pero tienen distintas representaciones, 
como puede verse si usamos la función `bits()`:

```jldoctest
julia> 0.0 == -0.0
true

julia> bits(0.0)
"0000000000000000000000000000000000000000000000000000000000000000"

julia> bits(-0.0)
"1000000000000000000000000000000000000000000000000000000000000000"
```

### Valores especiales en punto flotante

Hay tres valores especificados en el estándar de punto flotante para valores que no se corresponden 
con ningún punto en la línea de números reales:

| `Float16` | `Float32` | `Float64` | Name              | Description                                                     |
|:--------- |:--------- |:--------- |:----------------- |:--------------------------------------------------------------- |
| `Inf16`   | `Inf32`   | `Inf`     | positive infinity | a value greater than all finite floating-point values           |
| `-Inf16`  | `-Inf32`  | `-Inf`    | negative infinity | a value less than all finite floating-point values              |
| `NaN16`   | `NaN32`   | `NaN`     | not a number      | a value not `==` to any floating-point value (including itself) |

Para más información sobre cómo estos valores de punto flotante no finitos están ordenados entre 
sí y otros flotantes, vea [Comparaciones Numéricas](@ref). Mediante [estándar IEEE 754](https://en.wikipedia.org/wiki/IEEE_754-2008), estos valores de punto flotante son el resultado 
de ciertas operaciones aritméticas:

```jldoctest
julia> 1/Inf
0.0

julia> 1/0
Inf

julia> -5/0
-Inf

julia> 0.000001/0
Inf

julia> 0/0
NaN

julia> 500 + Inf
Inf

julia> 500 - Inf
-Inf

julia> Inf + Inf
Inf

julia> Inf - Inf
NaN

julia> Inf * Inf
Inf

julia> Inf / Inf
NaN

julia> 0 * Inf
NaN
```

Las funciones [`typemin()`](@ref) y [`typemax()`](@ref) también se aplican a los tipos en punto flotante:

```jldoctest
julia> (typemin(Float16),typemax(Float16))
(-Inf16, Inf16)

julia> (typemin(Float32),typemax(Float32))
(-Inf32, Inf32)

julia> (typemin(Float64),typemax(Float64))
(-Inf, Inf)
```

### Epsilon de máquina

La mayoría de los números reales no pueden representarse exactamente con números de coma flotante, 
por lo que para muchos propósitos es importante conocer la distancia entre dos números de punto 
flotante representables adyacentes, que a menudo se conoce como [epsilon de máquina](https://en.wikipedia.org/wiki/Machine_epsilon).

Julia proporciona [`eps()`](@ref), que da la distancia entre 1,0 y el siguiente valor de punto 
flotante representable más grande:

```jldoctest
julia> eps(Float32)
1.1920929f-7

julia> eps(Float64)
2.220446049250313e-16

julia> eps() # same as eps(Float64)
2.220446049250313e-16
```

Estos valores son `2.0^-23` y `2.0^-52` como valores [`Float32`](@ref) y [`Float64`](@ref), 
respectivamente. La función [`eps()`](@ref) también puede tomar un valor de punto flotante 
como un argumento y da la diferencia absoluta entre ese valor y el siguiente valor de punto 
flotante representable. Es decir, `eps(x)` produce un valor del mismo tipo que `x` tal que
`x` + `eps(x)` es el siguiente valor de punto flotante representable mayor que `x`:

```jldoctest
julia> eps(1.0)
2.220446049250313e-16

julia> eps(1000.)
1.1368683772161603e-13

julia> eps(1e-27)
1.793662034335766e-43

julia> eps(0.0)
5.0e-324
```

La distancia entre dos números de punto flotante representables adyacentes no es constante, 
pero es menor para valores más pequeños y mayor para valores mayores. En otras palabras, 
los números de punto flotante representables son más densos en la línea de números reales 
cerca de cero, y crecen exponencialmente dispersos a medida que uno se aleja de cero. Por
definición, `eps(1.0)` es el mismo que `eps(Float64)` ya que `1.0` es un valor de coma 
flotante de 64 bits.

Julia también proporciona las funciones [`nextfloat()`](@ref) y [`prevfloat()`](@ref) que 
devuelven el siguiente número de punto flotante representable más grande o más pequeño al 
argumento, respectivamente:

```jldoctest
julia> x = 1.25f0
1.25f0

julia> nextfloat(x)
1.2500001f0

julia> prevfloat(x)
1.2499999f0

julia> bits(prevfloat(x))
"00111111100111111111111111111111"

julia> bits(x)
"00111111101000000000000000000000"

julia> bits(nextfloat(x))
"00111111101000000000000000000001"
```

Este ejemplo resalta el principio general de que los números de punto flotante representables 
adyacentes también tienen representaciones binarias enteras adyacentes.

### Rounding modes

Si un número no tiene una representación de punto flotante exacta, debe redondearse a un valor 
representable apropiado. Sin embargo, si se desea, la forma en que se realiza este redondeo puede 
cambiarse de acuerdo con los modos de redondeo presentados en el 
[IEEE 754 standard](https://en.wikipedia.org/wiki/IEEE_754-2008).

```jldoctest
julia> x = 1.1; y = 0.1;

julia> x + y
1.2000000000000002

julia> setrounding(Float64,RoundDown) do
           x + y
       end
1.2
```

El modo predeterminado utilizado siempre es [`RoundNearest`](@ref), , que redondea al valor representable 
más cercano, con arcos redondeados hacia el valor más cercano con un bit menos significativo.

!!! warning
    El redondeo generalmente sólo es correcto para las funciones aritméticas básicas ([`+()`](@ref), 
    [`-()`](@ref), [`*()`](@ref), [`/()`](@ref) and [`sqrt()`](@ref)) y las operaciones de conversión 
    de tipos. Muchas otras funciones asumen que el modo por defecto [`RoundNearest`](@ref) está 
    establecido y pueden dar resultados erróneos al operar bajo otros modos de redondeo.

### Antecedentes y referencias

La aritmética de punto flotante supone muchas sutilezas que pueden sorprender a los usuarios que no 
están familiarizados con los detalles de implementación de bajo nivel. Sin embargo, estas sutilezas 
se describen en detalle en la mayoría de los libros sobre computación científica, y también en las 
siguientes referencias:

  * La guía definitiva para la aritmética de coma flotante es el estándar [IEEE 754-2008 (http://standards.ieee.org/findstds/standard/754-2008.html); Sin embargo, no está disponible 
  en línea gratis.
  * Para una presentación breve pero lúcida de cómo los números de punto flotante están 
  representados, vea el [artículo de John D. Cook](https://www.johndcook.com/blog/2009/04/06/anatomy-of-a-floating-point-number/) sobre el tema, así como su [introduction](https://www.johndcook.com/blog/2009/04/06/numbers-are-a-leaky-abstraction/)
    a algunas de las cuestiones que surgen de cómo esta representación difiere en el comportamiento de la abstracción idealizada de números reales.
    abstraction of real numbers.
  * También se recomienda la serie de [publicaciones de Bruce Dawson sobre números en punto flotante](https://randomascii.wordpress.com/2012/05/20/thats-not-normalthe-performance-of-odd-floats/).
  * Para un excelente y profundo análisis de los números de coma flotante y los problemas de precisión numérica encontrados al calcular con ellos, vea el artículo de David Goldberg [What Every Computer Scientist Should Know About Floating-Point Arithmetic](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.22.6768&rep=rep1&type=pdf).
  * Para una documentación aún más extensa de la historia de, la razón y las cuestiones con los números de punto flotante, así como la discusión de muchos otros temas en la computación numérica, ver los [escritos recolectados](https://people.eecs.berkeley.edu/~wkahan/)
    de [William Kahan](https://en.wikipedia.org/wiki/William_Kahan), comúnmente conocido como el "Padre de punto flotante". De interés particular puede ser [An Interview with the Old Man of Floating-Point](https://people.eecs.berkeley.edu/~wkahan/ieee754status/754story.html).

## Arbitrary Precision Arithmetic

Para permitir cálculos con enteros y números de coma flotante de precisión arbitraria, Julia envuelve la [Biblioteca Aritmética de Precisión Múltiple GNU (GMP)](https://gmplib.org) y la [biblioteca GNU MPFR](http://www.mpfr.org),
respectivamente. Los tipos [`BigInt`](@ref) y [`BigFloat`](@ref) están disponibles en Julia para números enteros de precisión arbitraria y números de coma flotante, respectivamente.

Existen constructores para crear estos tipos de tipos numéricos primitivos, y podemos también 
utilizar [`parse()`](@ref)
para construirlos a partir de `AbstractString`s.  Una vez creados, participan en la aritmética con todos los demás tipos numéricos gracias al [mecanismo de promotion y conversion de tipos](@ref conversion-and-promotion) de Julia:

```jldoctest
julia> BigInt(typemax(Int64)) + 1
9223372036854775808

julia> parse(BigInt, "123456789012345678901234567890") + 1
123456789012345678901234567891

julia> parse(BigFloat, "1.23456789012345678901")
1.234567890123456789010000000000000000000000000000000000000000000000000000000004

julia> BigFloat(2.0^66) / 3
2.459565876494606882133333333333333333333333333333333333333333333333333333333344e+19

julia> factorial(BigInt(40))
815915283247897734345611269596115894272000000000
```

Sin embargo, la promoción de tipos entre los tipos primitivos ya vistos y [`BigInt`](@ref)/[`BigFloat`](@ref) 
no es automática y debe ser establecida explícitamente: 

```jldoctest
julia> x = typemin(Int64)
-9223372036854775808

julia> x = x - 1
9223372036854775807

julia> typeof(x)
Int64

julia> y = BigInt(typemin(Int64))
-9223372036854775808

julia> y = y - 1
-9223372036854775809

julia> typeof(y)
BigInt
```

La precisión predeterminada (en número de bits del significado) y el modo de redondeo de las operaciones 
de [`BigFloat`](@ref) pueden cambiarse globalmente llamando [`setprecision()`](@ref) and [`setrounding()`](@ref),
y todos los cálculos adicionales tomarán en cuenta estos cambios. Alternativamente, la precisión o 
el redondeo se puede cambiar dentro sólo de la ejecución de un bloque particular de código 
utilizando las mismas funciones dentro de un bloque `do`:

```jldoctest
julia> setrounding(BigFloat, RoundUp) do
           BigFloat(1) + parse(BigFloat, "0.1")
       end
1.100000000000000000000000000000000000000000000000000000000000000000000000000003

julia> setrounding(BigFloat, RoundDown) do
           BigFloat(1) + parse(BigFloat, "0.1")
       end
1.099999999999999999999999999999999999999999999999999999999999999999999999999986

julia> setprecision(40) do
           BigFloat(1) + parse(BigFloat, "0.1")
       end
1.1000000000004
```

## [Coeficientes Literales Numéricos](@id man-numeric-literal-coefficients)

Para hacer más claras fórmulas numéricas y expresiones, Julia permite que las variables sean precedidas inmediatamente por un literal numérico, implicando la multiplicación. Esto hace que la escritura de las expresiones polinómicas sea mucho más limpias:

```jldoctest numeric-coefficients
julia> x = 3
3

julia> 2x^2 - 3x + 1
10

julia> 1.5x^2 - .5x + 1
13.0
```

También hace que escribir funciones exponenciales sea más elegante:

```jldoctest numeric-coefficients
julia> 2^2x
64
```

La precedencia de los coeficientes literales numéricos es la misma que la de los operadores unarios como la negación. Así que `2^3x` se analiza como `2^(3x)`, y `2x^3` se analiza como `2*(x ^ 3)`.

Los literales numéricos también funcionan como coeficientes de las expresiones entre paréntesis:

```jldoctest numeric-coefficients
julia> 2(x-1)^2 - 3(x-1) + 1
3
```

Además, las expresiones entre paréntesis se pueden utilizar como coeficientes a las variables, lo que implica la multiplicación de la expresión por la variable:

```jldoctest numeric-coefficients
julia> (x-1)x
6
```

Sin embargo, ni la yuxtaposición de dos expresiones entre paréntesis, ni la colocación de una variable antes de una expresión entre paréntesis puede ser usada para implicar multiplicación:

```jldoctest numeric-coefficients
julia> (x-1)(x+1)
ERROR: MethodError: objects of type Int64 are not callable

julia> x(x+1)
ERROR: MethodError: objects of type Int64 are not callable
```

Ambas expresiones se interpretan como la aplicación de una función: cualquier expresión que no sea un literal numérico, inmediatamente seguida de una entre paréntesis, se interpreta como una función aplicada a los valores entre paréntesis (ver [Functions](@ref) para más información sobre las funciones). Por lo tanto, en ambos casos, se produce un error, ya que el valor de la izquierda no es una función.

Las mejoras sintácticas anteriores reducen significativamente el ruido visual producido al escribir fórmulas matemáticas comunes. Obsérvese que ningún espacio en blanco puede encontrarse entre un coeficiente literal numérico y el identificador o la expresión entre paréntesis que multiplica.

### Conflictos de Sintaxis

La sintaxis de los coeficientes literales yuxtapuestos puede entrar en conflicto con dos sintaxis numéricas literales: literales enteros hexadecimales y notación ingenieril para literales de punto flotante. Aquí hay algunas situaciones donde surgen conflictos sintácticos:

* La expresión literal de enteros hexadecimales `0xff` podría interpretarse como el literal numérico `0` multiplicado por la variable `xff`.
* La expresión literal de punto flotante `1e10` podría interpretarse como el literal numérico `1` multiplicado por la variable `e10`, e igualmente con la forma `E` equivalente

En ambos casos, resolvemos la ambigüedad a favor de la interpretación como literales numéricos:

* Las expresiones que comienzan con `0x` siempre son literales hexadecimales.
* Las expresiones que empiezan con un literal numérico seguido por e o E siempre son literales de coma flotante.

## Literal zero and one

Julia proporciona funciones que devuelven los literales `0` y `1` correspondientes a un tipo especificado o al tipo de una variable dada.

| Function          | Description                                            |
|:----------------- |:------------------------------------------------------ |
| [`zero(x)`](@ref) | Literal cero del tipo `x` o del tipo de la variable `x`|
| [`one(x)`](@ref)  | Literal uno del tipo `x` o del tipo de la variable `x` |

Estas funciones son útiles en [Comparaciones Numéricas](@ref) para evitar la sobrecarga de una
[conversión de tipo](@ref conversion-and-promotion) innecesaria.

Ejemplos:

```jldoctest
julia> zero(Float32)
0.0f0

julia> zero(1.0)
0.0

julia> one(Int32)
1

julia> one(BigFloat)
1.000000000000000000000000000000000000000000000000000000000000000000000000000000
```
