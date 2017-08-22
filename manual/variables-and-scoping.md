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

## Global Scope

*Cada módulo introduce un nuevo espacio global*, separado del ámbito global de todos los otros 
módulos; no existen ámbitos globales compartidos por todos. Los módulos pueden introducir variables 
de otros módulos o en su ámbito a través del uso de las instrucciones [`using`](@ref modules) o 
[`import`](@ref modules) o a través de acceso cualificado usando la notación punto. En consecuencia, 
cada módulo es un espacio de nombres. Notese que los enlaces de nombres pueden sólo ser cambiados 
dentro de su ámbito global y no desde un módulo exterior.

*Each module introduces a new global scope*, separate from the global scope of all other modules;
there is no all-encompassing global scope. Modules can introduce variables of other modules into
their scope through the [using or import](@ref modules) statements or through qualified access using the
dot-notation, i.e. each module is a so-called *namespace*. Note that variable bindings can only
be changed within their global scope and not from an outside module.

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

### Soft Local Scope

> In a soft local scope, all variables are inherited from its parent scope unless a variable is
> specifically marked with the keyword `local`.

Soft local scopes are introduced by for-loops, while-loops, comprehensions, try-catch-finally-blocks,
and let-blocks. There are some extra rules for [Let Blocks](@ref) and for [For Loops and Comprehensions](@ref).

In the following example the `x` and `y` refer always to the same variables as the soft local
scope inherits both read and write variables:

```jldoctest
julia> x, y = 0, 1;

julia> for i = 1:10
           x = i + y + 1
       end

julia> x
12
```

Within soft scopes, the *global* keyword is never necessary, although allowed. The only case
when it would change the semantics is (currently) a syntax error:

```jldoctest
julia> let
           local j = 2
           let
               global j = 3
           end
       end
ERROR: syntax: `global j`: j is local variable in the enclosing scope
```

### Hard Local Scope

Hard local scopes are introduced by function definitions (in all their forms), struct type definition blocks,
and macro-definitions.

> In a hard local scope, all variables are inherited from its parent scope unless:
>
>   * an assignment would result in a modified *global* variable, or
>   * a variable is specifically marked with the keyword `local`.

Thus global variables are only inherited for reading but not for writing:

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

An explicit `global` is needed to assign to a global variable:

```jldoctest
julia> x = 1;

julia> function foobar()
           global x = 2
       end;

julia> foobar();

julia> x
2
```

Note that *nested functions* can behave differently to functions defined in the global scope as
they can modify their parent scope's *local* variables:

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

The distinction between inheriting global and local variables for assignment can lead to some
slight differences between functions defined in local vs. global scopes. Consider the modification
of the last example by moving `bar` to the global scope:

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

Note that above subtlety does not pertain to type and macro definitions as they can only appear
at the global scope. There are special scoping rules concerning the evaluation of default and
keyword function arguments which are described in the [Function section](@ref man-functions).

An assignment introducing a variable used inside a function, type or macro definition need not
come before its inner usage:

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

This behavior may seem slightly odd for a normal variable, but allows for named functions -- which
are just normal variables holding function objects -- to be used before they are defined. This
allows functions to be defined in whatever order is intuitive and convenient, rather than forcing
bottom up ordering or requiring forward declarations, as long as they are defined by the time
they are actually called. As an example, here is an inefficient, mutually recursive way to test
if positive integers are even or odd:

```jldoctest
julia> even(n) = n == 0 ? true : odd(n-1);

julia> odd(n) = n == 0 ? false : even(n-1);

julia> even(3)
false

julia> odd(3)
true
```

Julia provides built-in, efficient functions to test for oddness and evenness called [`iseven()`](@ref)
and [`isodd()`](@ref) so the above definitions should only be taken as examples.

### Hard vs. Soft Local Scope

Blocks which introduce a soft local scope, such as loops, are generally used to manipulate the
variables in their parent scope. Thus their default is to fully access all variables in their
parent scope.

Conversely, the code inside blocks which introduce a hard local scope (function, type, and macro
definitions) can be executed at any place in a program. Remotely changing the state of global
variables in other modules should be done with care and thus this is an opt-in feature requiring
the `global` keyword.

The reason to allow *modifying local* variables of parent scopes in nested functions is to allow
constructing [closures](https://en.wikipedia.org/wiki/Closure_%28computer_programming%29) which
have a private state, for instance the `state` variable in the following example:

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

See also the closures in the examples in the next two sections.

### Let Blocks

Unlike assignments to local variables, `let` statements allocate new variable bindings each time
they run. An assignment modifies an existing value location, and `let` creates new locations.
This difference is usually not important, and is only detectable in the case of variables that
outlive their scope via closures. The `let` syntax accepts a comma-separated series of assignments
and variable names:

```jldoctest
julia> x, y, z = -1, -1, -1;

julia> let x = 1, z
           println("x: $x, y: $y") # x is local variable, y the global
           println("z: $z") # errors as z has not been assigned yet but is local
       end
x: 1, y: -1
ERROR: UndefVarError: z not defined
```

The assignments are evaluated in order, with each right-hand side evaluated in the scope before
the new variable on the left-hand side has been introduced. Therefore it makes sense to write
something like `let x = x` since the two `x` variables are distinct and have separate storage.
Here is an example where the behavior of `let` is needed:

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

Here we create and store two closures that return variable `i`. However, it is always the same
variable `i`, so the two closures behave identically. We can use `let` to create a new binding
for `i`:

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

Since the `begin` construct does not introduce a new scope, it can be useful to use a zero-argument
`let` to just introduce a new scope block without creating any new bindings:

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

Since `let` introduces a new scope block, the inner local `x` is a different variable than the
outer local `x`.

### For Loops and Comprehensions

`for` loops and [Comprehensions](@ref) have the following behavior: any new variables introduced
in their body scopes are freshly allocated for each loop iteration. This is in contrast to `while`
loops which reuse the variables for all iterations. Therefore these constructs are similar to
`while` loops with `let` blocks inside:

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

`for` loops will reuse existing variables for its iteration variable:

```jldoctest
julia> i = 0;

julia> for i = 1:3
       end

julia> i
3
```

However, comprehensions do not do this, and always freshly allocate their iteration variables:

```jldoctest
julia> x = 0;

julia> [ x for x = 1:3 ];

julia> x
0
```

## Constants

A common use of variables is giving names to specific, unchanging values. Such variables are only
assigned once. This intent can be conveyed to the compiler using the `const` keyword:

```jldoctest
julia> const e  = 2.71828182845904523536;

julia> const pi = 3.14159265358979323846;
```

The `const` declaration is allowed on both global and local variables, but is especially useful
for globals. It is difficult for the compiler to optimize code involving global variables, since
their values (or even their types) might change at almost any time. If a global variable will
not change, adding a `const` declaration solves this performance problem.

Local constants are quite different. The compiler is able to determine automatically when a local
variable is constant, so local constant declarations are not necessary for performance purposes.

Special top-level assignments, such as those performed by the `function` and `struct` keywords,
are constant by default.

Note that `const` only affects the variable binding; the variable may be bound to a mutable object
(such as an array), and that object may still be modified.
