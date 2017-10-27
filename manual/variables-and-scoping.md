# [Ámbito de las variables](@id scope-of-variables)

El *ámbito* de una variable es la región de código donde dicha variable es visible. El ámbito de las 
variables ayuda a evitar conflictos de nombrado de variables. El concepto es intuitivo: dos funciones
pueden tener argumentos denominados `x` sin que las dos `x` se refieran a la misma cosa. De forma 
similar, hay muchos otros casos donde diferentes bloques de código pueden usar el mismo nombre sin 
referirse a la misma cosa. Las reglas para cuando el mismo nombre de variable se refiere o no a la 
misma cosa se llaman *reglas de ámbito*. Em este tema se analizan en detalle. 

Ciertas construcciones en el lenguaje introducen *bloques de ámbitos*, que son regiones de código 
que son elegibles para estar en el ámbito de algún conjunto de variables. El ámbito de una variable 
no puede ser un conjunto arbitrario de líneas de código; en lugar de ello, siempre se alinea con uno
de esos bloques. Hay dos tipos principales de ámbitos en Julia: *globales* y *locales*, pudiendo los
últimos estar anidados. Las construcciones que introducen estos bloques de ámbito son:

|Nombre de ámbito | Bloque/construcción que introduce este tipo de ámbito                                             |
|:---------------|:---------------------------------------------------------------------------------------------------|
| Ámbito Global  | `module`, `baremodule` y prompt interactivo (REPL)                                                 |
| Ámbito Local   | [Ámbito local blando](@ref): `for`, `while` comprensiones, bloques `try-catch-finally`, `let`      |
| Ámbito Local   | [Ámbito local duro](@ref): funciones (cualquier sintaxis, anónima y bloques do), `struct`, `macro` |


Dos notables ausencias en esta tabla son los [bloques begin](@ref man-compound-expressions) y los
[bloques if](@ref man-conditional-evaluation), que no introducen nuevos bloques de ámbito. Los tres
tipos de bloques siguen reglas un poco diferentes que serán explicadas más adelante, así como algunas 
reglas extra para ciertos bloques.

