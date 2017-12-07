# Tareas y Computación Paralela

## Tareas

```@docs
Core.Task
Base.current_task
Base.istaskdone
Base.istaskstarted
Base.yield
Base.yieldto
Base.task_local_storage(::Any)
Base.task_local_storage(::Any, ::Any)
Base.task_local_storage(::Function, ::Any, ::Any)
Base.Condition
Base.notify
Base.schedule
Base.@schedule
Base.@task
Base.sleep
Base.Channel
Base.put!(::Channel, ::Any)
Base.take!(::Channel)
Base.isready(::Channel)
Base.fetch(::Channel)
Base.close(::Channel)
Base.bind(c::Channel, task::Task)
Base.asyncmap
Base.asyncmap!
```

## Soporte General a la Computación Paralela

```@docs
Base.addprocs
Base.nprocs
Base.nworkers
Base.procs()
Base.procs(::Integer)
Base.workers
Base.rmprocs
Base.interrupt
Base.myid
Base.pmap
Base.RemoteException
Base.Future
Base.RemoteChannel(::Integer)
Base.RemoteChannel(::Function, ::Integer)
Base.wait
Base.fetch(::Any)
Base.remotecall(::Any, ::Integer, ::Any...)
Base.remotecall_wait(::Any, ::Integer, ::Any...)
Base.remotecall_fetch(::Any, ::Integer, ::Any...)
Base.remote_do(::Any, ::Integer, ::Any...)
Base.put!(::RemoteChannel, ::Any...)
Base.put!(::Future, ::Any)
Base.take!(::RemoteChannel, ::Any...)
Base.isready(::RemoteChannel, ::Any...)
Base.isready(::Future)
Base.WorkerPool
Base.CachingPool
Base.default_worker_pool
Base.clear!(::CachingPool)
Base.remote
Base.remotecall(::Any, ::Base.Distributed.AbstractWorkerPool, ::Any...)
Base.remotecall_wait(::Any, ::Base.Distributed.AbstractWorkerPool, ::Any...)
Base.remotecall_fetch(::Any, ::Base.Distributed.AbstractWorkerPool, ::Any...)
Base.remote_do(::Any, ::Base.Distributed.AbstractWorkerPool, ::Any...)
Base.timedwait
Base.@spawn
Base.@spawnat
Base.@fetch
Base.@fetchfrom
Base.@async
Base.@sync
Base.@parallel
Base.@everywhere
Base.clear!(::Any, ::Any; ::Any)
Base.remoteref_id
Base.channel_from_id
Base.worker_id_from_socket
Base.cluster_cookie()
Base.cluster_cookie(::Any)
```

## Arrays Compartidos

```@docs
Base.SharedArray
Base.procs(::SharedArray)
Base.sdata
Base.indexpids
Base.localindexes
```

## Multi-Threading

Este interfaz experimental soporta las capacidades multi-hilo de Julia. Los tipos y funciones descritos aquí pueden cambiar en el futuro (y probablemente lo harán).

```@docs
Base.Threads.threadid
Base.Threads.nthreads
Base.Threads.@threads
Base.Threads.Atomic
Base.Threads.atomic_cas!
Base.Threads.atomic_xchg!
Base.Threads.atomic_add!
Base.Threads.atomic_sub!
Base.Threads.atomic_and!
Base.Threads.atomic_nand!
Base.Threads.atomic_or!
Base.Threads.atomic_xor!
Base.Threads.atomic_max!
Base.Threads.atomic_min!
Base.Threads.atomic_fence
```

## ccall using a threadpool (Experimental)

```@docs
Base.@threadcall
```

## Primitivas de Sincronización

```@docs
Base.Threads.AbstractLock
Base.lock
Base.unlock
Base.trylock
Base.islocked
Base.ReentrantLock
Base.Threads.Mutex
Base.Threads.SpinLock
Base.Threads.RecursiveSpinLock
Base.Semaphore
Base.acquire
Base.release
```

## Interfaz de Administración de Cluster

Esta interfaz proporciona un mecanismo para lanzar y gestionar *workers* Julia sobre diferentes entornos cluster. Hay dos tipos de administrafores presentes en Base: `LocalManager`, para lanzar *workers* adicionales sobre el mismo host, y `SSHManager`, para lanzarlos sobre hosts remotos vía `ssh`. Para conectar y transportar mensajes entre procesos se usan los sockets TCP/IP. Es posible que los administradores de clusters proporcionen un transporte diferente.

```@docs
Base.launch
Base.manage
Base.kill(::ClusterManager, ::Int, ::WorkerConfig)
Base.connect(::ClusterManager, ::Int, ::WorkerConfig)
Base.init_worker
Base.start_worker
Base.process_messages
```
