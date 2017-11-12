# Manejando variaciones en el Sistema Operativo

Cuando se trata con librerías de plataforma, es frecuentemente necesario proporcionar casos especiales para distintas plataformas. La variable `Sys.KERNEL` puede utilizarse para escribir estos casos especiales. Hay varias funciones con intención de hacer esto más sencillo: `is_unix`, `is_linux`, `is_apple`, `is_bsd` e `is_windows`. Ellas pueden usarse de la siguiente forma:

```julia
if Sys.iswindows()
    some_complicated_thing(a)
end
```

Note que `is_linux`e `is_apple` son subconjuntos mutuamente exclusivos de `is_unix`. Adicionalmente, existe una macro `@static` que hace posible usar estas funciones para ocultar código inválido condicionalmente, como demuestran los siguientes ejemplos:

Bloques simples:

```
ccall((@static Sys.iswindows() ? :_fopen : :fopen), ...)
```

Bloques complejos:

```julia
@static if Sys.islinux()
    some_complicated_thing(a)
else
    some_different_thing(a)
end
```

Cuando se encadenan condicionales (incluyendo if/elsif/end) `@static` debe ser repetido por cada nivel (paréntesis opcional, pero recomendado por legibilidad):

```julia
@static Sys.iswindows() ? :a : (@static Sys.isapple() ? :b : :c)
```
