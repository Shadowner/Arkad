# 🎭 Arkad - Actor Runtime for Rust

> **A modern and ergonomic actor system for Rust, designed to simplify concurrent application development, with first-class support for proxies and distributed systems.**

## 🎯 Project Objective

Arkad was born out of a frustration: developing concurrent systems in Rust often requires writing hundreds of lines of boilerplate code to manage channels, synchronization, and component lifecycles.

**Our mission**: To provide an actor abstraction that is:
- **Simple**: 10x less code for the same functionality
- **Type-safe**: Leverage Rust's type system to the fullest
- **Production-ready**: Integrated supervision, metrics, and debugging
- **Performant**: Zero-copy and minimal overhead
- **Extensible**: Modular architecture with reusable patterns

## 🚀 Why Arkad ?

### The Problem

Let's take the example of a TCP proxy (such as a Minecraft proxy). With the traditional approach:

```rust
// Traditional code
let (tx1, rx1) = mpsc::channel(100);
let (tx2, rx2) = mpsc::channel(100);
let shutdown = Arc::new(AtomicBool::new(false));

// Complex manual thread management
tokio::spawn(async move {
    while !shutdown.load(Ordering::SeqCst) {
        tokio::select! {
            Some(msg) = rx1.recv() => { /* ... */ }
            // ... lots of code ...
        }
    }
});
// ... repeated for each component ...
```

### The Arkad Solution (What I aim for)

```rust
// 🎉 With Arkad
use arkad::prelude::*;

let system = ActorSystem::new("my-proxy");
let client = system.spawn(ClientActor::new(tcp_stream)).await;
let server = system.spawn(ServerActor::new(config)).await;
system.tunnel(client, server); // Automatic bidirectional tunnel!
```

## ✨ Key Features

### 🏗️ Core
- **Typed actors** with automatic lifecycle
- **Asynchronous messages** with tell/ask patterns
- **Supervision** with configurable restart strategies
- **Rich context** for each actor
- **Graceful shutdown** automatically propagated

### 🎨 Integrated Patterns
- **Router**: Load distribution (round-robin, hash, broadcast)
- **Pipeline**: Actor chaining with transformations
- **EventBus**: Decoupled pub/sub communication
- **Scheduler**: Delayed and periodic messages

### 🔌 Proxy Extensions (arkad-proxy)
- **Bidirectional tunnels** between actors
- **Hot-swapping**: Replace actors without interruption
- **Intelligent connection pooling**
- **Filters chain**: Intercept and modify messages
- **Metrics**: Latency, throughput, errors

### 🧪 Testing
- **TestKit**: Deterministic test framework
- **TestProbe**: Test actors for assertions
- **Time control**: Advance time in tests
- **Debugging**: Structured tracing of all messages

## 📚 API Goals

### Hello World

```rust
use arkad::prelude::*;

struct HelloActor;

impl Actor for HelloActor {
    type Msg = String;
    
    async fn handle(&mut self, msg: String, _ctx: &mut Context<Self>) -> Result<(), ActorError> {
        println!("Received: {}", msg);
        Ok(())
    }
}

#[tokio::main]
async fn main() {
    let system = ActorSystem::new("hello-system");
    let hello = system.spawn(HelloActor).await;
    
    hello.send("Hello, Arkad!".to_string()).unwrap();
    
    system.wait_for_termination().await;
}
```

### TCP Proxy with Hot-Swap

```rust
use arkad::prelude::*;
use arkad_proxy::prelude::*;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let system = ActorSystem::new("game-proxy");
    
    // Connected client
    let client = system.spawn(
        ProxyActor::new(client_stream)
            .with_filter(ChatFilter::new())
            .with_filter(AntiCheatFilter::new())
    ).await;
    
    // Initial server
    let server1 = system.spawn(
        ProxyActor::connect("server1.example.com:25565").await?
    ).await;
    
    // Bidirectional tunnel
    system.tunnel(client, server1.clone());
    
    // Hot-swap to another server (without disconnecting the client!)
    let server2 = system.spawn(
        ProxyActor::connect("server2.example.com:25565").await?
    ).await;
    
    system.replace_tunnel_endpoint(client, server1, server2).await?;
    
    system.wait_for_termination().await;
    Ok(())
}
```

### Processing Pipeline

```rust
use arkad::prelude::*;

let pipeline = system.pipeline()
    .source(TcpListener::new("0.0.0.0:8080"))
    .filter(|packet| packet.is_valid())
    .map(|packet| decompress(packet))
    .route_by(|packet| packet.session_id)
    .sink(DatabaseWriter::new(db_pool));

pipeline.start().await;
```



## 🎯 Use Cases

Arkad is perfect for:
- **Proxies**: TCP, HTTP, WebSocket with filtering and transformation
- **Game servers**: Concurrent state management with thousands of players
- **IoT Gateways**: Sensor data aggregation
- **Microservices**: Resilient inter-service communication
- **Chat/Messaging**: Real-time systems with pub/sub
- **Load Balancers**: Intelligent traffic distribution

## 🚧 Roadmap

- [ ] **v0.1** - MVP: Core, Basic Patterns, Testing
- [ ] **v0.2** - Persistence: Event sourcing, Snapshots
- [ ] **v0.3** - Clustering: Distributed actors, Gossip protocol
- [ ] **v0.4** - Streams: Backpressure, Flow control
- [ ] **v0.5** - WASM: WebAssembly support

## 🙏 Source of inspiration

Inspired by:
- [Actix](https://actix.rs/) (Rust)
- [Orleans](https://dotnet.github.io/orleans/) (.NET)
- [Erlang/OTP](https://www.erlang.org/)

## 📄 License

Dual-licensed under MIT or Apache 2.0. See [LICENSE](LICENSE) for more details.

---

<p align="center">
  Made with ❤️ for Rust<br>
  <sub>Simplifying concurrency, one actor at a time</sub>
</p>