# [Números Racionales y Complejos](@id complex-and-rational-numbers)

Julia se distribuye con tipos predefinidos que representan números complejos y racionales, 
y soporta todas las [Operaciones Matemáticas y Funciones Elementales](@ref) estándar sobre ellos. 
[Conversión and Promoción](@ref conversion-and-promotion) se definen de modo que las operaciones 
en cualquier combinación de tipos numéricos predefinidos, primitivos o compuestos, se comporten 
como se esperaba.

## Números Complejos

La constante global [`im`](@ref) está ligada al número complejo *i*, que representa la raíz 
cuadrada principal de -1. Se consideró nocivo para co-optar el nombre `i` para una constante 
global, ya que es un nombre de variable de índice popular. Como Julia permite que los literales 
numéricos se [yuxtapongan con identificadores como coeficientes](@ref man-numeric-literal-coefficients),
esta unión es suficiente para proporcionar sintaxis conveniente para números complejos, 
similar a la notación matemática tradicional:

```jldoctest
julia> 1 + 2im
1 + 2im
```

Podemos realizar todas las operaciones aritméticas estándar con los números complejos:

```jldoctest
julia> (1 + 2im)*(2 - 3im)
8 + 1im

julia> (1 + 2im)/(1 - 2im)
-0.6 + 0.8im

julia> (1 + 2im) + (1 - 2im)
2 + 0im

julia> (-3 + 2im) - (5 - 1im)
-8 + 3im

julia> (-1 + 2im)^2
-3 - 4im

julia> (-1 + 2im)^2.5
2.7296244647840084 - 6.960664459571898im

julia> (-1 + 2im)^(1 + 1im)
-0.27910381075826657 + 0.08708053414102428im

julia> 3(2 - 5im)
6 - 15im

julia> 3(2 - 5im)^2
-63 - 60im

julia> 3(2 - 5im)^-1.0
0.20689655172413796 + 0.5172413793103449im
```

El mecanismo de promoción asegura qur las combinaciones de operandos de distintos tipos funcionarán:

```jldoctest
julia> 2(1 - 1im)
2 - 2im

julia> (2 + 3im) - 1
1 + 3im

julia> (1 + 2im) + 0.5
1.5 + 2.0im

julia> (2 + 3im) - 0.5im
2.0 + 2.5im

julia> 0.75(1 + 2im)
0.75 + 1.5im

julia> (2 + 3im) / 2
1.0 + 1.5im

julia> (1 - 3im) / (2 + 2im)
-0.5 - 1.0im

julia> 2im^2
-2 + 0im

julia> 1 + 3/4im
1.0 - 0.75im
```

Nótese que `3/4im == 3/(4*im) == -(3/4*im)`, ya que un coeficiente literal se enlaza más 
fuerte que la división.

También se proporcionan las funciones estándar para manipular valores complejos:

```jldoctest
julia> z = 1 + 2im
1 + 2im

julia> real(1 + 2im) # real part of z
1

julia> imag(1 + 2im) # imaginary part of z
2

julia> conj(1 + 2im) # complex conjugate of z
1 - 2im

julia> abs(1 + 2im) # absolute value of z
2.23606797749979

julia> abs2(1 + 2im) # squared absolute value
5

julia> angle(1 + 2im) # phase angle in radians
1.1071487177940904
```

Como de costumbre, el valor absoluto ([`abs()`](@ref)) de un número complejo es su distancia a cero.
[`abs2()`](@ref) da el cuadrado del valor absoluto, y es de uso particular para los números complejos 
donde se evita tomar una raíz cuadrada. [`angle()`](@ref) devuelve el ángulo de fase en radianes 
(también conocido como *argumento* o función *arg*). La gama completa de otras [Funciones Elementales](@ref)
is also defined for complex numbers:

```jldoctest
julia> sqrt(1im)
0.7071067811865476 + 0.7071067811865475im

julia> sqrt(1 + 2im)
1.272019649514069 + 0.7861513777574233im

julia> cos(1 + 2im)
2.0327230070196656 - 3.0518977991518im

julia> exp(1 + 2im)
-1.1312043837568135 + 2.4717266720048188im

julia> sinh(1 + 2im)
-0.4890562590412937 + 1.4031192506220405im
```

Tenga en cuenta que las funciones matemáticas normalmente devuelven valores reales cuando se 
aplican a números reales y valores complejos cuando se aplican a números complejos. Por ejemplo, 
[`sqrt()`](@ref) se comporta de forma diferente cuando se aplica a `-1` que cuanso se aplica sobre
`-1 + 0im`, aunque `-1 == -1 + 0im`:

