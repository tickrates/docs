# Minecraft Server Memory (RAM) Allocation Guide

## 1. Fundamentals: The Over-Allocation Myth

Allocating excessive RAM to a Minecraft server does not increase processing speed or tick rate. In Java environments, oversized heap allocations create severe performance bottlenecks due to Garbage Collection (GC) mechanics.

### The GC Pause Cycle
1. **Memory Allocation:** The JVM allocates memory dynamically on the heap for temporary objects (entity ticks, block updates, packet buffers, mod registry calls).
2. **Heap Filling:** Unreferenced objects accumulate in the heap until a collection threshold is reached.
3. **Stop-The-World (STW) Spikes:** When clearing a massive heap (e.g., 16GB+ on standard G1GC), the collector must parse tens of millions of object references. This pauses the main server tick thread (`ServerThread`), causing visible tick drops and player rubber-banding.

**Rule of Thumb:** Allocate the minimum RAM required for your player count, simulation distance, and mod profile plus a 20–30% buffer for GC breathing room.

---

## 2. Modded Architecture & Memory Footprint

Mods alter the server memory profile in two distinct ways: **Static Registry Overhead** and **Dynamic Runtime Churn**.

### A. Static Registry & Metaspace Overhead
Custom blocks, items, biome IDs, entities, and recipes loaded by Fabric/NeoForge/Forge expand the static registry data in memory.
- **Metaspace:** The JVM region that stores loaded class definitions. Heavy modpacks (250+ mods) increase Metaspace requirements from 200MB up to 1GB+.
- **Data Structure Inflation:** Every custom block or item registered adds entries to global hash maps that persist in heap memory permanently, regardless of whether those items exist in active chunks.

### B. Dynamic Mod Runtime Churn
- **Tile/Block Entities:** Modded machines (e.g., Create kinetic networks, Applied Energistics 2 storage networks, Mekanism cables) continuously write custom NBT data and execute tick logic, drastically increasing memory allocations per tick.
- **Custom Entity AI:** Complex mob mods (e.g., Alex's Mobs) instantiate pathfinding algorithms that consume transient heap space.
- **Force-Loaded Chunks:** Chunk-loader blocks or modded claim systems (e.g., FTB Chunks) keep chunks, block entities, and mob AI anchored in RAM permanently across multiple dimensions.

---

## 3. Server Sizing Matrix (Vanilla, Paper, & Modded)

Memory requirements depend on server software, render distance, player concurrency, and total mod weight.

| Server Type | Players | Sim / View | Mod Profile | Recommended RAM |
| :--- | :--- | :--- | :--- | :--- |
| **Vanilla / Fabric** | 1 – 5 | 10 / 12 | 0–10 (Optimization) | **2 – 4 GB** |
| **Paper (Small)** | 10 – 30 | 6 / 10 | Plugins only | **6 – 8 GB** |
| **Paper (Large)** | 50 – 100 | 4 / 8 | Plugins only | **10 – 12 GB** |
| **Light Modpack** | 1 – 10 | 6 / 8 | 30–80 (Vanilla+) | **5 – 6 GB** |
| **Medium Modpack** | 5 – 15 | 6 / 8 | 100–200 (Tech/Magic) | **7 – 9 GB** |
| **Heavy Modpack** | 1 – 10 | 5 / 8 | 250–450+ (Expert/Kitchen Sink) | **10 – 14 GB** |
| **Proxy / Hub (Velocity)** | 500+ | N/A | None | **1 – 2 GB** |


---

## 4. Core Memory Drivers

### A. Simulation Distance (Primary Memory Scaler)
Simulation distance dictates the radius of chunks kept actively in memory around every player.

Active Chunks per Player = (2 × SimDistance + 1)²

- **Sim Distance 4:** 81 chunks per player context.
- **Sim Distance 8:** 289 chunks per player context (3.5× memory footprint increase).
- **Sim Distance 10:** 441 chunks per player context (5.4× memory footprint increase).

### B. Mod Classification Impact

- **Optimization Mods (Reduces Memory Usage):**
  - **FerriteCore:** Reduces RAM usage by deduplicating models and optimizing data structures.
  - **ModernFix:** Reduces launch time and fixes memory leaks in Forge/NeoForge registries.
  - **Lithium:** Optimizes data structure alignment and entity processing.
- **World Generation Mods (High Memory During Exploration):**
  - Mods like *Terralith*, *Tectonic*, or *Alex's Caves* increase chunk generation data size. If players explore ungenerated terrain, memory allocations spike dramatically. Pre-generation using Chunky is mandatory.
- **Tech / Automation Mods (High Continuous Memory Usage):**
  - Pipe networks, item transport, and automated crafting grids maintain large, volatile object reference trees in memory.

---

## 5. Off-Heap Overhead & Host System Requirements

Setting `-Xms8G -Xmx8G` does **not** cap the Java process at 8GB of system RAM. The total system RAM usage equals:

Total System RAM = Heap Allocation (-Xmx) + Off-Heap Memory + OS Overhead

### Off-Heap Components
1. **Metaspace:** Stores loaded class definitions (`-XX:MaxMetaspaceSize=512M` or higher for modded).
2. **Direct Byte Buffers:** High-throughput network packet management via Netty.
3. **Thread Stacks & JVM Code Cache:** Native memory allocated for thread handling and JIT compilation.

> **Hosting Provisioning Rule:** Reserve at least **2GB – 3GB of unallocated physical RAM** on the host machine for OS kernel tasks and JVM off-heap allocations. On a machine with 16GB total physical RAM, set `-Xms`/`-Xmx` to a maximum of **12GB – 13GB** to prevent Linux Out-Of-Memory (OOM) killer terminations.

---

## 6. Garbage Collector Selection by Memory Tier

Selection of the JVM Garbage Collector must adapt to the target heap size and mod weight.

### Tier 1: Small to Medium Heaps (4GB – 10GB) -> **G1GC**
Optimized for standard Paper servers or lightweight Fabric/Forge modpacks.
```bash
java -Xms6G -Xmx6G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -jar server.jar nogui
