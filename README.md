# Gunrock Concurrent Web Server

A high-performance, multi-threaded web server built in C++ designed to handle
concurrent HTTP requests efficiently. This server implements a thread pool
architecture with configurable worker threads and connection buffering to
maximize throughput while maintaining HTTP 1.1 compliance.

The `http_parse.c` file was written by [Ryan Dahl](https://github.com/ry) and is
licensed under the BSD license by Ryan. The server architecture is inspired by
modern web server design patterns and the excellent [OSTEP](http://ostep.org)
textbook on operating systems concepts.

# Quickstart

To compile and run the server, open a terminal and execute the following commands:

```bash
$ make
$ ./gunrock_web
```

The server can be tested using a web browser or command-line tools:

```bash
$ # get a basic HTML file
$ curl http://localhost:8080/hello_world.html
$ # get a basic HTML file with more detailed information
$ curl -v http://localhost:8080/hello_world.html
$ # head a basic HTML file
$ curl --head http://localhost:8080/hello_world.html
$ # test out a file that does not exist (404 status code)
$ curl -v http://localhost:8080/hello_world2.html
$ # test out a POST, which isn't supported currently (405 status code)
$ curl -v -X POST http://localhost:8080/hello_world.html
```

A full demo website is included for testing at: `http://localhost:8080/bootstrap.html`

# Features

This web server implements several key features for high-performance concurrent request handling:

- **Multi-threaded Architecture**: Thread pool with configurable worker threads
- **Connection Buffering**: Fixed-size buffer for managing incoming connections
- **HTTP 1.1 Compliance**: Full support for GET and HEAD requests
- **Static File Serving**: Efficient serving of HTML, CSS, JS, and other static content
- **FIFO Scheduling**: First-in-first-out request processing
- **Security Features**: Path traversal protection and connection management

## Architecture Goals

I designed the server with the following principles in mind:

- High concurrency through efficient thread management
- Scalable architecture that can handle varying loads
- Clean separation of concerns between networking and business logic
- Extensible design for adding new request handlers

My implementation draws from established operating systems concepts documented in:

- [Intro to threads](http://pages.cs.wisc.edu/~remzi/OSTEP/threads-intro.pdf)
- [Using locks](https://pages.cs.wisc.edu/~remzi/OSTEP/threads-locks.pdf)
- [Producer-consumer relationships](http://pages.cs.wisc.edu/~remzi/OSTEP/threads-cv.pdf)
- [Server concurrency architecture](http://pages.cs.wisc.edu/~remzi/OSTEP/threads-events.pdf)

# HTTP Background

This section provides a brief overview of how web servers work and the HTTP
protocol (version 1.1) used for communication. Although web browsers and
servers have [evolved significantly over the
years](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP),
the fundamental concepts remain the same and provide a solid foundation for
modern web technologies. My implementation abstracts away many of the
low-level networking details while keeping the core HTTP handling code
readable and maintainable.

Classic web browsers and web servers interact using a text-based protocol
called **HTTP** (**Hypertext Transfer Protocol**). A web browser opens a
connection to a web server and requests some content with HTTP. The web server
responds with the requested content and closes the connection. The browser
reads the content and displays it on the screen.

HTTP is built on top of the **TCP/IP** protocol suite provided by the
operating system. Together, TCP and IP ensure that messages are routed to
their correct destination, get from source to destination reliably in the face
of failure, and do not overly congest the network by sending too many messages
at once, among other features. For more information about networks, see [this free book](https://book.systemsapproach.org).

Each piece of content on the web server is associated with a file in the
server's file system. The simplest is _static_ content, in which a client
sends a request just to read a specific file from the server. Slightly more
complex is _dynamic_ content, in which a client requests that an executable
file be run on the web server and its output returned to the client.
Each file has a unique name known as a **URL** (**Universal Resource
Locator**).

As a simple example, let's say the client browser wants to fetch static
content (i.e., just some file) from a web server running on some machine. The
client might then type in the following URL to the browser:
`http://www.cs.wisc.edu/index.html`. This URL identifies that the HTTP
protocol is to be used, and that an HTML file in the root directory (`/`) of
the web server called `index.html` on the host machine `www.cs.wisc.edu`
should be fetched.

The web server is not just uniquely identified by which machine it is running
on but also the **port** it is listening for connections upon. Ports are a
communication abstraction that allow multiple (possibly independent) network
communications to happen concurrently upon a machine; for example, the web
server might be receiving an HTTP request upon port 80 while a mail server is
sending email out using port 25. By default, web servers are expected to run
on port 80 (the well-known HTTP port number), but sometimes (as in my
implementation), a different port number will be used. To fetch a file from a web
server running at a different port number (say 8000), specify the port number
directly in the URL, e.g., `http://www.cs.wisc.edu:8000/index.html`.

# The HTTP Request

When a client (e.g., a browser) wants to fetch a file from a machine, the
process starts by sending a message. The _request contents_ and subsequent
_reply contents_ are specified precisely by the HTTP protocol.

HTTP requests consist of a request line, followed by zero or more request
headers, and finally an empty text line. A request line has the form:
`method uri version`. The `method` is usually `GET`, which tells the web
server that the client wants to read the specified file; however, other
methods exist (e.g., `POST`). The `uri` is the file name, and perhaps optional
arguments (in the case of dynamic content). The `version` indicates
the version of the HTTP protocol that the web client is using (e.g.,
HTTP/1.1).

The HTTP response (from the server to the browser) consists of a response
line, zero or more response headers, an empty text line, and the response
body. A response line has the form version `status message`. The `status` is
a three-digit positive integer that indicates the state of the request; common
states include `200` for `OK`, `403` for `Forbidden` (the client can't access
that file), and `404` for `File Not Found`. Important header lines are
`Content-Type`, which specifies the type of content in the response body
(e.g., HTML or gif), and `Content-Length`, which indicates the file size in
bytes.

Understanding these HTTP fundamentals helps when working with the server code,
though my implementation handles most of the protocol details automatically.
The modular design allows for easy extension of HTTP functionality if needed.

# Building and Running

The web server can be compiled and run from this repository. Simply type `make`
to build the project. Use `make clean` to remove object files and executables
for a clean build.

The web server requires a port number to listen on. Ports below 1024 are
_reserved_ (see the list
[here](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml)),
so port numbers greater than 1023 should be used to avoid conflicts. The maximum
port number is 65535. On shared machines, port conflicts may occur, requiring
a different port number if the server fails to bind.

Connections to the server use the same port specified at startup.
For example, if running on `localhost` with port 8003:
`http://localhost:8003/favorite.html`

I designed the server to be lightweight and efficient, focusing on core
functionality. It handles `GET` and `HEAD` requests and serves various content
types. My implementation prioritizes performance and concurrency over extensive
feature sets, making it ideal for high-throughput static content serving.

# Multi-threaded Architecture

I implemented a sophisticated multi-threaded architecture designed to
handle concurrent requests efficiently. My design addresses the fundamental
limitation of single-threaded servers, which can only process one HTTP request
at a time, causing all other clients to wait.

## Thread Pool Design

Single-threaded web servers suffer from a fundamental performance bottleneck:
only one HTTP request can be serviced at a time. This becomes particularly
problematic when handling long-running requests or when files need to be read
from disk. My multi-threaded implementation solves this by allowing multiple
requests to be processed concurrently.

My implementation uses two main approaches for thread management:

**Per-Request Threading**: Spawning a new thread for each HTTP request allows
the OS to schedule threads independently. This approach ensures that short
requests don't wait for long ones, and blocked threads (waiting for disk I/O)
don't prevent other requests from being processed. However, this method incurs
significant overhead from thread creation and destruction.

**Thread Pool Architecture**: The preferred approach uses a fixed-size pool of
worker threads created at server startup. This design eliminates thread creation
overhead while maintaining concurrency. Worker threads wait for requests in a
shared buffer, and requests are queued when more arrive than can be immediately
processed.

## Implementation Details

The server architecture consists of:

- **Main Thread**: Accepts incoming connections and places them in a fixed-size
  buffer without reading from the connection
- **Worker Threads**: Process requests from the buffer, handling the complete
  request-response cycle
- **Connection Buffer**: A synchronized queue that manages incoming connections
- **Scheduling Policy**: Determines which request each worker thread processes

The main thread and worker threads operate in a producer-consumer relationship,
requiring careful synchronization of the shared buffer. My implementation uses
condition variables to ensure proper blocking behavior - the main thread blocks
when the buffer is full, and worker threads block when the buffer is empty.

## Scheduling Policies

I implemented a scheduling policy to determine which HTTP request each
worker thread should handle. While the OS controls which thread actually runs
at any given time, my server's scheduling policy determines the order in which
requests are processed from the buffer.

**First-in-First-out (FIFO)**: My implemented scheduling policy processes
requests in the order they arrive. When a worker thread becomes available, it
handles the oldest request in the buffer. Note that requests may not complete
in strict FIFO order due to OS thread scheduling, but the processing order
follows the arrival sequence.

## Security Considerations

Running a networked server requires careful attention to security. I implemented
several security measures to protect against common vulnerabilities:

**Connection Management**: I designed the server for testing and development
purposes. Avoid leaving it running in production environments without proper
security hardening.

**Path Traversal Protection**: My server constrains file requests to stay
within the sub-tree rooted at the server's working directory. This prevents
access to files outside the intended directory structure. My implementation
rejects any pathname containing `..` to avoid directory traversal attacks.

**Additional Security Measures**: For production use, consider implementing
more sophisticated security measures such as `chroot()` or containerization
to further isolate the server process.

# Configuration and Usage

## Command Line Arguments

The web server accepts the following command line arguments:

```bash
$ ./gunrock_web [-p port] [-t threads] [-b buffers]
```

**Arguments:**

- **-p port**: Port number for the web server to listen on (default: 8080)
- **-t threads**: Number of worker threads to create (default: 1)
- **-b buffers**: Number of connection buffers for queuing requests (default: 1)

**Example:**

```bash
$ ./gunrock_web -p 8003 -t 8 -b 16
```

This configuration runs the server on port 8003 with 8 worker threads and 16
connection buffers, allowing up to 16 concurrent connections to be queued
while 8 are actively being processed.

## Architecture Overview

I designed the server with extensibility in mind, making it easy to add new
request handlers. The `FileService.cpp` implements a simple service that reads
files from the `static` directory and serves them as HTML. New handlers can be
added by creating a service class inheriting from `HttpService`, adding the
source file to the `Makefile`, and registering the service with the main
`gunrock.cpp` file.

My main `gunrock.cpp` logic uses path prefix matching to route requests to
the appropriate service. Services handle the complete request-response cycle,
setting response bodies and status codes as needed.

## Threading Library

I used a custom threading library called `dthread` that provides
logging capabilities for thread operations. This library includes the following
key functions:

- `dthread_create()` - Create new threads
- `dthread_detach()` - Detach threads
- `dthread_mutex_lock()` / `dthread_mutex_unlock()` - Mutex operations
- `dthread_cond_wait()` / `dthread_cond_signal()` - Condition variable operations

Initialize mutexes and condition variables using the standard pthread macros:
`PTHREAD_MUTEX_INITIALIZER` and `PTHREAD_COND_INITIALIZER`.

## Core Components

The server consists of several key components that work together to provide
concurrent request handling:

- **gunrock.cpp** - Main server logic and request handling
- **FileService.cpp** - File serving implementation with static content handling
- **dthread** - Custom threading library with logging capabilities
- **HTTP** - High-level HTTP object interfacing with the parser
- **http_parser** - HTTP protocol parsing state machine
- **HTTPRequest** - Request object populated by the framework
- **HTTPResponse** - Response object filled by services
- **HttpUtils** - Utility functions for HTTP operations
- **MyServerSocket** - Server socket abstraction for accepting connections
- **MySocket** - Socket abstraction for reading requests and writing responses

## Testing and Development

My server includes comprehensive testing capabilities to ensure reliable
operation under various load conditions. The threading library provides
detailed logging of thread operations, mutex usage, and condition variable
interactions to aid in debugging and performance analysis.

## Development Notes

My server uses a custom threading library with specific requirements:

- `dthread` functions are used for all threading operations, providing
  logging capabilities and testing framework compatibility
- `set_log_file()` must be called before using any `dthread` functions
- The `accept` thread is designed to not block until after accepting a request
- `sync_print` functions are used for testing and monitoring:

```c++
sync_print("waiting_to_accept", "");
client = server->accept();
sync_print("client_accepted", "");
```
