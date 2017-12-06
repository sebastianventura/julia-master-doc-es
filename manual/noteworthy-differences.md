# Diferencias notables con otros idiomas

## Diferencias notables con MATLAB

Aunque los usuarios de MATLAB pueden encontrar la sintaxis de Julia familiar, Julia no es un clon de MATLAB. Hay importantes diferencias sintácticas y funcionales. Las siguientes son algunas diferencias notables que pueden hacer tropezar a los usuarios de Julia acostumbrados a MATLAB:

* Los arrays de Julia están indexados con corchetes, `A[i, j]`.
* Los arrays de Julia se asignan por referencia. Después de `A = B`, el cambio de elementos de `B` también modificará `A`.
* Los valores de Julia se pasan y se asignan por referencia. Si una función modifica una matriz, los cambios serán visibles en el código que la invoca.
* Julia no genera automáticamente matrices en una declaración de asignación. Mientras que en MATLAB `a(4) = 3.2` puede crear la matriz `a = [0 0 0 3.2]` y `a(5) = 7` puede crecer hasta `a = [0 0 0 3.2 7]`, la declaración correspondiente de Julia `a[5] = 7` arroja un error si la longitud de `a` es menor que 5 o si esta afirmación es el primer uso del identificador `a`. Julia tiene [`push!()`](@ref) y [`append!()`](@ref), que crecen `Vector`s mucho más eficientemente que `a(end + 1) = val` de MATLAB.
* La unidad imaginaria `sqrt(-1)` se representa en Julia como [`im`](@ref), no como `i` o `j` como en MATLAB.
* En Julia, los números literales sin un punto decimal (como `42`) crean números enteros en lugar de números de coma flotante. Se admiten literales enteros arbitrariamente grandes. Como resultado, algunas operaciones como `2^-1` arrojarán un error de dominio ya que el resultado no es un número entero (ver [la entrada de preguntas frecuentes sobre errores de dominio] (@ref faq-domain-errors) para más detalles).
* En Julia, los valores múltiples se devuelven y se asignan como tuplas, p. `(a, b) = (1, 2)` o `a, b = 1, 2`. `"nargout"` de MATLAB, que a menudo se usa en MATLAB para hacer trabajos opcionales basados en el número de valores devueltos, no existe en Julia. En cambio, los usuarios pueden usar argumentos opcionales y de palabras clave para lograr capacidades similares.
  
  * Julia tiene verdaderos arrays unidimensionales. Los vectores columna son de tamaño `N`, no` Nx1`. Por ejemplo, [`rand(N)`] (@ref) forma una matriz de 1 dimensión.
  * En Julia, `[x, y, z]` siempre construirá una matriz de 3 elementos que contiene `x`,` y` y `z`.
    - Para concatenar en la primera dimensión ("vertical") se usa [`vcat(x, y, z)`](@ref) o se separa con punto y coma (`[x; y; z]`).
    - Para concatenar en la segunda dimensión ("horizontal"), se usa [`hcat(x, y, z)`](@ref) o se separa con espacios (`[x y z]`).
    - Para construir matrices de bloques (concatenando en las dos primeras dimensiones), se usa [`hvcat()`](@ref) o se combinan espacios y puntos y comas (`[a b; c d]`).
