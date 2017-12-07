# E/S y Redes

## E/S General

```@docs
Base.STDOUT
Base.STDERR
Base.STDIN
Base.open
Base.IOBuffer
Base.take!(::Base.GenericIOBuffer)
Base.fdio
Base.flush
Base.close
Base.crc32c(::IO, ::Integer, ::UInt32)
Base.write
Base.read
Base.read!
Base.readbytes!
Base.unsafe_read
Base.unsafe_write
Base.position
Base.seek
Base.seekstart
Base.seekend
Base.skip
Base.mark
Base.unmark
Base.reset
Base.ismarked
Base.eof
Base.isreadonly
Base.iswritable
Base.isreadable
Base.isopen
Base.Serializer.serialize
Base.Serializer.deserialize
Base.Grisu.print_shortest
Base.fd
Base.redirect_stdout
Base.redirect_stdout(::Function, ::Any)
Base.redirect_stderr
Base.redirect_stderr(::Function, ::Any)
Base.redirect_stdin
Base.redirect_stdin(::Function, ::Any)
Base.readchomp
Base.truncate
Base.skipchars
Base.DataFmt.countlines
Base.PipeBuffer
Base.readavailable
Base.IOContext
Base.IOContext(::IO, ::Pair)
Base.IOContext(::IO, ::IOContext)
```

## E/S Texto

```@docs
Base.show(::Any)
Base.showcompact
Base.summary
Base.print
Base.println
Base.print_with_color
Base.info
Base.warn
Base.logging
Base.Printf.@printf
Base.Printf.@sprintf
Base.sprint
Base.showerror
Base.dump
Base.readline
Base.readuntil
Base.readlines
Base.eachline
Base.DataFmt.readdlm(::Any, ::Char, ::Type, ::Char)
Base.DataFmt.readdlm(::Any, ::Char, ::Char)
Base.DataFmt.readdlm(::Any, ::Char, ::Type)
Base.DataFmt.readdlm(::Any, ::Char)
Base.DataFmt.readdlm(::Any, ::Type)
Base.DataFmt.readdlm(::Any)
Base.DataFmt.writedlm
Base.DataFmt.readcsv
Base.DataFmt.writecsv
Base.Base64.Base64EncodePipe
Base.Base64.Base64DecodePipe
Base.Base64.base64encode
Base.Base64.base64decode
Base.displaysize
```

## E/S Multimedia

Del mismo modo que la salida de texto se realiza mediante [`print`](@ref) y los tipos definidos por el usuario pueden indicar su representación textual sobrecargando [`show`](@ref), Julia proporciona un mecanismo estandarizado para una salida multimedia enriquecida (como imágenes, texto formateado, o incluso audio y video) que consta de tres partes:

* Una función [`display(x)`](@ref) para solicitar la visualización multimedia más completa disponible de un objeto Julia `x` (con una reserva de texto sin formato).
* Sobrecargar [`show`](@ref) permite indicar representaciones multimedia arbitrarias (codificadas mediante tipos MIME estándar) de tipos definidos por el usuario.
* Los backends de visualización con capacidad multimedia pueden registrarse subclasificando un tipo genérico de `Display` y empujándolos a una pila de backends de visualización mediante [`pushdisplay`](@ref).

El tiempo de ejecución base de Julia proporciona solo visualización de texto sin formato, pero las pantallas más ricas pueden habilitarse cargando módulos externos o utilizando entornos gráficos de Julia (como el *notebook* IJulia, basado en IPython).

```@docs
Base.Multimedia.display
Base.Multimedia.redisplay
Base.Multimedia.displayable
Base.show(::Any, ::Any, ::Any)
Base.Multimedia.mimewritable
Base.Multimedia.reprmime
Base.Multimedia.stringmime
```

Como se mencionó anteriormente, también se pueden definir nuevos backends de pantalla. Por ejemplo, un módulo que puede mostrar imágenes PNG en una ventana puede registrar esta capacidad con Julia, de modo que llamar a [`display(x)`](@ref) en tipos con representaciones PNG mostrará automáticamente la imagen usando la ventana del módulo.

Para definir un nuevo backend de pantalla, primero se debe crear un subtipo `D` de la clase abstracta `Display`. Luego, para cada tipo MIME (cadena `mime`) que se puede mostrar en `D`, uno debe definir una función `display(d::D, ::MIME"mime", x) = ...` que muestra `x` como ese tipo MIME, generalmente llamando a [`reprmime(mime, x)`](@ref). Se debe lanzar `MethodError` si `x` no se puede mostrar como ese tipo MIME; esto es automático si uno llama [`reprmime`](@ref). Finalmente, se debe definir una función `display(d::D, x)` que consulte [`mimewritable(mime,x)`](@ref) para los tipos `mime` soportados por `D` y muestre el "mejor"; debe lanzarse un `MethodError` si no se encuentran tipos MIME compatibles para `x`. Del mismo modo, algunos subtipos pueden anular [`redisplay(d::D, ...)`](@ref Base.Multimedia.redisplay). (De nuevo, uno debe hacer `import Base.display` para agregar nuevos métodos a `display`.) Los valores de retorno de estas funciones dependen de la implementación (ya que en algunos casos puede ser útil devolver un "manejador" *handle* de visualización de algunos tipo). Las funciones de visualización para `D` se pueden llamar directamente, pero también se pueden invocar automáticamente desde [`display(x)`](@ref) simplemente presionando una nueva pantalla en la pila display-backend con:

```@docs
Base.Multimedia.pushdisplay
Base.Multimedia.popdisplay
Base.Multimedia.TextDisplay
Base.Multimedia.istextmime
```

## E/S Mapeada en Memoria

```@docs
Base.Mmap.Anonymous
Base.Mmap.mmap(::Any, ::Type, ::Any, ::Any)
Base.Mmap.mmap(::Any, ::BitArray, ::Any, ::Any)
Base.Mmap.sync!
```

## E/S por Red

```@docs
Base.connect(::TCPSocket, ::Integer)
Base.connect(::AbstractString)
Base.listen(::Any)
Base.listen(::AbstractString)
Base.getaddrinfo
Base.getsockname
Base.getpeername
Base.IPv4
Base.IPv6
Base.nb_available
Base.accept
Base.listenany
Base.Filesystem.poll_fd
Base.Filesystem.poll_file
Base.Filesystem.watch_file
Base.bind
Base.send
Base.recv
Base.recvfrom
Base.setopt
Base.ntoh
Base.hton
Base.ltoh
Base.htol
Base.ENDIAN_BOM
```
