# Álgebra Lineal

## Funciones Estándar

Las funciones de álgebra lineal en Julia está ampliamente implementadas llamando a funciones de [LAPACK](http://www.netlib.org/lapack/). Las factorizaciones *sparse* llaman a funciones de [SuiteSparse](http://faculty.cse.tamu.edu/davis/suitesparse.html).

```@docs
Base.:*(::AbstractArray, ::AbstractArray)
Base.:\(::AbstractArray, ::Any)
Base.LinAlg.dot
Base.LinAlg.vecdot
Base.LinAlg.cross
Base.LinAlg.factorize
Base.LinAlg.Diagonal
Base.LinAlg.Bidiagonal
Base.LinAlg.SymTridiagonal
Base.LinAlg.Tridiagonal
Base.LinAlg.Symmetric
Base.LinAlg.Hermitian
Base.LinAlg.LowerTriangular
Base.LinAlg.UpperTriangular
Base.LinAlg.UniformScaling
Base.LinAlg.lu
Base.LinAlg.lufact
Base.LinAlg.lufact!
Base.LinAlg.chol
Base.LinAlg.cholfact
Base.LinAlg.cholfact!
Base.LinAlg.lowrankupdate
Base.LinAlg.lowrankdowndate
Base.LinAlg.lowrankupdate!
Base.LinAlg.lowrankdowndate!
Base.LinAlg.ldltfact
Base.LinAlg.ldltfact!
Base.LinAlg.qr
Base.LinAlg.qr!
Base.LinAlg.qrfact
Base.LinAlg.qrfact!
Base.LinAlg.QR
Base.LinAlg.QRCompactWY
Base.LinAlg.QRPivoted
Base.LinAlg.lqfact!
Base.LinAlg.lqfact
Base.LinAlg.lq
Base.LinAlg.bkfact
Base.LinAlg.bkfact!
Base.LinAlg.eig
Base.LinAlg.eigvals
Base.LinAlg.eigvals!
Base.LinAlg.eigmax
Base.LinAlg.eigmin
Base.LinAlg.eigvecs
Base.LinAlg.eigfact
Base.LinAlg.eigfact!
Base.LinAlg.hessfact
Base.LinAlg.hessfact!
Base.LinAlg.schurfact
Base.LinAlg.schurfact!
Base.LinAlg.schur
Base.LinAlg.ordschur
Base.LinAlg.ordschur!
Base.LinAlg.svdfact
Base.LinAlg.svdfact!
Base.LinAlg.svd
Base.LinAlg.svdvals
Base.LinAlg.Givens
Base.LinAlg.givens
Base.LinAlg.triu
Base.LinAlg.triu!
Base.LinAlg.tril
Base.LinAlg.tril!
Base.LinAlg.diagind
Base.LinAlg.diag
Base.LinAlg.diagm
Base.LinAlg.scale!
Base.LinAlg.rank
Base.LinAlg.norm
Base.LinAlg.vecnorm
Base.LinAlg.normalize!
Base.LinAlg.normalize
Base.LinAlg.cond
Base.LinAlg.condskeel
Base.LinAlg.trace
Base.LinAlg.det
Base.LinAlg.logdet
Base.LinAlg.logabsdet
Base.inv(::AbstractMatrix)
Base.LinAlg.pinv
Base.LinAlg.nullspace
Base.repmat
Base.kron
Base.SparseArrays.blkdiag
Base.LinAlg.linreg
Base.LinAlg.expm
Base.LinAlg.logm
Base.LinAlg.sqrtm
Base.LinAlg.lyap
Base.LinAlg.sylvester
Base.LinAlg.issuccess
Base.LinAlg.issymmetric
Base.LinAlg.isposdef
Base.LinAlg.isposdef!
Base.LinAlg.istril
Base.LinAlg.istriu
Base.LinAlg.isdiag
Base.LinAlg.ishermitian
Base.LinAlg.RowVector
Base.LinAlg.ConjArray
Base.transpose
Base.transpose!
Base.ctranspose
Base.ctranspose!
Base.LinAlg.eigs(::Any)
Base.LinAlg.eigs(::Any, ::Any)
Base.LinAlg.svds
Base.LinAlg.peakflops
```

## Operaciones matriciales de bajo nivel

Las operaciones de matrices que involucran operaciones de transposición como `A' \ B` son convertidas por el analizador de Julia en llamadas a funciones especialmente nombradas como [`Ac_ldiv_B`](@ref). Si desea sobrecargar estas operaciones para sus propios tipos, será útil conocer los nombres de estas funciones.

Además, en muchos casos, hay versiones in situ de operaciones matriciales que le permiten suministrar un vector o matriz de salida preasignada. Esto es útil cuando se optimiza código crítico para evitar la sobrecarga de las asignaciones repetidas. Estas operaciones in situ tienen el sufijo `!` a continuación (por ejemplo, [`A_mul_B!`](@ref)) de acuerdo con la convención habitual de Julia.

```@docs
Base.LinAlg.A_ldiv_B!
Base.A_ldiv_Bc
Base.A_ldiv_Bt
Base.LinAlg.A_mul_B!
Base.A_mul_Bc
Base.A_mul_Bt
Base.A_rdiv_Bc
Base.A_rdiv_Bt
Base.Ac_ldiv_B
Base.LinAlg.Ac_ldiv_B!
Base.Ac_ldiv_Bc
Base.Ac_mul_B
Base.Ac_mul_Bc
Base.Ac_rdiv_B
Base.Ac_rdiv_Bc
Base.At_ldiv_B
Base.LinAlg.At_ldiv_B!
Base.At_ldiv_Bt
Base.At_mul_B
Base.At_mul_Bt
Base.At_rdiv_B
Base.At_rdiv_Bt
```

## Funciones BLAS

En Julia (como en gran parte de la computación científica), las operaciones  de álgebra lineal densa se basan en la [biblioteca LAPACK](http://www.netlib.org/lapack/), que a su vez se construye sobre bloques de construcción básicos de álgebra lineal conocidos como [BLAS](http://www.netlib.org/blas/). Hay implementaciones altamente optimizadas de BLAS disponibles para cada arquitectura de computadora, y algunas veces en rutinas de álgebra lineal de alto rendimiento, es útil llamar directamente a las funciones de BLAS.

`Base.LinAlg.BLAS` proporciona envoltorios para algunas de las funciones de BLAS. Esas funciones de BLAS que sobrescriben una de las matrices de entrada tienen nombres que terminan en `'!'`. Normalmente, una función BLAS tiene cuatro métodos definidos, para las arrays [`Float64`](@ref), [`Float32`](@ref), `Complex128` y` Complex64`.

### [BLAS Character Arguments](@id stdlib-blas-chars)
Muchas funciones BLAS aceptan argumentos que determinan si se debe transponer un argumento (`trans`), qué triángulo de una matriz referenciar (` uplo` o `ul`), si se puede suponer que la diagonal de una matriz triangular está formada por unos (`dA`) o a qué lado de una multiplicación de matrices pertenece el argumento de entrada (`side`). Las posibilidades son:

#### [Orden de Multiplicación](@id stdlib-blas-side)
| `side` | Meaning                                                             |
|:-------|:--------------------------------------------------------------------|
| `'L'`  | El argumento va al lado *izquierdo* de una operación matriz-matriz. |
| `'R'`  | El argumento va al lado *derecho* de una operación matriz-matriz.   |

#### [Referencia sobre el Triángulo](@id stdlib-blas-uplo)
| `uplo`/`ul` | Meaning                                               |
|:------------|:------------------------------------------------------|
| `'U'`       | Sólo se usará el triángulo *superior* de la matriz.   |
| `'L'`       | Sólo se usará el triángulo *inferior* de la matriz. |

#### [Operación de Transposición](@id stdlib-blas-trans)
| `trans`/`tX` | Meaning                                                  |
|:-------------|:---------------------------------------------------------|
| `'N'`        | La matriz de entrada `X` no es transpuesta ni conjugada. |
| `'T'`        | La matriz de entrada `X` será transpuesta.               |
| `'C'`        | La matriz de entrada `X` será conjugada y transpuesta.   |

#### [Unidades en la Diagonal](@id stdlib-blas-diag)
| `diag`/`dX` | Meaning                                                                     |
|:------------|:----------------------------------------------------------------------------|
| `'N'`       | Los valores diagonales de la matriz `X`serán leídos.                        |
| `'U'`       | Se supone que los elementos de la diagonal de la matriz `X` son todos unos. |

```@docs
Base.LinAlg.BLAS.dotu
Base.LinAlg.BLAS.dotc
Base.LinAlg.BLAS.blascopy!
Base.LinAlg.BLAS.nrm2
Base.LinAlg.BLAS.asum
Base.LinAlg.axpy!
Base.LinAlg.BLAS.scal!
Base.LinAlg.BLAS.scal
Base.LinAlg.BLAS.ger!
Base.LinAlg.BLAS.syr!
Base.LinAlg.BLAS.syrk!
Base.LinAlg.BLAS.syrk
Base.LinAlg.BLAS.her!
Base.LinAlg.BLAS.herk!
Base.LinAlg.BLAS.herk
Base.LinAlg.BLAS.gbmv!
Base.LinAlg.BLAS.gbmv
Base.LinAlg.BLAS.sbmv!
Base.LinAlg.BLAS.sbmv(::Any, ::Any, ::Any, ::Any, ::Any)
Base.LinAlg.BLAS.sbmv(::Any, ::Any, ::Any, ::Any)
Base.LinAlg.BLAS.gemm!
Base.LinAlg.BLAS.gemm(::Any, ::Any, ::Any, ::Any, ::Any)
Base.LinAlg.BLAS.gemm(::Any, ::Any, ::Any, ::Any)
Base.LinAlg.BLAS.gemv!
Base.LinAlg.BLAS.gemv(::Any, ::Any, ::Any, ::Any)
Base.LinAlg.BLAS.gemv(::Any, ::Any, ::Any)
Base.LinAlg.BLAS.symm!
Base.LinAlg.BLAS.symm(::Any, ::Any, ::Any, ::Any, ::Any)
Base.LinAlg.BLAS.symm(::Any, ::Any, ::Any, ::Any)
Base.LinAlg.BLAS.symv!
Base.LinAlg.BLAS.symv(::Any, ::Any, ::Any, ::Any)
Base.LinAlg.BLAS.symv(::Any, ::Any, ::Any)
Base.LinAlg.BLAS.trmm!
Base.LinAlg.BLAS.trmm
Base.LinAlg.BLAS.trsm!
Base.LinAlg.BLAS.trsm
Base.LinAlg.BLAS.trmv!
Base.LinAlg.BLAS.trmv
Base.LinAlg.BLAS.trsv!
Base.LinAlg.BLAS.trsv
Base.LinAlg.BLAS.set_num_threads
Base.LinAlg.I
```

## LAPACK Functions

`Base.LinAlg.LAPACK` proporciona *wrappers* para algunas de las funciones LAPACK para álgebra lineal. Las funciones que sobrescriben una de las matrices de entrada tienen nombres que terminan en `'!'`.

Por lo general, una función tiene 4 métodos definidos, uno para las arrays [`Float64`](@ref), [`Float32`](@ref), `Complex128` y `Complex64`.

Tenga en cuenta que la API LAPACK proporcionada por Julia puede y va a cambiar en el futuro. Dado que esta API no está orientada al usuario, no existe el compromiso de admitir/desaprobar este conjunto específico de funciones en futuras versiones.

```@docs
Base.LinAlg.LAPACK.gbtrf!
Base.LinAlg.LAPACK.gbtrs!
Base.LinAlg.LAPACK.gebal!
Base.LinAlg.LAPACK.gebak!
Base.LinAlg.LAPACK.gebrd!
Base.LinAlg.LAPACK.gelqf!
Base.LinAlg.LAPACK.geqlf!
Base.LinAlg.LAPACK.geqrf!
Base.LinAlg.LAPACK.geqp3!
Base.LinAlg.LAPACK.gerqf!
Base.LinAlg.LAPACK.geqrt!
Base.LinAlg.LAPACK.geqrt3!
Base.LinAlg.LAPACK.getrf!
Base.LinAlg.LAPACK.tzrzf!
Base.LinAlg.LAPACK.ormrz!
Base.LinAlg.LAPACK.gels!
Base.LinAlg.LAPACK.gesv!
Base.LinAlg.LAPACK.getrs!
Base.LinAlg.LAPACK.getri!
Base.LinAlg.LAPACK.gesvx!
Base.LinAlg.LAPACK.gelsd!
Base.LinAlg.LAPACK.gelsy!
Base.LinAlg.LAPACK.gglse!
Base.LinAlg.LAPACK.geev!
Base.LinAlg.LAPACK.gesdd!
Base.LinAlg.LAPACK.gesvd!
Base.LinAlg.LAPACK.ggsvd!
Base.LinAlg.LAPACK.ggsvd3!
Base.LinAlg.LAPACK.geevx!
Base.LinAlg.LAPACK.ggev!
Base.LinAlg.LAPACK.gtsv!
Base.LinAlg.LAPACK.gttrf!
Base.LinAlg.LAPACK.gttrs!
Base.LinAlg.LAPACK.orglq!
Base.LinAlg.LAPACK.orgqr!
Base.LinAlg.LAPACK.orgql!
Base.LinAlg.LAPACK.orgrq!
Base.LinAlg.LAPACK.ormlq!
Base.LinAlg.LAPACK.ormqr!
Base.LinAlg.LAPACK.ormql!
Base.LinAlg.LAPACK.ormrq!
Base.LinAlg.LAPACK.gemqrt!
Base.LinAlg.LAPACK.posv!
Base.LinAlg.LAPACK.potrf!
Base.LinAlg.LAPACK.potri!
Base.LinAlg.LAPACK.potrs!
Base.LinAlg.LAPACK.pstrf!
Base.LinAlg.LAPACK.ptsv!
Base.LinAlg.LAPACK.pttrf!
Base.LinAlg.LAPACK.pttrs!
Base.LinAlg.LAPACK.trtri!
Base.LinAlg.LAPACK.trtrs!
Base.LinAlg.LAPACK.trcon!
Base.LinAlg.LAPACK.trevc!
Base.LinAlg.LAPACK.trrfs!
Base.LinAlg.LAPACK.stev!
Base.LinAlg.LAPACK.stebz!
Base.LinAlg.LAPACK.stegr!
Base.LinAlg.LAPACK.stein!
Base.LinAlg.LAPACK.syconv!
Base.LinAlg.LAPACK.sysv!
Base.LinAlg.LAPACK.sytrf!
Base.LinAlg.LAPACK.sytri!
Base.LinAlg.LAPACK.sytrs!
Base.LinAlg.LAPACK.hesv!
Base.LinAlg.LAPACK.hetrf!
Base.LinAlg.LAPACK.hetri!
Base.LinAlg.LAPACK.hetrs!
Base.LinAlg.LAPACK.syev!
Base.LinAlg.LAPACK.syevr!
Base.LinAlg.LAPACK.sygvd!
Base.LinAlg.LAPACK.bdsqr!
Base.LinAlg.LAPACK.bdsdc!
Base.LinAlg.LAPACK.gecon!
Base.LinAlg.LAPACK.gehrd!
Base.LinAlg.LAPACK.orghr!
Base.LinAlg.LAPACK.gees!
Base.LinAlg.LAPACK.gges!
Base.LinAlg.LAPACK.trexc!
Base.LinAlg.LAPACK.trsen!
Base.LinAlg.LAPACK.tgsen!
Base.LinAlg.LAPACK.trsyl!
```