* En Julia, `a:b` y` a:b:c` construyen objetos `Range`. Para construir un vector completo como en MATLAB, use [`collect(a:b)`](@ref). En general, no es necesario llamar a `collect` aunque. `Range` actuará como un array normal en la mayoría de los casos, pero es más eficiente porque calcula perezosamente sus valores. Este patrón de creación de objetos especializados en lugar de matrices completas se usa con frecuencia, y también se ve en funciones como [`linspace`](@ref), o con iteradores como `enumerate` y `zip`. Los objetos especiales se pueden usar principalmente como si fueran matrices normales.
* Las funciones en Julia devuelven valores de su última expresión o la palabra clave `return` en lugar de enumerar los nombres de las variables a devolver en la definición de la función (ver [La palabra clave return](@ref) para más detalles).
* Un script de Julia puede contener cualquier cantidad de funciones, y todas las definiciones serán visibles externamente cuando se cargue el archivo. Las definiciones de funciones se pueden cargar desde archivos fuera del directorio de trabajo actual.
* En Julia, las reducciones como [`sum()`](@ref), [`prod()`](@ref), y [`max()`](@ref) se realizan sobre cada elemento de un array cuando se llama con un solo argumento, como en `sum(A)`, incluso si `A` tiene más de una dimensión.
* En Julia, las funciones como [`sort()`](@ref) que operan en forma de columnas por defecto (`sort(A)` es equivalente a `sort(A,1)`) no tienen un comportamiento especial para Conjuntos `1xN`; el argumento se devuelve sin modificar ya que todavía ejecuta `sort (A,1)`. Para ordenar una matriz `1xN` como un vector, use` sort(A,2) `.
* En Julia, los paréntesis se deben usar para llamar a una función con cero argumentos, como en [`tic()`](@ref) y [`toc()`](@ref).
* Julia desalienta el uso de punto y coma para finalizar las declaraciones. Los resultados de las declaraciones no se imprimen automáticamente (excepto en el aviso interactivo), y las líneas de código no necesitan terminar con punto y coma. [`println()`](@ref) o [`@printf()`](@ref) se pueden usar para imprimir resultados específicos.
* En Julia, si `A` y` B` son matrices, las operaciones de comparación lógica como `A == B` no devuelven una matriz de booleanos. En cambio, use `A == B`, y de manera similar para los otros operadores booleanos como [`<`](@ref), [`>`](@ref) y `=`.
* En Julia, los operadores [`&`](@ref), [`|`](@ref) y [`⊻`](@ref xor) ([`xor`](@ref)) realizan el operaciones a nivel de bit equivalentes a `y`,` o`, y `xor` respectivamente en MATLAB, y tienen precedencia similar a los operadores de bit a bit de Python (a diferencia de C). Pueden operar en escalas o en elementos a través de matrices y se pueden usar para combinar matrices lógicas, pero tenga en cuenta la diferencia en el orden de las operaciones: pueden ser necesarios paréntesis (por ejemplo, para seleccionar elementos de 'A' igual a 1 o 2 use `(A. == 1) | (A. == 2) `).
* En Julia, los elementos de una colección se pueden pasar como argumentos a una función usando el operador splat `...`, como en `xs = [1,2]; f(xs...) `.
* Julia [`svd()`](@ref) devuelve valores singulares como un vector en lugar de una matriz diagonal densa.
* En Julia, `...` no se usa para continuar líneas de código. En cambio, las expresiones incompletas continúan automáticamente en la siguiente línea.
* Tanto en Julia como en MATLAB, la variable `ans` se establece en el valor de la última expresión emitida en una sesión interactiva. En Julia, a diferencia de MATLAB, `ans` no se establece cuando el código de Julia se ejecuta en modo no interactivo.
* Los `type`s de Julia no son compatibles con la adición dinámica de campos en el tiempo de ejecución, a diferencia de `classes` es de MATLAB. En su lugar, use un [`Dict`](@ref).
* En Julia, cada módulo tiene su propio ámbito / espacio de nombres global, mientras que en MATLAB solo hay un ámbito global.
* En MATLAB, una forma idiomática de eliminar valores no deseados es usar la indexación lógica, como en la expresión `x (x> 3)` o en la declaración `x (x> 3) = []` para modificar `x` en -lugar. Por el contrario, Julia proporciona las funciones de orden superior [`filter()`](@ref) y [`filter!()`](@ref), permitiendo a los usuarios escribir `filter (z-> z> 3, x)` y `filter! (z-> z> 3, x)` como alternativas a las transliteraciones correspondientes `x [x.> 3]` y `x = x [x.> 3]`. El uso de [`filter! ()`](@ref) reduce el uso de matrices temporales.
* El análogo de extracción (o "desreferenciación") de todos los elementos de una matriz de celdas, p. en `vertcat(A {:})` en MATLAB, se escribe utilizando el operador splat en Julia, p. como `vcat(A ...)`.
  
