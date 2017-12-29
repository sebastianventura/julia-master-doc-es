# Linear algebra

Además de (y como parte de) su soporte a los arrays multidimensionales, Julia proporciona implementaciones nativas de muchas operaciones de álgebra lineal comunes y útiles. Las operaciones básicas tales como la traza ([`trace`](@ref)), el determinante ([`det`](@ref)) y la inversa ([`inv`](@ref)) están todas soportadas:

```jldoctest
julia> A = [1 2 3; 4 1 6; 7 8 1]
3×3 Array{Int64,2}:
 1  2  3
 4  1  6
 7  8  1

julia> trace(A)
3

julia> det(A)
104.0

julia> inv(A)
3×3 Array{Float64,2}:
 -0.451923   0.211538    0.0865385
  0.365385  -0.192308    0.0576923
  0.240385   0.0576923  -0.0673077
```

Así como otras operaciones útiles, como buscar autovalores o autovectores:

```jldoctest
julia> A = [1.5 2 -4; 3 -1 -6; -10 2.3 4]
3×3 Array{Float64,2}:
   1.5   2.0  -4.0
   3.0  -1.0  -6.0
 -10.0   2.3   4.0

julia> eigvals(A)
3-element Array{Complex{Float64},1}:
  9.31908+0.0im
 -2.40954+2.72095im
 -2.40954-2.72095im

julia> eigvecs(A)
3×3 Array{Complex{Float64},2}:
 -0.488645+0.0im  0.182546-0.39813im   0.182546+0.39813im
 -0.540358+0.0im  0.692926+0.0im       0.692926-0.0im
   0.68501+0.0im  0.254058-0.513301im  0.254058+0.513301im
```

Además, Julia proporciona muchas [factorizaciones](@ref man-linalg-factorizations) que pueden usarse para acelerar problemas como la resolución lineal o la exponenciación de matrices mediante la pre-factorización de una matriz en una forma más adecuada (por razones de rendimiento o memoria) ) al problema. Consulte la documentación en [`factorize`](@ref) para obtener más información. Como ejemplo:

```jldoctest
julia> A = [1.5 2 -4; 3 -1 -6; -10 2.3 4]
3×3 Array{Float64,2}:
   1.5   2.0  -4.0
   3.0  -1.0  -6.0
 -10.0   2.3   4.0

julia> factorize(A)
Base.LinAlg.LU{Float64,Array{Float64,2}} with factors L and U:
[1.0 0.0 0.0; -0.15 1.0 0.0; -0.3 -0.132196 1.0]
[-10.0 2.3 4.0; 0.0 2.345 -3.4; 0.0 0.0 -5.24947]
```

Como `A` no es hermítica, simétrica, triangular o bidiagonal, una factorización LU puede ser lo mejor que podemos hacer. Compara con:

```jldoctest
julia> B = [1.5 2 -4; 2 -1 -3; -4 -3 5]
3×3 Array{Float64,2}:
  1.5   2.0  -4.0
  2.0  -1.0  -3.0
 -4.0  -3.0   5.0

julia> factorize(B)
Base.LinAlg.BunchKaufman{Float64,Array{Float64,2}}
D factor:
3×3 Tridiagonal{Float64}:
 -1.64286   0.0   ⋅
  0.0      -2.8  0.0
   ⋅        0.0  5.0
U factor:
3×3 Base.LinAlg.UnitUpperTriangular{Float64,Array{Float64,2}}:
 1.0  0.142857  -0.8
 0.0  1.0       -0.6
 0.0  0.0        1.0
permutation:
3-element Array{Int64,1}:
 1
 2
 3
successful: true
```

Aquí, Julia fue capaz de detectar que `B` es de hecho simetrica, y usa una factorizacíon más apropiada. Frecuentemente es posible escribir código ms eficiente para una matriz de la que se conocen ciertas propiedades como que sea simétrica o diagonal. Julia proporciona algunos tipos especiales para que uno pueda "etiquetar" las matrices que tengan estas propiedades. Por ejemplo:

```jldoctest
julia> B = [1.5 2 -4; 2 -1 -3; -4 -3 5]
3×3 Array{Float64,2}:
  1.5   2.0  -4.0
  2.0  -1.0  -3.0
 -4.0  -3.0   5.0

julia> sB = Symmetric(B)
3×3 Symmetric{Float64,Array{Float64,2}}:
  1.5   2.0  -4.0
  2.0  -1.0  -3.0
 -4.0  -3.0   5.0
```

`sB` ha sido etiquetada como una matriz que es simétrica (real) por lo que para algunas operacioneas que podríamos hacer sobre ella, tal como la autofactorización o calbular productos matriz-vector, pueden encontrarse eficiencias sólo referenciando la mitad de ella. Por ejemplo:

