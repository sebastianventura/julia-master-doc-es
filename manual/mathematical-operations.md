# [Operaciones Matemáticas y Funciones Elementales](@id mathematical-operations)

Julia proporciona una colección completa de operadores aritméticos básicos y de operadores de bits para todos sus tipos numéricos primitivos, así como implementaciones portables y eficientes de una colección comprensiva de funciones matemática estándar.

## Arithmetic Operators

Los siguientes [operadores aritméticos](https://en.wikipedia.org/wiki/Arithmetic#Arithmetic_operations)
están soportados sobre todos los tipos primitivos:

| Expression | Name             | Description                            |
|:---------- |:---------------- |:-------------------------------------- |
| `+x`       | más unario       | Operación identidad                    |
| `-x`       | menos unario     | Inverso matemático de un número        |
| `x + y`    | suma binaria     | suma                                   |
| `x - y`    | menos binario    | resta                                  |
| `x * y`    | producto         | multiplicación                         |
| `x / y`    | división         | división                               |
| `x \ y`    | división inversa | Equivalente a `y / x`                  |
| `x ^ y`    | potencia         | eleva `x` a la `y`-ésima potencia      |
| `x % y`    | resto            | Equivalente a `rem(x,y)`               |

así como la negación sobre tipos [`Bool`](@ref):

| Expression | Name     | Description                         |
|:---------- |:-------- |:----------------------------------- |
| `!x`       | negación | Cambia `true` a `false` y viceversa |

El sistema de promoción de Julia hace que las operaciones aritméticas sobre mezclas de tipos de argumentos funcione de forma natural y automáticamente. Ver [Conversión y Promoción](@ref conversion-and-promotion) para los detalles del sistema de promoción.

He aquí algunos ejemplos simples de usar operadores aritméticos:

```jldoctest
julia> 1 + 2 + 3
6

julia> 1 - 2
-1

julia> 3*2/12
0.5
```

(Por convención, tendemos a los operadores de espacio más estrechamente si se aplican antes de otros operadores cercanos.Por ejemplo, generalmente escribimos `-x + 2` para reflejar que `x` primero se niega y, a continuación, `2` se agrega a ese resultado.)

## Bitwise Operators

Los siguientes [operadores bit a bit](https://en.wikipedia.org/wiki/Bitwise_operation#Bitwise_operators) son soportados sobre todos los tipos enteros primitivos:

| Expression | Name                                                                     |
|:---------- |:------------------------------------------------------------------------ |
| `~x`       | Negación bit a bit                                                       |
| `x & y`    | Conjunción (*and*) bit a bit                                             |
| `x \| y`   | Disyunción (*or*) bit a bit                                              |
| `x ⊻ y`    | *Or* exclusivo bit a bit (*xor*)                                         |
| `x >>> y`  | [Desplazamiento lógico](https://en.wikipedia.org/wiki/Logical_shift) hacia la derecha        |
| `x >> y`   | [Desplazamiento aritmético](https://en.wikipedia.org/wiki/Arithmetic_shift) hacia la derecha |
| `x << y`   | Desplazamiento hacia la izquierda lógico/aritmético                                          |

He aquí algunos ejemplos de uso de operadores bit a bit:

```jldoctest
julia> ~123
-124

julia> 123 & 234
106

julia> 123 | 234
251

julia> 123 ⊻ 234
145

julia> xor(123, 234)
145

julia> ~UInt32(123)
0xffffff84

julia> ~UInt8(123)
0x84
```

## Operaciones de actualización

Cada operador binario aritmético y bit a bit también tiene una versión de actualización que asigna el resultado de la operación de nuevo a su operando izquierdo. La versión de actualización del operador binario se forma colocando a = inmediatamente después del operador. Por ejemplo, escribir `x += 3` es equivalente a escribir `x = x + 3`:

```jldoctest
julia> x = 1
1

julia> x += 3
4

julia> x
4
```

Las versiones de actualización de todos los operadores binarios, aritméticos de bits son:

```
+=  -=  *=  /=  \=  ÷=  %=  ^=  &=  |=  ⊻=  >>>=  >>=  <<=
```

!!! nota
    Un operador de actualización reasigna la variable sobre la parte izquierda de la ecuación. Como resultado, el tipo de la variable puede cambiar:

    ```jldoctest
    julia> x = 0x01; typeof(x)
    UInt8

    julia> x *= 2 # Same as x = x * 2
    2

    julia> typeof(x)
    Int64
    ```

## [Operadores vectorizados con "punto"](@id man-dot-operators)

Para cada operación binaria como `^` hay su correspondiente operación "con punto" `.^` que se define *automáticamente* para realizar la operación `^` elemento a elemento sobre arrays. Por ejemplo, la operación `[1, 2, 3]^3` no está definidia, porque no hay un significado matemático estándar para calcular el cubo de un array, pero `[1, 2, 3].^3` si lo está como el cálculo de la operación cubo elemento a elemento (o vectorizada) `[1^3, 2^3, 3^3]`. Lo mismo puede decirse para operaciones unarios tales como `!` o `√`, que existe el correspondiente `.√` que aplica el operador elemento a elemento.

```jldoctest
julia> [1,2,3] .^ 3
3-element Array{Int64,1}:
  1
  8
 27
```

Más específicamente, `a .^ b` es analizado como la [llamada vectorizada ( o *"dot" call*)](@ref man-vectorized) `(^).(a,b)`, que realiza una operación de [retransmisión (*broadcast*)](@ref Broadcasting): ella puede combinar arrays y escalares, arrays del mismo tamaño (realizando la operación elemento a elemento), o incluso arrays de diferentes formas (por ejemplo, combinar vectores fila y columna para producir una matriz). Además, como todas las llamadas vectorizadas, estos operadores son *fusing*. Por ejemplo, si calculas `2 .* A.^2 .+ sin.(A)` (o, equivalentemente `@. 2A^2 + sin(A)`, usando la macro [`@.`](@ref @__dot__)) para un array `A`, se realiza un *único* bucle sobre `A`, computando `2a^2 + sin(a)` para cada elemento de `A`. En particular, las llamadas vectorizadas anidadas como `f.(g.(x))` son *fused*, y los operadores binarios adyacentes como `x .+ 3 .* x.^2` son equivalentes a llamadas vectorizadas anidadas `(+).(x, (*).(3, (^).(x, 2)))`.

Además, los operadores de actualización "vectorizados" como `a .+= b` (o `@. a += b`) son transformados en `a .= a .+ b`, donde `.=` es un opeador de asignación *fused* *in-place* (ver la [documentación de la sintaxis vectorizada](@ref man-vectorized)).

Nótese que la sintaxis de punto es también aplicable a operadores definidos por el usuario. Por ejemplo, si definimos el operador `⊗(A,B) = kron(A,B)` para dar una sintaxis infija `A ⊗ B` al producto de Kronecker ([`kron`](@ref)), entonces
`[A,B] .⊗ [C,D]` calculará  `[A⊗C, B⊗D]` sin ninguna codificación adicional.

Combinar operadores punto con literales numéricos puede ser ambiguo. Por ejemplo, no está claro si `1.+x` significa `1. + x` o `1 .+ x`. Por tanto, esta sintáxis no está permitida, y en estos casos deben usarse espacios alrededos del operador.

## Comparaciones Numéricas

Los operadores de comparación estándar están definidos para todos los tipos numéricos primitivos:


| Operador                     | Nombre                     |
|:---------------------------- |:------------------------ |
| [`==`](@ref)                 | Igualdad                 |
| [`!=`](@ref), [`≠`](@ref !=) | Desigualdad              |
| [`<`](@ref)                  | Menor que                |
| [`<=`](@ref), [`≤`](@ref <=) | Menor o igual que        |
| [`>`](@ref)                  | Mayor que                |
| [`>=`](@ref), [`≥`](@ref >=) | Mayor o igual que        |

He aquí algunos ejemplos:

```jldoctest
julia> 1 == 1
true

julia> 1 == 2
false

julia> 1 != 2
true

julia> 1 == 1.0
true

julia> 1 < 2
true

julia> 1.0 > 3
false

julia> 1 >= 1.0
true

julia> -1 <= 1
true

julia> -1 <= -1
true

julia> -1 <= -2
false

julia> 3 < -0.5
false
```

Los enteros se comparan de un modo estándar, mediante comparación de bits. Los números de punto flotante se comparan de acuerdo al [estándar IEEE 754](https://en.wikipedia.org/wiki/IEEE_754-2008):

* Los números finitos son ordenados del modo habitual.
* El cero positivo es igual pero no mayor que el cero negativo.
* `Inf` es igual a si mismo y mayor que todo excepto `NaN`
* `-Inf` es igual a si mismo y menor que todo excepto `NaN`
* `NaN` no es igual, mayor o menor a nadie, excepto a sí mismo.

Este último punto es potencialmente sorprendente y, por tanto, vale la pena señalar que:

```jldoctest
julia> NaN == NaN
false

julia> NaN != NaN
true

julia> NaN < NaN
false

julia> NaN > NaN
false
```

y puede causar dolores de cabeza especiales con [Arrays](@ref):

```jldoctest
julia> [1 NaN] == [1 NaN]
false
```

Julia proporciona funciones adicionales para comprobar números para valores especiales, lo cuál pueden ser útil en situaciones como las comparaciones de claves hash:

| Function                | Tests if                  |
|:----------------------- |:------------------------- |
| [`isequal(x, y)`](@ref) | `x` e `y` son idénticos   |
| [`isfinite(x)`](@ref)   | `x` es un número finito   |
| [`isinf(x)`](@ref)      | `x` es infinito           |
| [`isnan(x)`](@ref)      | `x` no es un número       |

[`isequal()`](@ref) considera los `NaN`s iguales entre sí:

```jldoctest
julia> isequal(NaN, NaN)
true

julia> isequal([1 NaN], [1 NaN])
true

julia> isequal(NaN, NaN32)
true
```

`isequal()` también puede usarse para distinguir los ceros con signo:

```jldoctest
julia> -0.0 == 0.0
true

julia> isequal(-0.0, 0.0)
false
```

Las comparaciones de tipo mixto entre enteros con signo, enteros sin signo y valores en punto flotante pueden ser complicadas. Se ha tomado mucho cuidado para asegurarse de que Julia los hace correctamente.

 `==()`,  `==()`.  `isequal(x, y)` implique `hash(x) == hash(y)`.

Para otros tipos, `isequal()` llama por defecto a [`==()`](@ref), así que si quieres definir la igualdad para nuestros propios tipos, solo tienes que agregar un método [`==()`](@ref).  Si defines tu propia función de igualdad, probablemente deberías definir un método [`hash()`](@ref) correspondiente para asegurarte de que `isequal(x,y)` implica `hash(x) == hash(y)`.

### Chaining comparisons

A diferencia de la mayoría de los idiomas, [con la notable excepción de Python (https://en.wikipedia.org/wiki/Python_syntax_and_semantics#Comparison_operators), las comparaciones pueden encadenarse arbitrariamente:

```jldoctest
julia> 1 < 2 <= 2 < 3 == 3 > 2 >= 1 == 1 < 3 != 5
true
```

El encadenamiento de comparaciones suele ser bastante conveniente en el código numérico. Las comparaciones encadenadas utilizan el operador `&&` para comparaciones escalares y el operador [`&`](@ref) para comparaciones elemento a elemento, lo que les permite trabajar en matrices. Por ejemplo, `0 .< A .< 1` da una matriz booleana cuyas entradas son verdaderas donde los elementos correspondientes de A están entre 0 y 1.

Nótese el comportamiento de evaluación de las comparaciones encadenadas:

```jldoctest
julia> v(x) = (println(x); x)
v (generic function with 1 method)

julia> v(1) < v(2) <= v(3)
2
1
3
true

julia> v(1) > v(2) <= v(3)
2
1
false
```

La expresión del medio sólo se evalúa una vez, en lugar de dos veces como lo sería si la expresión se escribiera como `v(1) < v(2) && v(2) <= v(3)`. Sin embargo, el orden de las evaluaciones en una comparación encadenada no está definido. Se recomienda encarecidamente no utilizar expresiones que puedan tener efectos secundarios (como la impresión) en comparaciones encadenadas. Si se requieren efectos secundarios, se debe utilizar explícitamente el operador de cortocircuito `&&` (ver [Evaluación en cortocircuito](short-circuit-evaluation)).

### Funciones elementales

Julia proporciona una colección completa de funciones matemáticas y operadores. Estas operaciones matemáticas se definen como una clase de valores numéricos que permiten definiciones sensibles, incluyendo enteros, números de punto flotante, racionales y complejos, dondequiera que tales definiciones tengan sentido.

Además, estas funciones (como cualquier función de Julia) se pueden aplicar de manera "vectorizada" a matrices y otras colecciones con la [sintaxis vectorizada](@ref man-vectorized)   `f.(A)`, por ejemplo, sin.(A) calculará el seno de cada elemento de una matriz `A`. 

## Precedencia de Operadores

Julia aplica el siguiente orden de operaciones, de mayor a menor precedencia:

| Category       | Operators                                                                                         |
|:-------------- |:------------------------------------------------------------------------------------------------- |
| Syntax         | `.` seguida por `::`                                                                              |
| Exponentiation | `^`                                                                                               |
| Fractions      | `//`                                                                                              |
| Multiplication | `* / % & \`                                                                                       |
| Bitshifts      | `<< >> >>>`                                                                                       |
| Addition       | `+ - \| ⊻`                                                                                        |
| Syntax         | `: ..` followed by `\|>`                                                                          |
| Comparisons    | `> < >= <= == === != !== <:`                                                                      |
| Control flow   | `&&` followed by `\|\|` followed by `?`                                                           |
| Assignments    | `= += -= *= /= //= \= ^= ÷= %= \|= &= ⊻= <<= >>= >>>=`                                            |

Para una lista completa de cada una de las precedencias de operadores de Julia, consultar el fichero
[`src/julia-parser.scm`](https://github.com/JuliaLang/julia/blob/master/src/julia-parser.scm)

También puede encontrarse la precedencia numérica pra cualquier operación dada mediante la función intrínseca `Base.operator_precedence` donde el número mayor corresponde a la operación con mayor precedencia.

```jldoctest
julia> Base.operator_precedence(:+), Base.operator_precedence(:*), Base.operator_precedence(:.)
(9, 11, 15)

julia> Base.operator_precedence(:+=), Base.operator_precedence(:(=))  # (Note the necessary parens on `:(=)`)
(1, 1)
```

## Conversiones Numéricas

Julia soporta tres formas de conversión numérica, que difieren en su manejo de las conversiones inexactas.

  * La notación `T(x)` o `convert(T,x)` convierte `x` a un valor de tipo `T`.


      * Si `T` es un tipo en punto flotante, el resultado es el valor más cercano representable, que podría ser infinito positivo o negativo.
      * Si `T` es un tipo entero, se lanzará un `InexactError` si `x`no es representable por `T`.
  *  `x % T`convierte un entero `x` a un valor de un tipo entero `T` congruente a `x` modulo `2^n`, donde `n` es el número de bits en `T`. En otras palabras, la representación binaria es truncada para ajustarse.
  * Las [Funciones de Redondeo](@ref rounding-functions) toman un tipo `T` como argumento opcional. Por ejemplo, `round(Int,x)` es una abreviatura de `Int(round(x))`.

Los siguientes ejemplos muestran las siguientes formas:

```jldoctest
julia> Int8(127)
127

julia> Int8(128)
ERROR: InexactError()
Stacktrace:
 [1] Int8(::Int64) at ./sysimg.jl:102

julia> Int8(127.0)
127

julia> Int8(3.14)
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Int8}, ::Float64) at ./float.jl:659
 [2] Int8(::Float64) at ./sysimg.jl:102

julia> Int8(128.0)
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Int8}, ::Float64) at ./float.jl:659
 [2] Int8(::Float64) at ./sysimg.jl:102

julia> 127 % Int8
127

julia> 128 % Int8
-128

julia> round(Int8,127.4)
127

julia> round(Int8,127.6)
ERROR: InexactError()
Stacktrace:
 [1] trunc(::Type{Int8}, ::Float64) at ./float.jl:652
 [2] round(::Type{Int8}, ::Float64) at ./float.jl:338
```

Ver [Conversión y Promoción](@ref conversion-and-promotion) para ver cómo definir tus propias conversiones y promociones.

### Funciones de Redondeo

| Function              | Description                        | Return type |
|:--------------------- |:---------------------------------- |:----------- |
| [`round(x)`](@ref)    | Redondea `x` al entero más cercano | `typeof(x)` |
| [`round(T, x)`](@ref) | Redondea `x` al entero más cercano | `T`         |
| [`floor(x)`](@ref)    | Redondea `x` hacia `-Inf`          | `typeof(x)` |
| [`floor(T, x)`](@ref) | Redondea `x` hacia `-Inf`          | `T`         |
| [`ceil(x)`](@ref)     | Redondea `x` hacia `+Inf`          | `typeof(x)` |
| [`ceil(T, x)`](@ref)  | Redondea `x` hacia `+Inf`          | `T`         |
| [`trunc(x)`](@ref)    | Redondea `x` hacia cero            | `typeof(x)` |
| [`trunc(T, x)`](@ref) | Redondea `x` hacia cero            | `T`         |

### Funciones de división

| Función               | Descripción                                                                                        |
|:--------------------- |:-------------------------------------------------------------------------------------------------- |
| [`div(x,y)`](@ref)    | División truncada; cociente redondeado hacia cero                                                  |
| [`fld(x,y)`](@ref)    | División *floored*; cociente redondeado hacia `-Inf`                                               |
| [`cld(x,y)`](@ref)    | División *ceiling*; cociente redondeado hacia `+Inf`                                               |
| [`rem(x,y)`](@ref)    | Resto; satisface `x == div(x,y)*y + rem(x,y)`; el signo se corresponde con el de `x`               |
| [`mod(x,y)`](@ref)    | Módulo; satisface `x == fld(x,y)*y + mod(x,y)`; el signo se corresponde con el de `y`              |
| [`mod1(x,y)`](@ref)   | Módulo con un desplazamiento de 1; devuelve `r∈(0,y]` para `y>0` o `r∈[y,0)` para `y<0`, donde `mod(r, y) == mod(x, y)` |
| [`mod2pi(x)`](@ref)   | Módulo con respecto a 2pi; `0 <= mod2pi(x)  < 2pi`                                                 |
| [`divrem(x,y)`](@ref) | Devuelve `(div(x,y),rem(x,y))`                                                                     |
| [`fldmod(x,y)`](@ref) | Devuelve `(fld(x,y),mod(x,y))`                                                                     |
| [`gcd(x,y...)`](@ref) | Máximo común divisor positivo de `x`, `y`,...                                                      |
| [`lcm(x,y...)`](@ref) | Mínimo común múltiplo positivo de `x`, `y`,...                                                     |

### Funciones de signo y valor absoluto

| Function                | Description                                                |
|:----------------------- |:---------------------------------------------------------- |
| [`abs(x)`](@ref)        | Un valor positivo con la magnitud de `x`                   |
| [`abs2(x)`](@ref)       | El cuadrado de la magnitud de `x`                          |
| [`sign(x)`](@ref)       | Indica el signo de `x`, devolviendo -1, 0, o +1            |
| [`signbit(x)`](@ref)    | Indica que si el bit de signo está en **on** (`true`) o en **off** (`false`)|
| [`copysign(x,y)`](@ref) | Indica un valor con la magnitud de `x` y el signo de `y    |
| [`flipsign(x,y)`](@ref) |  Indica un valor con la magnitud de `x` y el signo de `x*y`|

### Potencias, logaritmos y raíces

| Function                 | Description                                                                |
|:------------------------ |:-------------------------------------------------------------------------- |
| [`sqrt(x)`](@ref), `√x`  | Raíz cuadrada de `x`                                                       |
| [`cbrt(x)`](@ref), `∛x`  | Raíz cúbica de `x`                                                         |
| [`hypot(x,y)`](@ref)     | Hipotenusa del triángulo rectángulo cuyos catetos son de longitudes `x` e `y` |
| [`exp(x)`](@ref)         | Función exponencial natural sobre `x`                                      |
| [`expm1(x)`](@ref)       | Valor exacto de `exp(x)-1` para  `x` cercano a zero                        |
| [`ldexp(x,n)`](@ref)     | `x*2^n` calculado eficientemente para valores enteros de `n`               |
| [`log(x)`](@ref)         | Logaritmo neperiano de `x`                                                 |
| [`log(b,x)`](@ref)       | Logaritmo en base `b` de `x`                                               |
| [`log2(x)`](@ref)        | Logaritmo en base 2 de `x`                                                 |
| [`log10(x)`](@ref)       | Logaritmo decimal de `x`                                                   |
| [`log1p(x)`](@ref)       | Valor exacto de `log(1+x)` para `x` cercano a cero                         |
| [`exponent(x)`](@ref)    | Exponente binario de  `x`                                                  |
| [`significand(x)`](@ref) | Significando binario (alias *mantisa*) de un número en punto flotante `x`  |

Para una explicación de por qué son necesarias funciones como [`hypot()`](@ref), [`expm1()`](@ref), and [`log1p()`](@ref), véase el excelente par de artículos en el blog de John D. Cook's sobre el tema: [expm1, log1p, erfc](https://www.johndcook.com/blog/2010/06/07/math-library-functions-that-seem-unnecessary/),
e [hypot](https://www.johndcook.com/blog/2010/06/02/whats-so-hard-about-finding-a-hypotenuse/).

### Funciones Trigonométricas e Hiperbólicas

Todas las funciones trigonométricas e hiperbólicas estándar están también definidas:
```
sin    cos    tan    cot    sec    csc
sinh   cosh   tanh   coth   sech   csch
asin   acos   atan   acot   asec   acsc
asinh  acosh  atanh  acoth  asech  acsch
sinc   cosc   atan2
```

Son todas funciones de un solo argumento, con la excepción de  [atan2](https://en.wikipedia.org/wiki/Atan2),
que da el ángulo en [radians](https://en.wikipedia.org/wiki/Radian) entre el eje *x* y el punto especificado por sus argumentos, interpretado como sus coordenadas *x* e *y*.

Adicionalmente, se proporcionan [`sinpi(x)`](@ref) e [`cospi(x)`](@ref) para cálculos más exactos de [`sin(pi*x)`](@ref) y [`cos(pi*x)`](@ref) respectivamente.

Para computar funciones trigonométricas con grados en lugar de con rdianes, añada al nombre de la función el sufijo `d`. Por ejemplo, [`sind(x)`](@ref) calcula el seno de `x`, donde `x` se especifica en grados. La lista completa de funciones trigonométricas con variantes grados es:

```
sind   cosd   tand   cotd   secd   cscd
asind  acosd  atand  acotd  asecd  acscd
```

### Funciones Especiales

| Función                    | Descripción                        |
|:-------------------------- |:---------------------------------- |
| [`gamma(x)`](@ref)         | [Función gamma](https://en.wikipedia.org/wiki/Gamma_function) en `x`|
| [`lgamma(x)`](@ref)        | Valor exacto de `log(gamma(x))` para valores grandes de `x` |
| [`lfact(x)`](@ref)         | Valor exacto de  `log(factorial(x))` para valores grandes de  `x`; igual que `lgamma(x+1)` para `x > 1`, cero en otros caso            |
| [`beta(x,y)`](@ref)        | [Función beta](https://en.wikipedia.org/wiki/Beta_function) en `x,y` |
| [`lbeta(x,y)`](@ref)       | Valor exacto de  `log(beta(x,y))` para valores grandes de `x` o `y`  |