## Diferencias notables con R

Uno de los objetivos de Julia es proporcionar un lenguaje efectivo para el análisis de datos y la programación estadística. Para los usuarios que vienen a Julia de R, estas son algunas diferencias notables:

  * Las comillas simples de Julia encierran caracteres, no cadenas.
  * Julia puede crear subcadenas indexando en cadenas. En R, las cadenas deben convertirse en vectores de caracteres antes de crear subcadenas.
  * En Julia, como Python pero a diferencia de R, las cadenas se pueden crear con comillas triples `""" ... """`. Esta sintaxis es conveniente para construir cadenas que contienen saltos de línea.
  * En Julia, las variables se especifican utilizando el operador splat `...`, que siempre sigue el nombre de una variable específica, a diferencia de R, para el que `...` puede ocurrir de forma aislada.
  * En Julia, el módulo es `mod(a, b)`, no `a %% b`. `%` en Julia es el operador restante.
  * En Julia, no todas las estructuras de datos admiten la indexación lógica. Además, la indexación lógica en Julia solo se admite con vectores de longitud igual al objeto que se indexa. Por ejemplo:
  
      * En R, `c (1, 2, 3, 4) [c (TRUE, FALSE)]` es equivalente a `c (1, 3)`.
      * En R, `c (1, 2, 3, 4) [c (TRUE, FALSE, TRUE, FALSE)]` es equivalente a `c (1, 3)`.
      * En Julia, `[1, 2, 3, 4] [[verdadero, falso]]` arroja un [`BoundsError`] (@ ref).
      * En Julia, `[1, 2, 3, 4] [[verdadero, falso, verdadero, falso]]` produce `[1, 3]`.
  * Como en muchos lenguajes, Julia no siempre permite operaciones en vectores de diferentes longitudes, a diferencia de R, donde los vectores solo necesitan compartir un rango de índice común. Por ejemplo, `c(1, 2, 3, 4) + c (1, 2)` es válido R pero el equivalente `[1, 2, 3, 4] + [1, 2]` arrojará un error en Julia.
  * Julia [`map()`](@ref) toma primero la función, luego sus argumentos, a diferencia de `lapply (<structure>, function, ...)` en R. De forma similar, el equivalente de Julia de `apply(X, MARGIN , FUN, ...)` en R es[` mapslices ()`](@ref) donde la función es el primer argumento.
  * La aplicación multivariada en R, es decir, `mapply(choose, 11:13, 1: 3)`, se puede escribir como `broadcast(binomial, 11:13, 1:3)` en Julia. Equivalentemente, Julia ofrece una sintaxis de punto más corta para vectorizar funciones `binomial.(11:13, 1: 3)`.
  * Julia usa `end` para denotar el final de bloques condicionales, como` if`, bloques de bucle, como `while` /` for`, y funciones. En lugar de la sentencia de una línea `if (cond)`, Julia permite las declaraciones de la forma `if cond; declaración; end`, `cond && statement` y `!cond || declaración`. Las declaraciones de asignación en las dos últimas sintaxis deben estar explícitamente entrelazadas entre paréntesis, por ejemplo, `cond && (x = value)`.
  * En Julia, `<-`,` << - `y` -> `no son operadores de asignación.
  * En Julia, `->` crea una función anónima, como en Python.
  * Julia construye vectores usando corchetes. El `[1, 2, 3]` de Julia es el equivalente de R a `c(1, 2, 3)`.
  * El operador de Julia [`*`](@ref) puede realizar la multiplicación de la matriz, a diferencia de R. Si `A` y` B` son matrices, entonces `A * B` denota una multiplicación de matriz en Julia, equivalente a R `A %*% B`. En R, esta misma notación realizaría un producto de elemento (Hadamard). Para obtener la operación de multiplicación por elementos, debe escribir `A.*B` en Julia.
  * Julia realiza la transposición de la matriz utilizando el operador `.'` y la transposición conjugada con el operador `'`. Por lo tanto, `A.'` de Julia es equivalente a `t(A)`en R.
  * Julia no requiere paréntesis cuando escribe bucles `if` o` for` / `while`: use` for i en [1, 2, 3] `en lugar de` for (i en c (1, 2, 3) )` o `if i == 1` en lugar de `if (i == 1)`.
  * Julia no trata los números `0` y` 1` como booleanos. No puede escribir `if(1)` en Julia, porque las sentencias `if` solo aceptan booleanos. En su lugar, puede escribir `if true` , `if Bool(1) `, o` if 1 == 1`.
  * Julia no proporciona `nrow` y` ncol`. En su lugar, use `size(M,1)` para `nrow(M)` y `size(M,2)` para `ncol(M)`.
  * Julia tiene cuidado de distinguir escalares, vectores y matrices. En R, `1` y `c(1)` son lo mismo.
    En Julia, no se pueden usar indistintamente. Un resultado potencialmente confuso de esto es que `x'* y` para los vectores `x` e `y` es un vector de 1 elemento, no escalar. Para obtener un escalar, use [`dot(x, y)`](@ref).
  * Julia's [`diag()`] (@ref) y [`diagm()`](@ref) no son como R's.
  * Julia no puede asignar los resultados de llamadas a funciones en el lado izquierdo de una operación de asignación: no puede escribir `diag(M) = ones(n)`.
  * Julia desaconseja llenar el espacio de nombres principal con funciones. La mayoría de las funcionalidades estadísticas para Julia se encuentran en [paquetes](http://pkg.julialang.org/) bajo la [organización JuliaStats](https://github.com/JuliaStats). Por ejemplo:
  
      * Las funciones relacionadas con las distribuciones de probabilidad son proporcionadas por el [Paquete Distributions](https://github.com/JuliaStats/Distributions.jl).
      * El [paquete DataFrames](https://github.com/JuliaStats/DataFrames.jl) proporciona el tipo de datos `DataFrame`.
      * El [paquete GLM] (https://github.com/JuliaStats/GLM.jl) proporciona modelos lineales generalizados.
      
  * Julia proporciona tuplas y tablas hash reales, pero no listas R-style. Al devolver varios elementos, normalmente debe usar una tupla: en lugar de `list(a = 1, b = 2)`, use `(1, 2)`.
  * Julia anima a los usuarios a escribir sus propios tipos, que son más fáciles de usar que los objetos S3 o S4 de R. El sistema de despacho múltiple de Julia significa que `table(x::TypeA)` y `table(x::TypeB)` actúan como R `table.TypeA(x)` y `table.TypeB(x)`.
  * En Julia, los valores se pasan y se asignan por referencia. Si una función modifica una matriz, los cambios serán visibles en el código que invoca la función. Esto es muy diferente de R y permite que las nuevas funciones operen en estructuras de datos de gran tamaño de manera mucho más eficiente.
  * En Julia, los vectores y las matrices se concatenan usando [`hcat()`](@ref), [`vcat()`](@ref) y [`hvcat()`](@ref), no `c` , `rbind` y` cbind` como en R.
  * En Julia, un rango como `a:b` no es una abreviatura para un vector como en R, sino que es un `Range`especializado que se usa para la iteración sin una gran sobrecarga de memoria. Para convertir un rango en un vector, use [`collect(a:b)`](@ref).
  * Las funciones de Julia [`max()`](@ref) y [`min()`](@ref) son el equivalente de `pmax` y `pmin` respectivamente en R, pero ambos argumentos deben tener las mismas dimensiones. Mientras que [`maximum()`](@ref) y [`minimum()`](@ref) reemplazan `max` y `min` en R, hay diferencias importantes.
  * Las funciones de Julia  [`sum()`](@ref), [`prod()`](@ref), [`maximum()`](@ref), y [`minimum()`](@ref) son diferentes de sus contrapartes en R. Todos aceptan uno o dos argumentos. El primer argumento es una colección iterable tal como una matriz. Si hay un segundo argumento, este argumento indica las dimensiones sobre las cuales se lleva a cabo la operación. Por ejemplo, deje `A = [[1 2], [3 4]]` en Julia y `B = rbind(c(1,2), c(3,4))` sea la misma matriz en R. Luego `sum(A)` da el mismo resultado que `sum(B)`, pero `sum(A, 1)` es un vector de fila que contiene la suma sobre cada columna y `suma (A, 2)` es un vector de columna que contiene la suma sobre cada fila. Esto contrasta con el comportamiento de R, donde `suma (B, 1) = 11` y` suma (B, 2) = 12`. Si el segundo argumento es un vector, entonces especifica todas las dimensiones sobre las cuales se realiza la suma, por ejemplo, `suma (A, [1,2]) = 10`. Cabe señalar que no hay errores de comprobación con respecto al segundo argumento.
  * Julia tiene varias funciones que pueden mutar sus argumentos. Por ejemplo, tiene ambos [`sort ()`] (@ ref) y [`sort! ()`] (@ Ref).
  * En R, el rendimiento requiere vectorización. En Julia, casi todo lo contrario es cierto: el código de mejor rendimiento a menudo se logra mediante el uso de bucles devectorized.
  * Julia es evaluada con entusiasmo y no es compatible con la evaluación perezosa de estilo R. Para la mayoría de los usuarios, esto significa que hay muy pocas expresiones sin comillas o nombres de columnas.
  * Julia no admite el tipo `NULL`.
  * A Julia le falta el equivalente de "asignar" o "obtener" de R.
  * En Julia, `return` no requiere paréntesis.
  * En R, una forma idiomática de eliminar valores no deseados es usar indexación lógica, como en la expresión `x[x> 3]` o en la declaración `x = x[x > 3]` para modificar `x` lugar. Por el contrario, Julia proporciona las funciones de orden superior [`filter()`] (@ ref) y [`filter!()`] (@ Ref), permitiendo a los usuarios escribir `filter (z -> z>3, x) `y` filter! (z-> z> 3, x) `como alternativas a las transliteraciones correspondientes` x[x .> 3] `y` x = x [x.> 3] `. El uso de [`filter!()`](@ref) reduce el uso de matrices temporales.

## Diferencias notables con Python

* Julia requiere `end` para finalizar un bloque. A diferencia de Python, Julia no tiene la palabra clave `pass`.
* En Julia, la indexación de matrices, cadenas, etc. se basa en 1 y no en 0.
* La indexación de segmentos de Julia incluye el último elemento, a diferencia de Python. `a[2:3]` en Julia es `a[1:3]` en Python.
* Julia no admite índices negativos. En particular, el último elemento de una lista o matriz se indexa con `end` en Julia, no` -1` como en Python.
* Los bloques `for`,` if`, `while`, etc. de Julia terminan con la palabra clave `end`. El nivel de sangría no es significativo ya que está en Python.
* Julia no tiene sintaxis de continuación de línea: si, al final de una línea, la entrada hasta ahora es una expresión completa, se considera hecho; de lo contrario, la entrada continúa. Una forma de forzar que una expresión continúe es envolverla entre paréntesis.
* Los arrays de Julia son *column major* (orden de Fortran) mientras que los arrays de NumPy son *row major* (orden de C) por defecto. Para obtener un rendimiento óptimo al alternar sobre matrices, el orden de los bucles debe invertirse en Julia en relación con NumPy (consulte la sección correspondiente de [Sugerencias de rendimiento](@ref man-performance-tips)).
* Los operadores de actualización de Julia (por ejemplo, `+=`, `-=`, ...) *no están en el lugar* mientras que los de NumPy sí lo están. Esto significa `A = ones(4); B = A; B += 3` no cambia los valores en `A`, sino que vuelve a enlazar el nombre` B` con el resultado del lado derecho `B = B + 3`, que es una nueva matriz. Para la operación in-situ, use `B.+= 3` (vea también [operadores de punto](@ref man-dot-operators)), loops explícitos, o `InplaceOps.jl`.
* Julia evalúa los valores predeterminados de los argumentos de la función cada vez que se invoca el método, a diferencia de Python, donde los valores predeterminados se evalúan solo una vez cuando se define la función. Por ejemplo, la función `f(x = rand()) = x` devuelve un nuevo número aleatorio cada vez que se invoca sin argumentos. Por otro lado, la función `g (x = [1,2]) = push!(X, 3)` devuelve `[1,2,3]` cada vez que se llama como `g()`.
* En Julia `%` es el operador restante, mientras que en Python es el módulo.

## Noteworthy differences from C/C++

  * Julia arrays are indexed with square brackets, and can have more than one dimension `A[i,j]`.
    This syntax is not just syntactic sugar for a reference to a pointer or address as in C/C++. See
    the Julia documentation for the syntax for array construction (it has changed between versions).
  * In Julia, indexing of arrays, strings, etc. is 1-based not 0-based.
  * Julia arrays are assigned by reference. After `A=B`, changing elements of `B` will modify `A`
    as well. Updating operators like `+=` do not operate in-place, they are equivalent to `A = A + B`
    which rebinds the left-hand side to the result of the right-hand side expression.
  * Julia arrays are column major (Fortran ordered) whereas C/C++ arrays are row major ordered by
    default. To get optimal performance when looping over arrays, the order of the loops should be
    reversed in Julia relative to C/C++ (see relevant section of [Performance Tips](@ref man-performance-tips)).
  * Julia values are passed and assigned by reference. If a function modifies an array, the changes
    will be visible in the caller.
  * In Julia, whitespace is significant, unlike C/C++, so care must be taken when adding/removing
    whitespace from a Julia program.
  * In Julia, literal numbers without a decimal point (such as `42`) create signed integers, of type
    `Int`, but literals too large to fit in the machine word size will automatically be promoted to
    a larger size type, such as `Int64` (if `Int` is `Int32`), `Int128`, or the arbitrarily large
    `BigInt` type. There are no numeric literal suffixes, such as `L`, `LL`, `U`, `UL`, `ULL` to indicate
    unsigned and/or signed vs. unsigned. Decimal literals are always signed, and hexadecimal literals
    (which start with `0x` like C/C++), are unsigned. Hexadecimal literals also, unlike C/C++/Java
    and unlike decimal literals in Julia, have a type based on the *length* of the literal, including
    leading 0s. For example, `0x0` and `0x00` have type [`UInt8`](@ref), `0x000` and `0x0000` have type
    [`UInt16`](@ref), then literals with 5 to 8 hex digits have type `UInt32`, 9 to 16 hex digits type
    `UInt64` and 17 to 32 hex digits type `UInt128`. This needs to be taken into account when defining
    hexadecimal masks, for example `~0xf == 0xf0` is very different from `~0x000f == 0xfff0`. 64 bit `Float64`
    and 32 bit [`Float32`](@ref) bit literals are expressed as `1.0` and `1.0f0` respectively. Floating point
    literals are rounded (and not promoted to the `BigFloat` type) if they can not be exactly represented.
     Floating point literals are closer in behavior to C/C++. Octal (prefixed with `0o`) and binary
    (prefixed with `0b`) literals are also treated as unsigned.
  * String literals can be delimited with either `"`  or `"""`, `"""` delimited literals can contain
    `"` characters without quoting it like `"\""` String literals can have values of other variables
    or expressions interpolated into them, indicated by `$variablename` or `$(expression)`, which
    evaluates the variable name or the expression in the context of the function.
  * `//` indicates a [`Rational`](@ref) number, and not a single-line comment (which is `#` in Julia)
  * `#=` indicates the start of a multiline comment, and `=#` ends it.
  * Functions in Julia return values from their last expression(s) or the `return` keyword.  Multiple
    values can be returned from functions and assigned as tuples, e.g. `(a, b) = myfunction()` or
    `a, b = myfunction()`, instead of having to pass pointers to values as one would have to do in
    C/C++ (i.e. `a = myfunction(&b)`.
  * Julia does not require the use of semicolons to end statements. The results of expressions are
    not automatically printed (except at the interactive prompt, i.e. the REPL), and lines of code
    do not need to end with semicolons. [`println()`](@ref) or [`@printf()`](@ref) can be used to
    print specific output. In the REPL, `;` can be used to suppress output. `;` also has a different
    meaning within `[ ]`, something to watch out for. `;` can be used to separate expressions on a
    single line, but are not strictly necessary in many cases, and are more an aid to readability.
  * In Julia, the operator [`⊻`](@ref xor) ([`xor`](@ref)) performs the bitwise XOR operation, i.e.
    [`^`](@ref) in C/C++.  Also, the bitwise operators do not have the same precedence as C/++, so
    parenthesis may be required.
  * Julia's [`^`](@ref) is exponentiation (pow), not bitwise XOR as in C/C++ (use [`⊻`](@ref xor), or
    [`xor`](@ref), in Julia)
  * Julia has two right-shift operators, `>>` and `>>>`.  `>>>` performs an arithmetic shift, `>>`
    always performs a logical shift, unlike C/C++, where the meaning of `>>` depends on the type of
    the value being shifted.
  * Julia's `->` creates an anonymous function, it does not access a member via a pointer.
  * Julia does not require parentheses when writing `if` statements or `for`/`while` loops: use `for i in [1, 2, 3]`
    instead of `for (int i=1; i <= 3; i++)` and `if i == 1` instead of `if (i == 1)`.
  * Julia does not treat the numbers `0` and `1` as Booleans. You cannot write `if (1)` in Julia,
    because `if` statements accept only booleans. Instead, you can write `if true`, `if Bool(1)`,
    or `if 1==1`.
  * Julia uses `end` to denote the end of conditional blocks, like `if`, loop blocks, like `while`/
    `for`, and functions. In lieu of the one-line `if ( cond ) statement`, Julia allows statements
    of the form `if cond; statement; end`, `cond && statement` and `!cond || statement`. Assignment
    statements in the latter two syntaxes must be explicitly wrapped in parentheses, e.g. `cond && (x = value)`,
    because of the operator precedence.
  * Julia has no line continuation syntax: if, at the end of a line, the input so far is a complete
    expression, it is considered done; otherwise the input continues. One way to force an expression
    to continue is to wrap it in parentheses.
  * Julia macros operate on parsed expressions, rather than the text of the program, which allows
    them to perform sophisticated transformations of Julia code. Macro names start with the `@` character,
    and have both a function-like syntax, `@mymacro(arg1, arg2, arg3)`, and a statement-like syntax,
    `@mymacro arg1 arg2 arg3`. The forms are interchangable; the function-like form is particularly
    useful if the macro appears within another expression, and is often clearest. The statement-like
    form is often used to annotate blocks, as in the parallel `for` construct: `@parallel for i in 1:n; #= body =#; end`.
    Where the end of the macro construct may be unclear, use the function-like form.
  * Julia now has an enumeration type, expressed using the macro `@enum(name, value1, value2, ...)`
    For example: `@enum(Fruit, banana=1, apple, pear)`
  * By convention, functions that modify their arguments have a `!` at the end of the name, for example
    `push!`.
  * In C++, by default, you have static dispatch, i.e. you need to annotate a function as virtual,
    in order to have dynamic dispatch. On the other hand, in Julia every method is "virtual" (although
    it's more general than that since methods are dispatched on every argument type, not only `this`,
    using the most-specific-declaration rule).