```jldoctest
julia> B = [1.5 2 -4; 2 -1 -3; -4 -3 5]
3×3 Array{Float64,2}:
  1.5   2.0  -4.0
  2.0  -1.0  -3.0
 -4.0  -3.0   5.0

julia> sB = Symmetric(B)
3×3 Symmetric{Float64,Array{Float64,2}}:
  1.5   2.0  -4.0
  2.0  -1.0  -3.0
 -4.0  -3.0   5.0

julia> x = [1; 2; 3]
3-element Array{Int64,1}:
 1
 2
 3

julia> sB\x
3-element Array{Float64,1}:
 -1.73913
 -1.1087
 -1.45652
```
La operación `\` realiza aquí la resolución de la ecuación lineal. El analizador sintáctico de Julia proporciona un despacho conveniente para métodos especializados para la *transpuesta* de una matriz o una matriz dividida por la izquierda por un vector, o para las distintas combinaciones u operacioneas de transposición en soluciones matriz-matriz. Muchas de ellas son incluso más especializadas para ciertos tipos especiales de matrices. Por ejemplo, `A\B` acabará llamando a [`Base.LinAlg.A_ldiv_B!`](@ref) mientras que `A'\B` acabará llamando a [`Base.LinAlg.Ac_ldiv_B`](@ref), incluso aunque usáramos el mismo operador de división por la izquierda. Esto funcionea también para matrices: `A.'\B.'` invocará a [`Base.LinAlg.At_ldiv_Bt`](@ref). El operador de división por la izquierda es muy potente y es fácil escribir código compacto y bastante legible para resolver todo tipo de sistemas de ecuaciones lineales.

## Matrices especiales

