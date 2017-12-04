# Variables de Entorno

Julia se puede configurar con varias variables de entorno, ya sea de la manera habitual del sistema operativo o de manera portátil desde Julia. Supongamos que quiere establecer la variable de entorno `JULIA_EDITOR` como `vim`, para ello escribirá `ENV["JULIA_EDITOR"] = "vim"` en el REPL para llevar a cabo este cambio en la sesión actual, o agregará lo mismo en el archivo de configuración `.juliarc.jl` en el directorio de inicio del usuario para tener un efecto permanente. El valor actual de la misma variable de entorno se determina evaluando `ENV ["JULIA_EDITOR"]`.

Las variables de entorno que usa Julia generalmente comienzan con `JULIA`. Si [`Base.versioninfo`](@ref) se llama con `verbose` igual a `true`, la salida mostrará una lista de variables de entorno definidas relevantes para Julia, incluidas aquellas para las cuales` JULIA` aparece en el nombre.

## Localizaciones de fichero

### `JULIA_HOME`

La ruta absoluta del directorio que contiene el ejecutable de Julia, que establece la variable global [`Base.JULIA_HOME`](@ref). Si `$JULIA_HOME` no está configurado, entonces Julia determina el valor` Base.JULIA_HOME` en el tiempo de ejecución.

El ejecutable en sí es uno de

```
$ JULIA_HOME/julia
$ JULIA_HOME/julia-debug
```

por defecto.

La variable global `Base.DATAROOTDIR` determina una ruta relativa de `Base.JULIA_HOME` al directorio de datos asociado con Julia. Entonces el camino

```
$ JULIA_HOME / $ DATAROOTDIR / julia / base
```

determina el directorio en el que Julia inicialmente busca los archivos fuente (a través de `Base.find_source_file()`).

Del mismo modo, la variable global `Base.SYSCONFDIR` determina una ruta relativa al directorio del archivo de configuración. Entonces Julia busca un archivo `juliarc.jl` en

`` `
$JULIA_HOME/$SYSCONFDIR/julia/juliarc.jl
$JULIA_HOME/../etc/julia/juliarc.jl
`` `

por defecto (a través de `Base.load_juliarc()`).

Por ejemplo, una instalación de Linux con un ejecutable de Julia ubicado en `/bin/julia`, un` DATAROOTDIR` de `../ share`, y un` SYSCONFDIR` de `../ etc` tendrá `JULIA_HOME` establecido en `/bin`, una ruta de búsqueda de archivo fuente de

```
/share/julia/base
```

y una ruta de búsqueda de configuración global de

```
/etc/julia/juliarc.jl
```

### `JULIA_LOAD_PATH`

Una lista separada de rutas absolutas que se anexarán a la variable [`LOAD_PATH`](@ref). (En sistemas tipo Unix, el separador de ruta es `:`; en sistemas Windows, el separador de ruta es `;`.) La variable `LOAD_PATH` es donde [`Base.require`](@ref) y `Base.load_in_path()` busca el código; se predetermina a las rutas absolutas

```
$JULIA_HOME/../local/share/julia/site/v$(VERSION.major).$(VERSION.minor)
$JULIA_HOME/../share/julia/site/v$(VERSION.major).$(VERSION.minor)
```

de modo que, por ejemplo, la versión 0.6 de Julia en un sistema Linux con un ejecutable de Julia en `/bin/julia` tendrá un` LOAD_PATH` predeterminado de

```
/local/share/julia/site/v0.6
/share/julia/site/v0.6
```

### `JULIA_PKGDIR`

La ruta del directorio principal `Pkg.Dir._pkgroot()` para los repositorios de paquetes Julia específicos de la versión. Si la ruta es relativa, entonces se toma con respecto al directorio de trabajo. Si `$JULIA_PKGDIR` no está configurado, entonces` Pkg.Dir._pkgroot() `se establece por defecto en

```
$HOME/.julia
```

Entonces la localizacíon del repositorio [`Pkg.dir`](@ref) para una versin dada de Julia es

```
$JULIA_PKGDIR/v$(VERSION.major).$(VERSION.minor)
```

Por ejemplo, para un usuario Linux cuyo directorio home ea `/home/alice`, el directorio que contiene los repositorios de paquetes por defecto sería

```
/home/alice/.julia
```

y el repositorio de paquetes paa la versión 0.6 de Julia sería

```
/home/alice/.julia/v0.6
```

### `JULIA_HISTORY`

El camino absoluto `Base.REPL.find_hist_file()` del fichero de historia del REPL. Si `$JULIA_HISTORY` no está fijado, encontces `Base.REPL.find_hist_file()` tiene como valor por defecto

