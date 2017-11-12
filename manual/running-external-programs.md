# Ejecutando programas externos

Julia toma prestada la notación de tilde inversa para ejecutar mandatos de la shell y programas en Perl y en Ruby.  Sin embargo, en Julia, escribir

```jldoctest
julia> `echo hello`
`echo hello`
```

difiere en algunos aspectos del comportamiento en varios shells, Perl y Ruby:

   * En lugar de ejecutar el mandato inmediatamente, las tildes invertidas crean un objeto `Cmd` para representar el 
     mandato. Uno puede usar este objeto para conectar los mandatos a otros vía tuberías (*pipes*), ejecutarlo y leer 
     o escribir en él.
   * Cuando el mandato se está ejecutando, Julia no captura su salida a menos que uno lo organice específicamente. En 
     lugar de ello, la salida del mandato va por defecto a [`STDOUT`](@ref) como si se estuviera realizando una llamada 
     al sistema con la `libc`.
   * El mandato nunca es ejecutado con un shell. En su lugar, Julia analiza directamente la sintaxis del mandato, 
     interpola variables adecuadametne y dividiendo en palabras como el shell lo haría, respetando su sintaxis. 
     El mandato se ejecuta como un proceso hijo inmediato de Julia, usando las llamadas `fork` y `exec`.

He aquí un ejemplo sencillo de ejecución de un programa externo:

```jldoctest
julia> mycommand = `echo hello`
`echo hello`

julia> typeof(mycommand)
Cmd

julia> run(mycommand)
hello
```

El mensaje `hello` es la salida de este mandato `echo`, enviado a [`STDOUT`](@ref). El mensaje `hello` es la salida de este mandato `echo`, enviado a `STDOUT`. El método ejecutado devuelve en si mismo `nothing`, y lanza una [`ErrorException`](@ref) si el mandato externo falla en ejecutarse con éxito.

Si se desea leer la salida de un mandato externo puede usarse `readstring()`:  

```jldoctest
julia> a = read(`echo hello`, String)
"hello\n"

julia> chomp(a) == "hello"
true
```

Más generalmente, puedes usar [`open()`](@ref) para leer desde o escribir hacia un mandato externo.

```jldoctest
julia> open(`less`, "w", STDOUT) do io
           for i = 1:3
               println(io, i)
           end
       end
1
2
3
```

El nombre del programa y los argumentos individuales de un mandato pueden ser accedidos e iterados como si el mandato fuera un array de cadenas:

```jldoctest
julia> collect(`echo "foo bar"`)
2-element Array{String,1}:
 "echo"
 "foo bar"

julia> `echo "foo bar"`[2]
"foo bar"
```

## [Interpolación](@id command-interpolation)

Supongamos que uno quiere algo un poco más complicado y usa el nombre de un fichero en la variable `file` como argumento a un mandato. Se puede usar `$` para interpolar tal como lo haríamos con un literal cadena (ver la sección [Strings](@ref)):

```jldoctest
julia> file = "/etc/passwd"
"/etc/passwd"

julia> `sort $file`
`sort /etc/passwd`
```

Un error común es que cuando se ejecutan programas externos a través de un shell es que si el nombre del fichero contiene caracteres que son especiales para el shell, ellos pueden causar un comportamiento indeseaddo. Por ejemplo, en lugar de `/etc/passwd` se desea ordenar los contenidos del fichero `/volumes/External HD/data.csv`. Intentémoslo:

```jldoctest
julia> file = "/Volumes/External HD/data.csv"
"/Volumes/External HD/data.csv"

julia> `sort $file`
`sort '/Volumes/External HD/data.csv'`
```

¿Cómo se entrecomilla el nombre de fichero? Julia sabe que `file` va a ser interpolado por un solo argumento, por lo que él entrecomilla la cadena. De hecho, esto no es bastante exacto: el valor de `file` no va a ser interpretado por el shell nunca, por lo que no hay necesidad de entrecomillar. Las comillas se insertan sólo para presentar al usuario. Eso funcionará incluso si uno interpola un valor como parte de una palabra del shell:

```jldoctest
julia> path = "/Volumes/External HD"
"/Volumes/External HD"

julia> name = "data"
"data"

julia> ext = "csv"
"csv"

julia> `sort $path/$name.$ext`
`sort '/Volumes/External HD/data.csv'`
```

Como puedes ver, el espacio en la variable `path` es apropiadamente "escapado". Pero, ¿qué pasa si lo que uno desea es interpolar múltiples palabras? En este caso, se utilizará un array (u otro contenedor iterable):

