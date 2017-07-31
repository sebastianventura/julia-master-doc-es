# Getting Started

La instalación de Julia es sencilla, ya sea utilizando binarios precompilados o compilando desde la fuente. Descargue e instale Julia siguiendo las instrucciones en [https://julialang.org/downloads/](https://julialang.org/downloads/).

La forma más fácil de aprender y experimentar con Julia es iniciando una sesión interactiva (también conocida como *read-eval-print loop* o "REPL") haciendo doble clic en el ejecutable de Julia o ejecutando `julia` desde la línea de órdenes:

```
$ julia
               _
   _       _ _(_)_     |  A fresh approach to technical computing
  (_)     | (_) (_)    |  Documentation: https://docs.julialang.org
   _ _   _| |_  __ _   |  Type "?help" for help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 0.5.0-dev+2440 (2016-02-01 02:22 UTC)
 _/ |\__'_|_|_|\__'_|  |  Commit 2bb94d6 (11 days old master)
|__/                   |  x86_64-apple-darwin13.1.0

julia> 1 + 2
3

julia> ans
3
```

Para salir de la sesión interactiva, escriba `^D` (la tecla de control junto con la tecla `D`) o escriba
`quit()`. Cuando se ejecuta en modo interactivo, Julia muestra un banner y solicita al usuario la entrada. 
Una vez que el usuario ha introducido una expresión completa, como `1 + 2`, y pulsa *Enter*, la sesión 
interactiva evalúa la expresión y muestra su valor. Si se introduce una expresión en una sesión interactiva 
con un punto y coma al final, no se muestra su valor. La variable `ans` está enlazada al valor de la última 
expresión evaluada, sea mostrada o no. La variable `ans` sólo está enlazada a las sesiones interactivas, 
no cuando el código Julia se ejecuta de otras maneras.

Para evaluar expresiones escritas en un archivo de origen `file.jl`, escriba `include ("file.jl")`.

Para ejecutar código en un archivo de forma no interactiva, puede darlo como el primer argumento al mandato Julia:

```
$ julia script.jl arg1 arg2...
```

Como indica el ejemplo, los siguientes argumentos de línea de órdenes de Julia se toman como argumentos de 
línea de comandos al programa `script.jl` del programa, pasado a través de la constante global `ARGS`. El 
nombre del propio *script* se pasa como la variable global `PROGRAM_FILE`. Tenga en cuenta que `ARGS` 
también se establece cuando se da el código de script usando la opción `-e` en la línea de órdenes (vea 
la salida de ayuda de `julia` más abajo) pero `PROGRAM_FILE` estará vacío. Por ejemplo, para imprimir 
los argumentos que se le dan a un script, puede hacer esto:

```
$ julia -e 'println(PROGRAM_FILE); for x in ARGS; println(x); end' foo bar

foo
bar
```

O puede poner ese código en un script y ejecutarlo:

```
$ echo 'println(PROGRAM_FILE); for x in ARGS; println(x); end' > script.jl
$ julia script.jl foo bar
script.jl
foo
bar
```

El delimitador `--` puede usarse para separar argumentos en línea de mandatos al fichero del 
*script* a los argumentos de Julia:

```
$ julia --color=yes -O -- foo.jl arg1 arg2..
```

Julia can be started in parallel mode with either the `-p` or the `--machinefile` options. `-p n`
will launch an additional `n` worker processes, while `--machinefile file` will launch a worker
for each line in file `file`. The machines defined in `file` must be accessible via a passwordless
`ssh` login, with Julia installed at the same location as the current host. Each machine definition
takes the form `[count*][user@]host[:port] [bind_addr[:port]]` . `user` defaults to current user,
`port` to the standard ssh port. `count` is the number of workers to spawn on the node, and defaults
to 1. The optional `bind-to bind_addr[:port]` specifies the ip-address and port that other workers
should use to connect to this worker.

If you have code that you want executed whenever Julia is run, you can put it in `~/.juliarc.jl`:

```
$ echo 'println("Greetings! 你好! 안녕하세요?")' > ~/.juliarc.jl
$ julia
Greetings! 你好! 안녕하세요?

...
```

There are various ways to run Julia code and provide options, similar to those available for the
`perl` and `ruby` programs:

```
julia [switches] -- [programfile] [args...]
 -v, --version             Display version information
 -h, --help                Print this message

 -J, --sysimage <file>     Start up with the given system image file
 --precompiled={yes|no}    Use precompiled code from system image if available
 --compilecache={yes|no}   Enable/disable incremental precompilation of modules
 -H, --home <dir>          Set location of `julia` executable
 --startup-file={yes|no}   Load ~/.juliarc.jl
 --handle-signals={yes|no} Enable or disable Julia's default signal handlers

 -e, --eval <expr>         Evaluate <expr>
 -E, --print <expr>        Evaluate and show <expr>
 -L, --load <file>         Load <file> immediately on all processors

 -p, --procs {N|auto}      Integer value N launches N additional local worker processes
                           "auto" launches as many workers as the number of local cores
 --machinefile <file>      Run processes on hosts listed in <file>

 -i                        Interactive mode; REPL runs and isinteractive() is true
 -q, --quiet               Quiet startup (no banner)
 --color={yes|no}          Enable or disable color text
 --history-file={yes|no}   Load or save history

 --compile={yes|no|all|min}Enable or disable JIT compiler, or request exhaustive compilation
 -C, --cpu-target <target> Limit usage of cpu features up to <target>
 -O, --optimize={0,1,2,3}  Set the optimization level (default is 2 if unspecified or 3 if specified as -O)
 -g, -g <level>            Enable / Set the level of debug info generation (default is 1 if unspecified or 2 if specified as -g)
 --inline={yes|no}         Control whether inlining is permitted (overrides functions declared as @inline)
 --check-bounds={yes|no}   Emit bounds checks always or never (ignoring declarations)
 --math-mode={ieee,fast}   Disallow or enable unsafe floating point optimizations (overrides @fastmath declaration)

 --depwarn={yes|no|error}  Enable or disable syntax and method deprecation warnings ("error" turns warnings into errors)

 --output-o name           Generate an object file (including system image data)
 --output-ji name          Generate a system image data file (.ji)
 --output-bc name          Generate LLVM bitcode (.bc)
 --output-incremental=no   Generate an incremental output file (rather than complete)

 --code-coverage={none|user|all}, --code-coverage
                           Count executions of source lines (omitting setting is equivalent to "user")
 --track-allocation={none|user|all}, --track-allocation
                           Count bytes allocated by each source line
```

## Resources

In addition to this manual, there are various other resources that may help new users get started
with Julia:

  * [Julia and IJulia cheatsheet](http://math.mit.edu/~stevenj/Julia-cheatsheet.pdf)
  * [Learn Julia in a few minutes](https://learnxinyminutes.com/docs/julia/)
  * [Learn Julia the Hard Way](https://github.com/chrisvoncsefalvay/learn-julia-the-hard-way)
  * [Julia by Example](http://samuelcolvin.github.io/JuliaByExample/)
  * [Hands-on Julia](https://github.com/dpsanders/hands_on_julia)
  * [Tutorial for Homer Reid's numerical analysis class](http://homerreid.dyndns.org/teaching/18.330/JuliaProgramming.shtml)
  * [An introductory presentation](https://raw.githubusercontent.com/ViralBShah/julia-presentations/master/Fifth-Elephant-2013/Fifth-Elephant-2013.pdf)
  * [Videos from the Julia tutorial at MIT](https://julialang.org/blog/2013/03/julia-tutorial-MIT)
  * [YouTube videos from the JuliaCons](https://www.youtube.com/user/JuliaLanguage/playlists)
