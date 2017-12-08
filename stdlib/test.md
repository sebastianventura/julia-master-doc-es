# Haciendo Pruebas Unitarias

```@meta
DocTestSetup = quote
    using Base.Test
end
```

## Probando Julia Base

Julia está en rápido desarrollo y cuenta con un amplio conjunto de pruebas para verificar su funcionalidad en múltiples plataformas. Si compila Julia desde el origen, puede ejecutar este conjunto de pruebas con `make test`. En una instalación binaria, puede ejecutar el conjunto de pruebas utilizando `Base.runtests()`.

```@docs
Base.runtests
```

## Pruebas Unitarias Básicas

El módulo `Base.Test` proporciona una funcionalidad simple de *realización de pruebas unitarias*. Las pruebas unitarias son una forma de ver si su código es correcto al verificar que los resultados sean los esperados. Puede ser útil asegurarse de que su código aún funcione después de realizar los cambios, y se puede usar al desarrollarlo como una forma de especificar los comportamientos que su código debería tener cuando se complete.

Se pueden realizar pruebas unitarias simples con las macros `@test ()` y `@test_throws ()`:

```@docs
Base.Test.@test
Base.Test.@test_throws
```

Por ejemplo, supongamos que queremos comprobar que nuestra nueva función `foo(x)` funciona como se esperaba:

```jldoctest testfoo
julia> using Base.Test

julia> foo(x) = length(x)^2
foo (generic function with 1 method)
```

Si la condición es cierta, se devuelve un `Pass`:

```jldoctest testfoo
julia> @test foo("bar") == 9
Test Passed

julia> @test foo("fizz") >= 10
Test Passed
```

Si la condición es falsa, se devuelve un `Fail` y se lanza una excepción:

```jldoctest testfoo
julia> @test foo("f") == 20
Test Failed
  Expression: foo("f") == 20
   Evaluated: 1 == 20
ERROR: There was an error during testing
```

Si la condición no pudo ser evaluada porque se lanzó una excepción, lo que ocurre en este caso porque `length()` no está definido para símbolos, se devuelve un objeto `Error` y se lanza una excepción:

```julia-repl
julia> @test foo(:cat) == 1
Error During Test
  Test threw an exception of type MethodError
  Expression: foo(:cat) == 1
  MethodError: no method matching length(::Symbol)
  Closest candidates are:
    length(::SimpleVector) at essentials.jl:256
    length(::Base.MethodList) at reflection.jl:521
    length(::MethodTable) at reflection.jl:597
    ...
  Stacktrace:
   [...]
ERROR: There was an error during testing
```

Si esperamos que al evaluar una expresión *deberían* lanzarse una excepción, entonces podemos usar `@test_throws()` para comprobar que esto es lo que ocurre:

```jldoctest testfoo
julia> @test_throws MethodError foo(:cat)
Test Passed
      Thrown: MethodError
```

## Trabajando con Conjuntos de Test

Normalmente, se utiliza una gran cantidad de pruebas para garantizar que las funciones trabajan correctamente sobre distintas entradas. En el caso de que una prueba falle, el comportamiento predeterminado es lanzar una excepción de inmediato. Sin embargo, normalmente es preferible ejecutar el resto de las pruebas primero para obtener una mejor idea de cuántos errores hay en el código que se prueba.

La macro `@testset()` se puede usar para agrupar las pruebas en *conjuntos*. En un conjunto de pruebas, se ejecutarán variasy al final de su realización se imprimirá un resumen. Si alguna de las pruebas falla o no se puede evaluar debido a un error, el conjunto de prueba arrojará una `TestSetException`.

```@docs
Base.Test.@testset
```

Podemos poner nuestros tests para la función `foo(x)` en un conjuntos de tests:

```jldoctest testfoo
julia> @testset "Foo Tests" begin
           @test foo("a")   == 1
           @test foo("ab")  == 4
           @test foo("abc") == 9
       end;
Test Summary: | Pass  Total
Foo Tests     |    3      3
```

Los conjuntos de pruebas pueden también anidarse:

```jldoctest testfoo
julia> @testset "Foo Tests" begin
           @testset "Animals" begin
               @test foo("cat") == 9
               @test foo("dog") == foo("cat")
           end
           @testset "Arrays $i" for i in 1:3
               @test foo(zeros(i)) == i^2
               @test foo(ones(i)) == i^2
           end
       end;
Test Summary: | Pass  Total
Foo Tests     |    8      8
```

