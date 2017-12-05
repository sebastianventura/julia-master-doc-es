# Redes y Flujos

Julia proporciona una interfaz rica para tratar objetos que representen un flujo contínuo de E/S como terminales, tuberías y sockets TCP. Esta interfaz, aunque asíncrona a nivel del sistema, se presenta de forma síncrona al programador y normalmente no es necesario pensar en la operación asincrónica subyacente. Esto se logra haciendo un uso intensivo de la funcionalidad de los hilos cooperativos en Julia (o [corrutinas](@ref man-tasks)).

## Flujos de E/S básico

Todos los flujos en Julia exponen al menos un método [`read()`](@ref) y un [`write()`](@ref), tomando el flujo como su primer argumento, por ejemplo:

```julia-repl
julia> write(STDOUT,"Hello World");  # suppress return value 11 with ;
Hello World
julia> read(STDIN,Char)

'\n': ASCII/Unicode U+000a (category Cc: Other, control)
```

Tenga en cuenta que [`write()`](@ref) devuelve 11, el número de bytes (en `"Hello World"`) escrito en [`STDOUT`](@ref), pero este valor de retorno se suprime con `;`.

Aquí Enter fue presionado nuevamente para que Julia leyera la nueva línea. Ahora, como puede ver en este ejemplo, [`write()`] (@ref) toma los datos para escribir como su segundo argumento, mientras que [`read()`](@ref) toma el tipo de datos para ser leído como el segundo argumento.

Por ejemplo, para leer una matriz de bytes simple, podríamos hacer:

```julia-repl
julia> x = zeros(UInt8, 4)
4-element Array{UInt8,1}:
 0x00
 0x00
 0x00
 0x00

julia> read!(STDIN, x)
abcd
4-element Array{UInt8,1}:
 0x61
 0x62
 0x63
 0x64
```

Sin embargo, dado que esto es un poco engorroso, se proporcionan varios métodos de conveniencia. Por ejemplo, podríamos haber escrito lo anterior como:

```julia-repl
julia> read(STDIN,4)
abcd
4-element Array{UInt8,1}:
 0x61
 0x62
 0x63
 0x64
```

o si hubiéramos querido leer toda la línea en su lugar:

```julia-repl
julia> readline(STDIN)
abcd
"abcd"
```

Tenga en cuenta que, dependiendo de la configuración de su terminal, su TTY puede estar almacenado en línea y, por lo tanto, podría requerir una entrada adicional antes de enviar los datos a Julia.

Para leer cada línea desde [`STDIN`](@ref), puede usar [`eachline()`](@ref):

```julia
for line in eachline(STDIN)
    print("Found $line")
end
```

o [`read()`](@ref) si deseamos leer carácter a carácter en lugar de lo anterior:

```julia
while !eof(STDIN)
    x = read(STDIN, Char)
    println("Found: $x")
end
```

## Text I/O

Note que el método [`write()`](@ref) mencionado arriba opera sobre flujos binarios. En particular, los valores no son convertidos a ninguna representación de texto canónica sino que son escritas tal cual:

```jldoctest
julia> write(STDOUT,0x61);  # suppress return value 1 with ;
a
```

Note que `a` es escrito a [`STDOUT`](@ref) por la función [`write()`](@ref) y que el valor devuelto es `1` (ya que `0x61` es un byte).

Para E/S texto, use los métodos [`print()`](@ref) o [`show()`](@ref) methods, dependiendo de sus necesidades (ver la referencia de la librería estándar para una discusin detallada de la diferencia entre las dos):

```jldoctest
julia> print(STDOUT, 0x61)
97
```

## Propiedades contextuales de salida IO

En ocasiones, la salida de IO puede beneficiarse de la capacidad de pasar información contextual a los métodos de muestra. El objeto [`IOContext`](@ref) proporciona este marco para asociar metadatos arbitrarios con un objeto IO. Por ejemplo, [`showcompact`](@ref) agrega un parámetro de alusión al objeto IO para que el método del espectáculo invocado imprima un resultado más corto (si corresponde).

## Trabajando con Ficheros

Al igual que muchos otros entornos, Julia tiene una función [`open()`](@ref), que toma un nombre de archivo y devuelve un objeto `IOStream` que puede usar para leer y escribir cosas del archivo. Por ejemplo, si tenemos un archivo, `hello.txt`, cuyo contenido es `Hello, World!`:

```julia-repl
julia> f = open("hello.txt")
IOStream(<file hello.txt>)

julia> readlines(f)
1-element Array{String,1}:
 "Hello, World!"
```

Si desea escribir a un fichero, puede abrirlo con el flag de escritura (`"w"`):

```julia-repl
julia> f = open("hello.txt","w")
IOStream(<file hello.txt>)

julia> write(f,"Hello again.")
12
```

Si examina el contenido de `hello.txt` en este punto, notará que está vacío; no se ha escrito nada en el disco todavía. Esto se debe a que el `IOStream` debe cerrarse antes de que la escritura realmente se vacíe en el disco:

```julia-repl
julia> close(f)
```

Examinando `hello.txt` nuevamente mostrará que su contenido ha sido cambiado.

Abrir un archivo, hacer algo con su contenido y volver a cerrarlo es un patrón muy común. Para hacerlo más fácil, existe otra invocación de [`open()`](@ref) que toma una función como su primer argumento y nombre de archivo como su segundo, abre el archivo, llama a la función con el archivo como argumento, y luego lo cierra de nuevo. Por ejemplo, dada una función:

```julia
function read_and_capitalize(f::IOStream)
    return uppercase(read(f, String))
end
```

Uno puede llamar a:

```julia-repl
julia> open(read_and_capitalize, "hello.txt")
"HELLO AGAIN."
```

para abrir `hello.txt`, llamar `read_and_capitalize on it`, cerrar `hello.txt` y devolver los contenidos capitalizados.

Para incluso evitar tener que definir una función nombrada, puede usarse la sintaxis `do`, que crea una función anónima sobre la marcha:

```julia-repl
julia> open("hello.txt") do f
           uppercase(read(f, String))
       end
"HELLO AGAIN."
```

## Un ejemplo TCP simple

Saltemos directamente con un ejemplo simple que involucra sockets TCP. Primero creemos un servidor simple:

```julia-repl
julia> @async begin
           server = listen(2000)
           while true
               sock = accept(server)
               println("Hello World\n")
           end
       end
Task (runnable) @0x00007fd31dc11ae0
```

Para quienes estén familiarizados con la API de socket de Unix, los nombres de los métodos se sentirán familiares, aunque su uso es algo más simple que la API de socket Raw de Unix. La primera llamada a [`listen()`](@ref) creará un servidor en espera de conexiones entrantes en el puerto especificado (2000) en este caso. La misma función también se puede usar para crear otros tipos de servidores:

```julia-repl
julia> listen(2000) # Listens on localhost:2000 (IPv4)
Base.TCPServer(active)

julia> listen(ip"127.0.0.1",2000) # Equivalent to the first
Base.TCPServer(active)

julia> listen(ip"::1",2000) # Listens on localhost:2000 (IPv6)
Base.TCPServer(active)

julia> listen(IPv4(0),2001) # Listens on port 2001 on all IPv4 interfaces
Base.TCPServer(active)

julia> listen(IPv6(0),2001) # Listens on port 2001 on all IPv6 interfaces
Base.TCPServer(active)

julia> listen("testsocket") # Listens on a UNIX domain socket
Base.PipeServer(active)

julia> listen("\\\\.\\pipe\\testsocket") # Listens on a Windows named pipe
Base.PipeServer(active)
```

Tengase en cuenta que el tipo de retorno de la última invocación es diferente. Esto se debe a que este servidor no escucha en TCP, sino sobre una tubería nombrada (Windows) o socket de dominio UNIX. También tenga en cuenta que el formato de tubería nombrada de Windows debe ser un patrón específico tal que el prefijo del nombre (`\\.\pipe\`) identifique de manera única el [tipo de archivo](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365783(v=vs.85).aspx). La diferencia entre TCP y tuberías nombradas o sockets de dominio UNIX es sutil y tiene que ver con los métodos [`accept()`](@ref) y [`connect()`](@ref). El método [`accept()`](@ref) recupera una conexión con el cliente que se está conectando en el servidor que acabamos de crear, mientras que la función [`connect()`](@ref) se conecta a un servidor usando el método especificado. La función [`connect()`](@ ref) toma los mismos argumentos que [`listen()`](@ref), por lo tanto, suponiendo que el entorno (es decir, host, cwd, etc.) es el mismo uno debería capaz de pasar los mismos argumentos a [`connect()`](@ref) como lo se hizo para escuchar en el establecimiento de la conexión. Así que vamos a intentarlo (después de haber creado el servidor anterior):

```julia-repl
julia> connect(2000)
TCPSocket(open, 0 bytes waiting)

julia> Hello World
```

Como era de esperar, vimos "Hello World" impreso. Entonces, analicemos realmente lo que sucedió detrás de escena. Cuando llamamos a [`connect()`](@ref), nos conectamos al servidor que acabamos de crear. Mientras tanto, la función `accept` devuelve una conexión del lado del servidor al socket recién creado e imprime "Hello World" para indicar que la conexión fue exitosa.

Una gran fortaleza de Julia es que, dado que la API se expone sincrónicamente a pesar de que la E/S realmente está sucediendo de forma asíncrona, no tuvimos que preocuparnos de las devoluciones de llamadas ni siquiera de asegurarnos de que el servidor se ejecute. Cuando llamamos a [`connect()`](@ref) la tarea actual esperó a que se estableciera la conexión y solo continuó ejecutándose después de que se hizo. En esta pausa, la tarea del servidor reanudó la ejecución (porque una solicitud de conexión ya estaba disponible), aceptó la conexión, imprimió el mensaje y esperó al próximo cliente. Leer y escribir funciona de la misma manera. Para ver esto, considere el siguiente servidor de eco simple:

```julia-repl
julia> @async begin
           server = listen(2001)
           while true
               sock = accept(server)
               @async while isopen(sock)
                   write(sock,readline(sock))
               end
           end
       end
Task (runnable) @0x00007fd31dc12e60

julia> clientside = connect(2001)
TCPSocket(RawFD(28) open, 0 bytes waiting)

julia> @async while true
           write(STDOUT,readline(clientside))
       end
Task (runnable) @0x00007fd31dc11870

julia> println(clientside,"Hello World from the Echo Server")
Hello World from the Echo Server
```

Como con otros flujos, use [`close()`](@ref) para desconectar el socket:

```julia-repl
julia> close(clientside)
```

## Resolving IP Addresses

Uno de los métodos [`connect()`](@ref) que no sigue los métodos [`listen()`](@ref) es `connect(host::String, port)`, que intentará conectarse al host dado por el parámetro `host` en el puerto dado por el parámetro port. Te permite hacer cosas como:

```julia-repl
julia> connect("google.com",80)
TCPSocket(RawFD(30) open, 0 bytes waiting)
```

En la base de esta funcionalidad está [`getaddrinfo()`](@ref), que hará la resolución de dirección apropiada:

```julia-repl
julia> getaddrinfo("google.com")
ip"74.125.226.225"
```