```jldoctest
julia> files = ["/etc/passwd","/Volumes/External HD/data.csv"]
2-element Array{String,1}:
 "/etc/passwd"
 "/Volumes/External HD/data.csv"

julia> `grep foo $files`
`grep foo /etc/passwd '/Volumes/External HD/data.csv'`
```

Si interpolas un array como parte de una palabra de la shell, Julia emula la generación de argumentos de la shell `{a, b, c}`:

```jldoctest
julia> names = ["foo","bar","baz"]
3-element Array{String,1}:
 "foo"
 "bar"
 "baz"

julia> `grep xylophone $names.txt`
`grep xylophone foo.txt bar.txt baz.txt`
```

Además, si interpolas múltiples arrays en la misma palabra, se emula el comportamiento de generación del shell haciendo el producto cartesiano: 

```jldoctest
julia> names = ["foo","bar","baz"]
3-element Array{String,1}:
 "foo"
 "bar"
 "baz"

julia> exts = ["aux","log"]
2-element Array{String,1}:
 "aux"
 "log"

julia> `rm -f $names.$exts`
`rm -f foo.aux foo.log bar.aux bar.log baz.aux baz.log`
```

Como pudes interpolar arrays de literales, puedes usar esta funcionalidad generativa sin necesidad de crer objetos array temporales primero: 

```jldoctest
julia> `rm -rf $["foo","bar","baz","qux"].$["aux","log","pdf"]`
`rm -rf foo.aux foo.log foo.pdf bar.aux bar.log bar.pdf baz.aux baz.log baz.pdf qux.aux qux.log qux.pdf`
```

## Entrecomillado

Inevitablemente, uno quiere escribir mandatos que no sean tan simples, y se vueve necesario usar comillas. He aquí un ejemplo simple de un script Perl de una línea en el prompt del shell:

```
sh$ perl -le '$|=1; for (0..3) { print }'
0
1
2
3
```

La expresión Perl necesita estar entre comillas sencillas por dos razones: para que los espacios no rompan la expresión en múltiples palabras en el shell, y para que el uso de variables de Perl, como `$|` no cause interpolación. En otras instancias, puedes querer usar dobles comillas para que la interpolación SI tenga lugar:

```
sh$ first="A"
sh$ second="B"
sh$ perl -le '$|=1; print for @ARGV' "1: $first" "2: $second"
1: A
2: B
```

En general, la sintaxis de comillas invertidas de Julia está diseñada cuidadosamente para que puedas cortar y pegar comendos del shell, los pongas entre comillas y funcionen: los comportamientos del escape, las comillas y las interpolaciones son los mismos que los del shell. La única diferencia es que la interpolación está integrada y consciente de la noción de Julia de que es un valor de cadena simple, y qué es un contenedor para valores múltiples. Intentemos los dos ejemplos anteriores en Julia:

```jldoctest
julia> A = `perl -le '$|=1; for (0..3) { print }'`
`perl -le '$|=1; for (0..3) { print }'`

julia> run(A)
0
1
2
3

julia> first = "A"; second = "B";

julia> B = `perl -le 'print for @ARGV' "1: $first" "2: $second"`
`perl -le 'print for @ARGV' '1: A' '2: B'`

julia> run(B)
1: A
2: B
```

Los resultados son idénticos, y el comportamiento de interpolación de Julia imita el shell con algunas mejoras debido a que Julia soporta objetos iterables de primera clase mientras la mayoría de los shells usan división de cadenas mediante espacios para ésto, lo cuál introduce ambigüedades. Cuando intentamos portar mandatos del shell a Julia, intentemos cortar y pegar primero. Como Julia te muestra los mandatos antes de que los ejecutes, puedes examinar fácilmente y de forma segura su interpretación sin hacer ningún daño.

## Tuberías

Los metacaracteres del shell tales como `|`, `&`, and `>`, necesitan ser acotados o escapados dentro de las comillas invertidas de Julia:

```jldoctest
julia> run(`echo hello '|' sort`)
hello | sort

julia> run(`echo hello \| sort`)
hello | sort
```

Esta expresión invoca el mandato `echo` con tres palabras como argumentos, "hello", "|" y "sort". El resultados es que se imprime una sola línea "hello | sort". Dentro de las comillas traseras, el símbolo "|" no tiene un significado especial. ¿Cómo entonces, podemos construir una tubería? En lugar de usar el símbolo "|" dentro de la tubería, utilizaremos la función [`pipeline()`](@ref):

```jldoctest
julia> run(pipeline(`echo hello`, `sort`))
hello
```

Esto entuba la salida del mandato `echo` al mandato `sort`. Por supuesto, esto no es terriblemente interesante ya que sólo hay una línea que ordenar, pero podemos cosas mucho más interesantes:

