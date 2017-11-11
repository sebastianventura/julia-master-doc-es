# [Operaciones Matemáticas y Funciones Elementales](@id mathematical-operations)

Julia proporciona una colección completa de operadores aritméticos básicos y de operadores de bits para todos sus tipos numéricos primitivos, así como implementaciones portables y eficientes de una colección comprensiva de funciones matemática estándar.

## Arithmetic Operators

Los siguientes [operadores aritméticos](https://en.wikipedia.org/wiki/Arithmetic#Arithmetic_operations)
están soportados sobre todos los tipos primitivos:

| Expression | Name             | Description                            |
|:---------- |:---------------- |:-------------------------------------- |
| `+x`       | más unario       | Operación identidad                    |
| `-x`       | menos unario     | maps values to their additive inverses |
| `x + y`    | suma binaria     | performs addition                      |
| `x - y`    | menos binario    | performs subtraction                   |
| `x * y`    | producto         | performs multiplication                |
| `x / y`    | división         | performs division                      |
| `x \ y`    | división inversa | equivalent to `y / x`                  |
| `x ^ y`    | potencia         | raises `x` to the `y`th power          |
| `x % y`    | resto            | equivalent to `rem(x,y)`               |

así como la negación sobre tipos [`Bool`](@ref):

| Expression | Name     | Description                              |
|:---------- |:-------- |:---------------------------------------- |
| `!x`       | negation | changes `true` to `false` and vice versa |

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

Combining dot operators with numeric literals can be ambiguous.
For example, it is not clear whether `1.+x` means `1. + x` or `1 .+ x`.
Therefore this syntax is disallowed, and spaces must be used around
the operator in such cases.

## Numeric Comparisons

Standard comparison operations are defined for all the primitive numeric types:

| Operator                     | Name                     |
|:---------------------------- |:------------------------ |
| [`==`](@ref)                 | equality                 |
| [`!=`](@ref), [`≠`](@ref !=) | inequality               |
| [`<`](@ref)                  | less than                |
| [`<=`](@ref), [`≤`](@ref <=) | less than or equal to    |
| [`>`](@ref)                  | greater than             |
| [`>=`](@ref), [`≥`](@ref >=) | greater than or equal to |

Here are some simple examples:

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

Integers are compared in the standard manner -- by comparison of bits. Floating-point numbers
are compared according to the [IEEE 754 standard](https://en.wikipedia.org/wiki/IEEE_754-2008):

  * Finite numbers are ordered in the usual manner.
  * Positive zero is equal but not greater than negative zero.
  * `Inf` is equal to itself and greater than everything else except `NaN`.
  * `-Inf` is equal to itself and less then everything else except `NaN`.
  * `NaN` is not equal to, not less than, and not greater than anything, including itself.

The last point is potentially surprising and thus worth noting:

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

and can cause especial headaches with [Arrays](@ref):

```jldoctest
julia> [1 NaN] == [1 NaN]
false
```

Julia provides additional functions to test numbers for special values, which can be useful in
situations like hash key comparisons:

| Function                | Tests if                  |
|:----------------------- |:------------------------- |
| [`isequal(x, y)`](@ref) | `x` and `y` are identical |
| [`isfinite(x)`](@ref)   | `x` is a finite number    |
| [`isinf(x)`](@ref)      | `x` is infinite           |
| [`isnan(x)`](@ref)      | `x` is not a number       |

[`isequal()`](@ref) considers `NaN`s equal to each other:

```jldoctest
julia> isequal(NaN, NaN)
true

julia> isequal([1 NaN], [1 NaN])
true

julia> isequal(NaN, NaN32)
true
```

`isequal()` can also be used to distinguish signed zeros:

```jldoctest
julia> -0.0 == 0.0
true

julia> isequal(-0.0, 0.0)
false
```

Mixed-type comparisons between signed integers, unsigned integers, and floats can be tricky. A
great deal of care has been taken to ensure that Julia does them correctly.

For other types, `isequal()` defaults to calling [`==()`](@ref), so if you want to define
equality for your own types then you only need to add a [`==()`](@ref) method.  If you define
your own equality function, you should probably define a corresponding [`hash()`](@ref) method
to ensure that `isequal(x,y)` implies `hash(x) == hash(y)`.

### Chaining comparisons

Unlike most languages, with the [notable exception of Python](https://en.wikipedia.org/wiki/Python_syntax_and_semantics#Comparison_operators),
comparisons can be arbitrarily chained:

```jldoctest
julia> 1 < 2 <= 2 < 3 == 3 > 2 >= 1 == 1 < 3 != 5
true
```

Chaining comparisons is often quite convenient in numerical code. Chained comparisons use the
`&&` operator for scalar comparisons, and the [`&`](@ref) operator for elementwise comparisons,
which allows them to work on arrays. For example, `0 .< A .< 1` gives a boolean array whose entries
are true where the corresponding elements of `A` are between 0 and 1.

Note the evaluation behavior of chained comparisons:

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

The middle expression is only evaluated once, rather than twice as it would be if the expression
were written as `v(1) < v(2) && v(2) <= v(3)`. However, the order of evaluations in a chained
comparison is undefined. It is strongly recommended not to use expressions with side effects (such
as printing) in chained comparisons. If side effects are required, the short-circuit `&&` operator
should be used explicitly (see [Short-Circuit Evaluation](@ref)).

### Elementary Functions

Julia provides a comprehensive collection of mathematical functions and operators. These mathematical
operations are defined over as broad a class of numerical values as permit sensible definitions,
including integers, floating-point numbers, rationals, and complex numbers,
wherever such definitions make sense.

Moreover, these functions (like any Julia function) can be applied in "vectorized" fashion to
arrays and other collections with the [dot syntax](@ref man-vectorized) `f.(A)`,
e.g. `sin.(A)` will compute the sine of each element of an array `A`.

## Operator Precedence

Julia applies the following order of operations, from highest precedence to lowest:

| Category       | Operators                                                                                         |
|:-------------- |:------------------------------------------------------------------------------------------------- |
| Syntax         | `.` followed by `::`                                                                              |
| Exponentiation | `^`                                                                                               |
| Fractions      | `//`                                                                                              |
| Multiplication | `* / % & \`                                                                                       |
| Bitshifts      | `<< >> >>>`                                                                                       |
| Addition       | `+ - \| ⊻`                                                                                        |
| Syntax         | `: ..` followed by `\|>`                                                                          |
| Comparisons    | `> < >= <= == === != !== <:`                                                                      |
| Control flow   | `&&` followed by `\|\|` followed by `?`                                                           |
| Assignments    | `= += -= *= /= //= \= ^= ÷= %= \|= &= ⊻= <<= >>= >>>=`                                            |

For a complete list of *every* Julia operator's precedence, see the top of this file:
[`src/julia-parser.scm`](https://github.com/JuliaLang/julia/blob/master/src/julia-parser.scm)

You can also find the numerical precedence for any given operator via the built-in function `Base.operator_precedence`, where higher numbers take precedence:

```jldoctest
julia> Base.operator_precedence(:+), Base.operator_precedence(:*), Base.operator_precedence(:.)
(9, 11, 15)

julia> Base.operator_precedence(:+=), Base.operator_precedence(:(=))  # (Note the necessary parens on `:(=)`)
(1, 1)
```

## Numerical Conversions

Julia supports three forms of numerical conversion, which differ in their handling of inexact
conversions.

  * The notation `T(x)` or `convert(T,x)` converts `x` to a value of type `T`.

      * If `T` is a floating-point type, the result is the nearest representable value, which could be
        positive or negative infinity.
      * If `T` is an integer type, an `InexactError` is raised if `x` is not representable by `T`.
  * `x % T` converts an integer `x` to a value of integer type `T` congruent to `x` modulo `2^n`,
    where `n` is the number of bits in `T`. In other words, the binary representation is truncated
    to fit.
  * The [Rounding functions](@ref) take a type `T` as an optional argument. For example, `round(Int,x)`
    is a shorthand for `Int(round(x))`.

The following examples show the different forms.

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

See [Conversion and Promotion](@ref conversion-and-promotion) for how to define your own conversions and promotions.

### Rounding functions

| Function              | Description                      | Return type |
|:--------------------- |:-------------------------------- |:----------- |
| [`round(x)`](@ref)    | round `x` to the nearest integer | `typeof(x)` |
| [`round(T, x)`](@ref) | round `x` to the nearest integer | `T`         |
| [`floor(x)`](@ref)    | round `x` towards `-Inf`         | `typeof(x)` |
| [`floor(T, x)`](@ref) | round `x` towards `-Inf`         | `T`         |
| [`ceil(x)`](@ref)     | round `x` towards `+Inf`         | `typeof(x)` |
| [`ceil(T, x)`](@ref)  | round `x` towards `+Inf`         | `T`         |
| [`trunc(x)`](@ref)    | round `x` towards zero           | `typeof(x)` |
| [`trunc(T, x)`](@ref) | round `x` towards zero           | `T`         |

### Division functions

| Function              | Description                                                                                               |
|:--------------------- |:--------------------------------------------------------------------------------------------------------- |
| [`div(x,y)`](@ref)    | truncated division; quotient rounded towards zero                                                         |
| [`fld(x,y)`](@ref)    | floored division; quotient rounded towards `-Inf`                                                         |
| [`cld(x,y)`](@ref)    | ceiling division; quotient rounded towards `+Inf`                                                         |
| [`rem(x,y)`](@ref)    | remainder; satisfies `x == div(x,y)*y + rem(x,y)`; sign matches `x`                                       |
| [`mod(x,y)`](@ref)    | modulus; satisfies `x == fld(x,y)*y + mod(x,y)`; sign matches `y`                                         |
| [`mod1(x,y)`](@ref)   | `mod()` with offset 1; returns `r∈(0,y]` for `y>0` or `r∈[y,0)` for `y<0`, where `mod(r, y) == mod(x, y)` |
| [`mod2pi(x)`](@ref)   | modulus with respect to 2pi;  `0 <= mod2pi(x)    < 2pi`                                                   |
| [`divrem(x,y)`](@ref) | returns `(div(x,y),rem(x,y))`                                                                             |
| [`fldmod(x,y)`](@ref) | returns `(fld(x,y),mod(x,y))`                                                                             |
| [`gcd(x,y...)`](@ref) | greatest positive common divisor of `x`, `y`,...                                                          |
| [`lcm(x,y...)`](@ref) | least positive common multiple of `x`, `y`,...                                                            |

### Sign and absolute value functions

| Function                | Description                                                |
|:----------------------- |:---------------------------------------------------------- |
| [`abs(x)`](@ref)        | a positive value with the magnitude of `x`                 |
| [`abs2(x)`](@ref)       | the squared magnitude of `x`                               |
| [`sign(x)`](@ref)       | indicates the sign of `x`, returning -1, 0, or +1          |
| [`signbit(x)`](@ref)    | indicates whether the sign bit is on (true) or off (false) |
| [`copysign(x,y)`](@ref) | a value with the magnitude of `x` and the sign of `y`      |
| [`flipsign(x,y)`](@ref) | a value with the magnitude of `x` and the sign of `x*y`    |

### Powers, logs and roots

| Function                 | Description                                                                |
|:------------------------ |:-------------------------------------------------------------------------- |
| [`sqrt(x)`](@ref), `√x`  | square root of `x`                                                         |
| [`cbrt(x)`](@ref), `∛x`  | cube root of `x`                                                           |
| [`hypot(x,y)`](@ref)     | hypotenuse of right-angled triangle with other sides of length `x` and `y` |
| [`exp(x)`](@ref)         | natural exponential function at `x`                                        |
| [`expm1(x)`](@ref)       | accurate `exp(x)-1` for `x` near zero                                      |
| [`ldexp(x,n)`](@ref)     | `x*2^n` computed efficiently for integer values of `n`                     |
| [`log(x)`](@ref)         | natural logarithm of `x`                                                   |
| [`log(b,x)`](@ref)       | base `b` logarithm of `x`                                                  |
| [`log2(x)`](@ref)        | base 2 logarithm of `x`                                                    |
| [`log10(x)`](@ref)       | base 10 logarithm of `x`                                                   |
| [`log1p(x)`](@ref)       | accurate `log(1+x)` for `x` near zero                                      |
| [`exponent(x)`](@ref)    | binary exponent of `x`                                                     |
| [`significand(x)`](@ref) | binary significand (a.k.a. mantissa) of a floating-point number `x`        |

For an overview of why functions like [`hypot()`](@ref), [`expm1()`](@ref), and [`log1p()`](@ref)
are necessary and useful, see John D. Cook's excellent pair of blog posts on the subject: [expm1, log1p, erfc](https://www.johndcook.com/blog/2010/06/07/math-library-functions-that-seem-unnecessary/),
and [hypot](https://www.johndcook.com/blog/2010/06/02/whats-so-hard-about-finding-a-hypotenuse/).

### Trigonometric and hyperbolic functions

All the standard trigonometric and hyperbolic functions are also defined:

```
sin    cos    tan    cot    sec    csc
sinh   cosh   tanh   coth   sech   csch
asin   acos   atan   acot   asec   acsc
asinh  acosh  atanh  acoth  asech  acsch
sinc   cosc   atan2
```

These are all single-argument functions, with the exception of [atan2](https://en.wikipedia.org/wiki/Atan2),
which gives the angle in [radians](https://en.wikipedia.org/wiki/Radian) between the *x*-axis
and the point specified by its arguments, interpreted as *x* and *y* coordinates.

Additionally, [`sinpi(x)`](@ref) and [`cospi(x)`](@ref) are provided for more accurate computations
of [`sin(pi*x)`](@ref) and [`cos(pi*x)`](@ref) respectively.

In order to compute trigonometric functions with degrees instead of radians, suffix the function
with `d`. For example, [`sind(x)`](@ref) computes the sine of `x` where `x` is specified in degrees.
The complete list of trigonometric functions with degree variants is:

```
sind   cosd   tand   cotd   secd   cscd
asind  acosd  atand  acotd  asecd  acscd
```

### Special functions

| Function                                                      | Description                                                                                                                                                     |
|:------------------------------------------------------------- |:--------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`gamma(x)`](@ref)                                            | [gamma function](https://en.wikipedia.org/wiki/Gamma_function) at `x`                                                                                           |
| [`lgamma(x)`](@ref)                                           | accurate `log(gamma(x))` for large `x`                                                                                                                          |
| [`lfact(x)`](@ref)                                            | accurate `log(factorial(x))` for large `x`; same as `lgamma(x+1)` for `x > 1`, zero otherwise                                                                   |
| [`beta(x,y)`](@ref)                                           | [beta function](https://en.wikipedia.org/wiki/Beta_function) at `x,y`                                                                                           |
| [`lbeta(x,y)`](@ref)                                          | accurate `log(beta(x,y))` for large `x` or `y`                                                                                                                  |