```jldoctest
julia> sqrt(-1)
ERROR: DomainError:
sqrt will only return a complex result if called with a complex argument. Try sqrt(complex(x)).
Stacktrace:
 [1] sqrt(::Int64) at ./math.jl:447

julia> sqrt(-1 + 0im)
0.0 + 1.0im
```

La [notación de coeficiente numérico literal](@ref man-numeric-literal-coefficients) no funciona 
cuando se construye un número complejo a partir de variables. En su lugar, la multiplicación debe 
expresarse explícitamente:

```jldoctest
julia> a = 1; b = 2; a + b*im
1 + 2im
```

Sin embargo, esto no es lo recomendable; En su lugar, utilice la función [`complex()`](@ref) para 
construir un valor complejo directamente de sus partes real e imaginaria:

```jldoctest
julia> a = 1; b = 2; complex(a, b)
1 + 2im
```

Esta construcción evita las operaciones de multiplicación y adición.

[`Inf`](@ref) y [`NaN`](@ref) se propagan a través de números complejos en las partes real e 
imaginaria de un número complejo como se describe en la sección [valores especiales en punto flotante](@ref) 
section:

```jldoctest
julia> 1 + Inf*im
1.0 + Inf*im

julia> 1 + NaN*im
1.0 + NaN*im
```

## Números racionales

Julia tiene un tipo numérico racional para representar razones exactas de enteros. Los racionales 
se construyen usando el operador [`//`](@ref):

```jldoctest
julia> 2//3
2//3
```

Si el numerador y el denominador de un racional tienen factores comunes, ellos son reducidos a los 
términos mínimos tales que el denominador sea no negativo:

```jldoctest
julia> 6//9
2//3

julia> -4//8
-1//2

julia> 5//-15
-1//3

julia> -4//-12
1//3
```

Esta forma normalizada para una razón de enteros es única, por lo que la igualdad de valores 
racionales puede ser testada comprobando la igualdad del numerador y el denominador. El numerador 
estandarizado y el denominador de un valor racional pueden ser extraídos usando las funciones 
[`numerator()`](@ref) y [`denominator()`](@ref):

```jldoctest
julia> numerator(2//3)
2

julia> denominator(2//3)
3
```

La comparación directa de numerador y denominador no suele ser necesaria, ya que la aritmetica 
estándar y las operaciones de comparación están definidas para los valores racionales:

```jldoctest
julia> 2//3 == 6//9
true

julia> 2//3 == 9//27
false

julia> 3//7 < 1//2
true

julia> 3//4 > 2//3
true

julia> 2//4 + 1//6
2//3

julia> 5//12 - 1//4
1//6

julia> 5//8 * 3//12
5//32

julia> 6//5 / 10//7
21//25
```

Los racionales pueden convertirse fácilmente en número en punto flotante:

```jldoctest
julia> float(3//4)
0.75
```

La conversión de racional a punto flotante respeta la siguiente identifaf para cualesquiera valores 
enteros de `a` and `b`, con las excepciones de los casos `a == 0` and `b == 0`:

```jldoctest
julia> a = 1; b = 2;

julia> isequal(float(a//b), a/b)
true
```

Construir valores racionales infinitos es aceptable:

```jldoctest
julia> 5//0
1//0

julia> -3//0
-1//0

julia> typeof(ans)
Rational{Int64}
```

Sin embargo, no lo es tratar de construir un valor NaN [`NaN`](@ref) racional:

```jldoctest
julia> 0//0
ERROR: ArgumentError: invalid rational: zero(Int64)//zero(Int64)
Stacktrace:
 [1] Rational{Int64}(::Int64, ::Int64) at ./rational.jl:13
 [2] //(::Int64, ::Int64) at ./rational.jl:40
```

Como es natural, el sistema de promoción hace que las interacciones con otros tipos numéricos 
se hagan sin esfuerzo alguno:

```jldoctest
julia> 3//5 + 1
8//5

julia> 3//5 - 0.5
0.09999999999999998

julia> 2//7 * (1 + 2im)
2//7 + 4//7*im

julia> 2//7 * (1.5 + 2im)
0.42857142857142855 + 0.5714285714285714im

julia> 3//2 / (1 + 2im)
3//10 - 3//5*im

julia> 1//2 + 2im
1//2 + 2//1*im

julia> 1 + 2//3im
1//1 - 2//3*im

julia> 0.5 == 1//2
true

julia> 0.33 == 1//3
false

julia> 0.33 < 1//3
true

julia> 1//3 - 0.33
0.0033333333333332993
```