[Las matrices con simetrías y estructuras especiales](http://www2.imm.dtu.dk/pubdb/views/publication_details.php?id=3274) surgen a menudo en el álgebra lineal y frecuentemente se asocian con varias factorizaciones matriciales. Julia presenta una rica colección de tipos de matrices especiales, que permiten un cálculo rápido con rutinas especializadas que están especialmente desarrolladas para estos tipos particuales de matrices.

Las siguientes tablas resumen los tipos de matrices especiales que se han implementado en Julia, así como si están disponibles ganchos para varios métodos optimizados para ellos en LAPACK.

| Type                      | Description                                                                      |
|:------------------------- |:-------------------------------------------------------------------------------- |
| [`Symmetric`](@ref)       | [Symmetric matrix](https://en.wikipedia.org/wiki/Symmetric_matrix)               |
| [`Hermitian`](@ref)       | [Hermitian matrix](https://en.wikipedia.org/wiki/Hermitian_matrix)               |
| [`UpperTriangular`](@ref) | Upper [triangular matrix](https://en.wikipedia.org/wiki/Triangular_matrix)       |
| [`LowerTriangular`](@ref) | Lower [triangular matrix](https://en.wikipedia.org/wiki/Triangular_matrix)       |
| [`Tridiagonal`](@ref)     | [Tridiagonal matrix](https://en.wikipedia.org/wiki/Tridiagonal_matrix)           |
| [`SymTridiagonal`](@ref)  | Symmetric tridiagonal matrix                                                     |
| [`Bidiagonal`](@ref)      | Upper/lower [bidiagonal matrix](https://en.wikipedia.org/wiki/Bidiagonal_matrix) |
| [`Diagonal`](@ref)        | [Diagonal matrix](https://en.wikipedia.org/wiki/Diagonal_matrix)                 |
| [`UniformScaling`](@ref)  | [Uniform scaling operator](https://en.wikipedia.org/wiki/Uniform_scaling)        |

### Elementary operations

| Matrix type               | `+` | `-` | `*` | `\` | Other functions with optimized methods                              |
|:------------------------- |:--- |:--- |:--- |:--- |:------------------------------------------------------------------- |
| [`Symmetric`](@ref)       |     |     |     | MV  | [`inv()`](@ref), [`sqrtm()`](@ref), [`expm()`](@ref)                |
| [`Hermitian`](@ref)       |     |     |     | MV  | [`inv()`](@ref), [`sqrtm()`](@ref), [`expm()`](@ref)                |
| [`UpperTriangular`](@ref) |     |     | MV  | MV  | [`inv()`](@ref), [`det()`](@ref)                                    |
| [`LowerTriangular`](@ref) |     |     | MV  | MV  | [`inv()`](@ref), [`det()`](@ref)                                    |
| [`SymTridiagonal`](@ref)  | M   | M   | MS  | MV  | [`eigmax()`](@ref), [`eigmin()`](@ref)                              |
| [`Tridiagonal`](@ref)     | M   | M   | MS  | MV  |                                                                     |
| [`Bidiagonal`](@ref)      | M   | M   | MS  | MV  |                                                                     |
| [`Diagonal`](@ref)        | M   | M   | MV  | MV  | [`inv()`](@ref), [`det()`](@ref), [`logdet()`](@ref), [`/()`](@ref) |
| [`UniformScaling`](@ref)  | M   | M   | MVS | MVS | [`/()`](@ref)                                                       |

Legend:

| Key        | Description                                                   |
|:---------- |:------------------------------------------------------------- |
| M (matrix) | An optimized method for matrix-matrix operations is available |
| V (vector) | An optimized method for matrix-vector operations is available |
| S (scalar) | An optimized method for matrix-scalar operations is available |

### Matrix factorizations

| Matrix type               | LAPACK | [`eig()`](@ref) | [`eigvals()`](@ref) | [`eigvecs()`](@ref) | [`svd()`](@ref) | [`svdvals()`](@ref) |
|:------------------------- |:------ |:--------------- |:------------------- |:------------------- |:--------------- |:------------------- |
| [`Symmetric`](@ref)       | SY     |                 | ARI                 |                     |                 |                     |
| [`Hermitian`](@ref)       | HE     |                 | ARI                 |                     |                 |                     |
| [`UpperTriangular`](@ref) | TR     | A               | A                   | A                   |                 |                     |
| [`LowerTriangular`](@ref) | TR     | A               | A                   | A                   |                 |                     |
| [`SymTridiagonal`](@ref)  | ST     | A               | ARI                 | AV                  |                 |                     |
| [`Tridiagonal`](@ref)     | GT     |                 |                     |                     |                 |                     |
| [`Bidiagonal`](@ref)      | BD     |                 |                     |                     | A               | A                   |
| [`Diagonal`](@ref)        | DI     |                 | A                   |                     |                 |                     |

Legend:

| Key          | Description                                                                                                                     | Example              |
|:------------ |:------------------------------------------------------------------------------------------------------------------------------- |:-------------------- |
| A (all)      | An optimized method to find all the characteristic values and/or vectors is available                                           | e.g. `eigvals(M)`    |
| R (range)    | An optimized method to find the `il`th through the `ih`th characteristic values are available                                   | `eigvals(M, il, ih)` |
| I (interval) | An optimized method to find the characteristic values in the interval [`vl`, `vh`] is available                                 | `eigvals(M, vl, vh)` |
| V (vectors)  | An optimized method to find the characteristic vectors corresponding to the characteristic values `x=[x1, x2,...]` is available | `eigvecs(M, x)`      |

### El operador de escalado uniforme

Un operador [`UniformScaling`](@ref) representa un escalar multiplicado por el operador de identidad, `λ*I`. El operador de identidad `I` se define como una constante y es una instancia de `UniformScaling`. El tamaño de estos operadores es genérico y coincide con la otra matriz en las operaciones binarias [`+`](@ref), [`-`](@ref), [`*`](@ref) y [`\`](@ref). Para `A+I` y` A-I` esto significa que `A` debe ser cuadrado. La multiplicación con el operador de identidad `I` es un *noop* (excepto para comprobar que el factor de escala es uno) y, por lo tanto, casi sin sobrecarga.

## [Factorizaciones de matrices](@id man-linalg-factorizations)

[las factorizaciones de matrices (a.k.a. descomposiciones de matrices)](https://en.wikipedia.org/wiki/Matrix_decomposition)
calculan la factorizacin de una matriz en un producto de matrices, y son uno de los conceptos centrales del álgebra lineal.

La siguiente tabla resume los tipos de factorizaciones de matrices que han sido implementados en Julia. En la sección [Linear Algebra](@ref) de la documentación de la librería estándar pueden encontrarse más detalles de los métodos asociados.

| Type              | Description                                                                                                    |
|:----------------- |:-------------------------------------------------------------------------------------------------------------- |
| `Cholesky`        | [Cholesky factorization](https://en.wikipedia.org/wiki/Cholesky_decomposition)                                 |
| `CholeskyPivoted` | [Pivoted](https://en.wikipedia.org/wiki/Pivot_element) Cholesky factorization                                  |
| `LU`              | [LU factorization](https://en.wikipedia.org/wiki/LU_decomposition)                                             |
| `LUTridiagonal`   | LU factorization for [`Tridiagonal`](@ref) matrices                                                            |
| `UmfpackLU`       | LU factorization for sparse matrices (computed by UMFPack)                                                     |
| `QR`              | [QR factorization](https://en.wikipedia.org/wiki/QR_decomposition)                                             |
| `QRCompactWY`     | Compact WY form of the QR factorization                                                                        |
| `QRPivoted`       | Pivoted [QR factorization](https://en.wikipedia.org/wiki/QR_decomposition)                                     |
| `Hessenberg`      | [Hessenberg decomposition](http://mathworld.wolfram.com/HessenbergDecomposition.html)                          |
| `Eigen`           | [Spectral decomposition](https://en.wikipedia.org/wiki/Eigendecomposition_(matrix))                            |
| `SVD`             | [Singular value decomposition](https://en.wikipedia.org/wiki/Singular_value_decomposition)                     |
| `GeneralizedSVD`  | [Generalized SVD](https://en.wikipedia.org/wiki/Generalized_singular_value_decomposition#Higher_order_version) |

