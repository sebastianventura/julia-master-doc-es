# Control de Flujo

Julia proporciona una variedad de construcciones para control de flujo:

  * [Expresiones Compuestas](@ref man-compound-expressions): `begin` and `(;)`.
  * [Evaluación Condicional](@ref man-conditional-evaluation): `if`-`elseif`-`else` and `?:` (ternary operator).
  * [Evaluación en Cortocircuito](@ref): `&&`, `||` and chained comparisons. 
  * [Evaluación Repetida: Bucles](@ref man-loops): `while` and `for`.
  * [Manejo de Excepciones](@ref): `try`-`catch`, [`error()`](@ref) and [`throw()`](@ref).
  * [Tareas (también denominadas Coroutinas)](@ref man-tasks): [`yieldto()`](@ref).

Los cinco primeros mecanismos de control de flujo son estándar en los lenguajes de programación 
de alto nivel. Las tareas no son un mecanismo tan estándar: ellas proporcionan control de flujo 
no local, haciendo posible conmutar entre cálculos suspendidos temporalmente. Esta es una  
construcción potente: tanto el manejo de excepciones como la multitarea cooperativa se 
implementan en Julia usando tareas. La programación diaria no suele requerir el uso de tareas, 
pero ciertos problemas se resuelve de forma mucho más sencilla usando este mecanismo. 

## [Expresiones Compuestas](@id man-compound-expressions)

Algunas veces es conveniente tener una sola expresión que lleva nueve varias subexpresiones 
en orden, devolviendo el valor de la última subexpresión como su valor. Hay dos construcciones 
en Julia que llevan a cabo este trabajo: los bloques `begin` y las cadenas `;` El valor de 
ambas expresiones compuestas es el de la última subexpresión. He aquí un ejemplo del bloque `begin`:

```jldoctest
julia> z = begin
           x = 1
           y = 2
           x + y
       end
3
```

Como estas expresiones son bastante pequeñas, podrían ponerse con facilidad en una sola línea. 
Esta es precisamente la finalidad del `;`

```jldoctest
julia> z = (x = 1; y = 2; x + y)
3
```

Esta sintaxis es particularmente útil con la definición de función de una línea que introdujimos 
en el tema [Funciones](@ref). Aunque es típico, no hay obligación de que los bloques `begin` sean 
multilínea o de que las cadenas de punto y coma (`;`) tengan una única línea.

```jldoctest
julia> begin x = 1; y = 2; x + y end
3

julia> (x = 1;
        y = 2;
        x + y)
3
```

## [Conditional Evaluation](@id man-conditional-evaluation)

La evaluación condicional permite que porciones de código sean evaluadas o no evaluadas dependiendo 
del valor de una expresión booleana. Esta es la anatomía de la estructura de  `if`-`elseif`-`else`:

```julia
if x < y
    println("x is less than y")
elseif x > y
    println("x is greater than y")
else
    println("x is equal to y")
end
```

En el ejemplo anterior, si la condición `x<y` es verdadera, entonces se evaluará el bloque 
correspondiente. En caso contrario se evaluará la expresión `x>y`, y si esta es verdadera, se 
ejecutará el bloque correspondiente. Si la expresión también es falsa, se ejecutaría el bloque 
correspondiente al `else`. Veámoslo en acción:

```jldoctest
julia> function test(x, y)
           if x < y
               println("x is less than y")
           elseif x > y
               println("x is greater than y")
           else
               println("x is equal to y")
           end
       end
test (generic function with 1 method)

julia> test(1, 2)
x is less than y

julia> test(2, 1)
x is greater than y

julia> test(1, 1)
x is equal to y
```

Los bloques `elsif` y `else` son opcionales, y además pueden usarse tantos `elsif` como se 
deseen. Las expresiones condicionales del `if-elsif-else` serán evaluadas hasta que una de 
ellas se evalúe a `true`, después de lo cuál se evaluará el blqoue asociado, y ya no se 
evaluarán más expresiones condicionales.

Los bloques `if` son "permeables", es decir, no introducen un ámbito local. Eso significa 
que las variables que se definen dentro del bloque serán visibles fuera del mismo. Por 
tanto, podríamos haber definido la relación `test` de antes como...

```jldoctest
julia> function test(x,y)
           if x < y
               relation = "less than"
           elseif x == y
               relation = "equal to"
           else
               relation = "greater than"
           end
           println("x is ", relation, " y.")
       end
test (generic function with 1 method)

julia> test(2, 1)
x is greater than y.
```

