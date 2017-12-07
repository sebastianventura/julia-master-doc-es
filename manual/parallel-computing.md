# Computación Paralela

La mayoría de las computadoras modernas poseen más de una CPU, y varias computadoras pueden combinarse en un cluster. Aprovechar la potencia de estas múltiples CPU permite que muchos cálculos se completen más rápidamente. Hay dos factores principales que influyen en el rendimiento: la velocidad de las propias CPUs y la velocidad de su acceso a la memoria. En un clúster, es bastante obvio que una CPU dada tendrá acceso más rápido a la RAM dentro de la misma computadora (nodo). Quizás más sorprendentemente, problemas similares son relevantes en un portátil multicore típico, debido a las diferencias en la velocidad de la memoria principal y la [caché](https://www.akkadia.org/drepper/cpumemory.pdf). En consecuencia, un buen entorno de multiprocesamiento debe permitir el control sobre la "propiedad" de un trozo de memoria por una CPU particular. Julia proporciona un entorno de multiprocesamiento basado en el paso de mensajes para permitir que los programas se ejecuten en múltiples procesos en distintos dominios de memoria a la vez.

La implementación de Julia del paso del mensajes es diferente de otros entornos tales como MPI [^1]. Comunicación en Julia es generalmente "unilateral", lo que significa que el programador necesita explícitamente administrar sólo un proceso en una operación de dos procesos. Además, estas operaciones típicamente no se parecen a "envío de mensajes" y "recepción de mensajes", sino más bien se asemejan a operaciones de nivel superior como llamadas a funciones de usuario.

La programación paralela en Julia se basa en dos primitivas: *referencias remotas* y *llamadas remotas*. Una referencia remota es un objeto que puede utilizarse desde cualquier proceso para referirse a un objeto almacenado en un proceso determinado. Una llamada remota es una petición de un proceso para llamar a una determinada función en ciertos argumentos en otro proceso (posiblemente el mismo).

Las referencias remotas vienen en dos variedades: [`Future`](@ref) y [`RemoteChannel`](@ref).

Una llamada remota devuelve un [`Future`](@ref) a su resultado. Las llamadas remotas retornan inmediatamente; el proceso que hizo que la llamada procede a su siguiente operación mientras la llamada remota sucede en otro lugar. Podemos esperar a que una llamada remota termine llamando a [`wait()`](@ref) en el [`Future`](@ref) devuelto, y puede obtener el valor completo del resultado usando [`fetch()`](@ref).

Por otro lado, los objetos [`RemoteChannel`](@ref) son reescribibles. Por ejemplo, varios procesos pueden coordinar su procesamiento haciendo referencia al mismo canal (`Channel`) remoto.

Cada proceso tiene un identificador asociado. El proceso que proporciona el *prompt* interactivo de Julia siempre tiene un `id` igual a 1. Los procesos utilizados por defecto para operaciones paralelas se conocen como "workers". Cuando solo hay un proceso, el proceso 1 se considera trabajador. De lo contrario, se considera que los trabajadores son todos procesos distintos del proceso 1.

Vamos a probar esto. Comenzar con `julia -p n` proporciona `n` procesos *workers* en la máquina local. Generalmente tiene sentido igualar `n` al número de núcleos de la CPU en la máquina.

```julia
$ ./julia -p 2

julia> r = remotecall(rand, 2, 2, 2)
Future(2, 1, 4, Nullable{Any}())

julia> s = @spawnat 2 1 .+ fetch(r)
Future(2, 1, 5, Nullable{Any}())

julia> fetch(s)
2×2 Array{Float64,2}:
 1.18526  1.50912
 1.16296  1.60607
```

El primer argumento para [`remotecall()`](@ref) es la función para llamar. La mayoría de la programación paralela en Julia no hace referencia a procesos específicos ni a la cantidad de procesos disponibles, pero [`remotecall()`](@ref) se considera una interfaz de bajo nivel que proporciona un control más preciso. El segundo argumento para [`remotecall()`](@ref) es el `id` del proceso que hará el trabajo, y los argumentos restantes se pasarán a la función a la que se llama.

Como puede ver, en la primera línea le pedimos al proceso 2 que construyera una matriz aleatoria de 2 por 2, y en la segunda línea le pedimos que agregara 1 a ella. El resultado de ambos cálculos está disponible en los dos futuros, `r` y` s`. La macro [`@ spawnat`] (@ ref) evalúa la expresión en el segundo argumento sobre el proceso especificado por el primer argumento.

Ocasionalmente, es posible que desee un valor calculado de forma remota de inmediato. Esto suele ocurrir cuando lee desde un objeto remoto para obtener los datos que necesita la próxima operación local. La función [`remotecall_fetch ()`](@ref) existe para este propósito. Es equivalente a `fetch (remotecall (...))`
pero es más eficiente

```julia-repl
julia> remotecall_fetch(getindex, 2, r, 1, 1)
0.18526337335308085
```

Recuerde que [`getindex(r,1,1)`](@ref) es [equivalente](@ref man-array-indexing) a `r[1,1]`, por lo que esta llamada capta el primer elemento del futuro `r`.

La sintaxis de [`remotecall()`](@ref) no es especialmente conveniente. La macro [`@spawn`](@ref) hace las cosas más fáciles. Ella opera sobre una expresión en lugar de sobre una función, y elige dónde hacer la operación por ti:

```julia-repl
julia> r = @spawn rand(2,2)
Future(2, 1, 4, Nullable{Any}())

julia> s = @spawn 1 .+ fetch(r)
Future(3, 1, 5, Nullable{Any}())

julia> fetch(s)
2×2 Array{Float64,2}:
 1.38854  1.9098
 1.20939  1.57158
```

Note que usamos `1 .+ fetch(r)` en lugar de `1 .+ r`. Esto es debido a que no sabemos dónde se ejecutará el código, por lo que en general puede que se requiera un[`fetch()`](@ref) para mover `r` al proceso que esta realizando la suma. En este caso, [`@spawn`](@ref) es bastante inteligente como para realizar el computo sobre el proceso que posee `r`, por lo que la llamada [`fetch()`](@ref) será una no-op (no se realiza trabajo).

(Hay que notar que [`@spawn`](@ref) no es una función predefinida sino que está definida en Julia como una [macro](@ref man-macros). Es posible definir nuestras propias construcciones de ese tipo).

Una importante cosa a recordar es que, una vez traída, un [`Future`](@ref) cacheará su valor localmente. Las llamadas adicionales a [`fetch()`](@ref) calls no implican un salto de red. Una vez que todos los [`Future`](@ref)s que referencian ha sido recuperados, el valor remoto almacenado es borrado.

## Disponibiliad de Código y Carga de Paquetes

Nuestro código debe estar disponible sobre cualquir proceso que lo ejecuta. Por ejemplo, escriba esto en el *prompt* de Julia:

```julia-repl
julia> function rand2(dims...)
           return 2*rand(dims...)
       end

julia> rand2(2,2)
2×2 Array{Float64,2}:
 0.153756  0.368514
 1.15119   0.918912

julia> fetch(@spawn rand2(2,2))
ERROR: RemoteException(2, CapturedException(UndefVarError(Symbol("#rand2"))
[...]
```

El proceso 1 sabe dónde se encuentra la función `rand2`, pero el proceso 2 no.

Ms comunmente uno estará cargando código desde ficheros o paquetes, y tendrá una considerable flexibilidad para controlar qué procesos cargan código. Considere un fichero `DummyModule.jl`, que contenga el siguiente código:

```julia
module DummyModule

export MyType, f

mutable struct MyType
    a::Int
end

f(x) = x^2+1

println("loaded")

end
```

Si se arranca Julia con `julia -p 2`, se puede usar esto para verificar lo siguiente:

  * `include("DummyModule.jl")` carga elfichero sólo sobre un proceso (el que ejecuta la instrucción).
  * `using DummyModule` causa que el módulo sea cargado sobre todos los procesos; sin embargo, el módulo es llevado al ámbito sólo por el que ejecuta la instrucción.
  * En cuanto `DummyModule` sea cargado sobre el proceso 2, mandatos como

    ```julia
    rr = RemoteChannel(2)
    put!(rr, MyType(7))
    ```

    permiten almacenar un objeto de tipo `MyType` sobre el proceso 2 incluso aunque `DummyModule` no esté en 
    el ámbito del proceso 2.

Uno puede forzar que un mandato se ejecute sobre todos los procesos usando la macro [`@everywhere`](@ref). Por ejemplo, `@everywhere` puede también ser usado para definir directamente una función sobre todos los procesos:

```julia-repl
julia> @everywhere id = myid()

julia> remotecall_fetch(()->id, 2)
2
```

Un fichero puede también ser precargado sobre múltiples procesos al inicio, y puede usarse un *driver* para llevar a cabo ese cómputo:

```
julia -p <n> -L file1.jl -L file2.jl driver.jl
```

El proceso Julia que corre el programa *driver* del ejemplo anterior tiene un `id` igual a 1, justo como un proceso que proporciona un prompt interactivo.

La instalación base de Julia tiene soporte intríseco para dos tipos de clusters:

  * Un clúster local especificado con la opción `-p`, como se mostró anteriormente.
  * Un clúster que abarca máquinas utilizando la opción `--machinefile`. Esto utiliza un inicio de 
    sesión `ssh` sin contraseña para iniciar los procesos de trabajo de Julia (desde la misma ruta 
    que el servidor actual) en las máquinas especificadas.
  
Las funciones [`addprocs()`](@ref), [`rmprocs()`](@ref), [`workers()`](@ref), y otras están disponibles como una forma programática de añadir, borrar y consultar los procesos en un cluster.

Note que los *workers* no ejecutan un script `.juliarc.jl` de inicio, ni sincronizan su estado global (tal como variables globales, nuevas definiciones de métodos y módulos cargados) con cualquiera de los procesos que están ejecutando.

Pueden soportarse otros tipos de clústers escribiendo nuestro porpio `ClusterManager`, como se describe después en la sección [ClusterManagers](@ref) section.

## Movimiento de Datos

Enviar mensaje y mover datos constituye la mayor parte de la sobrecarga de un programa paralelo. Reducir el número de mensajes y la cantidad de datos enviados es crítico para conseguir rendimiento y escalabilidad. Para este fin, es importante comprender el movimiento de datos realizado por varias construcciones de programación paralela de Julia.

[`fetch()`](@ref) puede considerarse una operación explícita de movimiento de datos, ya que directamente pide que un objeto sea movido a la máquina local. [`@spawn`](@ref) (y unas pocas construcciones relacionadas) también mueve datos, pero esto no es tan obvio, por lo tanto, puede denominarse una operación implícita de movimiento de datos. Considere estos dos enfoques para construir y cuadrar una matriz aleatoria:

Method 1:

```julia-repl
julia> A = rand(1000,1000);

julia> Bref = @spawn A^2;

[...]

julia> fetch(Bref);
```

Method 2:

```julia-repl
julia> Bref = @spawn rand(1000,1000)^2;

[...]

julia> fetch(Bref);
```

La diferencia parece trivial, pero de hecho es bastante significativa debido al comportamiento de [`@spawn`](@ref). En el primer método, se construye localmente una matriz aleatoria, y después se manda a otro proceso que la eleva al cuadrado. En el segundo método, una matriz aleatoria es construida y elevada al cuadrado (ambas operaciones) sobre otro proceso. Por tanto, el segundo método envía muchos menos datos que el primero. 

En este ejemplo de juquete, los dos métodos son fáciles de distinguir y elegir. Sin embargo, en un programa real dieseñar movimiento de datos puede requerir ms reflexin y, probablemente, alguna medida. Por ejemplo, si el primero proceso necesita la matriz `A`, entonces el primer método sera mejor. O, si calcular `A` es costoso y sólo el proceso actual tiene que hacerlo, entonces mover la matriz al otro proceso puede ser inevitable. o, si el proceso actual tiene muy poco que hacer entre las instrucciones [`@spawn`](@ref) y `fetch(Bref)`, podría ser mejor eliminar el paralelismo por completo. O imagine que `rand(1000,1000)` es reemplazado con un a operación más costosa. Entonces podría tener sentido añadir ora instrucción  [`@spawn`](@ref) sólo para este paso.

# Variables Globales

Las expresiones ejecutadas remotamente vía `@spawn`, o los cierres especificados para ejecucin remota usando `remotecall` pueden referirse a variables globales. Las vinculaciones globales en el módulo `Main` son tratadas de un modo un poco diferente comparados a las vinculaciones globales en otros módulos. Considere el siguiente trozo de código:

```julia-repl
A = rand(10,10)
remotecall_fetch(()->norm(A), 2)
```

En este caso [`norm`](@ref) es una función que toma un array 2D como parámetro, y DEVE estar definido en el proceso remoto. Uno podría usar cualquier función distinta de `norm` siempre que ella esté definida en el proceso remoto y acepte el parámetro apropiado.

Tenga en cuenta que `A` es una variable global definida en el espacio de trabajo local. El *worker* 2 no tiene una variable llamada `A` dentro de ` Main`. Por tantom el acto de enviar el cierre `() -> norma (A)` al *worker* 2 da como resultado que `Main.A` se defina en 2. `Main.A` sigue existiendo en el worker 2 incluso después de que la llamada `remotecall_fetch` retorne. Las llamadas remotas con referencias globales integradas (solo bajo el módulo `Main`) administran los datos globales de la siguiente manera:


- Se crean nuevos enlaces globales en los trabajadores de destino si se hace referencia a ellos como parte de una llamada remota.

- Las constantes globales se declaran también como constantes en los nodos remotos.

- Globales se reenvían a un *worker* de destino solo en el contexto de una llamada remota, y solo si su valor ha cambiado. Además, el clúster no sincroniza las asignaciones globales entre nodos. Por ejemplo:

  ```julia
  A = rand(10,10)
  remotecall_fetch(()->norm(A), 2) # worker 2
  A = rand(10,10)
  remotecall_fetch(()->norm(A), 3) # worker 3
  A = nothing
  ```

  Ejecutar este trozo de código da como resultado que `Main.A` del *worker* 2 tenga un valor diferente del que tiene en 
  `Main.A` del *worker* 3, mientras que el valor de `Main.A` en el nodo 1 se fija a `nothing`.

Como se habrá dado cuenta, aunque la memoria asociada con los globales se puede recopilar cuando se reasignan en el maestro, dicha acción no se toma en los *workers* ya que los enlaces siguen siendo válidos. [`clear!`](@ref) se puede usar para reasignar manualmente globales específicos en nodos remotos a `nothing` una vez que ya no sean necesarios. Esto liberará cualquier memoria asociada con ellos como parte de un ciclo de recolección de basura regular.

Por lo tanto, los programas deben ser cuidadosos al hacer referencia a los globales en las llamadas remotas. De hecho, es preferible evitarlos por completo si es posible. Si hay que hacer referencia a globales, considere usar bloques `let` para localizar variables globales.

For example:

```julia-repl
julia> A = rand(10,10);

julia> remotecall_fetch(()->A, 2);

julia> B = rand(10,10);

julia> let B = B
           remotecall_fetch(()->B, 2)
       end;

julia> @spawnat 2 whos();

julia>  From worker 2:                               A    800 bytes  10×10 Array{Float64,2}
        From worker 2:                            Base               Module
        From worker 2:                            Core               Module
        From worker 2:                            Main               Module
```

Como puede verse, la variable global `A` es definidia en el *worker* 2, pero `B` es capturada como una variable local y por tanto no existe una asignacin para `B` en *worker* 2.


## Map y Bucles Paralelos

Afortunadamente, muchos cálculos paralelos no requieren movimiento de datos. Un ejemplo común es las simulaciones Monte Carlo, donde muchos procesos pueden manejar simultáneamente pruebas de simulación independientes. Podemos usar [`@spawn`](@ref) para lanzar monedas sobre dos procesos. Primero, se escribiría la siguiente función en `count_heads.jl`:

```julia
function count_heads(n)
    c::Int = 0
    for i = 1:n
        c += rand(Bool)
    end
    c
end
```

La función `count_heads` simplemente añade juntos `n` bits aleatorios. He aqui cónmo pueden realizarse algunas pruebas sobre dos máquinas, y añadir juntos los resultados:

```julia-repl
julia> @everywhere include_string(Main, $(read("count_heads.jl", String)), "count_heads.jl")

julia> a = @spawn count_heads(100000000)
Future(2, 1, 6, Nullable{Any}())

julia> b = @spawn count_heads(100000000)
Future(3, 1, 7, Nullable{Any}())

julia> fetch(a)+fetch(b)
100001564
```

Este ejemplo demuestra un patrón de programación paralela potente y frecuentemente usado. Muchas iteraciones se ejecutan independientemente sobre varios porocesos, y entonces sus resultados se combinan usando alguna función. El proceso de combinación se denomina *reducción* ya que suele ser la reduccin de rango de un tensor: un vector de números es reducido a un solo número o una matriz es reducida a una sola fila o columna, etc. En código esto suele tener el aspecto del patrón `x = f(x,v[i])`, donde `x` es el acumulador, `f` es la funcin de reducción, y los `v[i]` son los elementos que se reducirán. Es deseable que `f` sea asociativa, para que no importe el orden en el que se realizan las operaciones.

Notese que nuestro uso de este patrón con `count_heads` puede ser generalizado. Se utilizaron dos instrucciones [`@spawn`](@ref) explícitas, que limitan el paralelismo a dos procesos. Para ejecutar sobre cualquier número de procesos, se puede usar el *bucle for paralelo* que puede escribirse en Julia usando la macro [`@parallel`](@ref) como en este ejemplo:

```julia
nheads = @parallel (+) for i = 1:200000000
    Int(rand(Bool))
end
```

Esta construcción implementa el patrón de asignar iteraciones a múltiples procesos, y combinarlos con una reducción especificada (en este caso `(+)`). El resultado de cada iteración es tomado como el valor de la última expresión dentro del bucle. La expresión total del bucle paralelo en sí misma se evalúa a la respuesta final.

Debe notar que, aunque los bucles for paralelos tienen un aspecto muy parecido al de los bucles for seriales, su comportamiento es dramaticamente diferente. En particular, las iteraciones no tiene lugar en un orden especificado, y l escritura a variables o arrays no será globalmente visible ya que las iteraciones se ejecutan sobre procesos distintos. Cualquier variable usada dentro del bucle paralelo será copiada y retransmitida a cada proceso.

Por ejemplo, el siguiente código no trabajará como se esperaba:

```julia
a = zeros(100000)
@parallel for i = 1:100000
    a[i] = i
end
```

Este código no inicializará todo `a`, ya que cada proceso tendrá una copia separada de él. Los bucles for paralelos como éste deben ser evitados. Afortunadamente, podemos usar los [arrays compartidos](@ref man-shared-arrays) para sortear esta limitación:

```julia
a = SharedArray{Float64}(10)
@parallel for i = 1:10
    a[i] = i
end
```

Usar variables "forasteras" en los bucles paralelos es perfectamente razonable si las variables son de sólo lectura:

```julia
a = randn(1000)
@parallel (+) for i = 1:100000
    f(a[rand(1:end)])
end
```

En este ejemplo, cada iteración aplica `f` a una muestra elegida aleatoriamente de un vector `a` compartido por todos los procesos.

Como podía ver, el operador de reducción puede ser omitido si no se necesita En este caso, el bucle se ejecuta de forma asíncrona, es decir, engendra tareas independientes sobre todos los *workers* disponibles y devuelve un array de objetos [`Future`](@ref) inmediatamente sin esperar a su terminación. El código invocador puede esperar a la terminación de los [`Future`](@ref) en un punto posterior mediante una llamada a [`fetch()`](@ref) sobre ellos, o esperar la terminancin al final del bucle prefijándolo con [`@sync`](@ref), como `@sync @parallel for`.

En algunos casos no se necesita un operador de reducción, y simplemente deseamos aplicar una funcin a todos los enteros en algún rango (o, de forma más general, a todos los elementos de una colección). Esta es otra operación útil llamada *parallel map*, implementada en con la función [`pmap()`](@ref). Por ejemplo, podríamos computar los valores singularees de varias matrices aleatorias en paralelo de la siguiente forma:

```julia-repl
julia> M = Matrix{Float64}[rand(1000,1000) for i = 1:10];

julia> pmap(svd, M);
```

La función [`pmap()`](@ref) de Julia está diseñada para el caso de que cada llamada a función realice una gran cantidad de trabajo. En contraste `@parallel for` puede manejar situaciones donde cada iteración es pequeña, quizás incluso sumar dos números. Tanto las funciones [`pmap()`](@ref) como `@parallel for` usan exclusivamente procesos *worker* para la computación paralela. En el caso de `@parallel for`, la reducción final se realiza sobre el proceso principal.

## Sincronización con Referencias Remotas

## Planificación

La plataforma de programación paralela de Julia usa [Tareas (también conocidas como Coroutinas)](@ref man-tasks) para alternar entre múltiples cálculos. Cada vez que el código realiza una operación de comunicación como [`fetch()`](@ref) o [`wait()`](@ref), la tarea actual se suspende y un planificador elige otra tarea para ejecutar. Una tarea se reinicia cuando finaliza el evento que está esperando.

Para muchos problemas, no es necesario pensar en las tareas directamente. Sin embargo, pueden usarse para esperar múltiples eventos al mismo tiempo, lo que proporciona una *planificación dinámica*. En la planificación dinámica, un programa decide qué calcular o dónde calcularlo en función de cuándo finalizan otros trabajos. Esto es necesario para cargas de trabajo impredecibles o desequilibradas, donde queremos asignar más trabajo a los procesos solo cuando finalizan sus tareas actuales.

Como ejemplo, considere calcular los valores singulares de matrices de diferentes tamaños:

```julia-repl
julia> M = Matrix{Float64}[rand(800,800), rand(600,600), rand(800,800), rand(600,600)];

julia> pmap(svd, M);
```

Si un proceso maneja la dos matrices de 800 × 800 y otro maneja las dos matrices de 600 × 600, no obtendremos la mayor escalabilidad posible. La solución es hacer una tarea local para "alimentar" el trabajo a cada proceso cuando completa su tarea actual. Por ejemplo, considere una implementación simple [`pmap()`](@ref):

```julia
function pmap(f, lst)
    np = nprocs()  # determine the number of processes available
    n = length(lst)
    results = Vector{Any}(n)
    i = 1
    # function to produce the next work item from the queue.
    # in this case it's just an index.
    nextidx() = (idx=i; i+=1; idx)
    @sync begin
        for p=1:np
            if p != myid() || np == 1
                @async begin
                    while true
                        idx = nextidx()
                        if idx > n
                            break
                        end
                        results[idx] = remotecall_fetch(f, p, lst[idx])
                    end
                end
            end
        end
    end
    results
end
```

[`@async`](@ref) es similar a [`@spawn`](@ref), pero solo ejecuta tareas en el proceso local. Lo usamos para crear una tarea   "alimentador" para cada proceso. Cada tarea selecciona el siguiente índice que debe calcularse, luego espera a que termine su proceso, y luego se repite hasta que nos quedemos sin índices. Tenga en cuenta que las tareas "alimentadoras no" comienzan a ejecutarse hasta que la tarea principal llega al final del bloque [`@sync`](@ref), momento en el cual se somete al control y espera a que se completen todas las tareas locales antes de regresar de la función. Las tareas del "alimentador" pueden compartir el estado a través de `nextidx()` porque todas se ejecutan en el mismo proceso. No se requiere bloqueo, ya que los hilos están programados de forma cooperativa y no apropiativa. Esto significa que los cambios de contexto solo ocurren en puntos bien definidos: en este caso, cuando se llama a [`remotecall_fetch()`](@ref).

## Canales

La sección sobre tareas ([`Task`](@ref)) en [Control de flujo](@ref) discutió la ejecución de múltiples funciones de forma cooperativa. Los canales ([`Channel`](@ref)) pueden ser bastante útiles para pasar datos entre tareas en ejecución, particularmente aquellas que involucran operaciones de E/S.

Ejemplos de operaciones que implican E/S incluyen la lectura/escritura en archivos, acceso a servicios web, ejecución de programas externos, etc. En todos estos casos, el tiempo de ejecución general puede mejorarse si se pueden ejecutar otras tareas mientras se lee un archivo, o mientras se espera a que se complete un servicio o programa externo.

Un canal se puede visualizar como un conducto, es decir, tiene un extremo de escritura y un extremo de lectura.

   * Varios escritores en diferentes tareas pueden escribir en el mismo canal concurrentemente a través de llamadas [`put!()`](@ref).
   * Varios lectores en diferentes tareas pueden leer datos simultáneamente a través de llamadas [`take!()`](@ref).
   * Como ejemplo:

    ```julia
    # Given Channels c1 and c2,
    c1 = Channel(32)
    c2 = Channel(32)

    # and a function `foo()` which reads items from from c1, processes the item read
    # and writes a result to c2,
    function foo()
        while true
            data = take!(c1)
            [...]               # process data
            put!(c2, result)    # write out result
        end
    end

    # we can schedule `n` instances of `foo()` to be active concurrently.
    for _ in 1:n
        @schedule foo()
    end
    ```
 
   * Los canales se crean a través del constructor `Channel{T}(sz)`. El canal solo tendrá objetos de tipo `T`. Si no se especifica el tipo, el canal puede contener objetos de cualquier tipo. `sz` se refiere a la cantidad máxima de elementos que pueden mantenerse en el canal en cualquier momento. Por ejemplo, `Channel(32)` crea un canal que puede contener un máximo de 32 objetos de cualquier tipo. Un `Channel{MyType}(64)` puede contener hasta 64 objetos de `MyType` en cualquier momento.
   * Si un [`Channel`](@ref) está vacío, los lectores (en una llamada [`take!()`](@ref)) se bloquearán hasta que los datos estén disponibles.
   * Si un [`Channel`](@ref) está lleno, los escritores (en una llamada [`put!()`](@ref)) se bloquearán hasta que haya espacio disponible.
   * [`isready()`](@ref) comprueba la presencia de cualquier objeto en el canal, mientras que [`wait()`](@ref) espera a que un objeto esté disponible.
   * Un [`Channel`](@ref) está inicialmente en un estado abierto. Esto significa que puede leerse y escribirse libremente a través de llamadas [`take!()`](@ref) y [`put!()`](@ref). [`close()`](@ref) cierra un [`Channel`](@ref). En un [`Channel`](@ref) cerrado, la función [`put!()`](@ref) fallará. Por ejemplo:
  
```julia-repl
julia> c = Channel(2);

julia> put!(c, 1) # `put!` on an open channel succeeds
1

julia> close(c);

julia> put!(c, 2) # `put!` on a closed channel throws an exception.
ERROR: InvalidStateException("Channel is closed.",:closed)
[...]
```

* [`take!()`](@ref) y [`fetch()`](@ref) (que recupera pero no elimina el valor) en un canal cerrado devuelve con éxito cualquier valor existente hasta que se vacíe. Continuando con el ejemplo anterior:

```julia-repl
julia> fetch(c) # Any number of `fetch` calls succeed.
1

julia> fetch(c)
1

julia> take!(c) # The first `take!` removes the value.
1

julia> take!(c) # No more data available on a closed channel.
ERROR: InvalidStateException("Channel is closed.",:closed)
[...]
```

Un canal se puede usar como un objeto iterable en un bucle `for`, en cuyo caso el bucle se ejecuta mientras el canal tenga datos o esté abierto. La variable del bucle toma todos los valores agregados al canal. El bucle `for` finaliza una vez que el canal se cierra y se vacía.

Por ejemplo, lo siguiente haría que el bucle `for` esperara más datos:

```julia-repl
julia> c = Channel{Int}(10);

julia> foreach(i->put!(c, i), 1:3) # add a few entries

julia> data = [i for i in c]
```

mientras que esto retornará después de leer todos los datos:

```julia-repl
julia> c = Channel{Int}(10);

julia> foreach(i->put!(c, i), 1:3); # add a few entries

julia> close(c);                    # `for` loops can exit

julia> data = [i for i in c]
3-element Array{Int64,1}:
 1
 2
 3
```

Consideremos un ejemplo simple utilizando canales para la comunicación entre tareas. Comenzamos 4 tareas para procesar los datos de un solo canal `jobs`. Los trabajos, identificados por un identificador (`job_id`), se escriben en el canal. Cada tarea en esta simulación lee un `job_id`, espera una cantidad aleatoria de tiempo y escribe una tupla de `job_id` y el tiempo simulado en el canal de resultados. Finalmente todos los `resultados` se imprimen.


```julia-repl
julia> const jobs = Channel{Int}(32);

julia> const results = Channel{Tuple}(32);

julia> function do_work()
           for job_id in jobs
               exec_time = rand()
               sleep(exec_time)                # simulates elapsed time doing actual work
                                               # typically performed externally.
               put!(results, (job_id, exec_time))
           end
       end;

julia> function make_jobs(n)
           for i in 1:n
               put!(jobs, i)
           end
       end;

julia> n = 12;

julia> @schedule make_jobs(n); # feed the jobs channel with "n" jobs

julia> for i in 1:4 # start 4 tasks to process requests in parallel
           @schedule do_work()
       end

julia> @elapsed while n > 0 # print out results
           job_id, exec_time = take!(results)
           println("$job_id finished in $(round(exec_time,2)) seconds")
           n = n - 1
       end
4 finished in 0.22 seconds
3 finished in 0.45 seconds
1 finished in 0.5 seconds
7 finished in 0.14 seconds
2 finished in 0.78 seconds
5 finished in 0.9 seconds
9 finished in 0.36 seconds
6 finished in 0.87 seconds
8 finished in 0.79 seconds
10 finished in 0.64 seconds
12 finished in 0.5 seconds
11 finished in 0.97 seconds
0.029772311
```

La versión actual de Julia multiplexa todas las tareas en un solo hilo del sistema operativo. Por lo tanto, aunque las tareas que implican operaciones de E/S se benefician de la ejecución en paralelo, las tareas vinculadas a la computación se ejecutan efectivamente de forma secuencial en un solo hilo del sistema operativo. Las versiones futuras de Julia pueden soportar la planificación de tareas en múltiples subprocesos, en cuyo caso las tareas vínculadas al cálculo  también verán los beneficios de la ejecución en paralelo.

## Referencias remotas y AbstractChannels

Las referencias remotas siempre se refieren a una implementación de un `AbstractChannel`.

Se requiere una implementación concreta de un `AbstractChannel` (como `Channel`) para implementar [`put!()`](@ref), [`take!()`](@ref), [`fetch()`](@ref), [`isready()`](@ref) y [`wait()`](@ref). El objeto remoto al que se hace referencia con un [`Future`](@ref) se almacena en `Channel{Any}(1) `, es decir, un` Channel` de tamaño 1 capaz de contener objetos del tipo `Any`.

[`RemoteChannel`](@ref), que es reescribible, puede señalar cualquier tipo y tamaño de canales, o cualquier otra implementación de `AbstractChannel`.

El constructor `RemoteChannel(f::Function, pid)()` permite construir referencias a canales que contienen más de un valor de un tipo específico. `f()` es una función ejecutada en un `pid` y debe devolver un `AbstractChannel`.

Por ejemplo, `RemoteChannel(()->Channel{Int}(10), pid)`, devolverá una referencia a un canal de tipo `Int` y tamaño 10. El canal existe sobre el *worker* `pid`.

Los métodos [`put!()`](@ref), [`take!()`](@ref), [`fetch()`](@ref), [`isready()`](@ref) y [`wait()`](@ref) de un [`RemoteChannel`](@ref) son proxys en el backing store en el proceso remoto.

[`RemoteChannel`](@ref) se puede usar para referirse a los objetos `AbstractChannel` implementados por el usuario. Un ejemplo simple de esto se proporciona en `examples/dictchannel.jl que usa un diccionario como su almacén remoto.

## Channels and RemoteChannels

  * A [`Channel`](@ref) is local to a process. Worker 2 cannot directly refer to a `Channel` on worker 3 and
    vice-versa. A [`RemoteChannel`](@ref), however, can put and take values across workers.
  * A [`RemoteChannel`](@ref) can be thought of as a *handle* to a `Channel`.
  * The process id, `pid`, associated with a [`RemoteChannel`](@ref) identifies the process where
    the backing store, i.e., the backing `Channel` exists.
  * Any process with a reference to a [`RemoteChannel`](@ref) can put and take items from the channel.
    Data is automatically sent to (or retrieved from) the process a [`RemoteChannel`](@ref) is associated
    with.
  * Serializing  a `Channel` also serializes any data present in the channel. Deserializing it therefore
    effectively makes a copy of the original object.
  * On the other hand, serializing a [`RemoteChannel`](@ref) only involves the serialization of an
    identifier that identifies the location and instance of `Channel` referred to by the handle. A
    deserialized [`RemoteChannel`](@ref) object (on any worker), therefore also points to the same
    backing store as the original.

The channels example from above can be modified for interprocess communication,
as shown below.

We start 4 workers to process a single `jobs` remote channel. Jobs, identified by an id (`job_id`),
are written to the channel. Each remotely executing task in this simulation reads a `job_id`,
waits for a random amount of time and writes back a tuple of `job_id`, time taken and its own
`pid` to the results channel. Finally all the `results` are printed out on the master process.

```julia-repl
julia> addprocs(4); # add worker processes

julia> const jobs = RemoteChannel(()->Channel{Int}(32));

julia> const results = RemoteChannel(()->Channel{Tuple}(32));

julia> @everywhere function do_work(jobs, results) # define work function everywhere
           while true
               job_id = take!(jobs)
               exec_time = rand()
               sleep(exec_time) # simulates elapsed time doing actual work
               put!(results, (job_id, exec_time, myid()))
           end
       end

julia> function make_jobs(n)
           for i in 1:n
               put!(jobs, i)
           end
       end;

julia> n = 12;

julia> @schedule make_jobs(n); # feed the jobs channel with "n" jobs

julia> for p in workers() # start tasks on the workers to process requests in parallel
           remote_do(do_work, p, jobs, results)
       end

julia> @elapsed while n > 0 # print out results
           job_id, exec_time, where = take!(results)
           println("$job_id finished in $(round(exec_time,2)) seconds on worker $where")
           n = n - 1
       end
1 finished in 0.18 seconds on worker 4
2 finished in 0.26 seconds on worker 5
6 finished in 0.12 seconds on worker 4
7 finished in 0.18 seconds on worker 4
5 finished in 0.35 seconds on worker 5
4 finished in 0.68 seconds on worker 2
3 finished in 0.73 seconds on worker 3
11 finished in 0.01 seconds on worker 3
12 finished in 0.02 seconds on worker 3
9 finished in 0.26 seconds on worker 5
8 finished in 0.57 seconds on worker 4
10 finished in 0.58 seconds on worker 2
0.055971741
```

## Remote References and Distributed Garbage Collection

Objects referred to by remote references can be freed only when *all* held references
in the cluster are deleted.

The node where the value is stored keeps track of which of the workers have a reference to it.
Every time a [`RemoteChannel`](@ref) or a (unfetched) [`Future`](@ref) is serialized to a worker,
the node pointed to by the reference is notified. And every time a [`RemoteChannel`](@ref) or
a (unfetched) [`Future`](@ref) is garbage collected locally, the node owning the value is again
notified. This is implemented in an internal cluster aware serializer. Remote references are only
valid in the context of a running cluster. Serializing and deserializing references to and from
regular `IO` objects is not supported.

The notifications are done via sending of "tracking" messages--an "add reference" message when
a reference is serialized to a different process and a "delete reference" message when a reference
is locally garbage collected.

Since [`Future`](@ref)s are write-once and cached locally, the act of [`fetch()`](@ref)ing a
[`Future`](@ref) also updates reference tracking information on the node owning the value.

The node which owns the value frees it once all references to it are cleared.

With [`Future`](@ref)s, serializing an already fetched [`Future`](@ref) to a different node also
sends the value since the original remote store may have collected the value by this time.

It is important to note that *when* an object is locally garbage collected depends on the size
of the object and the current memory pressure in the system.

In case of remote references, the size of the local reference object is quite small, while the
value stored on the remote node may be quite large. Since the local object may not be collected
immediately, it is a good practice to explicitly call [`finalize()`](@ref) on local instances
of a [`RemoteChannel`](@ref), or on unfetched [`Future`](@ref)s. Since calling [`fetch()`](@ref)
on a [`Future`](@ref) also removes its reference from the remote store, this is not required on
fetched [`Future`](@ref)s. Explicitly calling [`finalize()`](@ref) results in an immediate message
sent to the remote node to go ahead and remove its reference to the value.

Once finalized, a reference becomes invalid and cannot be used in any further calls.

## [Shared Arrays](@id man-shared-arrays)

Shared Arrays use system shared memory to map the same array across many processes. While there
are some similarities to a [`DArray`](https://github.com/JuliaParallel/DistributedArrays.jl), the
behavior of a [`SharedArray`](@ref) is quite different. In a [`DArray`](https://github.com/JuliaParallel/DistributedArrays.jl),
each process has local access to just a chunk of the data, and no two processes share the same
chunk; in contrast, in a [`SharedArray`](@ref) each "participating" process has access to the
entire array.  A [`SharedArray`](@ref) is a good choice when you want to have a large amount of
data jointly accessible to two or more processes on the same machine.

[`SharedArray`](@ref) indexing (assignment and accessing values) works just as with regular arrays,
and is efficient because the underlying memory is available to the local process. Therefore,
most algorithms work naturally on [`SharedArray`](@ref)s, albeit in single-process mode. In cases
where an algorithm insists on an [`Array`](@ref) input, the underlying array can be retrieved
from a [`SharedArray`](@ref) by calling [`sdata()`](@ref). For other `AbstractArray` types, [`sdata()`](@ref)
just returns the object itself, so it's safe to use [`sdata()`](@ref) on any `Array`-type object.

The constructor for a shared array is of the form:

```julia
SharedArray{T,N}(dims::NTuple; init=false, pids=Int[])
```

which creates an `N`-dimensional shared array of a bits type `T` and size `dims` across the processes specified
by `pids`. Unlike distributed arrays, a shared array is accessible only from those participating
workers specified by the `pids` named argument (and the creating process too, if it is on the
same host).

If an `init` function, of signature `initfn(S::SharedArray)`, is specified, it is called on all
the participating workers. You can specify that each worker runs the `init` function on a distinct
portion of the array, thereby parallelizing initialization.

Here's a brief example:

```julia-repl
julia> addprocs(3)
3-element Array{Int64,1}:
 2
 3
 4

julia> S = SharedArray{Int,2}((3,4), init = S -> S[Base.localindexes(S)] = myid())
3×4 SharedArray{Int64,2}:
 2  2  3  4
 2  3  3  4
 2  3  4  4

julia> S[3,2] = 7
7

julia> S
3×4 SharedArray{Int64,2}:
 2  2  3  4
 2  3  3  4
 2  7  4  4
```

[`Base.localindexes()`](@ref) provides disjoint one-dimensional ranges of indexes, and is sometimes
convenient for splitting up tasks among processes. You can, of course, divide the work any way
you wish:

```julia-repl
julia> S = SharedArray{Int,2}((3,4), init = S -> S[indexpids(S):length(procs(S)):length(S)] = myid())
3×4 SharedArray{Int64,2}:
 2  2  2  2
 3  3  3  3
 4  4  4  4
```

Since all processes have access to the underlying data, you do have to be careful not to set up
conflicts. For example:

```julia
@sync begin
    for p in procs(S)
        @async begin
            remotecall_wait(fill!, p, S, p)
        end
    end
end
```

would result in undefined behavior. Because each process fills the *entire* array with its own
`pid`, whichever process is the last to execute (for any particular element of `S`) will have
its `pid` retained.

As a more extended and complex example, consider running the following "kernel" in parallel:

```julia
q[i,j,t+1] = q[i,j,t] + u[i,j,t]
```

In this case, if we try to split up the work using a one-dimensional index, we are likely to run
into trouble: if `q[i,j,t]` is near the end of the block assigned to one worker and `q[i,j,t+1]`
is near the beginning of the block assigned to another, it's very likely that `q[i,j,t]` will
not be ready at the time it's needed for computing `q[i,j,t+1]`. In such cases, one is better
off chunking the array manually. Let's split along the second dimension.
Define a function that returns the `(irange, jrange)` indexes assigned to this worker:

```julia-repl
julia> @everywhere function myrange(q::SharedArray)
           idx = indexpids(q)
           if idx == 0 # This worker is not assigned a piece
               return 1:0, 1:0
           end
           nchunks = length(procs(q))
           splits = [round(Int, s) for s in linspace(0,size(q,2),nchunks+1)]
           1:size(q,1), splits[idx]+1:splits[idx+1]
       end
```

Next, define the kernel:

```julia-repl
julia> @everywhere function advection_chunk!(q, u, irange, jrange, trange)
           @show (irange, jrange, trange)  # display so we can see what's happening
           for t in trange, j in jrange, i in irange
               q[i,j,t+1] = q[i,j,t] + u[i,j,t]
           end
           q
       end
```

We also define a convenience wrapper for a `SharedArray` implementation

```julia-repl
julia> @everywhere advection_shared_chunk!(q, u) =
           advection_chunk!(q, u, myrange(q)..., 1:size(q,3)-1)
```

Now let's compare three different versions, one that runs in a single process:

```julia-repl
julia> advection_serial!(q, u) = advection_chunk!(q, u, 1:size(q,1), 1:size(q,2), 1:size(q,3)-1);
```

one that uses [`@parallel`](@ref):

```julia-repl
julia> function advection_parallel!(q, u)
           for t = 1:size(q,3)-1
               @sync @parallel for j = 1:size(q,2)
                   for i = 1:size(q,1)
                       q[i,j,t+1]= q[i,j,t] + u[i,j,t]
                   end
               end
           end
           q
       end;
```

and one that delegates in chunks:

```julia-repl
julia> function advection_shared!(q, u)
           @sync begin
               for p in procs(q)
                   @async remotecall_wait(advection_shared_chunk!, p, q, u)
               end
           end
           q
       end;
```

If we create `SharedArray`s and time these functions, we get the following results (with `julia -p 4`):

```julia-repl
julia> q = SharedArray{Float64,3}((500,500,500));

julia> u = SharedArray{Float64,3}((500,500,500));
```

Run the functions once to JIT-compile and [`@time`](@ref) them on the second run:

```julia-repl
julia> @time advection_serial!(q, u);
(irange,jrange,trange) = (1:500,1:500,1:499)
 830.220 milliseconds (216 allocations: 13820 bytes)

julia> @time advection_parallel!(q, u);
   2.495 seconds      (3999 k allocations: 289 MB, 2.09% gc time)

julia> @time advection_shared!(q,u);
        From worker 2:       (irange,jrange,trange) = (1:500,1:125,1:499)
        From worker 4:       (irange,jrange,trange) = (1:500,251:375,1:499)
        From worker 3:       (irange,jrange,trange) = (1:500,126:250,1:499)
        From worker 5:       (irange,jrange,trange) = (1:500,376:500,1:499)
 238.119 milliseconds (2264 allocations: 169 KB)
```

The biggest advantage of `advection_shared!` is that it minimizes traffic among the workers, allowing
each to compute for an extended time on the assigned piece.

## Shared Arrays and Distributed Garbage Collection

Like remote references, shared arrays are also dependent on garbage collection on the creating
node to release references from all participating workers. Code which creates many short lived
shared array objects would benefit from explicitly finalizing these objects as soon as possible.
This results in both memory and file handles mapping the shared segment being released sooner.

## ClusterManagers

The launching, management and networking of Julia processes into a logical cluster is done via
cluster managers. A `ClusterManager` is responsible for

  * launching worker processes in a cluster environment
  * managing events during the lifetime of each worker
  * optionally, providing data transport

A Julia cluster has the following characteristics:

  * The initial Julia process, also called the `master`, is special and has an `id` of 1.
  * Only the `master` process can add or remove worker processes.
  * All processes can directly communicate with each other.

Connections between workers (using the in-built TCP/IP transport) is established in the following
manner:

  * [`addprocs()`](@ref) is called on the master process with a `ClusterManager` object.
  * [`addprocs()`](@ref) calls the appropriate [`launch()`](@ref) method which spawns required number
    of worker processes on appropriate machines.
  * Each worker starts listening on a free port and writes out its host and port information to [`STDOUT`](@ref).
  * The cluster manager captures the [`STDOUT`](@ref) of each worker and makes it available to the
    master process.
  * The master process parses this information and sets up TCP/IP connections to each worker.
  * Every worker is also notified of other workers in the cluster.
  * Each worker connects to all workers whose `id` is less than the worker's own `id`.
  * In this way a mesh network is established, wherein every worker is directly connected with every
    other worker.

While the default transport layer uses plain `TCPSocket`, it is possible for a Julia cluster to
provide its own transport.

Julia provides two in-built cluster managers:

  * `LocalManager`, used when [`addprocs()`](@ref) or [`addprocs(np::Integer)`](@ref) are called
  * `SSHManager`, used when [`addprocs(hostnames::Array)`](@ref) is called with a list of hostnames

`LocalManager` is used to launch additional workers on the same host, thereby leveraging multi-core
and multi-processor hardware.

Thus, a minimal cluster manager would need to:

  * be a subtype of the abstract `ClusterManager`
  * implement [`launch()`](@ref), a method responsible for launching new workers
  * implement [`manage()`](@ref), which is called at various events during a worker's lifetime (for
    example, sending an interrupt signal)

[`addprocs(manager::FooManager)`](@ref addprocs) requires `FooManager` to implement:

```julia
function launch(manager::FooManager, params::Dict, launched::Array, c::Condition)
    [...]
end

function manage(manager::FooManager, id::Integer, config::WorkerConfig, op::Symbol)
    [...]
end
```

As an example let us see how the `LocalManager`, the manager responsible for starting workers
on the same host, is implemented:

```julia
struct LocalManager <: ClusterManager
    np::Integer
end

function launch(manager::LocalManager, params::Dict, launched::Array, c::Condition)
    [...]
end

function manage(manager::LocalManager, id::Integer, config::WorkerConfig, op::Symbol)
    [...]
end
```

The [`launch()`](@ref) method takes the following arguments:

  * `manager::ClusterManager`: the cluster manager that [`addprocs()`](@ref) is called with
  * `params::Dict`: all the keyword arguments passed to [`addprocs()`](@ref)
  * `launched::Array`: the array to append one or more `WorkerConfig` objects to
  * `c::Condition`: the condition variable to be notified as and when workers are launched

The [`launch()`](@ref) method is called asynchronously in a separate task. The termination of
this task signals that all requested workers have been launched. Hence the [`launch()`](@ref)
function MUST exit as soon as all the requested workers have been launched.

Newly launched workers are connected to each other and the master process in an all-to-all manner.
Specifying the command line argument `--worker[=<cookie>]` results in the launched processes
initializing themselves as workers and connections being set up via TCP/IP sockets.

All workers in a cluster share the same [cookie](#cluster-cookie) as the master. When the cookie is
unspecified, i.e, with the `--worker` option, the worker tries to read it from its standard input.
 `LocalManager` and `SSHManager` both pass the cookie to newly launched workers via their
 standard inputs.

By default a worker will listen on a free port at the address returned by a call to `getipaddr()`.
A specific address to listen on may be specified by optional argument `--bind-to bind_addr[:port]`.
This is useful for multi-homed hosts.

As an example of a non-TCP/IP transport, an implementation may choose to use MPI, in which case
`--worker` must NOT be specified. Instead, newly launched workers should call `init_worker(cookie)`
before using any of the parallel constructs.

For every worker launched, the [`launch()`](@ref) method must add a `WorkerConfig` object (with
appropriate fields initialized) to `launched`

```julia
mutable struct WorkerConfig
    # Common fields relevant to all cluster managers
    io::Nullable{IO}
    host::Nullable{AbstractString}
    port::Nullable{Integer}

    # Used when launching additional workers at a host
    count::Nullable{Union{Int, Symbol}}
    exename::Nullable{AbstractString}
    exeflags::Nullable{Cmd}

    # External cluster managers can use this to store information at a per-worker level
    # Can be a dict if multiple fields need to be stored.
    userdata::Nullable{Any}

    # SSHManager / SSH tunnel connections to workers
    tunnel::Nullable{Bool}
    bind_addr::Nullable{AbstractString}
    sshflags::Nullable{Cmd}
    max_parallel::Nullable{Integer}

    connect_at::Nullable{Any}

    [...]
end
```

Most of the fields in `WorkerConfig` are used by the inbuilt managers. Custom cluster managers
would typically specify only `io` or `host` / `port`:

  * If `io` is specified, it is used to read host/port information. A Julia worker prints out its
    bind address and port at startup. This allows Julia workers to listen on any free port available
    instead of requiring worker ports to be configured manually.
  * If `io` is not specified, `host` and `port` are used to connect.
  * `count`, `exename` and `exeflags` are relevant for launching additional workers from a worker.
    For example, a cluster manager may launch a single worker per node, and use that to launch additional
    workers.

      * `count` with an integer value `n` will launch a total of `n` workers.
      * `count` with a value of `:auto` will launch as many workers as the number of cores on that machine.
      * `exename` is the name of the `julia` executable including the full path.
      * `exeflags` should be set to the required command line arguments for new workers.
  * `tunnel`, `bind_addr`, `sshflags` and `max_parallel` are used when a ssh tunnel is required to
    connect to the workers from the master process.
  * `userdata` is provided for custom cluster managers to store their own worker-specific information.

`manage(manager::FooManager, id::Integer, config::WorkerConfig, op::Symbol)` is called at different
times during the worker's lifetime with appropriate `op` values:

  * with `:register`/`:deregister` when a worker is added / removed from the Julia worker pool.
  * with `:interrupt` when `interrupt(workers)` is called. The `ClusterManager` should signal the
    appropriate worker with an interrupt signal.
  * with `:finalize` for cleanup purposes.

## Cluster Managers with Custom Transports

Replacing the default TCP/IP all-to-all socket connections with a custom transport layer is a
little more involved. Each Julia process has as many communication tasks as the workers it is
connected to. For example, consider a Julia cluster of 32 processes in an all-to-all mesh network:

  * Each Julia process thus has 31 communication tasks.
  * Each task handles all incoming messages from a single remote worker in a message-processing loop.
  * The message-processing loop waits on an `IO` object (for example, a `TCPSocket` in the default
    implementation), reads an entire message, processes it and waits for the next one.
  * Sending messages to a process is done directly from any Julia task--not just communication tasks--again,
    via the appropriate `IO` object.

Replacing the default transport requires the new implementation to set up connections to remote
workers and to provide appropriate `IO` objects that the message-processing loops can wait on.
The manager-specific callbacks to be implemented are:

```julia
connect(manager::FooManager, pid::Integer, config::WorkerConfig)
kill(manager::FooManager, pid::Int, config::WorkerConfig)
```

The default implementation (which uses TCP/IP sockets) is implemented as `connect(manager::ClusterManager, pid::Integer, config::WorkerConfig)`.

`connect` should return a pair of `IO` objects, one for reading data sent from worker `pid`, and
the other to write data that needs to be sent to worker `pid`. Custom cluster managers can use
an in-memory `BufferStream` as the plumbing to proxy data between the custom, possibly non-`IO`
transport and Julia's in-built parallel infrastructure.

A `BufferStream` is an in-memory `IOBuffer` which behaves like an `IO`--it is a stream which can
be handled asynchronously.

Folder `examples/clustermanager/0mq` contains an example of using ZeroMQ to connect Julia workers
in a star topology with a 0MQ broker in the middle. Note: The Julia processes are still all *logically*
connected to each other--any worker can message any other worker directly without any awareness
of 0MQ being used as the transport layer.

When using custom transports:

  * Julia workers must NOT be started with `--worker`. Starting with `--worker` will result in the
    newly launched workers defaulting to the TCP/IP socket transport implementation.
  * For every incoming logical connection with a worker, `Base.process_messages(rd::IO, wr::IO)()`
    must be called. This launches a new task that handles reading and writing of messages from/to
    the worker represented by the `IO` objects.
  * `init_worker(cookie, manager::FooManager)` MUST be called as part of worker process initialization.
  * Field `connect_at::Any` in `WorkerConfig` can be set by the cluster manager when [`launch()`](@ref)
    is called. The value of this field is passed in in all [`connect()`](@ref) callbacks. Typically,
    it carries information on *how to connect* to a worker. For example, the TCP/IP socket transport
    uses this field to specify the `(host, port)` tuple at which to connect to a worker.

`kill(manager, pid, config)` is called to remove a worker from the cluster. On the master process,
the corresponding `IO` objects must be closed by the implementation to ensure proper cleanup.
The default implementation simply executes an `exit()` call on the specified remote worker.

`examples/clustermanager/simple` is an example that shows a simple implementation using UNIX domain
sockets for cluster setup.

## Network Requirements for LocalManager and SSHManager

Julia clusters are designed to be executed on already secured environments on infrastructure such
as local laptops, departmental clusters, or even the cloud. This section covers network security
requirements for the inbuilt `LocalManager` and `SSHManager`:

  * The master process does not listen on any port. It only connects out to the workers.
  * Each worker binds to only one of the local interfaces and listens on an ephemeral port number
    assigned by the OS.
  * `LocalManager`, used by `addprocs(N)`, by default binds only to the loopback interface. This means
    that workers started later on remote hosts (or by anyone with malicious intentions) are unable
    to connect to the cluster. An `addprocs(4)` followed by an `addprocs(["remote_host"])` will fail.
    Some users may need to create a cluster comprising their local system and a few remote systems.
    This can be done by explicitly requesting `LocalManager` to bind to an external network interface
    via the `restrict` keyword argument: `addprocs(4; restrict=false)`.
  * `SSHManager`, used by `addprocs(list_of_remote_hosts)`, launches workers on remote hosts via SSH.
    By default SSH is only used to launch Julia workers. Subsequent master-worker and worker-worker
    connections use plain, unencrypted TCP/IP sockets. The remote hosts must have passwordless login
    enabled. Additional SSH flags or credentials may be specified via keyword argument `sshflags`.
  * `addprocs(list_of_remote_hosts; tunnel=true, sshflags=<ssh keys and other flags>)` is useful when
    we wish to use SSH connections for master-worker too. A typical scenario for this is a local laptop
    running the Julia REPL (i.e., the master) with the rest of the cluster on the cloud, say on Amazon
    EC2. In this case only port 22 needs to be opened at the remote cluster coupled with SSH client
    authenticated via public key infrastructure (PKI). Authentication credentials can be supplied
    via `sshflags`, for example ```sshflags=`-e <keyfile>` ```.

    In an all-to-all topology (the default), all workers connect to each other via plain TCP sockets.
    The security policy on the cluster nodes must thus ensure free connectivity between workers for
    the ephemeral port range (varies by OS).

    Securing and encrypting all worker-worker traffic (via SSH) or encrypting individual messages
    can be done via a custom ClusterManager.

## Cluster Cookie

All processes in a cluster share the same cookie which, by default, is a randomly generated string
on the master process:

  * [`Base.cluster_cookie()`](@ref) returns the cookie, while `Base.cluster_cookie(cookie)()` sets
    it and returns the new cookie.
  * All connections are authenticated on both sides to ensure that only workers started by the master
    are allowed to connect to each other.
  * The cookie may be passed to the workers at startup via argument `--worker=<cookie>`. If argument
    `--worker` is specified without the cookie, the worker tries to read the cookie from its
    standard input (STDIN). The STDIN is closed immediately after the cookie is retrieved.
  * ClusterManagers can retrieve the cookie on the master by calling [`Base.cluster_cookie()`](@ref).
    Cluster managers not using the default TCP/IP transport (and hence not specifying `--worker`)
    must call `init_worker(cookie, manager)` with the same cookie as on the master.

Note that environments requiring higher levels of security can implement this via a custom `ClusterManager`.
For example, cookies can be pre-shared and hence not specified as a startup argument.

## Specifying Network Topology (Experimental)

The keyword argument `topology` passed to `addprocs` is used to specify how the workers must be
connected to each other:

  * `:all_to_all`, the default: all workers are connected to each other.
  * `:master_slave`: only the driver process, i.e. `pid` 1, has connections to the workers.
  * `:custom`: the `launch` method of the cluster manager specifies the connection topology via the
    fields `ident` and `connect_idents` in `WorkerConfig`. A worker with a cluster-manager-provided
    identity `ident` will connect to all workers specified in `connect_idents`.

Keyword argument `lazy=true|false` only affects `topology` option `:all_to_all`. If `true`, the cluster
starts off with the master connected to all workers. Specific worker-worker connections are established
at the first remote invocation between two workers. This helps in reducing initial resources allocated for
intra-cluster communication. Connections are setup depending on the runtime requirements of a parallel
program. Default value for `lazy` is `true`.

Currently, sending a message between unconnected workers results in an error. This behaviour,
as with the functionality and interface, should be considered experimental in nature and may change
in future releases.

## Multi-Threading (Experimental)

In addition to tasks, remote calls, and remote references, Julia from `v0.5` forwards will natively
support multi-threading. Note that this section is experimental and the interfaces may change
in the future.

### Setup

By default, Julia starts up with a single thread of execution. This can be verified by using the
command [`Threads.nthreads()`](@ref):

```julia-repl
julia> Threads.nthreads()
1
```

The number of threads Julia starts up with is controlled by an environment variable called `JULIA_NUM_THREADS`.
Now, let's start up Julia with 4 threads:

```bash
export JULIA_NUM_THREADS=4
```

(The above command works on bourne shells on Linux and OSX. Note that if you're using a C shell
on these platforms, you should use the keyword `set` instead of `export`. If you're on Windows,
start up the command line in the location of `julia.exe` and use `set` instead of `export`.)

Let's verify there are 4 threads at our disposal.

```julia-repl
julia> Threads.nthreads()
4
```

But we are currently on the master thread. To check, we use the command [`Threads.threadid()`](@ref)

```julia-repl
julia> Threads.threadid()
1
```

### The `@threads` Macro

Let's work a simple example using our native threads. Let us create an array of zeros:

```jldoctest
julia> a = zeros(10)
10-element Array{Float64,1}:
 0.0
 0.0
 0.0
 0.0
 0.0
 0.0
 0.0
 0.0
 0.0
 0.0
```

Let us operate on this array simultaneously using 4 threads. We'll have each thread write its
thread ID into each location.

Julia supports parallel loops using the [`Threads.@threads`](@ref) macro. This macro is affixed
in front of a `for` loop to indicate to Julia that the loop is a multi-threaded region:

```julia-repl
julia> Threads.@threads for i = 1:10
           a[i] = Threads.threadid()
       end
```

The iteration space is split amongst the threads, after which each thread writes its thread ID
to its assigned locations:

```julia-repl
julia> a
10-element Array{Float64,1}:
 1.0
 1.0
 1.0
 2.0
 2.0
 2.0
 3.0
 3.0
 4.0
 4.0
```

Note that [`Threads.@threads`](@ref) does not have an optional reduction parameter like [`@parallel`](@ref).

## @threadcall (Experimental)

All I/O tasks, timers, REPL commands, etc are multiplexed onto a single OS thread via an event loop. A patched version of libuv ([http://docs.libuv.org/en/v1.x/](http://docs.libuv.org/en/v1.x/)) provides this functionality. Yield points provide for co-operatively scheduling multiple tasks onto the same OS thread. I/O tasks and timers yield implicitly while waiting for the event to occur. Calling [`yield()`](@ref) explicitly allows for other tasks to be scheduled.

Thus, a task executing a [`ccall`](@ref) effectively prevents the Julia scheduler from executing any other tasks till the call returns. This is true for all calls into external libraries. Exceptions are calls into custom C code that call back into Julia (which may then yield) or C code that calls `jl_yield()` (C equivalent of [`yield()`](@ref)).

Note that while Julia code runs on a single thread (by default), libraries used by Julia may launch their own internal threads. For example, the BLAS library may start as many threads as there are cores on a machine.

The `@threadcall` macro addresses scenarios where we do not want a `ccall` to block the main Julia event loop. It schedules a C function for execution in a separate thread. A threadpool with a default size of 4 is used for this. The size of the threadpool is controlled via environment variable `UV_THREADPOOL_SIZE`. While waiting for a free thread, and during function execution once a thread is available, the requesting task (on the main Julia event loop) yields to other tasks. Note that
`@threadcall` does not return till the execution is complete. From a user point of view, it is therefore a blocking call like other Julia APIs.

It is very important that the called function does not call back into Julia.

`@threadcall` may be removed/changed in future versions of Julia.

[^1]:
    In this context, MPI refers to the MPI-1 standard. Beginning with MPI-2, the MPI standards committee
    introduced a new set of communication mechanisms, collectively referred to as Remote Memory Access
    (RMA). The motivation for adding RMA to the MPI standard was to facilitate one-sided communication
    patterns. For additional information on the latest MPI standard, see [http://mpi-forum.org/docs](http://mpi-forum.org/docs/).