```
$HOME/.julia_history
```

### `JULIA_PKGRESOLVE_ACCURACY`

Un `Int` positivo que determina cuánto tiempo la subrutina de suma máxima` MaxSum.maxsum () `del resolvedor de dependencia de paquetes [` Base.Pkg.resolve`](@ref) dedicará a intentar las restricciones satisfactorias antes de abandonar: este valor es por defecto `1`, y los valores más grandes corresponden a mayores cantidades de tiempo.

Supongamos que el valor de `$ JULIA_PKGRESOLVE_ACCURACY` es `n`. Entonces

* el número de iteraciones de pre-decimación es `20*n`,
* el número de iteraciones entre los pasos de aniquilación es `10*n`, y
* en los pasos de aniquilación, como máximo se destruye uno de cada paquetes `20*n`.


## Aplicaciones externas

### `JULIA_SHELL`

La ruta absoluta del shell con el que Julia debe ejecutar comandos externos (a través de `Base.repl_cmd ()`). Se predetermina a la variable de entorno `$SHELL`, y vuelve a `/bin/sh` si `$SHELL` está desactivado.

!!! note

     En Windows, esta variable de entorno se ignora y los comandos externos se ejecutan directamente.

### `JULIA_EDITOR`

El editor devuelto por `Base.editor()` y utilizado en, por ejemplo, [`Base.edit`] (@ref), refiriéndose al comando del editor preferido, por ejemplo` vim`.

`$JULIA_EDITOR` tiene prioridad sobre` $VISUAL`, que a su vez tiene prioridad sobre `$EDITOR`. Si no se establece ninguna de estas variables de entorno, entonces el editor se considera `abierto` en Windows y OS X, o` /etc/alternatives/editor` si
existe, o `emacs` de lo contrario.

!!! note

    `$JULIA_EDITOR` *no se usa* en la determinación del editor para [`Base.Pkg.edit`](@ref): esta función verifica `$VISUAL` y` $EDITOR` solo.

## Paralelización

### `JULIA_CPU_CORES`

Sobreescribe la variable global [`Base.Sys.CPU_CORES`](@ref), el número de núcleos de CPU lógicos disponible.

### `JULIA_WORKER_TIMEOUT`

Un [`Float64`](@ref) que establece el valor de` Base.worker_timeout()` (predeterminado:` 60.0`). Esta función proporciona la cantidad de segundos que un proceso de trabajo esperará un proceso maestro para establecer una conexión antes de morir.

### `JULIA_NUM_THREADS`

Un entero sin signo de 64 bits (`uint64_t`) que establece el número máximo de subprocesos disponiblew para Julia. Si `$JULIA_NUM_THREADS` excede la cantidad disponiblede núcleos de CPU físicos, el número de subprocesos se establece en la cantidad de núcleos. Si `$JULIA_NUM_THREADS` no es positivo o no está configurado, o si el número de núcleos de CPU no se puede determinar a través de llamadas al sistema, entonces la cantidad de hilos es establecida en `1`.

### `JULIA_THREAD_SLEEP_THRESHOLD`

Si se fija a una cadena que comienza con la subcadena insensible a mayúsculas y minúsculas `"infinite"`, entonces los hilos vivos nunva duermen. En caso contrario, `$JULIA_THREAD_SLEEP_THRESHOLD` es interpretado como un entero sin signo de 64 bits (`uint64_t`) y da, en nanosegundos, la cantidad de tiempo después del cual los hilos deben dormir.

### `JULIA_EXCLUSIVE`

Si se establece en algo además de `0`, entonces la política de hilos de Julia es consistente con la ejecución en una máquina dedicada: el hilo maestro está en proc 0, y los hilos están affinitizados. De lo contrario, Julia deja que el sistema operativo maneje la política de hilos.

## Formateo del REPL

