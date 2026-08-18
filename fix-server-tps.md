# Minecraft Server TPS & MSPT Optimization Guide

## 1. Fundamentals: TPS vs. MSPT

- **TPS (Ticks Per Second):** Capped at 20.0. A falling TPS means tick duration exceeds 50ms.
- **MSPT (Milliseconds Per Tick):** The actual measure of server health.
  - `< 40ms`: Healthy buffer.
  - `50ms`: Maximum threshold for stable 20.0 TPS.
  - `> 50ms`: Server is losing ticks (lag).

Do not tune server configurations blindly based on symptoms. Always profile first.

---

## 2. Diagnostic Protocol

1. **Run Spark Profiler:**
   ```bash
   /spark profiler --start
   # Allow to run for 10-15 minutes under normal player load
   /spark profiler --stop
   ```
2. **Analyze Output:** Look for dominant tick consumers in the call stack.
   - For detailed stack interpretation, see [Spark Profiler Analysis Guide](./spark-profiler-guide.md).
3. **Monitor Memory & Threading:**
   ```bash
   /spark healthcheck
   /spark heap
   ```

---

## 3. Core Engine & Runtime Setup

### Server Software
- **Avoid:** CraftBukkit, Spigot, Vanilla.
- **Recommended:** Paper or Purpur (Purpur exposes fine-grained tick frequency toggles).
- **Multithreading Workloads:** For massive player bases on 1.20+, evaluate Folia (requires multi-threaded compatible plugins).

### Java Runtime & GC Configuration
Use Java 21 LTS or newer. Do not allocate excessive RAM; over-allocation increases GC collection times.

**Recommended JVM Arguments (Generational ZGC for Java 21+):**
```bash
java -Xms6G -Xmx6G -XX:+UseZGC -XX:+ZGenerational -XX:+AlwaysPreTouch -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -jar server.jar nogui
```
For legacy G1GC tuning or detailed flag explanations, refer to [JVM & GC Tuning Guide](./jvm-flags-guide.md).

---

## 4. Key Configuration Tweaks

### A. View & Simulation Distance
World loading and entity processing scale exponentially with simulation distance.
- `server.properties`:
  - `view-distance`: `8` - `12` (No-tick view distance handles client visual render)
  - `simulation-distance`: `4` - `6` (Stops entity/redstone processing beyond this radius)

### B. Entity Activation & Ticking
In `paper-world-defaults.yml` or `spigot.yml`:
- Reduce `entity-activation-range`:
  - `animals`: `16`
  - `monsters`: `24`
  - `raiders`: `24`
  - `misc`: `8`
- Enable `tick-inactive-villagers: false` or increase villager brain tick intervals.
- Set `despawn-ranges`:
  - `soft`: `32`
  - `hard`: `96`

### C. Redstone & Hoppers
Hoppers checking for item entities above them every tick cause high CPU overhead.
- In `paper-global.yml`:
  - `hopper.disable-move-event: true`
- In `purpur.yml` (if using Purpur):
  - Set custom hopper transfer cooldowns or alternate redstone algorithms (e.g., Alternate Current).

For a complete key-by-key breakdown, see [Paper & Purpur Configuration Matrix](./paper-purpur-optimization.md).

---

## 5. World Pre-Generation

Generating chunks dynamically during player exploration consumes significant main-thread CPU time and disk I/O.

1. Install **Chunky**.
2. Set world border and pre-generate prior to public launch:
   ```bash
   /worldborder set 10000
   /chunky radius 5000
   /chunky start
   ```

---

## 6. Related Documentation

- [Spark Profiler Deep-Dive & Flamegraph Interpretation](./spark-profiler-guide.md)
- [Paper & Purpur Advanced Config Reference](./paper-purpur-optimization.md)
- [JVM Flag Tuning for High-Memory Environments](./jvm-flags-guide.md)
- [Entity & Mob Farm Control Strategies](./entity-management.md)
