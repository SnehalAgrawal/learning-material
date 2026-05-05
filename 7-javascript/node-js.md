# Node.js: Streams, Clustering, & Event-Driven I/O

### 1. Overview
Node.js is a runtime built on V8 that uses an Event-Driven, non-blocking I/O model. For senior engineers, Node.js is about managing system resources efficiently, handling massive amounts of data through streams, and scaling across multiple CPU cores via clustering.

### 2. Key Concepts
*   **Event-Driven Architecture**: Node.js relies on an `EventEmitter` to signal that an operation is complete, allowing the main thread to remain free for other tasks.
*   **Streams**: Handling data in small "Chunks" instead of loading the entire file/buffer into RAM.
    *   *Readable*: Can be read from (e.g., `fs.createReadStream`).
    *   *Writable*: Can be written to (e.g., `res` object in an HTTP server).
    *   *Duplex/Transform*: Both read and write (e.g., Gzip compression).
*   **Buffer**: Raw binary data outside the V8 heap, used for handling binary streams (TCP, Files).
*   **Clustering / Worker Threads**:
    *   *Clustering*: Running multiple copies of the entire app, each on its own CPU core, to share the load.
    *   *Worker Threads*: Running multiple threads within the *same* process for CPU-heavy tasks.

### 3. Real-World Usage
*   **Large File Uploads/Downloads**: Using `.pipe()` to stream a 5GB file from a user to S3 without it ever using more than 50MB of server RAM.
*   **Socket.io / WebSockets**: Leveraging the Event-Driven model to handle 10,000+ persistent, bi-directional connections on a single server.
*   **Data Pipelines**: Using `Transform` streams to parse a 10 million row CSV, transform the fields, and write to a DB in real-time.
*   **Multicore Utilization**: Using the `cluster` module to spawn one worker process for every physical CPU core on a standard AWS `m5.large` instance.

### 4. Tradeoffs
*   **Non-Blocking vs. Throughput**: Node.js excellent for I/O but a single CPU-heavy function (like a large `JSON.parse` or loop) will block *every* other client connected to that process.
*   **Streams vs. Simple FS**: Streams are much more memory-efficient but require significantly more complex error handling (`.on('error')`, `destroy()`, `pipeline`).
*   **Clustering vs. Memory**: Clustering is simple but every process has its own memory heap. If you have 8 cores and your app uses 500MB, you need 4GB of RAM just for the app instances.

### 5. When NOT to Use
*   **CPU-Heavy Backend Architecture**: If your backend is 80% complex math, encryption, or ML inference, Node.js is the wrong choice. Use Go, Rust, or Java.
*   **Shared State Environments**: Since Node clusters don't share memory, you cannot use global variables to track "logged in users" across processes. You MUST use a shared store like Redis.

### 6. Interview Focus
*   **Stream Mechanics**: "How would you read a 10GB file on a machine with 2GB of RAM using Node.js?"
    ```javascript
    const fs = require('fs');
    const readable = fs.createReadStream('large-file.txt', { highWaterMark: 64 * 1024 }); // 64KB chunks
    let totalBytes = 0;
    
    readable.on('data', (chunk) => {
        totalBytes += chunk.length;
        // Process chunk without loading entire file
        // For example, write to another stream or process in memory
        console.log(`Read ${chunk.length} bytes, total: ${totalBytes}`);
    });
    
    readable.on('error', (err) => {
        console.error('Error reading file:', err);
    });
    ```
*   **The Cluster Module**: "Explain why `cluster.fork()` is different from multi-threading in other languages."
    * **Answer**: In Node.js, `cluster.fork()` creates a new *process* (child process), not a thread. This means each worker has its own memory space and event loop. This is different from traditional multi-threading where multiple threads share the same memory space within a single process.
    * **Example**:
    ```javascript
    const cluster = require('cluster');
    const http = require('http');
    const numCPUs = require('os').cpus().length;
    
    if (cluster.isPrimary) {
        console.log(`Primary ${process.pid} is running`);
        
        // Fork workers
        for (let i = 0; i < numCPUs; i++) {
            cluster.fork();
        }
        
        cluster.on('exit', (worker, code, signal) => {
            console.log(`Worker ${worker.process.pid} died`);
            cluster.fork(); // Restart worker
        });
    } else {
        // Workers share TCP connections
        http.createServer((req, res) => {
            res.writeHead(200);
            res.end('hello\n');
        }).listen(8000);
        
        console.log(`Worker ${process.pid} started`);
    }
    ```
*   **Backpressure in Streams**: "What happens if a Readable stream is faster than a Writable stream? How does Node.js handle this automatically?" (.pipe() vs manual .write()).
    * **Backpressure**: Backpressure occurs when a source of data (Readable stream) produces data faster than the destination (Writable stream) can consume it. This can lead to memory overflow as data accumulates in buffers.
    * **Node.js Handling**: Node.js handles backpressure through the `drain` event. When a Writable stream's buffer is full, it emits a `drain` event. The producer (Readable stream) should pause emitting data until it receives this event, indicating that the consumer is ready for more.
    * **Example**:
    ```javascript
    const fs = require('fs');
    const readable = fs.createReadStream('large-file.txt');
    const writable = fs.createWriteStream('output.txt');
    
    readable.on('data', (chunk) => {
        // write() returns false if the buffer is full
        const stillWriting = writable.write(chunk);
        if (!stillWriting) {
            // Pause reading until consumer is ready
            readable.pause();
        }
    });
    
    writable.on('drain', () => {
        // Resume reading when buffer has space
        readable.resume();
    });
    
    readable.on('end', () => {
        writable.end();
    });
    ```
    * **The `pipeline` utility**: The `stream.pipeline` function automatically handles backpressure and error propagation for you, making it the recommended way to pipe streams in modern Node.js.
    ```javascript
    const { pipeline } = require('stream');
    const fs = require('fs');
    
    pipeline(
        fs.createReadStream('large-file.txt'),
        fs.createWriteStream('output.txt'),
        (err) => {
            if (err) {
                console.error('Pipeline failed:', err);
            } else {
                console.log('Pipeline succeeded');
            }
        }
    );
    ```

### 7. Common Mistakes
*   **Loading Full Files into Memory**: Using `fs.readFile` for large production files instead of `fs.createReadStream`.
*   **Blocking the Event Loop**: Running a `while` loop that takes 2 seconds, which makes the server unresponsive to all other incoming HTTP requests during that time.
*   **Zomibe Processes**: Spawning child processes or worker threads and failing to kill them when no longer needed, leading to massive memory leaks.