En el caso de que un conjunto de pruebas anidado no tenga fallos, como pasa aquí, ello se ocultará en el resumen. Si tenemos un test que falle, sólo se mostrarán los detalles para este conjunto de tests que ha fallado:

```julia-repl
julia> @testset "Foo Tests" begin
           @testset "Animals" begin
               @testset "Felines" begin
                   @test foo("cat") == 9
               end
               @testset "Canines" begin
                   @test foo("dog") == 9
               end
           end
           @testset "Arrays" begin
               @test foo(zeros(2)) == 4
               @test foo(ones(4)) == 15
           end
       end

Arrays: Test Failed
  Expression: foo(ones(4)) == 15
   Evaluated: 16 == 15
Stacktrace:
    [...]
Test Summary: | Pass  Fail  Total
Foo Tests     |    3     1      4
  Animals     |    2            2
  Arrays      |    1     1      2
ERROR: Some tests did not pass: 3 passed, 1 failed, 0 errored, 0 broken.
```

## Otras Macros para Tests

Como los cálculos sobre valores en punto flotane pueden ser imprecisos, podemos realizar comprobaciones de igualdad aproximada usando `@test a ≈ b` (donde `≈`, se obtiene mediante terminación con tabulador de `\approx`, es la función [`isapprox()`](@ref)) o usar directamente [`isapprox()`](@ref).

```jldoctest
julia> @test 1 ≈ 0.999999999
Test Passed

julia> @test 1 ≈ 0.999999
Test Failed
  Expression: 1 ≈ 0.999999
   Evaluated: 1 ≈ 0.999999
ERROR: There was an error during testing
```

```@docs
Base.Test.@inferred
Base.Test.@test_warn
Base.Test.@test_nowarn
```

## Tests Rotos

Si un test falla consistentemente puede ser cambiado para utilizar la macro `@test_broken()`. Esto denotará el test como Roto  (`Broken`) si el test continua fallando y alterta al usuaria a traves de un `Error` si el test tiene éxito.


```@docs
Base.Test.@test_broken
```

`@test_skip()` está también disponible para saltar un test sin evaluación, pero contando el test que se ha saltado en el informe del conjunto de tests. El test no se ejecutará pero da un `Broken` `Result`.

```@docs
Base.Test.@test_skip
```

## Creando Tipos `AbstractTestSet` Personalizados

Los paquetes pueden crear sus propios subtipos `AbstractTestSet` implementando los métodos `record` y `finish`. El subtipo debe tener un constructor de un argumento que tome una cadena de descripción, con todas las opciones pasadas como argumentos  palabra clave.

```@docs
Base.Test.record
Base.Test.finish
```

`Base.Test` asume la responsabilidad de mantener una pila de conjuntos de pruebas anidados a medida que se ejecutan, pero cualquier acumulación de resultados es responsabilidad del subtipo` AbstractTestSet`. Puede acceder a esta pila con los métodos `get_testset` y` get_testset_depth`. Tenga en cuenta que estas funciones no se exportan.

```@docs
Base.Test.get_testset
Base.Test.get_testset_depth
```

`Base.Test` también se asegura de que las invocaciones `@testset` anidadas utilicen el mismo subtipo `AbstractTestSet` que sus padres a menos que se establezca explícitamente. Él no propaga ninguna propiedad del conjunto de pruebas. El comportamiento de herencia de opciones se puede implementar mediante paquetes que usan la infraestructura de pila que proporciona `Base.Test`.

La definición de un subtipo básico de 'AbstractTestSet` podría verse así:

```julia
import Base.Test: record, finish
using Base.Test: AbstractTestSet, Result, Pass, Fail, Error
using Base.Test: get_testset_depth, get_testset
struct CustomTestSet <: Base.Test.AbstractTestSet
    description::AbstractString
    foo::Int
    results::Vector
    # constructor takes a description string and options keyword arguments
    CustomTestSet(desc; foo=1) = new(desc, foo, [])
end

record(ts::CustomTestSet, child::AbstractTestSet) = push!(ts.results, child)
record(ts::CustomTestSet, res::Result) = push!(ts.results, res)
function finish(ts::CustomTestSet)
    # just record if we're not the top-level parent
    if get_testset_depth() > 0
        record(get_testset(), ts)
    end
    ts
end
```

Y usar este conjunto de test tiene el siguiente aspecto:

```julia
@testset CustomTestSet foo=4 "custom testset inner 2" begin
    # this testset should inherit the type, but not the argument.
    @testset "custom testset inner" begin
        @test true
    end
end
```

```@meta
DocTestSetup = nothing
```
