# Spark Profiler Analysis Guide
[Download Spark](https://spark.lucko.me/download)

## 1. Sampling & Execution Protocol

Profiling measures thread activity via sampling. Short profiles capture noise; always sample during active load.

### Commands
```bash
# Standard CPU profiling (run for 10-15 minutes during peak load)
/spark profiler --start
/spark profiler --stop

# Allocation profiling (tracks memory churn & garbage creation)
/spark profiler --start --alloc
```

### Sampling Guidelines
- Do not profile during initial server startup or pre-generation.
- Minimum profiling duration: 5 minutes. Recommended: 10–15 minutes under live player load.
- Ensure the main thread (`ServerThread`) remains the focal point for tick-rate lag diagnostics.

---

## 2. Flamegraph Mechanics

Spark flamegraphs display call stacks visually:
- **X-Axis (Width):** Percentage of sample time consumed by a method call. Broader blocks represent greater CPU time spent.
- **Y-Axis (Depth):** Method invocation call stack depth (parent methods on bottom, child methods on top).

### Reading Direction
1. Locate `ServerThread` at the base.
2. Follow the stack upward to trace execution flow.
3. Identify wide horizontal "plateaus" near top levels—these represent actual work executed by CPU cores.

---

## 3. Classifying Tick Lag Bottlenecks

### A. Entity & Mob AI (`net.minecraft.world.entity`)
- **Signatures:** `ServerLevel.tick`, `Entity.tick`, `Mob.tick`, `PathfinderGoal`.
- **Primary Causes:** Large concentrations of villagers, farm animals, or aggressive mob pathfinding in un-optimized farms.
- **Resolution:** Tighten activation ranges and brain tick rates. See [TPS Optimization Guide](./tps-optimization-guide.md#4-key-configuration-tweaks).

### B. Block Entities & Redstone (`net.minecraft.world.level.block.entity`)
- **Signatures:** `BlockEntity.tick`, `HopperBlockEntity.tick`, `RedstoneWireBlock`.
- **Primary Causes:** Massive hopper chains, sorting systems, auto-crafters, or constant redstone update cascades.
- **Resolution:** Disable hopper move events in `paper-global.yml` or implement Alternate Current via Purpur/fabric mods.

### C. Chunk I/O & World Generation (`net.minecraft.server.level`)
- **Signatures:** `ChunkMap`, `RegionFileStorage`, `AnvilChunkLoader`.
- **Primary Causes:** Players exploring ungenerated terrain using elytra/Riptide tridents on main thread.
- **Resolution:** Pre-generate world boundaries using Chunky. See [TPS Optimization Guide](./tps-optimization-guide.md#5-world-pre-generation).

### D. Plugin Overhead
- **Signatures:** Method calls containing third-party package namespaces (e.g., `com.example.plugin.listeners`).
- **Primary Causes:** Synchronous database operations (SQLite/MySQL on main thread), un-indexed inventory searches, dense particle loops on `PlayerMoveEvent`.
- **Resolution:** Report stack trace to plugin developer or move work to async tasks.

---

## 4. Allocation Profiling (Garbage Collection Lag)

When CPU usage appears low but micro-stutters occur, memory allocation rate may be triggering frequent Stop-The-World (STW) GC pauses.

1. Run `/spark profiler --start --alloc`.
2. Inspect methods producing high allocations per second (MB/s).
3. If GC pause times remain high despite clean code, adjust JVM heap flags. Refer to [JVM & GC Tuning Guide](./jvm-flags-guide.md).

---
