# nodejs-scaling-applications

Project with techniques that can help you scale your Node.js applications

## skills

* Cloning
* The scale cube
* Scaling the x-axis, z-axis, and y-axis
* Forking processes
* Implementing a database instance
* Database scaling
* Setting up horizontal partitioning
* Decomposing your app into microservices

## Cloning

### With Fork

The `child_process.fork()` method is a special case of `child_process.spawn()` used specifically to spawn new Node.js processes.
Like `child_process.spawn()`, a `ChildProcess` object is returned. The returned `ChildProcess` will have an additional communication channel built-in that allows messages to be passed back and forth between the parent and child.

Keep in mind that spawned Node.js child processes are independent of the parent with exception of the IPC communication channel that is established between the two. Each process has its own memory, with their own V8 instances. Because of the additional resource allocations required, spawning a large number of child Node.js processes is not recommended.

By default, `child_process.fork()` will spawn new Node.js instances using the `process.execPath` of the parent process. The execPath property in the options object allows for an alternative execution path to be used.

Node.js processes launched with a custom execPath will communicate with the parent process using the file descriptor (fd) identified using the environment variable NODE_CHANNEL_FD on the child process.

Unlike the fork POSIX system call, `child_process.fork()` does not clone the current process.

[Files example](https://github.com/laissonsilveira/nodejs-scaling-applications/blob/main/cloning/fork)

### With Cluster

A single instance of Node.js runs in a single thread. To take advantage of multi-core systems, the user will sometimes want to launch a cluster of Node.js processes to handle the load.

The cluster module allows easy creation of child processes that all share server ports.

[Files example](https://github.com/laissonsilveira/nodejs-scaling-applications/blob/main/cloning/cluster)


#### How it works

The worker processes are spawned using the `child_process.fork()` method, so that they can communicate with the parent via IPC and pass server handles back and forth.

The cluster module supports two methods of distributing incoming connections.

The first one (and the default one on all platforms except Windows), is the round-robin approach, where the primary process listens on a port, accepts new connections and distributes them across the workers in a round-robin fashion, with some built-in smarts to avoid overloading a worker process.

The second approach is where the primary process creates the listen socket and sends it to interested workers. The workers then accept incoming connections directly.

The second approach should, in theory, give the best performance. In practice however, distribution tends to be very unbalanced due to operating system scheduler vagaries. Loads have been observed where over 70% of all connections ended up in just two processes, out of a total of eight.

Because `server.listen()` hands off most of the work to the primary process, there are three cases where the behavior between a normal Node.js process and a cluster worker differs:

1. `server.listen({fd: 7})` Because the message is passed to the primary, file descriptor 7 in the parent will be listened on, and the handle passed to the worker, rather than listening to the worker's idea of what the number 7 file descriptor references.
2. `server.listen(handle)` Listening on handles explicitly will cause the worker to use the supplied handle, rather than talk to the primary process.
3. `server.listen(0)` Normally, this will cause servers to listen on a random port. However, in a cluster, each worker will receive the same "random" port each time they do `listen(0)`. In essence, the port is random the first time, but predictable thereafter. To listen on a unique port, generate a port number based on the cluster worker ID.
Node.js does not provide routing logic. It is, therefore important to design an application such that it does not rely too heavily on in-memory data objects for things like sessions and login.

Because workers are all separate processes, they can be killed or re-spawned depending on a program's needs, without affecting other workers. As long as there are some workers still alive, the server will continue to accept connections. If no workers are alive, existing connections will be dropped and new connections will be refused. Node.js does not automatically manage the number of workers, however. It is the application's responsibility to manage the worker pool based on its own needs.

Although a primary use case for the `cluster` module is networking, it can also be used for other use cases requiring worker processes.

### With PM2

Execute `pm2 start app.js -i <number of instance>`

## Database Scaling

## Incorporating a database

[Files example](https://github.com/laissonsilveira/nodejs-scaling-applications/blob/main/database-scaling/incorporating)

## Horizontal partitioning

A partition is a division of a logical database or its constituent elements into distinct independent parts. Database partitioning is normally done for manageability, performance or availability reasons, or for load balancing. It is popular in distributed database management systems, where each partition may be spread over multiple nodes, with users at the node performing local transactions on the partition. This increases performance for sites that have regular transactions involving certain views of data, whilst maintaining availability and security.

![z-axis](https://github.com/laissonsilveira/nodejs-scaling-applications/blob/main/database-scaling/horizontal/z-axis.png)

[Files example](https://github.com/laissonsilveira/nodejs-scaling-applications/blob/main/database-scaling/horizontal)

### When to use

* Too much data
* More write operations than the server can handle
* Slow performance
* Often cheaper to host shards than one database

### How to use

* MongoDB (mongos)
* Postres (Postgres-XL)
* MySQL (Fabric, Router)
* Elasticsearch (Lucene)
* Redis (Redis Cluster, Amazon ElastiCache for Redis)

## Microservices

This project have 3 services (shows, reservations and orchestration of the 2 others services, named api)

[Project example](https://github.com/laissonsilveira/nodejs-scaling-applications/blob/main/microservices)