Julia usa un [ámbito léxico](https://en.wikipedia.org/wiki/Scope_%28computer_science%29#Lexical_scoping_vs._dynamic_scoping),
lo que significa que el ámbito de una función no hereda del ámbito que lo invocó, pero si del ámbito 
en que la función fue definida. Por ejemplo, en el siguiente código, `x` deontro de `foo` se refiere 
a la `x` que hay en el ámbito local de su módulo `Bar`:

```jldoctest moduleBar
julia> module Bar
           x = 1
           foo() = x
       end;
```

y no a la `x` en el ámbito en que se ha usado `foo`:

```jldoctest moduleBar
julia> import .Bar

julia> x = -1;

julia> Bar.foo()
1
```

Por tanto, *ámbito léxico* significa que el ámbito de las variables puede ser inferido del código 
fuente sin más. 

## Ámbito Global

*Cada módulo introduce un nuevo espacio global*, separado del ámbito global de todos los otros 
módulos; no existen ámbitos globales compartidos por todos. Los módulos pueden introducir variables 
de otros módulos o en su ámbito a través del uso de las instrucciones [`using`](@ref modules) o 
[`import`](@ref modules) o a través de acceso cualificado usando la notación punto. En consecuencia, 
cada módulo es un espacio de nombres. Notese que los enlaces de nombres pueden sólo ser cambiados 
dentro de su ámbito global y no desde un módulo exterior.

```jldoctest
julia> module A
           a = 1 # a global in A's scope
       end;

julia> module B
           module C
               c = 2
           end
           b = C.c    # can access the namespace of a nested global scope
                      # through a qualified access
           import ..A # makes module A available
           d = A.a
       end;

julia> module D
           b = a # errors as D's global scope is separate from A's
       end;
ERROR: UndefVarError: a not defined

julia> module E
           import ..A # make module A available
           A.a = 2    # throws below error
       end;
ERROR: cannot assign variables in other modules
```

Nótese que el prompt interactivo (REPL) está en el ámbito global del módulo `Main`.

## Ámbito Local

La mayoría de los bloques de código introducen un nuevo ámbito local. Los ámbitos locales 
suelen heredar todas las variables de su ámbito padre, tanto para lectura como para escritura. 
Hay dos subtipos de ámbitos locales, denominados *duros* y *blandos*, con reglas ligeramente 
distintas en relación a qué variables son heredadas. A diferencia de los ámbitos globales, los 
ámbitos locales no son espacios de nombres, por lo que las variables de un ámbito más interno 
no pueden ser recuperadas de uno más externo a través de alguna clase de acceso cualificado.

Las siguientes reglas y ejemplos pertenecen tanto a los ámbitos locales como globales. Una 
variable introducida de nuevo en un ámbito local no se retroprogada a su ámbito padre. Por 
ejemplo, la `z` no se introduce en el ámbito de nivel superior:

```jldoctest
julia> for i = 1:10
           z = i
       end

julia> z
ERROR: UndefVarError: z not defined
```

(Nótese que En este ejemplo y los siguientes se supone que el ámbito de nivel superior es un ámbito 
global con un espacio de trabajo limpio, por ejemplo un REPL arrancado de nuevo.)

Dentro de un ámbito local una variable puede ser forzada a ser una variables local usando la palabra 
clave `local`.

```jldoctest
julia> x = 0;

julia> for i = 1:10
           local x
           x = i + 1
       end

julia> x
0
```

Dentro de un ámbito local puede definirse una nueva variable global usando la palabra clave `global`:

```jldoctest
julia> for i = 1:10
           global z
           z = i
       end

julia> z
10
```
La localización de las palabras clave `local` y  `global` dentro del bloque del ámbito es irrelevante. 
El siguiente código es totalmente equivalente al ejemplo anterior: 

```jldoctest
julia> for i = 1:10
           z = i
           global z
       end

julia> z
10
```

Las palabras clave `local` y `global` pueden también usarse en asignaciones desestructuradas, por ejemplo,
`local x, y = 1, 2`. En este caso la palabra clave afecta a toas las variables listadas.

### Ámbito local blando

En el ámbito local blando, todas las variables son heredadas de su ámbito padre a menos que una 
variable haya sido marcada específicamente con la palabra `local`.

Los ámbitos locales blandos se introducen en los bucles `for`, bucles `while`, comprensiones, bloques 
`try-catch-finally` y bloques `let`. Hay algunas reglas extra para los bloques `let` y para los bucles 
`for` y comprensiones.

En el siguiente ejemplo, `x` e `y` se refieren siempre a la misma variable dado que el ámbito local 
blando heredan ambas variables de lectura y escritura:

```jldoctest
julia> x, y = 0, 1;

julia> for i = 1:10
           x = i + y + 1
       end

julia> x
12
```

Dentro de los ámbitos blandos, la palabra clave *global* no es nunca necesaria, aunque está permitida. 
El único caso donde podría cambiar la semántica es en un error sintáctico:

```jldoctest
julia> let
           local j = 2
           let
               global j = 3
           end
       end
ERROR: syntax: `global j`: j is local variable in the enclosing scope
```

### Ámbito local duro

Los ámbitos locales duros se introducen mediante las definiciones de función (en todas sus formas) 
bloques de tipos e inmutables y definiciones de macros.

> En el ámbito local duro, todas las variables son heredades de su ámbito padre a menos que:
>
>   * Una asignación daría como resultado una variable `global`
>   * Una variable sea marcada específicamente con la palabra clave `local`.

Por tanto, las variables globales son sólo heredadas para lectura pero no para escritura::

```jldoctest
julia> x, y = 1, 2;

julia> function foo()
           x = 2        # assignment introduces a new local
           return x + y # y refers to the global
       end;

julia> foo()
4

julia> x
1
```

Se necesita un `global` explícito para asignar a una variable global:

```jldoctest
julia> x = 1;

julia> function foobar()
           global x = 2
       end;

julia> foobar();

julia> x
2
```

Note que las *funciones anidadas* pueden comportarse diferentemente de las funciones definidas 
en el ámbito global como si ellas pudieran variar las variables locales del ámbito padre.

```jldoctest
julia> x, y = 1, 2;

julia> function baz()
           x = 2 # introduces a new local
           function bar()
               x = 10       # modifies the parent's x
               return x + y # y is global
           end
           return bar() + x # 12 + 10 (x is modified in call of bar())
       end;

julia> baz()
22

julia> x, y
(1, 2)
```

La distinción entre heredar variables locales y globales para asignación puede llevar a ligeras 
diferencias entre funciones definidas en ámbitos locales y/o globales. Considere la modificación
del último ejemplo moviendo `bar` al ámbito global:

```jldoctest
julia> x, y = 1, 2;

julia> function bar()
           x = 10 # local
           return x + y
       end;

julia> function quz()
           x = 2 # local
           return bar() + x # 12 + 2 (x is not modified)
       end;

julia> quz()
14

julia> x, y
(1, 2)
```

Notemos que lo anterior sutilmente no pertenece a las definiciones de tipo y de macro, por lo que 
ellas sólo pueden aparecer en el ámbito global. Hay reglas de ámbito especiales relacionadas con 
la evaluación de argumentos de función por defecto y palabra clave que se describen en la sección 
de [Funciones](@ref man-functions).

Una asignación que introduce una variable usada dentro de una definicíon de función, tipo o macro 
no necesita ir antes de su uso interno: 

```jldoctest
julia> f = y -> y + a
(::#1) (generic function with 1 method)

julia> f(3)
ERROR: UndefVarError: a not defined
Stacktrace:
 [1] (::##1#2)(::Int64) at ./none:1

julia> a = 1
1

julia> f(3)
4
```

Este comportamiento puede parecer un poco raro para una variable normal, pero está permitido 
para que las funciones nombradas sean usadas antes de ser definidas (las funciones nombradas 
son exactamente variables normales que almacenan objetos función). Esto permite a las funciones 
ser definidas en cualquier orden que sea intuitivo y conveniente en lugar de forzar un 
ordenamiento de abajo a arriba o requerir declaraciones hacia delante, mientras que ellas sean 
definidas en el momento que sean usadas. Por ejemplo, he aquí una forma ineficiente, mutuamente 
recursiva de comprobar si un entero positivo es par o impar:

```jldoctest
julia> even(n) = n == 0 ? true : odd(n-1);

julia> odd(n) = n == 0 ? false : even(n-1);

julia> even(3)
false

julia> odd(3)
true
```

Julia proporciona funciones eficientes y predefinidas para comprobar la paridad o imparidad, 
llamadas [`iseven()`](@ref) e [`isodd()`](@ref). Por tanto, las definiciones anteriores 
deberían ser sólo tomadas como ejemplos.

### Ámbitos locales duro vs. blando

Los bloques que introducen un ámbito local blando, como los bucles, se suelen usar para 
manipular las variables en su ámbito padre. Por tanto, su defecto es acceder completamente a 
todas las variables de su ámbito padre.

A la inversa, el código dentro de los bloques que introduce un ámbito local duro (definiciones 
de función, tipo o macro) pueden ser ejecutados en cualquier parte del programa. Cambiar 
remotamente el estado de variables locales en otros módulos debería ser realizado con cuidado 
y por tanto esta es una característica de optimización que requiere la palabra clave `global`.

La razón para permitir *modificar local* variables de ámbitos padres en funciones anidadas
es permitir la construccin de [cierres](https://en.wikipedia.org/wiki/Closure_%28computer_programming%29) 
que tienen un estado privado, por ejemplo la variable `state` del siguiente ejemplo:

```jldoctest
julia> let
           state = 0
           global counter
           counter() = state += 1
       end;

julia> counter()
1

julia> counter()
2
```

Ver también los cierres en los ejemplos de las dos siguientes secciones.

### Bloques Let

A diferencia de las asignaciones a variables locales, las instrucciones `let` asignan nuevas 
asociaciones de variables cada vez que se ejecutan. Una asignación modifica el valor de una
localización existente, y `let` crea nuevas localizaciones. Esta diferencia no suele ser importante, 
y es sólo detectable en el caso de variables que sobreviven a sus ámbitos vía cierres. La sintaxis 
de `let` acepta una serie de asignaciones y nombres de variables separados por comas:

```jldoctest
julia> x, y, z = -1, -1, -1;

julia> let x = 1, z
           println("x: $x, y: $y") # x is local variable, y the global
           println("z: $z") # errors as z has not been assigned yet but is local
       end
x: 1, y: -1
ERROR: UndefVarError: z not defined
```

Las asignaciones se evalúan en orden, con cada término derecho evaluado en el ámbito antes de que 
se introduzca la nueva variable a través del término izquierdo. Por tanto, tiene sentido escribir 
algo como `let x = x` ya que las dos variables son distintas y tienen almacenamiento separado. Este
es un ejemplo de dónde se necesita el comportamiento de `let`: 

```jldoctest
julia> Fs = Array{Any}(2); i = 1;

julia> while i <= 2
           Fs[i] = ()->i
           i += 1
       end

julia> Fs[1]()
3

julia> Fs[2]()
3
```

Aquí creamos y almacenamos dos cierres que devuelven la variable `i`. Sin embargo, es siempre 
la misma variable `i`, por lo que los dos cierres se comportan de forma idéntica. Podemos usar 
`let` para crear una nueva correspondencia para `i`:

```jldoctest
julia> Fs = Array{Any}(2); i = 1;

julia> while i <= 2
           let i = i
               Fs[i] = ()->i
           end
           i += 1
       end

julia> Fs[1]()
1

julia> Fs[2]()
2
```

Como la construcción `begin` no construye un nuevo ámbito, puede ser útil usar un `let` con cero
argumentos para introducir un nuevo bloque de ámbito sin crear ninguna nueva correspondencia:

```jldoctest
julia> let
           local x = 1
           let
               local x = 2
           end
           x
       end
1
```

Como `let` introduce un nuevo bloque de ámbito, la variable local interna `x` es diferente de la
externa local `x`.

### Blucles for y compresiones

Los bucles `for` y las [comprensiones](@ref) tiene el siguiente comportamiento: cualquier nueva 
variable introducida en sus ámbitos se reservan de nuevo para cada nueva iteración del bucle. 
Esto contrasta con los bucles `while` que reservan las variables para todas las iteraciones. 
or tanto, estas construcciones son similares a bucles `while` con bloques `let` dentro de ellos:

```jldoctest
julia> Fs = Array{Any}(2);

julia> for j = 1:2
           Fs[j] = ()->j
       end

julia> Fs[1]()
1

julia> Fs[2]()
2
```

Los bucles `for` reusarán las variables existentes para su variable iteración:

```jldoctest
julia> i = 0;

julia> for i = 1:3
       end

julia> i
3
```

Sin embargo, las comprensiones no hacen esto, y siempre asignan de nuevo sus variables de 
iteración:

```jldoctest
julia> x = 0;

julia> [ x for x = 1:3 ];

julia> x
0
```

## Constantes

Un uso común de las variables es darle nombres de valores específicos, no cambiantes. Tales 
variables sólo se asignan una vez. Esta intención puede ser transportada al compilador usando 
la palabra clave `const`:

```jldoctest
julia> const e  = 2.71828182845904523536;

julia> const pi = 3.14159265358979323846;
```

La declaración `const` es permitida sobre variables globales y locales, pero es especialmente 
útil para las globales. Es difícil para el compilador optimizar´código en el que están 
implicadas las variables globales, ya que sus valores (o incluso sus tipos) podrían cambiar en 
cualquier momento. Si una variable global no va a cambiar, añadir una declaración `const` 
resolverá este problema de rendimiento.

Las constantes locales son bastante diferentes. El compilador es capaz de determinar cuando 
una variable local es constante, por lo que las declaraciones de constante local no son 
necesarias para mejorar el rendimiento.

Las asignaciones especiales de nivel superior, tales como las realizadas por las palabras clave 
`function` y `type` son constantes por defecto. 

Nótese que `const` sólo afecta a la asociación de variables:  la variable puede ser asociada 
a un objeto mutable (tal com un array) y el objeto puede aún ser modificado.
