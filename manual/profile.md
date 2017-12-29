# *Profiling*

El módulo `Profile` proporciona herramientas para ayudar a los desarrolladores a mejorar el rendimiento de su
código. Cuando se usa, toma medidas de código en ejecución y produce resultados que lo ayudan a comprender
cuánto tiempo se gasta en línea(s) individual(es). El uso más común es identificar "cuellos de botella"
como objetivos para la optimización.

`Profile` implementa lo que se conoce como "sampling" o [*profiler* estadístico](https://en.wikipedia.org/wiki/Profiling_ (computer_programming)). Funciona periódicamente tomando una traza inversa durante la ejecución de cualquier tarea. Cada traz inversa captura la función que se está ejecutando actualmente y el número de línea, más la cadena completa de llamadas a función que llevó a esta línea, y por lo tanto es una "instantánea" del estado actual de ejecución.

Si se gasta una gran parte de su tiempo de ejecución al ejecutar una línea particular de código, esta línea aparecerá con frecuencia en el conjunto de todas las trazas inversas. En otras palabras, el "costo" de una línea determinada -o, en realidad, el costo de la secuencia de llamadas de función hasta e incluyendo esta línea- es proporcional a la frecuencia con que aparece en el conjunto de todas las trazas inversas.

Un generador de perfiles de muestreo (*sampling profiler*) no proporciona una cobertura completa línea por línea, porque las trazas inversas se producen a intervalos (por defecto, 1 ms en sistemas Unix y 10 ms en Windows, aunque la programación real está sujeta a la carga del sistema operativo). Además, como se analiza más adelante, como las muestras se recogen en un subconjunto disperso de todos los puntos de ejecución, los datos recopilados por un generador de perfiles de muestreo están sujetos a ruido estadístico.

A pesar de estas limitaciones, los perfiles de muestreo tienen fortalezas sustanciales:

  * No tiene que hacer ninguna modificación en su código para tomar medidas de temporización (en c
  ontraste con la alternativa [instrumenting profiler](https://github.com/timholy/IProfile.jl)).  
  * Puede perfilarse en el código central de Julia e incluso (opcionalmente) en las bibliotecas C y Fortran.
  * Al ejecutarse "con poca frecuencia", hay muy poca sobrecarga de rendimiento; durante el perfilado, 
    su código puede ejecutarse a una velocidad casi nativa.

Por estas razones, se recomienda que intente utilizar el generador de perfiles de muestreo incorporado antes de considerar cualquier alternativa.

## Uso básico

Trabajemos con un simple caso de test:

```julia-repl
julia> function myfunc()
           A = rand(200, 200, 400)
           maximum(A)
       end
```

Es buena idea ejecutar primero el código que se intenta analizar al menos una vez (a menos que uno quiera analizar el compilardor JIT de Julia):

```julia-repl
julia> myfunc() # run once to force compilation
```

Ahora estamos listos para analizar esta función:

```julia-repl
julia> @profile myfunc()
```

Para ver los resultados del *profiler* hay disponible un [navegador gráfico](https://github.com/timholy/ProfileView.jl), pero aquí usaremos la pantalla basada en texto que viene con la librería estándar:

```julia-repl
julia> Profile.print()
80 ./event.jl:73; (::Base.REPL.##1#2{Base.REPL.REPLBackend})()
 80 ./REPL.jl:97; macro expansion
  80 ./REPL.jl:66; eval_user_input(::Any, ::Base.REPL.REPLBackend)
   80 ./boot.jl:235; eval(::Module, ::Any)
    80 ./<missing>:?; anonymous
     80 ./profile.jl:23; macro expansion
      52 ./REPL[1]:2; myfunc()
       38 ./random.jl:431; rand!(::MersenneTwister, ::Array{Float64,3}, ::Int64, ::Type{B...
        38 ./dSFMT.jl:84; dsfmt_fill_array_close_open!(::Base.dSFMT.DSFMT_state, ::Ptr{F...
       14 ./random.jl:278; rand
        14 ./random.jl:277; rand
         14 ./random.jl:366; rand
          14 ./random.jl:369; rand
      28 ./REPL[1]:3; myfunc()
       28 ./reduce.jl:270; _mapreduce(::Base.#identity, ::Base.#scalarmax, ::IndexLinear,...
        3  ./reduce.jl:426; mapreduce_impl(::Base.#identity, ::Base.#scalarmax, ::Array{F...
        25 ./reduce.jl:428; mapreduce_impl(::Base.#identity, ::Base.#scalarmax, ::Array{F...
```

Cada línea de esta pantalla representa un punto particular (número de línea) en el código. La sangría se usa para indicar la secuencia anidada de llamadas a funciones, con líneas más sangradas que son más profundas en la secuencia de llamadas. En cada línea, el primer "campo" es el número de trazas inversas (muestras) tomadas *en esta línea o en cualquier función ejecutada por esta línea*. El segundo campo es el nombre del archivo y el número de línea, y el tercer campo es el nombre de la función. Tenga en cuenta que los números de línea específicos pueden cambiar como los cambios de código de Julia; si quieres seguir, es mejor que ejecutes este ejemplo tú mismo.

En este ejemplo, podemos ver que la función de nivel superior llamada está en el archivo `event.jl`. Esta es la función que ejecuta REPL cuando se lanza Julia. Si se examina la línea 97 de `REPL.jl`, verá que aquí es donde se llama a la función `eval_user_input()`. Esta es la función que evalúa lo que escribes en el REPL, y dado que estamos trabajando de forma interactiva estas funciones se invocaron cuando ingresamos `@profile myfunc()`. La siguiente línea refleja las acciones tomadas en la macro [`@ profile`](@ref).

La primera línea muestra que se tomaron 80 trazas inversas en la línea 73 de `event.jl`, pero no es que esta línea fuera "costosa" por sí misma: la tercera línea revela que las 80 trazas inversas se desencadenaron dentro de su llamada a `eval_user_input`, y así sucesivamente. Para averiguar qué operaciones se están tomando realmente el tiempo, necesitamos buscar más profundamente en la cadena de llamadas.

La primera línea "importante" en esta salida es esta:

```
52 ./REPL[1]:2; myfunc()
```

`REPL` se refiere al hecho de que definimos` myfunc` en REPL, en lugar de ponerlo en un archivo; si hubiéramos usado un archivo, esto mostraría el nombre del archivo. El `[1]` muestra que la función `myfunc` fue la primera expresión evaluada en esta sesión REPL. La línea 2 de `myfunc()` contiene la llamada a `rand`, y hubo 52 (de 80) trazas inversas que ocurrieron en esta línea. Debajo de eso, puede ver una llamada a `dsfmt_fill_array_close_open!` Dentro de `dSFMT.jl`.

Un poco más abajo, ves:

```
28 ./REPL[1]:3; myfunc()
```


La línea 3 de `myfunc` contiene la llamada a` maximum`, y hubo 28 (de 80) trazas inversas tomadas aquí. Debajo de eso, puede ver los lugares específicos en `base/reduce.jl` que llevan a cabo las operaciones que consumen mucho tiempo en la función` maximum` para este tipo de datos de entrada.

En general, podemos concluir tentativamente que generar los números aleatorios es aproximadamente el doble de costoso que encontrar el elemento máximo. Podríamos aumentar nuestra confianza en este resultado recopilando más muestras:

```julia-repl
julia> @profile (for i = 1:100; myfunc(); end)

julia> Profile.print()
[....]
 3821 ./REPL[1]:2; myfunc()
  3511 ./random.jl:431; rand!(::MersenneTwister, ::Array{Float64,3}, ::Int64, ::Type...
   3511 ./dSFMT.jl:84; dsfmt_fill_array_close_open!(::Base.dSFMT.DSFMT_state, ::Ptr...
  310  ./random.jl:278; rand
   [....]
 2893 ./REPL[1]:3; myfunc()
  2893 ./reduce.jl:270; _mapreduce(::Base.#identity, ::Base.#scalarmax, ::IndexLinea...
   [....]
```

En general, si tiene `N` muestras recopiladas en una línea, puede esperar una incertidumbre en el orden de `sqrt(N)` (excluyendo otras fuentes de ruido, como cuán ocupada está la computadora con otras tareas). La principal excepción a esta regla es la recolección de basura, que se ejecuta con poca frecuencia pero tiende a ser bastante costosa. (Dado que el recolector de basura de Julia está escrito en C, tales eventos pueden ser detectados usando el modo de salida `C = true` descrito a continuación, o usando [ProfileView.jl](https://github.com/timholy/ProfileView.jl).)

Esto ilustra el volcado de "árbol" predeterminado; una alternativa es el volcado "plano", que acumula conteos independientemente de su anidación:

```julia-repl
julia> Profile.print(format=:flat)
 Count File          Line Function
  6714 ./<missing>     -1 anonymous
  6714 ./REPL.jl       66 eval_user_input(::Any, ::Base.REPL.REPLBackend)
  6714 ./REPL.jl       97 macro expansion
  3821 ./REPL[1]        2 myfunc()
  2893 ./REPL[1]        3 myfunc()
  6714 ./REPL[7]        1 macro expansion
  6714 ./boot.jl      235 eval(::Module, ::Any)
  3511 ./dSFMT.jl      84 dsfmt_fill_array_close_open!(::Base.dSFMT.DSFMT_s...
  6714 ./event.jl      73 (::Base.REPL.##1#2{Base.REPL.REPLBackend})()
  6714 ./profile.jl    23 macro expansion
  3511 ./random.jl    431 rand!(::MersenneTwister, ::Array{Float64,3}, ::In...
   310 ./random.jl    277 rand
   310 ./random.jl    278 rand
   310 ./random.jl    366 rand
   310 ./random.jl    369 rand
  2893 ./reduce.jl    270 _mapreduce(::Base.#identity, ::Base.#scalarmax, :...
     5 ./reduce.jl    420 mapreduce_impl(::Base.#identity, ::Base.#scalarma...
   253 ./reduce.jl    426 mapreduce_impl(::Base.#identity, ::Base.#scalarma...
  2592 ./reduce.jl    428 mapreduce_impl(::Base.#identity, ::Base.#scalarma...
    43 ./reduce.jl    429 mapreduce_impl(::Base.#identity, ::Base.#scalarma...
```

Si nuestro código tiene recursión, un punto potencialmente confso es que una línea en una función "hija" puede acumular ms cuentas que hay de trazas inversas en total.Considere las siguientes definiciones de función:

```julia
dumbsum(n::Integer) = n == 1 ? 1 : 1 + dumbsum(n-1)
dumbsum3() = dumbsum(3)
```

Si tuviéramos que analizar  `dumbsum3`, y una traza inversa fuera tomada mientras estaba ejecutándose `dumbsum(1)`, la traza inversa tendría este aspecto:

```julia
dumbsum3
    dumbsum(3)
        dumbsum(2)
            dumbsum(1)
```

En consecuencia, esta función hija realiza tres cuentas incluso aunque la padre sólo realiza una. La representación en "árbol" hace esto mucho ms claro, y por esta razón (entre otras) es probablemente la forma más útil de ver los resultados.

## Acumulación y Limpieza

Los resultados de [`@profile`](@ref) se acumulan en un buffer; si ejecuta varios fragmentos de código en [`@profile`](@ref), entonces [`Profile.print()`](@ref) le mostrará los resultados combinados. Esto puede ser muy útil, pero a veces desea comenzar de nuevo; puede hacerlo con [`Profile.clear()`](@ref).

## Opciones para controlar la visión de los resultados del análisis

[`Profile.print()`](@ref) tienes más opciones de las que se han descrito hasta ahora. Veamos la declaración completa:

```julia
function print(io::IO = STDOUT, data = fetch(); kwargs...)
```

Primero discutamos los dos argumentos posicionales, y luego los argumentos *keyword*:

  * `io` -- Le permite guardar los resultados en un búfer, por ejemplo, un archivo, pero el predeterminado es 
    imprimir en `STDOUT` (la consola)..
  * `data` -- Contiene los datos que desea analizar; de forma predeterminada, se obtiene de [`Profile.fetch()`](@ref), 
    que extrae trazas inversas de un búfer preasignado. Por ejemplo, si desea perfilar el generador de perfiles, 
    podría decir:
  
    ```julia
    data = copy(Profile.fetch())
    Profile.clear()
    @profile Profile.print(STDOUT, data) # Prints the previous results
    Profile.print()                      # Prints results from Profile.print()
    ```

The argumentos *keyword* pueden ser cualquier combinación de:

  * `format` -- Introducido anteriormente, determina si se imprimen trazas inversas con (por defecto, `: árbol`) 
    o sin (`: plano`) indentación que indica la estructura en árbol.
  * `C` -- Si` true`, se muestran trazas inversas de C y Fortran (normalmente están excluidas). Intente ejecutar 
    el ejemplo introductorio con `Profile.print(C = true)`. Esto puede ser extremadamente útil para decidir si 
    es el código Julia o el código C lo que está causando un cuello de botella; establecer `C = true` también 
    mejora la interpretabilidad del anidamiento, a coste de unos listados de perfil más largos. 
  * `combine` -- Algunas líneas de código contienen múltiples operaciones; por ejemplo, `s + = A [i]` contiene 
    una referencia de matriz (`A[i]`) y una operación de suma. Estos corresponden a diferentes líneas en el 
    código máquina generado, y por lo tanto puede haber dos o más direcciones diferentes capturadas durante 
    las trazas inversas en esta línea. `combine = true` los agrupa, y es probablemente lo que generalmente 
    desea, pero puede generar un resultado por separado para cada puntero de instrucción con` combine = false`.
  * `maxdepth` -- Limita los marcos a una profundidad mayor que` maxdepth` en el formato `:tree`.
  * `sortedby` -- Controla el orden en formato `:flat`. `:filefuncline` (predeterminado) ordena por la línea 
    de origen, mientras que `:count` se ordena según el número de muestras recolectadas.
  * `noisefloor` -- Límita los marcos que están debajo del umbral de ruido heurístico de la muestra (solo se aplica
    al formato`: tree`). Un valor sugerido para intentar esto es 2.0 (el valor predeterminado es 0). Este parámetro 
    oculta muestras para las cuales `n <= noisefloor * √N`, donde` n` es el número de muestras en esta línea, y 
    `N` es el número de muestras para el método invocado.
  * `mincount` -- Limita marcos con menos de `mincount` ocurrencias.

Los nombres de archivo/función a veces se truncan (con `...`), y la sangría se trunca con un `+n` al principio, donde `n` es el número de espacios adicionales que se habrían insertado, si hubiera habido espacio . Si desea un perfil completo de código profundamente anidado, a menudo una buena idea es guardar en un archivo usando un ancho "tamaño de pantalla" en un [`IOContext`] (@ref):

```julia
open("/tmp/prof.txt", "w") do s
    Profile.print(IOContext(s, :displaysize => (24, 500)))
end
```

## Configuración

[`@ profile`](@ref) solo acumula *backtraces*, y el análisis ocurre cuando usted llama a [`Profile.print()`](@ref). Para un cálculo de larga ejecución, es muy posible que se llene el búfer preasignado para almacenar *backtraces*. Si eso sucede, las trazas inversas se detienen pero su cálculo continúa. Como consecuencia, es posible que omitan algunos datos importantes de generación de perfiles (recibirá una advertencia cuando eso suceda).

Puede obtener y configurar los parámetros relevantes de esta manera:

```julia
Profile.init() # returns the current settings
Profile.init(n = 10^7, delay = 0.01)
```

`n` es la cantidad total de punteros de instrucción que puede almacenar, con un valor predeterminado de `10 ^ 6`. Si su traza inversa típica es de 20 punteros de instrucción, puede recopilar 50000 trazas inversas, lo que sugiere una incertidumbre estadística de menos del 1%. Esto puede ser lo suficientemente bueno para la mayoría de las aplicaciones.

En consecuencia, es más probable que necesite modificar el `retraso`, expresado en segundos, que establece la cantidad de tiempo que Julia obtiene entre las instantáneas para realizar los cálculos solicitados. Un trabajo de larga duración puede no necesitar trazas frecuentes. La configuración predeterminada es `delay = 0.001`. Por supuesto, puede disminuir la demora y aumentarla; sin embargo, la sobrecarga de los perfiles crece una vez que la demora se vuelve similar a la cantidad de tiempo necesaria para tomar una traza inversa (~ 30 microsegundos en la computadora portátil del autor).

# Análisis de la asignación de memoria

Una de las técnicas más comunes para mejorar el rendimiento es reducir la asignación de memoria. La cantidad total de  asignación se puede medir con [`@time`](@ref) y [`@assigned`](@ref), y las líneas específicas que desencadenan la asignación a  pueden a menudo inferirse a partir del perfil a través del costo de la recolección de basura en la que incurren estas líneas. Sin embargo, a veces es más eficiente medir directamente la cantidad de memoria asignada por cada línea de código.

Para medir la asignación línea por línea, inicie Julia con la opción de línea de comando `--track-allocation = <setting>`, para la cual puede elegir `none` (el predeterminado, no medir la asignación), `user` (mida la asignación de memoria en todas partes excepto el código central de Julia), o `todo` (mida la asignación de memoria en cada línea del código Julia). La asignación se mide para cada línea de código compilado. Cuando sale de Julia, los resultados acumulativos se escriben en archivos de texto con `.mem` adjunto después del nombre del archivo, que residen en el mismo directorio que el archivo de origen. Cada línea enumera la cantidad total de bytes asignados. El [paquete `Coverage`](https://github.com/JuliaCI/Coverage.jl) contiene algunas herramientas de análisis elementales, por ejemplo, para ordenar las líneas en orden de cantidad de bytes asignados.

Al interpretar los resultados, hay algunos detalles importantes. Bajo la configuración `user`, la primera línea de cualquier función directamente llamada desde REPL exhibirá asignación debido a eventos que ocurren en el código REPL. Más significativamente, la compilación de JIT también se suma a los recuentos de asignación, porque gran parte del compilador de Julia está escrito en Julia (y la compilación generalmente requiere asignación de memoria). El procedimiento recomendado es forzar la compilación ejecutando todos los comandos que desea analizar, luego llame a [`Profile.clear_malloc_data()`](@ref) para restablecer todos los contadores de asignación. Finalmente, ejecute los comandos deseados y salga de Julia para activar la generación de los archivos `.mem`.
