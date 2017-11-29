# Metaprogramación

El legado más fuerte de Lisp en el lenguaje Julia es su soporte a la metaprogramación. Al igual que Lisp, Julia representa su propio código como una estructura de datos del propio lenguaje. Dado que el código está representado por objetos que pueden ser creados y manipulados desde dentro del lenguaje, es posible que un programa pueda transformar y generar su propio código. Esto permite una sofisticada generación de código sin pasos de construcción adicionales, y también permite las verdaderas macros de estilo Lisp que operan a nivel de los [árboles sintácticos abstractos](https://en.wikipedia.org/wiki/Abstract_syntax_tree). En contraste, los sistemas de preprocesador "macro" como el de C y C++, realizan la manipulación y sustitución textual antes de que se realice cualquier análisis o interpretación real. Debido a que todos los tipos de datos y código en Julia están representados por las estructuras de datos Julia, hay disponibles poderosas capacidades de [reflexión](https://en.wikipedia.org/wiki/Reflection_%28computer_programming%29) están disponibles para explorar las características internas de un programa y sus tipos al igual que cualquier otro dato.

## Representación de programas

Cada programa en Julia comienza su vida como una cadena:

```jldoctest prog
julia> prog = "1 + 1"
"1 + 1"
```

**¿Qué sucede después?**

El siguiente paso es [analizar sintácticamente](https://en.wikipedia.org/wiki/Parsing#Computer_languages) cada cadena en un objeto denominado una expresión, representado por el tipo `Expr` de Julia:

```jldoctest prog
julia> ex1 = parse(prog)
:(1 + 1)

julia> typeof(ex1)
Expr
```

Los objetos `Expr` contienen tres partes:

* Un `Symbol` identificando la clase de expresión. Un símbolo es un [identificador de cadena internado](https://en.wikipedia.org/wiki/String_interning) (más información a continuación).

```jldoctest prog
julia> ex1.head
:call
```

  * Los argumentos de expresión, que pueden ser símbolos, otras expresiones o valores literales:

```jldoctest prog
julia> ex1.args
3-element Array{Any,1}:
  :+
 1
 1
```

  * Finalmente, el tipo de resultado de la expresión, que puede ser anotado por el usuario o inferido por el 
    compilador (y puede ser ignorado completamente para los propósitos de este capítulo):

```jldoctest prog
julia> ex1.typ
Any
```

Las expresiones pueden también ser construidas directamente en [notación prefija](https://en.wikipedia.org/wiki/Polish_notation):

```jldoctest prog
julia> ex2 = Expr(:call, :+, 1, 1)
:(1 + 1)
```

Las dos expresiones construidas antes (mediante análisis sintáctico y mediante construcción directa) son equivalentes:

```jldoctest prog
julia> ex1 == ex2
true
```

**El punto clave aquí es que el código Julia se representa internamente como una estructura de datos que es accesible desde el propio lenguaje.**

La función [`dump()`](@ref) proporciona una visualización indentada y anotada de objetos `Expr`:

```jldoctest prog
julia> dump(ex2)
Expr
  head: Symbol call
  args: Array{Any}((3,))
    1: Symbol +
    2: Int64 1
    3: Int64 1
  typ: Any
```

Los objetos `Expr` puede ser también anidados:

```jldoctest ex3
julia> ex3 = parse("(4 + 4) / 2")
:((4 + 4) / 2)
```

Otra forma de ver expresiones es con Meta.show_sexpr, que muestra la [expresión-S](https://en.wikipedia.org/wiki/S-expression) de una `Expr` dada, que puede resultar muy familiar a los usuarios de Lisp. He aquí un ejemplo que visualiza una expresión (`Expr`) anidada:

```jldoctest ex3
julia> Meta.show_sexpr(ex3)
(:call, :/, (:call, :+, 4, 4), 2)
```

### Simbolos

El carácter `:` tiene dos propósitos sintácticos en Julia. La primera forma crea un símbolo (`Symbol`), una [cadena internada](https://en.wikipedia.org/wiki/String_interning) usada como un bloque constructivo de expresiones:

```jldoctest
julia> :foo
:foo

julia> typeof(ans)
Symbol
```

El constructor [`Symbol`](@ref) toma cualquier número de argumentos y crea un símbolo concatenando sus representaciones de cadena juntas:

```jldoctest
julia> :foo == Symbol("foo")
true

julia> Symbol("func",10)
:func10

julia> Symbol(:var,'_',"sym")
:var_sym
```

En el contexto de una expresión, los símbolos se utilizan para indicar el acceso a variables; Cuando se evalúa una expresión, se sustituye un símbolo por el valor asociado a ese símbolo en el [ámbito](@ref scope-of-variables) apropiado.

A veces son necesarios paréntesis adicionales alrededor del argumento a: para evitar la ambigüedad en el análisis:

```jldoctest
julia> :(:)
:(:)

julia> :(::)
:(::)
```

## Expresiones y evaluación

###  Citación

El segundo propósito sintáctico del carácter `:` es crear objetos expresión sin utilizar el constructor `Expr` explícito. Esto se conoce como *citación*. El carácter `:`, seguido de pares de paréntesis alrededor de una sola declaración de código Julia, produce un objeto `Expr` basado en el código incluido. He aquí un ejemplo de la forma corta utilizada para citar una expresión aritmética:

```jldoctest
julia> ex = :(a+b*c+1)
:(a + b * c + 1)

julia> typeof(ex)
Expr
```

(para ver la estructura de esta expresión, podemos usar `ex.head` y `ex.args` o `dump()`como antes)

Nótese que pueden construirse expresiones equivalentes usando [`parse()`](@ref) o la forma directa `Expr`:

```jldoctest
julia>      :(a + b*c + 1)  ==
       parse("a + b*c + 1") ==
       Expr(:call, :+, :a, Expr(:call, :*, :b, :c), 1)
true
```

Las expresiones proporcionadas por el analizador sólo suelen tener símbolos, otras expresiones y valores literales como sus args, mientras que las expresiones construidas por el código Julia pueden tener valores de ejecución arbitrarios sin formas literales como args. En este ejemplo específico, `+` y `a` son símbolos, `*(b,c)` es una subexpresión, y `1` un entero literal de 64 bits.

Hay una segunda forma sintáctica de citar para expresiones múltiples: bloques de código encerrados en `quote ... end`. Note que esta forma introduce elementos `QuoteNode` al árbol de expresión que deben considerarse cuando se manipule directamente un árbol de expresiones generado a partir de bloques `quote`. Para otros propósitos, los bloques `:(...)` y `quote...end` se tratan de forma idéntica.

```jldoctest
julia> ex = quote
           x = 1
           y = 2
           x + y
       end
quote
    #= none:2 =#
    x = 1
    #= none:3 =#
    y = 2
    #= none:4 =#
    x + y
end

julia> typeof(ex)
Expr
```

### Interpolación

La construcción directa de objetos `Expr` con valores como argumentos es potente, pero los constructores objetos `Expr` pueden ser tediosos comparados con la sintaxis "normal" de Julia. Como alternativa, Julia permite "empalme" o interpolación de literales o expresiones en expresiones citadas. La interpolación se indica con el prefijo `$`.

En este ejemplo el valor literal de `a` es interpolado:

```jldoctest interp1
julia> a = 1;

julia> ex = :($a + b)
:(1 + b)
```

Interpolar en una expresión no citada (*quoted*) no se admite y causará un error en tiempo de compilación: 

```jlcodtest interp1
julia> $a + b
ERROR: unsupported or misplaced expression $
 ...
```

En este ejemplo, la tupla `(1,2,3)` es interpolada como una expresión en un test condicional:

```jldoctest interp1
julia> ex = :(a in $:((1,2,3)) )
:(a in (1, 2, 3))
```

Interpolar símbolos en una expresión anidada requiere encerrar cada símbolo en un bloque de cita que lo encierre:

```julia-repl
julia> :( :a in $( :(:a + :b) ) )
                   ^^^^^^^^^^
                   quoted inner expression
```

El uso de `$` para la interpolación de la expresión recuerda intencionalmente a la [interpolación de cadenas](@ref string-interpolation) y a la [interpolación de mandatos](@ref command-interpolation). La interpolación de expresiones permite la construcción programática conveniente y legible de expresiones Julia complejas.

### [`eval()`](@ref) and efectos

Dado un objeto expresión, uno puede causar que Julia lo evalúe (ejecute) en un ámbito global usando [`eval()`](@ref):

```jldoctest interp1
julia> :(1 + 2)
:(1 + 2)

julia> eval(ans)
3

julia> ex = :(a + b)
:(a + b)

julia> eval(ex)
ERROR: UndefVarError: b not defined
[...]

julia> a = 1; b = 2;

julia> eval(ex)
3
```

Cada [módulo](@ref modules) tiene su propia función [`eval()`](@ref) que evalúa expresiones en su ámbito global. Las expresiones pasadas a [`eval()`](@ref) no están limitadas a valores de retorno (ellas también pueden tener efetos colaterales que alteren el estado del entorno del módulo que las encierra:

```jldoctest
julia> ex = :(x = 1)
:(x = 1)

julia> x
ERROR: UndefVarError: x not defined

julia> eval(ex)
1

julia> x
1
```

Aquí, la evaluación de un objeto expresión causa que se asigne un valor a la variable global `x`.

Como las expresiones no son más que objetos `Expr` que pueden ser construidos programáticamente y después evaluados, es posible generar dinámicamente código arbitrario que pueda ser ejecutado luego mediante [`eval()`](@ref). He aquí
Since expressions are just `Expr` objects which can be constructed programmatically and then evaluated, un ejemplo sencillo:

```julia-repl
julia> a = 1;

julia> ex = Expr(:call, :+, a, :b)
:(1 + b)

julia> a = 0; b = 2;

julia> eval(ex)
3
```

El valor de `a` se ha usado para construir la expresión `ex` que aplica la función `+` al valor `1` y a la variable `b`. Note la distinción importante entre la forma en que se usan las variables `a` y `b`:

* El valor de la *variable* `a` se utiliza como valor inmediato en la expresión en el tiempo de construcción de la 
  expresión. Por lo tanto, una vez que la expresión es evaluada, el valor de a ya no importa: el valor en la expresión 
  ya es 1, independientemente de lo que pueda ser ahora  el valor de `a`.
* Por otro lado, el símbolo `:b` se utiliza en la construcción de la expresión, por lo que el valor de la variable `b` 
  en ese momento es irrelevante - :b es sólo un símbolo y la variable b ni siquiera necesita ser definida. En el 
  momento de la evaluación de la expresión, sin embargo, el valor del símbolo `:b` se resuelve buscando el valor de 
  la variable `b`.

### Funcciones sobre `Expr`esiones

Como se ha sugerido anteriormente, una característica muy útil de Julia es la capacidad de generar y manipular código Julia dentro del propio Julia. Ya hemos visto un ejemplo de una función que devuelve objetos `Expr`: la función [`parse()`](@ref), que toma una cadena de código Julia y devuelve la `Expr` correspondiente. Una función también puede tomar uno o más objetos `Expr` como argumentos, y devolver otro `Expr`. Aquí hay un ejemplo simple y motivador:

```jldoctest
julia> function math_expr(op, op1, op2)
           expr = Expr(:call, op, op1, op2)
           return expr
       end
math_expr (generic function with 1 method)

julia>  ex = math_expr(:+, 1, Expr(:call, :*, 4, 5))
:(1 + 4 * 5)

julia> eval(ex)
21
```

Otro ejemplo puede ser esta función que dobla cualquier argumento numérico, pero deja las expresiones solas:

```jldoctest
julia> function make_expr2(op, opr1, opr2)
           opr1f, opr2f = map(x -> isa(x, Number) ? 2*x : x, (opr1, opr2))
           retexpr = Expr(:call, op, opr1f, opr2f)
           return retexpr
       end
make_expr2 (generic function with 1 method)

julia> make_expr2(:+, 1, 2)
:(2 + 4)

julia> ex = make_expr2(:+, 1, Expr(:call, :*, 5, 8))
:(2 + 5 * 8)

julia> eval(ex)
42
```

## [Macros](@id man-macros)

Las macros proporcionan un método para incluir el código generado en el cuerpo final de un programa. Una macro asigna una tupla de argumentos a una expresión devuelta, y la expresión resultante se compila directamente en lugar de requerir una llamada [`eval()`](@ref) de ejecución. Los argumentos de macro pueden incluir expresiones, valores literales y símbolos.

### Básico

He aquí una macro extraordinariamente simple:

```jldoctest sayhello
julia> macro sayhello()
           return :( println("Hello, world!") )
       end
@sayhello (macro with 1 method)
```

Las macros tienen un carácter dedicado en la sintaxis de Julia: el `@` (at-sign), seguido por el nombre único declarado en un bloque `macro NAME ... end`. En este ejemplo, el compilador reemplazará todas las instancias de `@sayhello` con:

```julia
:( println("Hello, world!") )
```

Cuando `@sayhello` se llama en el REPL, la expresión se ejecuta inmediatamente, por lo tanto solo vemos el resultado de la evaluación:   

```jldoctest sayhello
julia> @sayhello()
Hello, world!
```

Ahora, considere una macro un poco más compleja:

```jldoctest sayhello2
julia> macro sayhello(name)
           return :( println("Hello, ", $name) )
       end
@sayhello (macro with 1 method)
```

Esta macro toma un argumento: `name`. Cuando se encuentra `@sayhello`, la expresión citada se *expande* para interpolar el valor del argumento en la expresión final:

```jldoctest sayhello2
julia> @sayhello("human")
Hello, human
```

Podemos ver la expresión de retorno entre comillas usando la función [`macroexpand()`](@ref) (**nota importante:** esta es una herramienta extremadamente útil para depurar macros):

```jldoctest sayhello2
julia> ex = macroexpand( :(@sayhello("human")) )
:((println)("Hello, ", "human"))

julia> typeof(ex)
Expr
```

We can see that the `"human"` literal has been interpolated into the expression.

También existe una macro [`@macroexpand`](@ref) que quizás sea un poco más conveniente que la función `macroexpand`:

```jldoctest sayhello2
julia> @macroexpand @sayhello "human"
:((println)("Hello, ", "human"))
```

### Un momento. ¿Por qué las macros?

Ya hemos visto una función `f(:: Expr ...) -> Expr` en una sección anterior. De hecho, [`macroexpand()`](@ref) es también una función. Entonces, ¿por qué existen macros?

Las macros son necesarias porque se ejecutan cuando se analiza el código, por lo tanto, las macros permiten al programador generar e incluir fragmentos de código personalizado antes de ejecutar el programa completo. Para ilustrar la diferencia, considere el siguiente ejemplo:

```jldoctest whymacros
julia> macro twostep(arg)
           println("I execute at parse time. The argument is: ", arg)
           return :(println("I execute at runtime. The argument is: ", $arg))
       end
@twostep (macro with 1 method)

julia> ex = macroexpand( :(@twostep :(1, 2, 3)) );
I execute at parse time. The argument is: $(Expr(:quote, :((1, 2, 3))))
```

La primera llamada a [`println()`](@ref) se ejecuta cuando se invoca [`macroexpand()`](@ref). La expresión resultante contiene *sólo* el segundo `println`:

```jldoctest whymacros
julia> typeof(ex)
Expr

julia> ex
:((println)("I execute at runtime. The argument is: ", $(Expr(:copyast, :($(QuoteNode(:((1, 2, 3)))))))))

julia> eval(ex)
I execute at runtime. The argument is: (1, 2, 3)
```

### Invocación de macros

Las macros son invocadas con la siguiente sintaxis general:



```julia
@name expr1 expr2 ...
@name(expr1, expr2, ...)
```

Note la `@` antes del nombre de macro y la falta de domas entre las expresiones de los argumentos en la primera forma, y la falta de espacios en balnco después de `@name` en la segunda forma. Los dos estilos no deberían mezclarse. Por ejemplo, la siguiente sintaxis es diferente que la de los ejemplos anteriores; ella pasa la tupla `(expr1, expr2, ...)` como argumento a la macro:

```julia
@name (expr1, expr2, ...)
```

Es importante enfatizar que las macros reciben sus argumentos como expresiones, literales o símbolos. Una forma de explorar los argumentos de las macros es llamar a la función `show()` dentro del cuerpo de la macro:

```jldoctest
julia> macro showarg(x)
           show(x)
           # ... remainder of macro, returning an expression
       end
@showarg (macro with 1 method)

julia> @showarg(a)
:a

julia> @showarg(1+1)
:(1 + 1)

julia> @showarg(println("Yo!"))
:(println("Yo!"))
```

Además de la lista de argumentos dada, a cada macro se le pasan dos argumentos adicionales llamados `__source__` and `__module__`.

El argumento `__source__` proporciona información (en forma de un objeto `LineNumberNode`) sobre la ubicación del analizador del signo `@` de invocación de la macro . Esto permite que las macros incluyan una mejor información de diagnóstico de errores, y es comúnmente utilizada por el registro, las macros del analizador de cadenas y los documentos, por ejemplo, así como para implementar las macros `@__ LINE__`,`@__ FILE__` y `@__ DIR__` .

Se puede acceder a la información de ubicación haciendo referencia a `__source __. Line` y` __source __. File`:

```jldoctest
julia> macro __LOCATION__(); return QuoteNode(__source__); end
@__LOCATION__ (macro with 1 method)

julia> dump(
            @__LOCATION__(
       ))
LineNumberNode
  line: Int64 2
  file: Symbol none
```

El argumento `__module__` proporciona información (en forma de un objeto` Module`) sobre el contexto de expansión de invocacin de la macro. Esto permite que las macros busquen información contextual, como enlaces existentes, o que inserten el valor como un argumento adicional a una llamada de función en tiempo de ejecución que realiza la autorreflexión en el módulo actual.

### Construir una macro avanzada

He aquí una versión simplificada de la macro `@assert` de Julia:

```jldoctest building
julia> macro assert(ex)
           return :( $ex ? nothing : throw(AssertionError($(string(ex)))) )
       end
@assert (macro with 1 method)
```

La macro puede ser usada de esta forma:

```jldoctest building
julia> @assert 1 == 1.0

julia> @assert 1 == 0
ERROR: AssertionError: 1 == 0
```

En lugar de la sintaxis escrita, la llamada a macro es expandida en tiempo de análisis para que devuelva un resutlado. Esto es equivalente a escribir:


```julia
1 == 1.0 ? nothing : throw(AssertionError("1 == 1.0"))
1 == 0 ? nothing : throw(AssertionError("1 == 0"))
```

Es decir, en la primera llamada, la expresión `:(1 == 1.0)` se empalma en la ranura de condición de prueba, mientras que el valor de `string(:( 1 == 1.0))` se empalma en la ranura de mensaje de aserción. Toda la expresión, así construida, se coloca en el árbol de sintaxis donde se produce la llamada de macro `@assert`. Entonces, en el tiempo de ejecución, si la expresión de prueba se evalúa como verdadera, entonces se devuelve `nothing`, mientras que si la prueba es falsa, se genera un error indicando la expresión afirmada que es falsa. Observe que no sería posible escribir esto como una función, ya que sólo está disponible el valor de la condición y sería imposible mostrar la expresión que lo calculó en el mensaje de error.

La definición real de `@assert` en la biblioteca estándar es más complicada. Permite al usuario especificar opcionalmente su propio mensaje de error, en lugar de simplemente imprimir la expresión fallida. Al igual que en las funciones con un número variable de argumentos, esto se especifica con elipses después del último argumento:

```jldoctest assert2
julia> macro assert(ex, msgs...)
           msg_body = isempty(msgs) ? ex : msgs[1]
           msg = string(msg_body)
           return :($ex ? nothing : throw(AssertionError($msg)))
       end
@assert (macro with 1 method)
```

Ahora `@assert` tiene dos modos de operación, dependiendo del número de argumentos que recibe! Si sólo hay un argumento, la tupla de expresiones capturadas por `msgs` estará vacía y se comportará igual que la definición más simple anterior. Ahora bien, si el usuario especifica un segundo argumento, se imprime en el cuerpo del mensaje en lugar de la expresión que falla. Puede examinar el resultado de una expansión de macro con la función [`macroexpand()`](@ref) correctamente denominada:

```jldoctest assert2
julia> macroexpand(:(@assert a == b))
:(if a == b
        nothing
    else
        (throw)((AssertionError)("a == b"))
    end)

julia> macroexpand(:(@assert a==b "a should equal b!"))
:(if a == b
        nothing
    else
        (throw)((AssertionError)("a should equal b!"))
    end)
```

Hay otro caso que la versión real de `@assert` maneja: ¿qué pasa si, además de imprimir "a should be equal b", queremos imprimir sus valores? Uno podría ingenuamente intentar usar interpolación de cadena en el mensaje personalizado, por ejemplo, ` @assert a==b "a ($a) should equal b ($b)!"`, pero esto no funcionará como se esperaba con la macro anterior. ¿Puedes ver por qué? Recuerda de [string interpolation](@ref string-interpolation) que una cadena interpolada se reescribe a una llamada a [`string()`](@ref). Compare:

```jldoctest
julia> typeof(:("a should equal b"))
String

julia> typeof(:("a ($a) should equal b ($b)!"))
Expr

julia> dump(:("a ($a) should equal b ($b)!"))
Expr
  head: Symbol string
  args: Array{Any}((5,))
    1: String "a ("
    2: Symbol a
    3: String ") should equal b ("
    4: Symbol b
    5: String ")!"
  typ: Any
```

Así que ahora en lugar de obtener una cadena sencilla en `msg_body`, la macro está recibiendo una expresión completa que necesitará ser evaluada para mostrarse como se esperaba. Esto puede ser empalmado directamente en la expresión devuelta como un argumento a la llamada [`string()`](@ref); Vea [`error.jl`](https://github.com/JuliaLang/julia/blob/master/base/error.jl) para la implementación completa.

La macro `@assert` hace un gran uso del empalme en expresiones entre comillas para simplificar la manipulación de expresiones dentro del cuerpo de la macro.

### Higiene

Un problema que surge en las macros más complejas es el de la [higiene](https://en.wikipedia.org/wiki/Hygienic_macro).En resumen, las macros deben asegurarse de que las variables que introducen en sus expresiones devueltas no chocan accidentalmente con las variables existentes en el código circundante en el que se expanden. A la inversa, a menudo se espera que las expresiones que se pasan a una macro como argumentos evalúen en el contexto del código circundante, interactuando con y modificando las variables existentes. Otra preocupación surge del hecho de que una macro puede ser llamada en un módulo diferente desde donde se definió. En este caso, debemos asegurarnos de que todas las variables globales se resuelvan en el módulo correcto. Julia ya tiene una gran ventaja sobre los lenguajes con expansión de macro textual (como C) en que sólo necesita considerar la expresión devuelta. Todas las demás variables (como `msg` en `@assert` arriba) siguen el [comportamiento normal del bloque de ámbito](@ref scope-of-variables).

Para demostrar estos problemas, consideremos la posibilidad de escribir una macro `@time` que toma una expresión como su argumento, registra el tiempo, evalúa la expresión, registra el tiempo de nuevo, imprime la diferencia entre los tiempos antes y después y luego tiene el valor de La expresión como su valor final. La macro podría tener este aspecto:

```julia
macro time(ex)
    return quote
        local t0 = time()
        local val = $ex
        local t1 = time()
        println("elapsed time: ", t1-t0, " seconds")
        val
    end
end
```

Aquí, queremos que `t0`, `t1` y `val` sean variables temporales privadas, y queremos que `time` se refiera a la función [`time()`](@ref) de la biblioteca estándar, no a cualquier variable de tiempo que el usuario pueda tener (lo mismo se aplica a `println`). Imagine los problemas que podrían ocurrir si la expresión de usuario `ex` también contuviera asignaciones a una variable denominada `t0`, o definiese su propia variable `time`. Podríamos obtener errores o comportamiento misteriosamente incorrecto.


El expansor de macro de Julia resuelve estos problemas de la siguiente manera. En primer lugar, las variables dentro de un resultado de macro se clasifican como locales o globales. Una variable se considera local si es asignada (y no se declara global), se declara local o se utiliza como un nombre de argumento de función. De lo contrario, se considera global. Las variables locales son renombradas como únicas (utilizando la función [`gensym()`](@ref), que genera nuevos símbolos), y las variables globales se resuelven dentro del entorno de definición de macro. Por lo tanto, ambas preocupaciones se manejan; Los locales de la macro no entrarán en conflicto con ninguna variable de usuario, y `time` y `println` se referirán a las definiciones de la biblioteca estándar.

Sin embargo, queda un problema. Considere el siguiente uso de esta macro:

```julia
module MyModule
import Base.@time

time() = ... # compute something

@time time()
end
```

Aquí la expresión de usuario `ex` es una llamada a `time`, pero no a la misma función `time` que usa la macro, sino que se refiere claramente a `MyModule.time`. Por tanto, debemos arreglar pora que el código en `ex` sea resuelto en el entorno de llamada de la macro. Esto se hace usando [`esc()`](@ref) para "escapar" la expresión:

```julia
macro time(ex)
    ...
    local val = $(esc(ex))
    ...
end
```

Una expresión envuelta de esta manera es dejada sola por el expansor de macros y simplemente pegada en la salida. Por tanto, será resuelta en el entorno de llamada de la macro.

El mecanismo de "escapar" puede ser usado para "violar" la higiene cuando sea necesario, para introducir o manipular variables de usuario. Por ejemplo, la siguiente macro fija `x` a cero en el entorno de llamada:

```jldoctest
julia> macro zerox()
           return esc(:(x = 0))
       end
@zerox (macro with 1 method)

julia> function foo()
           x = 1
           @zerox
           return x # is zero
       end
foo (generic function with 1 method)

julia> foo()
0
```

Esta clase de manipulación de variables debería ser usada juiciosamente, pero es ocasionalmente bastante útil.

Obtener las normas de higiene correctas puede ser un desafío formidable. Antes de usar una macro, es posible que desee considerar si un cierre de función sería suficiente. Otra estrategia útil es diferir tanto trabajo como sea posible para el tiempo de ejecución. Por ejemplo, muchas macros simplemente envuelven sus argumentos en un QuoteNode u otro Expr similar. Algunos ejemplos de esto incluyen `@task body` que simplemente devuelve `schedule (Task(() -> $ body))`, y `@eval expr`, que simplemente devuelve `eval (QuoteNode (expr))`.

Obtener las normas de higiene correctas puede ser un desafío formidable. Antes de usar una macro, es posible que desee considerar si un cierre de función sería suficiente. Otra estrategia útil es diferir tanto trabajo como sea posible para el tiempo de ejecución. Por ejemplo, muchas macros simplemente envuelven sus argumentos en un QuoteNode u otro Expr similar. Algunos ejemplos de esto incluyen `@task body` que simplemente devuelve` schedule (Task (() -> $ body)) `, y` @eval expr`, que simplemente devuelve `eval (QuoteNode (expr))`.

Para demostrarlo, podríamos reescribir el ejemplo `@time` anterior como:

```julia
macro time(expr)
    return :(timeit(() -> $(esc(expr))))
end
function timeit(f)
    t0 = time()
    val = f()
    t1 = time()
    println("elapsed time: ", t1-t0, " seconds")
    return val
end
```

Sin embargo, no hacemos esto por una buena razón: al envolver el `expr` en un nuevo bloque de alcance (la función anónima) también cambia ligeramente el significado de la expresión (el alcance de cualquier variable en él), mientras que queremos` @ time` para ser utilizable con un impacto mínimo en el código ajustado.

## Generación de Código

Cuando se requiere una cantidad significativa de código repetitivo, es común generarlo programáticamente para evitar la redundancia. En la mayoría de los lenguajes, esto requiere un paso de construcción adicional y un programa separado para generar el código repetitivo. En Julia, la interpolación de expresión y [`eval()`](@ref) permiten que dicha generación de código tenga lugar en el curso normal de la ejecución del programa. Por ejemplo, el siguiente código define una serie de operadores en tres argumentos en términos de sus formas de 2 argumentos:

```julia
for op = (:+, :*, :&, :|, :$)
    eval(quote
        ($op)(a,b,c) = ($op)(($op)(a,b),c)
    end)
end
```

De este modo, Julia actúa como su propio [preprocesador](https://en.wikipedia.org/wiki/Preprocessor), y permite la generación de código desde dentro del lenguaje. El código anterior debería ser escrito ligeramente más secamente usando la forma prefija de citación `:`

```julia
for op = (:+, :*, :&, :|, :$)
    eval(:(($op)(a,b,c) = ($op)(($op)(a,b),c)))
end
```

En este tipo de generación de código dentro del lenguaje utilizando el patrón `eval(quote(...))` es bastante común, sin embargo, que Julia venga con una macro para abreviar este patrón:

```julia
for op = (:+, :*, :&, :|, :$)
    @eval ($op)(a,b,c) = ($op)(($op)(a,b),c)
end
```

La macro [`@eval`](@ref) reescribe esta llamada para ser precisamente equivalente a las versiones largas anteriores. Para bloques de código generado más grandes, el argumento expresión dado a [`@eval`](@ref) puede ser un bloque:

```julia
@eval begin
    # multiple lines
end
```

## Literales de cadena no estándar

Recuerde de [Strings](@ref non-standard-string-literals) que los literales de cadena prefijados por un identificador se llaman literales de cadena no estándar y pueden tener semántica distinta que los literales de cadena no prefijados. Por ejemplo:

* `r"^\s*(?:#|$)"` produces un objeto expresión regular en lugar de una cadena.
* `b"DATA\xff\u2200"` es un literal array bytepara `[68,65,84,65,255,226,136,128]`.

Tal vez sorprendentemente, estos comportamientos no están codificados en el analizador de Julia o en el compilador. En su lugar, son comportamientos personalizados proporcionados por un mecanismo general que cualquiera puede utilizar: los literales de cadenas prefijados se analizan como llamadas a macros de nombre especial. Por ejemplo, la macro de expresiones regulares es sólo la siguiente:

```julia
macro r_str(p)
    Regex(p)
end
```

Eso es todo. Esta macro dice que el contenido literal de la cadena literal `r"^\s*(?:#|$)"` debe ser pasado a la macro `@r_str` y que el resultado de esa expansión debe colocarse en el árbol de sintaxis donde tiene lugar la cadena literal. En otras palabras, la expresión `r"^\s*(?:#|$)"` equivale a colocar el siguiente objeto directamente en el árbol de sintaxis:

```julia
Regex("^\\s*(?:#|\$)")
```

No sólo la forma literal de la cadena es más corta y mucho más conveniente, sino que también es más eficiente: puesto que la expresión regular se compila y el objeto Regex se crea realmente *cuando el código es compilado*, la compilación se produce sólo una vez, Se ejecuta el código. Considere si la expresión regular se produce en un bucle:


```julia
for line = lines
    m = match(r"^\s*(?:#|$)", line)
    if m === nothing
        # non-comment
    else
        # comment
    end
end
```

Como la expresión regular `r"^\s*(?:#|$)"` Se compila e inserta en el árbol de sintaxis cuando se analiza este código, la expresión sólo se compila una vez en lugar de cada vez que se ejecuta el bucle. Para lograr esto sin macros, uno tendría que escribir este bucle así:

```julia
re = Regex("^\\s*(?:#|\$)")
for line = lines
    m = match(re, line)
    if m === nothing
        # non-comment
    else
        # comment
    end
end
```

Por otra parte, si el compilador no pudiera determinar que el objeto regex era constante en todos los bucles, ciertas optimizaciones podrían no ser posibles, haciendo esta versión aún menos eficiente que la forma literal más conveniente de arriba. Por supuesto, todavía hay situaciones en las que la forma no literal es más conveniente: si se necesita interpolar una variable en la expresión regular, se debe tomar este enfoque más detallado; En los casos en que el patrón de expresión regular mismo es dinámico, cambiando potencialmente en cada iteración del bucle, un nuevo objeto expresión regular debe ser construido en cada iteración. Sin embargo, en la gran mayoría de los casos de uso, las expresiones regulares no se construyen sobre la base de datos de tiempo de ejecución. En esta mayoría de casos, la capacidad de escribir expresiones regulares como valores en tiempo de compilación es valiosísima.

Al igual que los literales de cadena no estándar, existen literales de comandos no estándar que usan una variante prefijada de la sintaxis literal del comando. El comando literal ```custom `literal` ``` se analiza como `@custom_cmd "literal"`. Julia por sí misma no contiene ningún literal de comando no estándar, pero los paquetes pueden hacer uso de esta sintaxis. Aparte de la sintaxis diferente y el sufijo `_cmd` en lugar del sufijo` _str`, los literales de comandos no estándar se comportan exactamente como los literales de cadena no estándar.

En el caso de que dos módulos proporcionen cadenas o literales de comando con el mismo nombre, es posible calificar la cadena o literal de comando con un nombre de módulo. Por ejemplo, si tanto `Foo` como` Bar` proporcionan literal de cadena no estándar `@x_str`, entonces uno puede escribir `Foo.x "literal"` o `Bar.x "literal" `para desambiguar entre los dos.

El mecanismo para literales de cadena definidos por el usuario es profundo, profundamente poderoso. No sólo son literales no estándar de Julia implementados usándolos, sino que también se implementa la sintaxis literal de comandos (``` `echo "Hello, $person"` ```)


The mechanism for user-defined string literals is deeply, profoundly powerful. Not only are Julia's
non-standard literals implemented using it, but also the command literal syntax (``` `echo "Hello, $person"` ```)
se implementa con la siguiente macro de aspecto inofensivo:

```julia
macro cmd(str)
    :(cmd_gen($(shell_parse(str)[1])))
end
```

Por supuesto, una gran cantidad de complejidad se oculta en las funciones utilizadas en esta definición de macro, pero son sólo funciones, escritas íntegramente en Julia. Usted puede leer su fuente y ver exactamente lo que hacen -y todo lo que hacen es construir objetos de expresión para ser insertados en el árbol de sintaxis de su programa.

## Funciones Generadas

Una macro muy especial es `@generated`, que permite definir las llamadas *funciones generadas*. Éstas tienen la capacidad de generar código especializado dependiendo de los tipos de sus argumentos con más flexibilidad y/o menos código que lo que se puede lograr con el despacho múltiple. Mientras las macros trabajan con expresiones al momento de analizar y no pueden acceder a los tipos de sus entradas, una función generada se amplía en un momento en que se conocen los tipos de los argumentos, pero la función aún no se ha compilado.

En lugar de realizar algún cálculo o acción, una declaración de función generada devuelve una expresión entre comillas que luego forma el cuerpo para el método correspondiente a los tipos de los argumentos. Cuando se llama, la expresión del cuerpo se compila (o se extrae de una caché, en las llamadas posteriores) y sólo se evalúa la expresión devuelta, y no el código que lo generó. Así, las funciones generadas proporcionan un marco flexible para mover el trabajo desde el tiempo de ejecución hasta el tiempo de compilación.

Cuando se definen las funciones generadas, hay tres diferencias principales con las funciones ordinarias:

1. Uno anota la declaración de función con la macro `@generated`. Esto agrega cierta información a la AST que permite al
   compilador saber que se trata de una función generada.
2. En el cuerpo de la función generada sólo tiene acceso a los *tipos* de los argumentos, no a sus valores – y cualquier
   función que fuera definida *antes* de la definición de la función generada.
3. En lugar de calcular algo o realizar alguna acción, devuelve una *expresión citada* que, cuando se evalúa, hace lo
   que uno quiere.
4. Las funciones generadas no deben *mutar* ni *observar* ningún estado global no constante (incluidos, por ejemplo, 
   IO, bloqueos, diccionarios no locales o que usen `method_exists`). Esto significa que solo pueden leer constantes 
   globales y no pueden tener ningún efecto secundario. En otras palabras, deben ser completamente puros. Debido a 
   una limitación de implementación, esto también significa que actualmente no pueden definir un cierre o un generador 
   sin tipo.

Es más fácil ilustrar esto con un ejemplo. Podemos declarar una función generada `foo` como:

```jldoctest generated
julia> @generated function foo(x)
           Core.println(x)
           return :(x * x)
       end
foo (generic function with 1 method)
```

Tenga en cuenta que el cuerpo devuelve una expresión entre comillas, a saber `:(x * x)`, en lugar de sólo el valor de `x * x`.

Desde la perspectiva del llamador, son muy similares a las funciones regulares; de hecho, no tienes que saber si estás llamando a una función regular o generada  -la sintaxis y el resultado de la llamada son iguales. Veamos cómo se comporta `foo`:

```jldoctest generated
julia> x = foo(2); # note: output is from println() statement in the body
Int64

julia> x           # now we print x
4

julia> y = foo("bar");
String

julia> y
"barbar"
```

Así, vemos que en el cuerpo de la función generada, `x` es el tipo del argumento pasado, y el valor devuelto por la función generada es el resultado de la evaluación de la expresión citada que devolvimos de la definición, ahora con el *valor* de `x`.

¿Qué pasa si evaluamos foo de nuevo con un tipo que ya hemos utilizado?

```jldoctest generated
julia> foo(4)
16
```

Tenga en cuenta que no hay ninguna impresión de [`Int64`](@ref). El cuerpo de la función generada sólo se ejecuta una vez (no es enteramente cierto, véase la nota a continuación) cuando se compila el método para ese conjunto específico de tipos de argumentos. Después de eso, la expresión devuelta de la función generada en la primera invocación se vuelve a utilizar como el cuerpo del método.

La cantidad de veces que se genera una función generada *podría* ser solo una vez, pero *podría* también ser más frecuente o no aparecer en absoluto. Como consecuencia, *nunca* debe escribir una función generada con efectos secundarios: cuándo y con qué frecuencia ocurren los efectos secundarios no está definida. (Esto también es válido para las macros, y al igual que para las macros, el uso de [`eval()`](@ref) en una función generada es una señal de que estás haciendo algo incorrecto.) Sin embargo, a diferencia de las macros , el sistema de tiempo de ejecución no puede manejar correctamente una llamada a [`eval()`](@ref), por lo que no se permite.

También es importante ver cómo las funciones `@generated` interactúan con la redefinición del método. Siguiendo el principio de que una función correcta `@ generated` no debe observar ningún estado mutable ni causar ninguna mutación del estado global, vemos el siguiente comportamiento. Observe que la función generada *no puede* llamar a ningún método que no se haya definido antes de la * definición * de la función generada en sí.

Inicialmente `f (x)` tiene una definición:

```jldoctest redefinition
julia> f(x) = "original definition";
```

Define otras operaciones que usan `f(x)`:

```jldoctest redefinition
julia> g(x) = f(x);

julia> @generated gen1(x) = f(x);

julia> @generated gen2(x) = :(f(x));
```

Ahora añadimos algunas definiciones nuevas para `f(x)`:

```jldoctest redefinition
julia> f(x::Int) = "definition for Int";

julia> f(x::Type{Int}) = "definition for Type{Int}";
```

y comparamos cómo difieren estos resultados:

```jldoctest redefinition
julia> f(1)
"definition for Int"

julia> g(1)
"definition for Int"

julia> gen1(1)
"original definition"

julia> gen2(1)
"definition for Int"
```

Cada método de una función generada tiene su propia visión de funciones definidas:

```jldoctest redefinition
julia> @generated gen1(x::Real) = f(x);

julia> gen1(1)
"definition for Type{Int}"
```
La función de ejemplo `foo` anterior no hizo nada de lo que una función normal` foo(x) = x*x` no pudo hacer (excepto imprimir el tipo en la primera invocación y incurrir en una sobrecarga más alta). Sin embargo, el poder de una función generada radica en su capacidad para calcular diferentes expresiones entrecomilladas según los tipos que se le pasen:

```jldoctest
julia> @generated function bar(x)
           if x <: Integer
               return :(x ^ 2)
           else
               return :(x)
           end
       end
bar (generic function with 1 method)

julia> bar(4)
16

julia> bar("baz")
"baz"
```

(although por supuesto este ejemplo artificial se implementa facilmente usando despacho múltiple...

Podemos, por supuesto, abusar de esto para producir algún comportamiento interesante:


```jldoctest
julia> @generated function baz(x)
           if rand() < .9
               return :(x^2)
           else
               return :("boo!")
           end
       end
baz (generic function with 1 method)
```

ado que el cuerpo de la función generada es no determinista, su comportamiento es indefinido.

*¡No copie estos ejemplos!*

Estos ejemplos sirven para ilustrar cómo funcionan las funciones generadas, tanto en el extremo de la definición como en el sitio de llamada; Sin embargo, no los copie, por las siguientes razones:

  * La función `foo` tiene efectos secundarios (la llamada a `Core.println`) y no está definida exactamente cuándo, 
    con qué frecuencia o cuántas veces se producirán estos efectos secundarios
  * La función `bar` resuelve un problema que se resuelve mejor con el despacho múltiple - definiendo `bar(x) = x` 
    y `bar(x :: Integer) = x ^ 2` hará la misma cosa, pero es más simple y más rápido.
  * La función `baz` es patológicamente insana

Tenga en cuenta que el conjunto de operaciones que no se deben intentar en una función generada no tiene límites, y el sistema de tiempo de ejecución solo puede detectar actualmente un subconjunto de las operaciones no válidas. Hay muchas otras operaciones que simplemente corromperán el sistema de tiempo de ejecución sin notificación, por lo general de maneras sutiles que obviamente no están conectadas a la mala definición. Debido a que el generador de funciones se ejecuta durante la inferencia, debe respetar todas las limitaciones de ese código.

Algunas operaciones que no deberían intentarse incluyen:

1. Almacenamiento en caché de punteros nativos.
2. Interactuar de cualquier manera con los contenidos o métodos de Core.Inference.
3. Observar cualquier estado mutable.

     * La inferencia sobre la función generada se puede ejecutar en *cualquier* momento, incluso cuando el código intenta
       observar o modificar este estado.

4. Tomar cualquier bloqueo: código C que llame a puede utilizar bloqueos internos, (por ejemplo, no es problemático 
   para llamar `malloc`, a pesar de que la mayoría de las implementaciones requieren bloqueos internos), pero no 
   pretenden mantener o adquirir cualquier tiempo ejecutando el código de Julia.
5. Llamar a cualquier función que esté definida después del cuerpo de la función generada. Esta condición se relaja 
   para los módulos precompilados de carga incremental para permitir llamar a cualquier función en el módulo.

De acuerdo, ahora que tenemos una mejor comprensión de cómo funcionan las funciones generadas, utilicémoslas para construir algunas funcionalidades más avanzadas (y válidas) ...

### An advanced example

La biblioteca base de Julia tiene una función  [`sub2ind()`](@ref) para calcular un índice lineal en una matriz n-dimensional, basada en un conjunto de n índices multilineales - en otras palabras, para calcular el índice `i` que se puede usar para indexar en una matriz `A` usando `A[i]`, en lugar de `A[x, y, z, ...]`. Una posible implementación es la siguiente:

```jldoctest sub2ind
julia> function sub2ind_loop(dims::NTuple{N}, I::Integer...) where N
           ind = I[N] - 1
           for i = N-1:-1:1
               ind = I[i]-1 + dims[i]*ind
           end
           return ind + 1
       end
sub2ind_loop (generic function with 1 method)

julia> sub2ind_loop((3, 5), 1, 2)
4
```

Lo mismo puede hacerse usando recursión:

```jldoctest
julia> sub2ind_rec(dims::Tuple{}) = 1;

julia> sub2ind_rec(dims::Tuple{}, i1::Integer, I::Integer...) =
           i1 == 1 ? sub2ind_rec(dims, I...) : throw(BoundsError());

julia> sub2ind_rec(dims::Tuple{Integer, Vararg{Integer}}, i1::Integer) = i1;

julia> sub2ind_rec(dims::Tuple{Integer, Vararg{Integer}}, i1::Integer, I::Integer...) =
           i1 + dims[1] * (sub2ind_rec(Base.tail(dims), I...) - 1);

julia> sub2ind_rec((3, 5), 1, 2)
4
```

Ambas implementaciones, aunque diferentes, hacen esencialmente lo mismo: un bucle de tiempo de ejecución sobre las dimensiones de la matriz, recogiendo el desplazamiento en cada dimensión en el índice final.

Sin embargo, toda la información que necesitamos para el bucle está incrustada en la información de tipo de los argumentos. Por lo tanto, podemos utilizar las funciones generadas para mover la iteración a tiempo de compilación; en la jerga del compilador, usamos las funciones generadas para desenrollar manualmente el bucle. El cuerpo se vuelve casi idéntico, pero en vez de calcular el índice lineal, construimos una *expresión* que calcula el índice:

```jldoctest sub2ind_gen
julia> @generated function sub2ind_gen(dims::NTuple{N}, I::Integer...) where N
           ex = :(I[$N] - 1)
           for i = (N - 1):-1:1
               ex = :(I[$i] - 1 + dims[$i] * $ex)
           end
           return :($ex + 1)
       end
sub2ind_gen (generic function with 1 method)

julia> sub2ind_gen((3, 5), 1, 2)
4
```

**¿Qué código generará esto?**

Una forma sencilla de averiguarlo es extraer el cuerpo en otra función (regular):

```jldoctest sub2ind_gen2
julia> @generated function sub2ind_gen(dims::NTuple{N}, I::Integer...) where N
           return sub2ind_gen_impl(dims, I...)
       end
sub2ind_gen (generic function with 1 method)

julia> function sub2ind_gen_impl(dims::Type{T}, I...) where T <: NTuple{N,Any} where N
           length(I) == N || return :(error("partial indexing is unsupported"))
           ex = :(I[$N] - 1)
           for i = (N - 1):-1:1
               ex = :(I[$i] - 1 + dims[$i] * $ex)
           end
           return :($ex + 1)
       end
sub2ind_gen_impl (generic function with 1 method)
```

Ahora podemos ejecutar `sub2ind_gen_impl` y examinar la expresión que devuelve

```jldoctest sub2ind_gen2
julia> sub2ind_gen_impl(Tuple{Int,Int}, Int, Int)
:(((I[1] - 1) + dims[1] * (I[2] - 1)) + 1)
```

Por lo tanto, el cuerpo de método que se utilizará aquí no incluye un bucle en absoluto - sólo indexación en las dos tuplas, multiplicación y suma/resta. Todo el bucle se realiza en tiempo de compilación, y evitamos el bucle durante la ejecución por completo. Por lo tanto, sólo se realiza el bucle una vez por tipo, en este caso una vez por `N` (excepto en casos de borde donde la función se genera más de una vez - véase la exención de responsabilidad anterior).