La variable `relation` se ha declarado dentro del bloque `if`,  pero se usa fuera. Sin 
embargo, cuando se hace uso de este tipo de variables, hay que asegurarse de que todos los 
caminos de código definen un valor para la variable. La siguiente función no lo tiene en 
cuenta y genera un error en tiempo de ejecución.

```jldoctest
julia> function test(x,y)
           if x < y
               relation = "less than"
           elseif x == y
               relation = "equal to"
           end
           println("x is ", relation, " y.")
       end
test (generic function with 1 method)

julia> test(1,2)
x is less than y.

julia> test(2,1)
ERROR: UndefVarError: relation not defined
Stacktrace:
 [1] test(::Int64, ::Int64) at ./none:7
```

Los bloques `if` también devuelven un valor, lo que puede no parecer intuitivo para quienes 
proceden de otros lenguajes de programación no funcionales. Este valor no es más que el  
devuelto por la última instrucción en la rama que fue elegida. Por tanto:

```jldoctest
julia> x = 3
3

julia> if x > 0
           "positive!"
       else
           "negative..."
       end
"positive!"
```

Nótese que las instrucciones condicionales muy cortas (de una línea) se suelen expresar en 
Julia mediante evaluación en cortocircuito, como se verá en la siguiente sección.

A diferencia de C, MATLAB, Perl, Python y Ruby (pero como en Java y en otros lenguajes 
tipados, más estrictos) en Julia se produce un error si el valor de una expresión condicional 
es algo que no sea `true` o `false`.

```jldoctest
julia> if 1
           println("true")
       end
ERROR: TypeError: non-boolean (Int64) used in boolean context
```

This error indicates that the conditional was of the wrong type: [`Int64`](@ref) rather
than the required [`Bool`](@ref).

Este error indica que el condicional fue de un tipo incorrecto. 

El llamado *operador ternario* (`?`) está muy relacionado con la sintaxis de `if-elsif-else`, 
pero se usa donde hay que hacer una elección condicional entre expresiones sencillas, a 
diferencia de la ejecución condicional de grandes bloques de código. Su nombre se debe a que 
es el único operador que toma tres operandos en la mayoría de los lenguajes de programación:

```julia
a ? b : c
```

La expresión `a` delante del `?` es una expresión condicional, y la operación ternaria evalúa 
la expresión `b` (la que está delante del símbolo `:`)  si la condición `a` es `true` o la 
expresión `c` si la condición `a` es `false`. Nótese que los espacios alrededor de `?` y `:` 
çson obligatorios: una expresión como `a?b:c` no es una expresin ternaria válida (aunque se
pueden utilizar saltos de línea entre los símbolos `?` y `:`).

La forma más fácil de comprender este comportamiento es ver un ejemplo. En el ejemplo anterior,
la llamada a `println` es compartida por las tres ramas: la única elección real es qué cadena
literal imprimir. Esto podría haberse escrito de forma más concisa usando el operador ternario.
En aras de la claridad, intentemos primero la versin con dos caminos:

```jldoctest
julia> x = 1; y = 2;

julia> println(x < y ? "less than" : "not less than")
less than

julia> x = 1; y = 0;

julia> println(x < y ? "less than" : "not less than")
not less than
```

En los ejemplos anteriores, si `x < y` es `true` se devolverá la cadena `"less than"` y, en caso 
contrario, se devolverá la cadena `"not less than"`. El ejemplo original, que tiene tres opciones, 
requeriría el uso encadenado del operador `?`:

```jldoctest
julia> test(x, y) = println(x < y ? "x is less than y"    :
                            x > y ? "x is greater than y" : "x is equal to y")
test (generic function with 1 method)

julia> test(1, 2)
x is less than y

julia> test(2, 1)
x is greater than y

julia> test(1, 1)
x is equal to y
```

Para facilitar el encadenamiento, el operador `?` asocia de derecha a izquierda.

Es también significativo que, como en la construcción `if-elsif-else` las expresiones anterior y 
posterior al símbolo `:` sólo se evalúan si la expresión condicional es evaluada a `true` o `false`, 
respectivamente.

```jldoctest
julia> v(x) = (println(x); x)
v (generic function with 1 method)

julia> 1 < 2 ? v("yes") : v("no")
yes
"yes"

julia> 1 > 2 ? v("yes") : v("no")
no
"no"
```

## Evaluación en Cortocircuito

