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
  
## Noteworthy differences from R

One of Julia's goals is to provide an effective language for data analysis and statistical programming.
For users coming to Julia from R, these are some noteworthy differences:

  * Julia's single quotes enclose characters, not strings.
  * Julia can create substrings by indexing into strings. In R, strings must be converted into character
    vectors before creating substrings.
  * In Julia, like Python but unlike R, strings can be created with triple quotes `""" ... """`. This
    syntax is convenient for constructing strings that contain line breaks.
  * In Julia, varargs are specified using the splat operator `...`, which always follows the name
    of a specific variable, unlike R, for which `...` can occur in isolation.
  * In Julia, modulus is `mod(a, b)`, not `a %% b`. `%` in Julia is the remainder operator.
  * In Julia, not all data structures support logical indexing. Furthermore, logical indexing in Julia
    is supported only with vectors of length equal to the object being indexed. For example:

      * In R, `c(1, 2, 3, 4)[c(TRUE, FALSE)]` is equivalent to `c(1, 3)`.
      * In R, `c(1, 2, 3, 4)[c(TRUE, FALSE, TRUE, FALSE)]` is equivalent to `c(1, 3)`.
      * In Julia, `[1, 2, 3, 4][[true, false]]` throws a [`BoundsError`](@ref).
      * In Julia, `[1, 2, 3, 4][[true, false, true, false]]` produces `[1, 3]`.
  * Like many languages, Julia does not always allow operations on vectors of different lengths, unlike
    R where the vectors only need to share a common index range.  For example, `c(1, 2, 3, 4) + c(1, 2)`
    is valid R but the equivalent `[1, 2, 3, 4] + [1, 2]` will throw an error in Julia.
  * Julia's [`map()`](@ref) takes the function first, then its arguments, unlike `lapply(<structure>, function, ...)`
    in R. Similarly Julia's equivalent of `apply(X, MARGIN, FUN, ...)` in R is [`mapslices()`](@ref)
    where the function is the first argument.
  * Multivariate apply in R, e.g. `mapply(choose, 11:13, 1:3)`, can be written as `broadcast(binomial, 11:13, 1:3)`
    in Julia. Equivalently Julia offers a shorter dot syntax for vectorizing functions `binomial.(11:13, 1:3)`.
  * Julia uses `end` to denote the end of conditional blocks, like `if`, loop blocks, like `while`/
    `for`, and functions. In lieu of the one-line `if ( cond ) statement`, Julia allows statements
    of the form `if cond; statement; end`, `cond && statement` and `!cond || statement`. Assignment
    statements in the latter two syntaxes must be explicitly wrapped in parentheses, e.g. `cond && (x = value)`.
  * In Julia, `<-`, `<<-` and `->` are not assignment operators.
  * Julia's `->` creates an anonymous function, like Python.
  * Julia constructs vectors using brackets. Julia's `[1, 2, 3]` is the equivalent of R's `c(1, 2, 3)`.
  * Julia's [`*`](@ref) operator can perform matrix multiplication, unlike in R. If `A` and `B` are
    matrices, then `A * B` denotes a matrix multiplication in Julia, equivalent to R's `A %*% B`.
    In R, this same notation would perform an element-wise (Hadamard) product. To get the element-wise
    multiplication operation, you need to write `A .* B` in Julia.
  * Julia performs matrix transposition using the `.'` operator and conjugated transposition using
    the `'` operator. Julia's `A.'` is therefore equivalent to R's `t(A)`.
  * Julia does not require parentheses when writing `if` statements or `for`/`while` loops: use `for i in [1, 2, 3]`
    instead of `for (i in c(1, 2, 3))` and `if i == 1` instead of `if (i == 1)`.
  * Julia does not treat the numbers `0` and `1` as Booleans. You cannot write `if (1)` in Julia,
    because `if` statements accept only booleans. Instead, you can write `if true`, `if Bool(1)`,
    or `if 1==1`.
  * Julia does not provide `nrow` and `ncol`. Instead, use `size(M, 1)` for `nrow(M)` and `size(M, 2)`
    for `ncol(M)`.
  * Julia is careful to distinguish scalars, vectors and matrices.  In R, `1` and `c(1)` are the same.
    In Julia, they can not be used interchangeably. One potentially confusing result of this is that
    `x' * y` for vectors `x` and `y` is a 1-element vector, not a scalar. To get a scalar, use [`dot(x, y)`](@ref).
  * Julia's [`diag()`](@ref) and [`diagm()`](@ref) are not like R's.
  * Julia cannot assign to the results of function calls on the left hand side of an assignment operation:
    you cannot write `diag(M) = ones(n)`.
  * Julia discourages populating the main namespace with functions. Most statistical functionality
    for Julia is found in [packages](http://pkg.julialang.org/) under the [JuliaStats organization](https://github.com/JuliaStats).
    For example:

      * Functions pertaining to probability distributions are provided by the [Distributions package](https://github.com/JuliaStats/Distributions.jl).
      * The [DataFrames package](https://github.com/JuliaStats/DataFrames.jl) provides data frames.
      * Generalized linear models are provided by the [GLM package](https://github.com/JuliaStats/GLM.jl).
  * Julia provides tuples and real hash tables, but not R-style lists. When returning multiple items,
    you should typically use a tuple: instead of `list(a = 1, b = 2)`, use `(1, 2)`.
  * Julia encourages users to write their own types, which are easier to use than S3 or S4 objects
    in R. Julia's multiple dispatch system means that `table(x::TypeA)` and `table(x::TypeB)` act
    like R's `table.TypeA(x)` and `table.TypeB(x)`.
  * In Julia, values are passed and assigned by reference. If a function modifies an array, the changes
    will be visible in the caller. This is very different from R and allows new functions to operate
    on large data structures much more efficiently.
  * In Julia, vectors and matrices are concatenated using [`hcat()`](@ref), [`vcat()`](@ref) and
    [`hvcat()`](@ref), not `c`, `rbind` and `cbind` like in R.
  * In Julia, a range like `a:b` is not shorthand for a vector like in R, but is a specialized `Range`
    that is used for iteration without high memory overhead. To convert a range into a vector, use
    [`collect(a:b)`](@ref).
  * Julia's [`max()`](@ref) and [`min()`](@ref) are the equivalent of `pmax` and `pmin` respectively
    in R, but both arguments need to have the same dimensions.  While [`maximum()`](@ref) and [`minimum()`](@ref)
    replace `max` and `min` in R, there are important differences.
  * Julia's [`sum()`](@ref), [`prod()`](@ref), [`maximum()`](@ref), and [`minimum()`](@ref) are different
    from their counterparts in R. They all accept one or two arguments. The first argument is an iterable
    collection such as an array.  If there is a second argument, then this argument indicates the
    dimensions, over which the operation is carried out.  For instance, let `A=[[1 2],[3 4]]` in Julia
    and `B=rbind(c(1,2),c(3,4))` be the same matrix in R.  Then `sum(A)` gives the same result as
    `sum(B)`, but `sum(A, 1)` is a row vector containing the sum over each column and `sum(A, 2)`
    is a column vector containing the sum over each row.  This contrasts to the behavior of R, where
    `sum(B,1)=11` and `sum(B,2)=12`.  If the second argument is a vector, then it specifies all the
    dimensions over which the sum is performed, e.g., `sum(A,[1,2])=10`.  It should be noted that
    there is no error checking regarding the second argument.
  * Julia has several functions that can mutate their arguments. For example, it has both [`sort()`](@ref)
    and [`sort!()`](@ref).
  * In R, performance requires vectorization. In Julia, almost the opposite is true: the best performing
    code is often achieved by using devectorized loops.
  * Julia is eagerly evaluated and does not support R-style lazy evaluation. For most users, this
    means that there are very few unquoted expressions or column names.
  * Julia does not support the `NULL` type.
  * Julia lacks the equivalent of R's `assign` or `get`.
  * In Julia, `return` does not require parentheses.
  * In R, an idiomatic way to remove unwanted values is to use logical indexing, like in the expression
    `x[x>3]` or in the statement `x = x[x>3]` to modify `x` in-place. In contrast, Julia provides
    the higher order functions [`filter()`](@ref) and [`filter!()`](@ref), allowing users to write
    `filter(z->z>3, x)` and `filter!(z->z>3, x)` as alternatives to the corresponding transliterations
    `x[x.>3]` and `x = x[x.>3]`. Using [`filter!()`](@ref) reduces the use of temporary arrays.

## Noteworthy differences from Python

  * Julia requires `end` to end a block. Unlike Python, Julia has no `pass` keyword.
  * In Julia, indexing of arrays, strings, etc. is 1-based not 0-based.
  * Julia's slice indexing includes the last element, unlike in Python. `a[2:3]` in Julia is `a[1:3]`
    in Python.
  * Julia does not support negative indexes. In particular, the last element of a list or array is
    indexed with `end` in Julia, not `-1` as in Python.
  * Julia's `for`, `if`, `while`, etc. blocks are terminated by the `end` keyword. Indentation level
    is not significant as it is in Python.
  * Julia has no line continuation syntax: if, at the end of a line, the input so far is a complete
    expression, it is considered done; otherwise the input continues. One way to force an expression
    to continue is to wrap it in parentheses.
  * Julia arrays are column major (Fortran ordered) whereas NumPy arrays are row major (C-ordered)
    by default. To get optimal performance when looping over arrays, the order of the loops should
    be reversed in Julia relative to NumPy (see relevant section of [Performance Tips](@ref man-performance-tips)).
  * Julia's updating operators (e.g. `+=`, `-=`, ...) are *not in-place* whereas NumPy's are. This
    means `A = ones(4); B = A; B += 3` doesn't change values in `A`, it rather rebinds the name `B`
    to the result of the right-hand side `B = B + 3`, which is a new array. For in-place operation, use `B .+= 3`
    (see also [dot operators](@ref man-dot-operators)), explicit loops, or `InplaceOps.jl`.
  * Julia evaluates default values of function arguments every time the method is invoked, unlike
    in Python where the default values are evaluated only once when the function is defined. For example,
    the function `f(x=rand()) = x` returns a new random number every time it is invoked without argument.
    On the other hand, the function `g(x=[1,2]) = push!(x,3)` returns `[1,2,3]` every time it is called
    as `g()`.
  * In Julia `%` is the remainder operator, whereas in Python it is the modulus.

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