```julia-repl
julia> run(pipeline(`cut -d: -f3 /etc/passwd`, `sort -n`, `tail -n5`))
210
211
212
213
214
```

Esto imprime los cinco identificadores de usuario mayores dentro de un sistema UNIX. Los mandatos `cut`, `sort` y `tail` son "criados" como hijos inmediatos del proceso `julia` actual, sin que intervenga el proceso shell. Julia en sí mismo hace el trabajo (normalmente hecho por el shell) de inicializar las tuberías y conectar los descriptores de fichero.  Como Julia hace este trabajo, retiene un mejor control y puede hacer algunas cosas que el shell no puede.

Julia puede ejecutar múltiples órdenes en paralelo:

```julia-repl
julia> run(`echo hello` & `echo world`)
world
hello
```

El orden de esta salida es no determinista debido a que los dos procesos `echo` se lanzan casi simultáneamente, y compiten para hacer la primera escritura al descriptor [`STDOUT`](@ref) que ambas comparten con el proceso padre `julia`.  Julia te premite entubar la salida desde estos procesos a otro programa:

```jldoctest
julia> run(pipeline(`echo world` & `echo hello`, `sort`))
hello
world
```

En términos de fontanería UNIX, lo que está pasando aquí es que un único objeto tubería de UNIX se ha creado y es escrito por dos procesos `echo` y el otro extremo al final de la tubería es leído por la orden `sort`.

La redirección de la E/S puede conseguirse pasando los argumentos clave stdin, stdout y stderr a la función `pipeline`:

```julia
pipeline(`do_work`, stdout=pipeline(`sort`, "out.txt"), stderr="errs.txt")
```

### Evitar interbloqueos en tuberías

Cuando leemos y escribirmos en los dos extremos de una tubería desde un solo proceso, es importante evitar forzar el núcleo a almacenar todos los datos en el buffer.

Por ejemplo, cuando leemos toda la salida de un mandato, llamamos a `readstring(out)`, no `wait(process)`, ya que el primero consumirá activamente todos los datos exritos por el proceso, mientras que el último interntará almacenar los datos en los búferes del kernel mientras espera a que un lector esté conectado. 

Otra solución común es separar el lector y el escritor de la tuberían en tareas separadas:

```julia
writer = @async write(process, "data")
reader = @async do_compute(read(process, String))
wait(process)
fetch(reader)
```

### Ejemplo complicado

La combinación de un lenguaje de programación de alto nivel, una abstracción de mandatos de primera clase y la inicialización automática de tuberías entre procesos es muy poderos. Para dar algun sentido a las tuberías complejas que pueden ser creadas facilmente, he aquí algunos ejemplos más sofisticado, con nuestras disculpas por el excesito uso de scripts Perl con una sola línea:

```julia-repl
julia> prefixer(prefix, sleep) = `perl -nle '$|=1; print "'$prefix' ", $_; sleep '$sleep';'`;

julia> run(pipeline(`perl -le '$|=1; for(0..9){ print; sleep 1 }'`, prefixer("A",2) & prefixer("B",2)))
A 0
B 1
A 2
B 3
A 4
B 5
A 6
B 7
A 8
B 9
```

Este es el ejemplo típico de un solo productor que alimenta dos consumidores concurrentes: un proceso `perl` genera líneas con los número 0 a 9, el otro con la letra "B". Qué consumidor llega el primero es no determinista, pero una vez que se ha ganado la carrera, las líneas son consumidas alternativamente primero por un proceso y después por el otro. (Fijas `$|=1` en Perl causa que cada instrucción de impresión vuelque al flujo [`STDOUT`](@ref), lo cuál es necesario para que este ejemplo funcione. En caso contrario toda la salida va a un buffer y sería impresa en la tubería de una vez, para ser leída por un solo proceso consumidor).

He aquí un ejemplo incluso más complicado de productor consumidor multi-etapa:

```julia-repl
julia> run(pipeline(`perl -le '$|=1; for(0..9){ print; sleep 1 }'`,
           prefixer("X",3) & prefixer("Y",3) & prefixer("Z",3),
           prefixer("A",2) & prefixer("B",2)))
A X 0
B Y 1
A Z 2
B X 3
A Y 4
B Z 5
A X 6
B Y 7
A Z 8
B X 9
```

Este ejemplo es similar al anterior, excepto en que hay dos etapas de consumidores y las estapas tiene diferente latencia por lo que usan un número de workers paralelos diferrentes, para mantener saturado el throughput .

Se recomienda intentar todos estos ejemplos y ver cómo funcionan.
