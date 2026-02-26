# CS gRPC Service - RPC with C# (Client/Server)

This repository shows a simple **Remote Procedure Call (RPC)** communication using **gRPC** in C#:

- `csgrpcserver`: Provides RPC methods
- `csgrpcclient`: Calls the methods over the network
- `Protos/SimpleCommunication.proto`: Contract (interface) between client and server

## Project Goal

The project demonstrates how to design function calls between two processes/applications so they behave like local method calls.

Instead of just sending bytes over a socket connection, gRPC provides:

- clearly defined interfaces,
- serialized data types,
- and strongly typed calls between client and server.

---

## What is RPC - explained simply

**RPC (Remote Procedure Call)** means:

> A function is called in another process (often on another machine) as if it were local.

Technically, the call is a network request, but from a developer perspective it looks similar to a normal method call:

1. The client calls a method (e.g. `GetMessage(...)`)
2. The request is serialized and sent over the network
3. The server executes the method
4. The response is serialized and sent back
5. The client receives the result

### Why gRPC is often chosen for this

- based on HTTP/2
- uses Protocol Buffers (compact, fast)
- strongly typed contracts via `.proto`
- automatic code generation for client/server stubs

---

## When is RPC useful?

RPC is especially useful when:

1. **Application logic should run separately** (client and service can be deployed separately)
2. **Multiple clients** (desktop, web, backend) should use the same service
3. **Scalability** matters (server can be scaled horizontally on its own)
4. **Clear API contracts** are required (versioning via protos)
5. **Cross-language communication** is relevant (e.g. C#, Go, Java, Python)
6. **Internal service communication** is needed in distributed systems

Typical use cases:

- microservices
- central business services
- internal enterprise APIs
- high-performance service-to-service communication

---

## Comparison: RPC vs. raw TCP vs. local call/return (DLL or external process)

## 1) RPC (e.g. gRPC)

### Advantages

- **Method-level abstraction** instead of manually handling a byte protocol
- **Strongly typed** through `.proto` contracts
- **Automatic code generation** reduces boilerplate
- **Good interoperability** across languages/platforms
- **Scalable and deployable** as its own service
- **Uniform contract** for all clients

### Disadvantages

- **Network dependency** (latency, timeouts, outages)
- **More complex operating model** than pure in-process calls
- **Versioning discipline required** (avoid breaking changes)
- **Distributed debugging** is often more complex than local code

---

## 2) Raw TCP (sockets, custom protocol)

### Advantages

- **Maximum control** over protocol and data flow
- **Very flexible** for special cases
- Potentially **very efficient** if implemented well

### Disadvantages

- **High development effort** (framing, serialization, error cases)
- **More error-prone** due to custom protocol design
- **Poor maintainability** if the specification is not maintained strictly
- No standardized stub/contract workflow like gRPC

### When does it make sense?

- When a proprietary low-level protocol is required
- When extremely specific transport requirements exist

---

## 3) Normal local call/return (referencing a DLL)

This means: The C# program references a DLL and calls functions directly in the same process.

### Advantages

- **Very fast** (no network, no external serialization)
- **Simple debugging** in the same process
- **Low runtime complexity**

### Disadvantages

- **Tight coupling** between the calling app and the library
- **Deployment coupling** (versions must match)
- **Weaker isolation**: an error/crash in the DLL can affect the whole process
- **Limited scaling** to the local process/host

### When does it make sense?

- For local, high-performance function libraries
- When no process or machine separation is required

---

## 4) External programs via call-and-return (start process, collect result)

This means: The main program starts a separate tool/program, passes parameters, and waits for a result (exit code, output, file).

### Advantages

- **Strong isolation** between the main process and the tool
- Can reuse large existing tools
- Errors in the tool usually affect the host process less directly

### Disadvantages

- **Startup cost per call** (process startup is expensive)
- **Cumbersome data transfer** (arguments, stdout, files, pipes)
- **Limited interactivity** for many small calls
- **Operations/monitoring** become complex with many tool calls

### When does it make sense?

- For infrequent, heavy batch tasks
- When existing CLI tools must be integrated

---

## Short conclusion: Which technique when?

- **gRPC/RPC**: Best choice for stable, typed communication between separate applications/services.
- **Raw TCP**: Only when you deliberately need your own protocol and full low-level control.
- **Local DLL**: Best choice for maximum local performance and tight integration in the same process.
- **External process call**: Good for loosely coupled tool integration and isolated batch steps with higher runtime overhead.

---

## Decision guide (practical)

Ask yourself for new features:

1. Does the function need to be usable **over the network/between processes**? -> more likely **RPC**
2. Must it run **extremely fast in the same process**? -> more likely **DLL**
3. Does an existing large tool only need to be used occasionally? -> more likely **external process**
4. Do you need a **custom binary protocol with special requirements**? -> possibly **direct TCP**

---

## Project Structure

- `csgrpcclient/CSgRPCClient/Program.cs`: Example client
- `csgrpcserver/CSgRPCServer/Program.cs`: Example server
- `Protos/SimpleCommunication.proto`: RPC contract

---

## Practical note on error handling in RPC

Unlike a local function call, RPC should always consider:

- timeouts/deadlines
- retries (controlled)
- idempotency
- clean error codes
- telemetry/logging for distributed error analysis

These points in particular are a core difference between a "local call" and a "remote call".

---

## One-sentence summary

RPC is ideal when functions should be provided **as a service**; local DLL calls are ideal for **in-process performance**, and external process calls are suited for **loosely coupled tool integration** with higher runtime overhead.