La evaluación en cortocircuito es bastante similar a la evaluación condicional. Este comportamiento 
aparece en la mayoría de los lenguajes de programación imperativos que tiene los operadores booleanos 
`&&` y `||`.  En una serie de expresiones booleanas conectadas por estos operadores, sólo se evalúa 
el número mínimo de expresiones necesarios para determinar el valor booleano final de la cadena 
completa. Explícitamente, esto significa que:

* En la expresión `a && b` la subexpresión `b` sólo se evalúa si la subexpresión `a` es evaluada a 
`true`.
* En la expresión `a || b` la subexpresión `b` sólo se evalúa si la subexpresión `a` es evaluada a 
`false`.

El razonamiento es que `a && b` debe ser `false` si `a` is `false`, independientemente del valor de `b`
y, análogamente, el valor de `a || b` debe ser cierto si `a` es `true`, independientemente del valor de
`b`. Tanto `&&` como `||` asocian a la derecha, pero `&&` tiene mayore precedencia que `||`. Es fácil 
experimentar con este comportamiento:

```jldoctest tandf
julia> t(x) = (println(x); true)
t (generic function with 1 method)

julia> f(x) = (println(x); false)
f (generic function with 1 method)

julia> t(1) && t(2)
1
2
true

julia> t(1) && f(2)
1
2
false

julia> f(1) && t(2)
1
false

julia> f(1) && f(2)
1
false

julia> t(1) || t(2)
1
true

julia> t(1) || f(2)
1
true

julia> f(1) || t(2)
1
2
true

julia> f(1) || f(2)
1
2
false
```

Se puede experimentar de forma parecida con la asociatividad y la precedencia de varias combinaciones de 
operadores `&&` y `||`.

Este comportamiento se utiliza en Julia con frecuencia para formar una alternativa a instrucciones `if` 
muy cortas. En lugar de usar la construcción `if <cond> && <instrucción>` uno puede escribir `<cond> && <instrucción>` 
que puede leerse como `<cond>` y entonces `<instrucción>`. de forma similar, uno puede escribir 
`<cond> || <instrucción>`, que se leería como `<cond>` o sino `<instrucción>` en lugar de `if !<cond> || <instrucción>`.

Por ejemplo, una rutina recursiva para obtener un factorial podría ser definida como:

```jldoctest
julia> function fact(n::Int)
           n >= 0 || error("n must be non-negative")
           n == 0 && return 1
           n * fact(n-1)
       end
fact (generic function with 1 method)

julia> fact(5)
120

julia> fact(0)
1

julia> fact(-1)
ERROR: n must be non-negative
Stacktrace:
 [1] fact(::Int64) at ./none:2
```

Las operaciones booleanas sin cortocircuito podrían llevarse a cabo con los operadores a nivel de bit 
introducidos en la sección [Operaciones Matemáticas y Funciones Elementales](@ref): `&` and `|`. Estas 
son funciones normales, que suportan la sintaxis infija de los operadores, pero que siempre evalúan 
sus argumentos:

```jldoctest tandf
julia> f(1) & t(2)
1
2
false

julia> t(1) | t(2)
1
2
true
```

Como en el caso de las expresiones condicionales usadas en `if`, `elsif` o el operador 
ternario `?`, los operandos de `&&` y de `||` deben ser valores booleanos.  Usar un valor
no booleanos en cualquier lugar distinto de la última entrada en una cadena condicional 
producirá un error.

```jldoctest
julia> 1 && true
ERROR: TypeError: non-boolean (Int64) used in boolean context
```

Por otra parte, cualquire tipo de expresión puede ser usada al final de una cadena 
condicional. Ella será evaluada y devuelta dependiendo de los condicionales precedentes:

```jldoctest
julia> true && (x = (1, 2, 3))
(1, 2, 3)

julia> false && (x = (1, 2, 3))
false
```

## [Evaluación Repetida: Bucles](@id man-loops)

Hay dos construcciones que realizan la evaluación repetida de expresiones: el bucle 
`while` y el bucle `for`. He aquí un ejemplo del bucle `while`:

```jldoctest
julia> i = 1;

julia> while i <= 5
           println(i)
           i += 1
       end
1
2
3
4
5
```

El bucle `while` evalúa la expresión condicional (en el ejemplo `i<=5`) y, mientras que 
esta se evalúe a `true`, sigue evaluando el cuerpo del bucle `while`. Si la expresión se 
evalúa a `false` la primera vez en que se alcanza el bucle, su cuerpo nunca será evaluado.

El bucle `for` facilita la repetición. Dado que contar arriba y abajo (como en el ejemplo 
anterior del bucle `while`) es tan común, podemos expresar esto de una forma muy concisa 
con un bucle `for`:

```jldoctest
julia> for i = 1:5
           println(i)
       end
1
2
3
4
5
```

En el ejemplo anterior, `1:5` es un objeto `Range` que representa una secuencia de números. 
El bucle `for` itera sobre estos valores, asignando cada uno de ellos por turno a la variable
`i`.  Una distinción importante ente esta construcción (`for`) y la construcción anterior 
(`while`) es el ámbito durante el cuál la variable es visible. Si la variable `i` no ha 
sido introducida en otro ámbito, el bucle `for` la verá sólo en su interior y no 
posteriormente. Para demostrar esto necesitaremos una nueva sesión interactiva o usar un 
nombre de variable distinto:

```jldoctest
julia> for j = 1:5
           println(j)
       end
1
2
3
4
5

julia> j
ERROR: UndefVarError: j not defined
```

Ver [Ámbito de Variables](@ref scope-of-variables) para una explicación detallada de los ámbitos
de las variables y cómo funcionan en Julia.

En general, la construcción `for` puede iterar sobre cualquier contenedor. En estos casos, la 
palara clave alternativa (pero totalmente equivalente `in` o `∈` es usada en lugar de `=`, dado 
que hace que la lectura del código sea más clara.

```jldoctest
julia> for i in [1,4,0]
           println(i)
       end
1
4
0

julia> for s ∈ ["foo","bar","baz"]
           println(s)
       end
foo
bar
baz
```

En otras secciones del manual se introducirán y discutirán varios tipos de contenedores iterables 
(ver, por ejemplo, [Arrays Multi-dimensionales](@ref man-multi-dim-arrays)).

Algunas veces es conveniente terminar la repetición de un `while` antes de chequear la condición 
de test o partar de iterar en un bucle `for` antes de que se alcance el final del objeto iterable. 
Esto puede conseguirse usando la palabra clave `break`:

```jldoctest
julia> i = 1;

julia> while true
           println(i)
           if i >= 5
               break
           end
           i += 1
       end
1
2
3
4
5

julia> for i = 1:1000
           println(i)
           if i >= 5
               break
           end
       end
1
2
3
4
5
```

Si no existiera la palabra clave `break`, el bucle `while` anterior nunca finalizará por si 
mismo,  y el bucle `for` iteraría hasta 10000. Si hacemos uso de la instrucción `break` 
conseguiremos abandonar el bucle mucho antes.

En otras circunstancias es útil ser capaz de detener una iteración y moverse a la siguiente 
de forma inmediata. Para ello, se utiliza la palabra clave `continue`:

```jldoctest
julia> for i = 1:10
           if i % 3 != 0
               continue
           end
           println(i)
       end
3
6
9
```

Este es un ejemplo un tanto artificial, ya que podríamos obtener el mismo comportamiento de 
forma mucho más clara negando las condiciones y colocando la llamada a `println` dentro del 
bloque `if`. En usos más reales hay más código que evaluar después del `continue`, y con 
frecuencia hay muchos puntos desde los que uno puede llamar a esta instrucción.

Podemos anidar múltiples bucles for en un solo bucle externo, formando el producto cartesiano 
de sus iterables:

```jldoctest
julia> for i = 1:2, j = 3:4
           println((i, j))
       end
(1, 3)
(1, 4)
(2, 3)
(2, 4)
```

Una instrucción `break` dentro de tal bucle sale del anidamiento de bucles completo, no sólo 
del más interior.

## Manejo de Excepciones

Cuando tiene lugar una condición inesperada, una funció puede ser incapaz de devolver un valor 
razonable al código que la invoca. En tales casos, puede ser mejor para la condición excepcional 
terminar el programa, imprimiendo un mensaje de error diagnóstico, o si el programador ha 
proporcionado código para manejar tales circunstancias excepcionales, permitiendo que el código 
tome la acción apropiada.

### Excepciones predefinidas

Las excepciones se lanzan cuando ocurre una condición inesperada. En la siguiente tabla se 
muestran todas la excepciones predefinidas, que interrumplen todas el flujo de control normal.

| `Exception`                   |
|:----------------------------- |
| [`ArgumentError`](@ref)       |
| [`BoundsError`](@ref)         |
| `CompositeException`          |
| [`DivideError`](@ref)         |
| [`DomainError`](@ref)         |
| [`EOFError`](@ref)            |
| [`ErrorException`](@ref)      |
| [`InexactError`](@ref)        |
| [`InitError`](@ref)           |
| [`InterruptException`](@ref)  |
| `InvalidStateException`       |
| [`KeyError`](@ref)            |
| [`LoadError`](@ref)           |
| [`OutOfMemoryError`](@ref)    |
| [`ReadOnlyMemoryError`](@ref) |
| [`RemoteException`](@ref)     |
| [`MethodError`](@ref)         |
| [`OverflowError`](@ref)       |
| [`ParseError`](@ref)          |
| [`SystemError`](@ref)         |
| [`TypeError`](@ref)           |
| [`UndefRefError`](@ref)       |
| [`UndefVarError`](@ref)       |
| `UnicodeError`                |

Por ejemplo, la función [`sqrt()`](@ref) lanza un [`DomainError`](@ref) si se aplica sobre un valor 
real negativo:

```jldoctest
julia> sqrt(-1)
ERROR: DomainError:
sqrt will only return a complex result if called with a complex argument. Try sqrt(complex(x)).
Stacktrace:
 [1] sqrt(::Int64) at ./math.jl:447
```

Uno puede definir sus propias excepciones de la siguiente manera:
```jldoctest
julia> struct MyCustomException <: Exception end
```

### La función [`throw()`](@ref)

Las excepciones pueden crearse explícitamente con  [`throw()`](@ref). Por ejemplo, una función 
definida sólo para número no negativos podría escribirse para que lanzara un
[`DomainError`](@ref) si el argumento es negativo:

```jldoctest
julia> f(x) = x>=0 ? exp(-x) : throw(DomainError())
f (generic function with 1 method)

julia> f(1)
0.36787944117144233

julia> f(-1)
ERROR: DomainError:
Stacktrace:
 [1] f(::Int64) at ./none:1
```

Notese que [`DomainError`](@ref) sin paréntesis no es una excepción, sino un tipo de excepción. 
Ella necesita ser invocada para obtener un objeto `Exception`:

```jldoctest
julia> typeof(DomainError()) <: Exception
true

julia> typeof(DomainError) <: Exception
false
```

Adicionalmente, algunos tipos de excepción toman uno o más argumentos que se utilizan para reportar 
errores.

```jldoctest
julia> throw(UndefVarError(:x))
ERROR: UndefVarError: x not defined
```

Este mecanismo puede ser fácilmente implementado mediante los tipos de excepción personalizados 
que sigan la forma en que se escribe [`UndefVarError`](@ref):

```jldoctest
julia> struct MyUndefVarError <: Exception
           var::Symbol
       end

julia> Base.showerror(io::IO, e::MyUndefVarError) = print(io, e.var, " not defined")
```

!!! note
    Cuando se escribe un mensaje de error, es preferible que la primera palabra sea minúscula. 
    Por ejemplo,
    
    `size(A) == size(B) || throw(DimensionMismatch("size of A not equal to size of B"))`

    es preferible a 

    `size(A) == size(B) || throw(DimensionMismatch("Size of A not equal to size of B"))`.

    Sin embargo, algunas veces tiene sentido mantener la primera letra en mayúscula, por 
    ejemplo, si un argumento a función es una letra mayúscula: 
    `size(A,1) == size(B,2) || throw(DimensionMismatch("A has first dimension..."))`.

### Errors

La función [`error()`](@ref) se usa para producir una [`ErrorException`](@ref) que interrumpe el
flujo de control normal.

Supóngase que deseamos detener la ejecución inmediatamente si se toma la raíz cuadrad de un número 
negativo. Para hacer ésto, podemos definir una versión "quisquillosa" de la función  [`sqrt()`](@ref) 
que lanza un error si recibe un número negativo:

```jldoctest fussy_sqrt
julia> fussy_sqrt(x) = x >= 0 ? sqrt(x) : error("negative x not allowed")
fussy_sqrt (generic function with 1 method)

julia> fussy_sqrt(2)
1.4142135623730951

julia> fussy_sqrt(-1)
ERROR: negative x not allowed
Stacktrace:
 [1] fussy_sqrt(::Int64) at ./none:1
```

Si `fussy_sqrt()` es invocada con un valor negativo desde otra función, en lugar de intentar 
continuar la ejecución de la función que la invocó, retorna inmediatamente, mostrando el mensaje 
de error en la sesión interactiva.:

```jldoctest fussy_sqrt
julia> function verbose_fussy_sqrt(x)
           println("before fussy_sqrt")
           r = fussy_sqrt(x)
           println("after fussy_sqrt")
           return r
       end
verbose_fussy_sqrt (generic function with 1 method)

julia> verbose_fussy_sqrt(2)
before fussy_sqrt
after fussy_sqrt
1.4142135623730951

julia> verbose_fussy_sqrt(-1)
before fussy_sqrt
ERROR: negative x not allowed
Stacktrace:
 [1] fussy_sqrt at ./none:1 [inlined]
 [2] verbose_fussy_sqrt(::Int64) at ./none:3
```

### Warnings and informational messages

Julia también proporciona otras funciones que escriben mensajes a la salida de error 
estándar, pero no lanzan ninguna `Exception` y, por tanto, no interrumpen la ejecución:

```jldoctest
julia> info("Hi"); 1+1
INFO: Hi
2

julia> warn("Hi"); 1+1
WARNING: Hi
2

julia> error("Hi"); 1+1
ERROR: Hi
Stacktrace:
 [1] error(::String) at ./error.jl:21
```

### La instrucción `try/catch` 

La intrucción `try/ catch` permite comprobar a aparición de excepciones. Por ejemplo, 
puede escribirse una función personalizada para calcular la raíz cuadrada que invoque 
automáticamente al método de cálculo de la raíz de valores reales y/o complejos en 
función de la excepción:

```jldoctest
julia> f(x) = try
           sqrt(x)
       catch
           sqrt(complex(x, 0))
       end
f (generic function with 1 method)

julia> f(1)
1.0

julia> f(-1)
0.0 + 1.0im
```

Es importante notar que en el código real que computa esta función, uno podría comparar `x` 
con vero en lugar de atrapar la excepción. De hcho, la opción de la excepción ees mucho 
más lenta de comparar y ramificar.

Las instrucciones `try / catch` también permiten salvar la excepción en una variable. En 
este ejemplo artificial, se calcula la raíz cuadrada del segundo elemento de `x`.  Si `x` 
es indexavle, en caso contrario asume que `x` es un número real y devuelve su raíz cuadrada:

```jldoctest
julia> sqrt_second(x) = try
           sqrt(x[2])
       catch y
           if isa(y, DomainError)
               sqrt(complex(x[2], 0))
           elseif isa(y, BoundsError)
               sqrt(x)
           end
       end
sqrt_second (generic function with 1 method)

julia> sqrt_second([1 4])
2.0

julia> sqrt_second([1 -4])
0.0 + 2.0im

julia> sqrt_second(9)
3.0

julia> sqrt_second(-9)
ERROR: DomainError:
Stacktrace:
 [1] sqrt_second(::Int64) at ./none:7
```

Note que el símbolo que sigue al `catch` siempre será interpretado como el nombre 
para la excepción, por lo que hay que tener cuidado cuando se escriben expresiones
`try / catch` en una sola línea. El siguiente código no funcionará para devolver el
valor de `x` en caso de error:

```julia
try bad() catch x end
```

En su lugar, es mejor usar un punto u coma o insertar un salto de línea después del `catch`:

```julia
try bad() catch; x end

try bad()
catch
    x
end
```

La cláusula `catch` no es estrictamente necesaria; cuando se omite el valor de retorno 
por defecto es `nothing`.

```jldoctest
julia> try error() end # Returns nothing
```

La potencia de la construcción `try / catch` estriba  en la capacidad de desplegar inmediatamente
un  cálculo profundamente anidado de hasta un nivel mucho más elevado en la pila de llamadas a
función. Hay situacionces donde no ha ocurrido error, pero la capacidad de desplegar la pila 
y pasar un valor a un nivel superior es deseable. Julia proporciona las funciones 
 [`rethrow()`](@ref), [`backtrace()`](@ref) and [`catch_backtrace()`](@ref) para un manejo de
 errores más avanzado.

### Cláusulas `finally` 

En código que realiza cambios de estado o usa recursos como ficheros, hay típicamente un trabajo 
de limpieza (tal como cerrar ficheros) que necesita ser realizado cuando el código finaliza. Las 
excepciones complican potencialmente esta tarea, ya que pueden causar que un bloque de código salga 
antes de alcanzar su final normal. La palabra clave `finally` proporciona una forma de ejecutar 
código cuando existe un blqoue de código dado, sin preocuparse de cómo salga.

Por ejemplo, aquí podemos garantizar que un fichero abierto se cierra:

```julia
f = open("file")
try
    # operate on file f
finally
    close(f)
end
```

Cuando el contro deja el bloque `try` (por ejemplo, debido a un `return`, o finalizando normalmente) 
se ejecutará `close()`.  Si el bloque `try` saliera debido a una excepción, la excepción continuará 
propagándose. Un bloque `catch` puede ser combinada con `try` y `finally` también. En este caso el
bloque `finally` ejecutará después de que `catch` haya manejado el error.

## [Tareas (aka Corutinas)](@id man-tasks)

Las tareas son una característica de control de flujo que permite que los cálculos sean 
suspendidos y continuados de una forma flexible. Esta característica es llamada algunas veces con
otros nombres, tales como corrutinas simétricas, hilos de peso ligero, multitarea cooperativa o
continuaciones de un disparo.

Cuando una pieza de trabajo de cómputo (en la práctica, ejecutar una función particular) es designada
como tarea ([`Task`](@ref)), se hace posible interrumplirla intercambiándola por otra tarea. La tarea 
original puede ser continuada después, en el punto en que se encontraba justo cuando fue detenida. A 
primera vista, esto puede parecer similar a una llamada a función. Sin embargo, hay dos diferencias 
clave. Primero, conmutar tareas no usa ningún espacio, por lo que puede tener lugar cualquier número
de intercambios de tarea sin que se consuma la pila de llamadas. Segundo, conmutar entre tareas puede
ocurrir en cualquier orden, a diferencia de lo que pasa en las llamadas a función, donde la función 
invocada debe terminar la ejecución antes de que el control retorne a la función que la llamó.

Esta clase de flujo de control puede hacer mucho más fácil resolver ciertos problemas. En algunos
problemas, las distintas piezas de trabajo requerido no están relacionadas naturalmente mediante 
llamadas a función: no hay un obvio llamador o llamado entre los trabajos que necesitan ser 
realizados. Un ejemplo es el problema del productor-consumidor, donde un procedimiento complejo 
está generando valores u otro procedimiento complejo los está consumiendo. El consumidor no puede
simplemente llamar a la función productora para obtener un valor, debido a que el productor puede 
tener más valores que generar y, por tanto, podría no estar listo todavía para retornar. Con las 
tareas, el productor y el consumidor pueden ambos ejecutarse mientas que lo necesiten, pasando 
valores adelante y detrás cuando sea necesario.

Julia proporciona un mecanismo denominado "canal" ([`Channel`](@ref) para resolver este problema. 
Un canal es una cola FIFO (primero en entrar, primero en salir) que puede tener múltiples tareas 
leyendo de y escribiendo en ella. 

Definamos una tarea productor, que produce valores a través de una llamada  [`put!`](@ref).
Para consumir valores, necesitamos planificar un productor que ejecute una nueva tarea. 
Para ejecutar una tarea asociada a un canal utilizaremos un constructor especial [`Channel`](@ref)
que recibe como argumento una función de un argumento. Podemos entonce tomar valores repetidamente 
del objeto canar mediante llamadas a [`take!()`](@ref):

```jldoctest producer
julia> function producer(c::Channel)
           put!(c, "start")
           for n=1:4
               put!(c, 2n)
           end
           put!(c, "stop")
       end;

julia> chnl = Channel(producer);

julia> take!(chnl)
"start"

julia> take!(chnl)
2

julia> take!(chnl)
4

julia> take!(chnl)
6

julia> take!(chnl)
8

julia> take!(chnl)
"stop"
```

Una forma de pensar en este comportamiento es que el `producer` era capaz de retornar múltiples
veces. Entre las llamadas a [`put!()`](@ref), la ejecución del productor se ha suspendido y el 
consumidor tiene el control.

El objeto [`Channel`](@ref) devuelto puede ser usado como un objeto iterable dentro de un bucle
`for` loop,  en cuyo caso las variable del bucle tomará todos los objetos producidos. El bucle 
será terminado cuando el canal se haya cerrado.

```jldoctest producer
julia> for x in Channel(producer)
           println(x)
       end
start
2
4
6
8
stop
```

Note que nosotros no tuvimos que cerrar explícitamente el canal en el productor. Esto es
debido a que el acto de enlazar un canal ([`Channel`](@ref)) a una tarea ([`Task()`](@ref)) 
asocia el tiempo de vida abierto de un canal con el de la tarea asociada. El objeto canal 
se cierra automáticamente cuando la tarea termina. Podemos enlazar múltiplos canales a una
tarea, y viceversa.

Aunque el constructor de [`Task()`](@ref) espere una función sin argumentos, el método 
[`Channel()`](@ref) que crea un enlace entre un canal y una tarea espera una función que 
acepta un solo argumento de tipo [`Channel`](@ref). Un patrón común es que el productor
esté parametrizado, en cuyo caso se neceesita una aplicación de función parcial para crear 
una [función anónima](@ref man-anonymous-functions) con 1 ó 0 argumentos..

Para objetos [`Task()`](@ref) esto puede hacerse bien directamente o mediante el uso de una
macro conveniente:

```julia
function mytask(myarg)
    ...
end

taskHdl = Task(() -> mytask(7))
# or, equivalently
taskHdl = @task mytask(7)
```

Para orquestar patrones de distribucióin más avanzados, pueden usarse [`bind()`](@ref) y 
[`schedule()`](@ref) en conjunción con los constructores de [`Task()`](@ref) y [`Channel()`](@ref)
para enlazar explícitamente un conjunto de canales con un conjunto de tareas productor/consumidor.

Note que en la actualidad las tareas Julia no son planificadas para que ejecuten sobre núcleos de CPU separados. Los verdaderos hilos del núcleo se discutirán en la sección [Computación Paralela](@ref).

### Core task operations

Let us explore the low level construct [`yieldto()`](@ref) to underestand how task switching works.
`yieldto(task,value)` suspends the current task, switches to the specified `task`, and causes
that task's last [`yieldto()`](@ref) call to return the specified `value`. Notice that [`yieldto()`](@ref)
is the only operation required to use task-style control flow; instead of calling and returning
we are always just switching to a different task. This is why this feature is also called "symmetric
coroutines"; each task is switched to and from using the same mechanism.

[`yieldto()`](@ref) is powerful, but most uses of tasks do not invoke it directly. Consider why
this might be. If you switch away from the current task, you will probably want to switch back
to it at some point, but knowing when to switch back, and knowing which task has the responsibility
of switching back, can require considerable coordination. For example, [`put!()`](@ref) and [`take!()`](@ref)
are blocking operations, which, when used in the context of channels maintain state to remember
who the consumers are. Not needing to manually keep track of the consuming task is what makes [`put!()`](@ref)
easier to use than the low-level [`yieldto()`](@ref).

In addition to [`yieldto()`](@ref), a few other basic functions are needed to use tasks effectively.

  * [`current_task()`](@ref) gets a reference to the currently-running task.
  * [`istaskdone()`](@ref) queries whether a task has exited.
  * [`istaskstarted()`](@ref) queries whether a task has run yet.
  * [`task_local_storage()`](@ref) manipulates a key-value store specific to the current task.

### Tasks and events

Most task switches occur as a result of waiting for events such as I/O requests, and are performed
by a scheduler included in the standard library. The scheduler maintains a queue of runnable tasks,
and executes an event loop that restarts tasks based on external events such as message arrival.

The basic function for waiting for an event is [`wait()`](@ref). Several objects implement [`wait()`](@ref);
for example, given a `Process` object, [`wait()`](@ref) will wait for it to exit. [`wait()`](@ref)
is often implicit; for example, a [`wait()`](@ref) can happen inside a call to [`read()`](@ref)
to wait for data to be available.

In all of these cases, [`wait()`](@ref) ultimately operates on a [`Condition`](@ref) object, which
is in charge of queueing and restarting tasks. When a task calls [`wait()`](@ref) on a [`Condition`](@ref),
the task is marked as non-runnable, added to the condition's queue, and switches to the scheduler.
The scheduler will then pick another task to run, or block waiting for external events. If all
goes well, eventually an event handler will call [`notify()`](@ref) on the condition, which causes
tasks waiting for that condition to become runnable again.

A task created explicitly by calling [`Task`](@ref) is initially not known to the scheduler. This
allows you to manage tasks manually using [`yieldto()`](@ref) if you wish. However, when such
a task waits for an event, it still gets restarted automatically when the event happens, as you
would expect. It is also possible to make the scheduler run a task whenever it can, without necessarily
waiting for any events. This is done by calling [`schedule()`](@ref), or using the [`@schedule`](@ref)
or [`@async`](@ref) macros (see [Parallel Computing](@ref) for more details).

### Task states

Tasks have a `state` field that describes their execution status. A [`Task`](@ref) `state` is one of the following
symbols:

| Symbol      | Meaning                                            |
|:----------- |:-------------------------------------------------- |
| `:runnable` | Currently running, or available to be switched to  |
| `:waiting`  | Blocked waiting for a specific event               |
| `:queued`   | In the scheduler's run queue about to be restarted |
| `:done`     | Successfully finished executing                    |
| `:failed`   | Finished with an uncaught exception                |