Variables de entorno que determinan cómo debe formatearse la salida REPL en el terminal. En general, estas variables deben establecerse en [seciencias de scape de terminal ANSI](http://ascii-table.com/ansi-escape-sequences.php). Julia proporciona
una interfaz de alto nivel con gran parte de la misma funcionalidad: ver la sección sobre [Interacción con Julia](@ref).

### `JULIA_ERROR_COLOR`

El formato `Base.error_color()` (predeterminado: rojo claro, `"\033[91m "`) que los errores deberían tener en la terminal.

### `JULIA_WARN_COLOR`

El formato `Base.warn_color()` (preeterminado: amarillo, `"\033[93m"`) que deberían tener las advertencias en el terminal.

### `JULIA_INFO_COLOR`

El formato `Base.info_color()` (predeterminado: cyan, `"\033[36m"`) que debería tener la info en el terminal. 

### `JULIA_INPUT_COLOR`

El formato `Base.input_color()` (predeterminado: normal, `"\033[0m"`) que debería haber en el terminal.

### `JULIA_ANSWER_COLOR`

El formato `Base.answer_color()` (predeterminado:  normal, `"\033[0m"`) que debería haber en el terminal..

### `JULIA_STACKFRAME_LINEINFO_COLOR`

El formato `Base.stackframe_lineinfo_color()` (predeterminado:  bold, `"\033[1m"`) que la línea info debería tener durante una traza de la pila en el terminal.

### `JULIA_STACKFRAME_FUNCTION_COLOR`

El formato `Base.stackframe_function_color()` (predeterminado:  bold, `"\033[1m"`) que las llamadas a funcin deberían tener durante una traza de la pila en el terminal.

## Depuración y profiling

### `JULIA_GC_ALLOC_POOL`, `JULIA_GC_ALLOC_OTHER`, `JULIA_GC_ALLOC_PRINT`

Si se establecen, estas variables de entorno toman cadenas que opcionalmente comienzan con el carácter `'r'`, seguido por una interpolación de cadenas de una lista separada por dos puntos de tres enteros de 64 bits (`int64_t`). Este triplete de enteros `a:b:c` representa la secuencia aritmética `a`, `a + b`,` a + 2*b`, ... `c`.

 * Si es la "n" vez que se ha llamado a `jl_gc_pool_alloc ()`, y `n` pertenece a la secuencia aritmética 
   representada por` $ JULIA_GC_ALLOC_POOL`, entonces se forzará la recolección de elementos no utilizados.
 * Si es la `n`th vez que` maybe_collect () `ha sido llamado, y `n` pertenece a la secuencia aritmética 
   representada por `$JULIA_GC_ALLOC_OTHER`, entonces se forzará la recolección de elementos no utilizados.
 * Si es la `n` th vez que` jl_gc_collect()` ha sido llamado, y `n` pertenece a la secuencia aritmética 
   representada por `$JULIA_GC_ALLOC_PRINT`, entonces cuenta para el número de llamadas a `jl_gc_pool_alloc()` y
   `maybe_collect()` están impresos.

Si el valor de la variable de entorno comienza con el carácter `'r'`, entonces el intervalo entre los eventos de recolección de basura es aleatorio.

!!! note

    Estas variables de entorno solo tienen un efecto si Julia se compiló con la 
    depuración de la recolección de basura (es decir, si `WITH_GC_DEBUG_ENV` 
    se fija a `1` en la configuración de compilación).

### `JULIA_GC_NO_GENERATIONAL`

Si se fija a algo por encima de `0`, entonces el recolector de basura de Julia nunca realizará barridos rápidos de memoria.

!!! note

    Esta variable de entorno solo tiene un efecto si Julia se compiló con depuración de recolección de elementos 
    no utilizados (es decir, si `WITH_GC_DEBUG_ENV` está establecido en` 1` en la configuración de compilación).
    
### `JULIA_GC_WAIT_FOR_DEBUGGER`

Si se fija a algo por encima de `0`, entonces el recolector de basura esperará a que un depurador la enlace en lugar de abortar cuando haya un error crítico.

!!! note

    Esta variable de entorno solo tiene un efecto si Julia se compiló con depuración de recolección de elementos 
    no utilizados (es decir, si `WITH_GC_DEBUG_ENV` está establecido en` 1` en la configuración de compilación).

### `ENABLE_JITPROFILING`

Si se fija a un valor por encima de `0`, entonces el compilador creará y registrará un detector de evaentos (*event listener*)  para *for just-in-time (JIT) profiling*.

!!! note

    Esta variable de entorno sólo tiene efecto su Julia se compiló con soporte de JIT profiling usando 

*   [VTune™ Amplifier](https://software.intel.com/en-us/intel-vtune-amplifier-xe) de Intel
    (`USE_INTEL_JITEVENTS` set to `1` in the build configuration), o bien
*   [OProfile](http://oprofile.sourceforge.net/news/) (`USE_OPROFILE_JITEVENTS` set to `1`
    in the build configuration).

### `JULIA_LLVM_ARGS`

Arguments para pasar al backend de LLVM.

!!! note

    Esta variable de entorno sólo tiene efecto su Julia se compiló con
    `JL_DEBUG_BUILD` fijado — en particular, el ejecutable `julia-debug` siempre es compilado con
    esta variable.

### `JULIA_DEBUG_LOADING`

Si se establece, entonces Julia imprime información detallada sobre la caché en el proceso de carga de [`Base.require`](@ref).

