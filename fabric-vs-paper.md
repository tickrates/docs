# Fabric vs. Paper & Purpur: Server Software Comparison

## 1. Key Differences

### Paper & Purpur (Engine Patchers)
Paper and Purpur fork Spigot/CraftBukkit. They rewrite, replace, or skip Vanilla tick logic, chunk loading, and entity behaviors directly at the server code level.
- **Goal:** Maximizing player capacity and preventing server crashes at all costs.
- **Ecosystem:** Bukkit / Spigot / Paper API (`.jar` plugins).
- **Modification Strategy:** Hardcodes performance shortcuts into the server engine, often changing Vanilla gameplay behavior.

### Fabric (Lightweight Mod Loader)
Fabric is not a server software by itself; it is a lightweight mod loader. Out of the box, Fabric runs unpatched Vanilla server code. Performance gains are achieved by adding standalone server-side optimization mods.
- **Goal:** Moddability and exact Vanilla mechanic preservation.
- **Ecosystem:** Fabric Mod API (`.jar` mods).
- **Modification Strategy:** Retains 100% Vanilla game logic while replacing inefficient internal algorithms via optimization mods.

---

## 2. Performance & Scaling Profile

| Vector | Paper / Purpur | Fabric + Optimization Stack |
| :--- | :--- | :--- |
| **Primary Bottleneck Fix** | Entity AI throttling, async chunk generation, despawn aggressive tuning | Algorithmic optimizations (Lithium), networking (Krypton), memory usage (FerriteCore) |
| **High Player Counts (50+)** | **Superior.** Handles high player density via aggressive entity activation ranges and chunk unloading. | **Requires Tuning.** Scales well with C2ME and VMP (Very Many Players), but entity processing remains heavier due to Vanilla parity. |
| **Low-to-Medium Counts (< 30)** | **Fast.** Low resource consumption, but alters Vanilla tick behavior. | **Fastest.** Fabric + Lithium often achieves lower MSPT and RAM overhead than Paper on raw Vanilla workloads. |
| **Redstone Execution** | **Altered.** Uses optimized algorithms (e.g., Alternate Current) that can break directional/micro-tick redstone wiring. | **100% Vanilla.** Zero changes to update order, quasi-connectivity, or item duplication mechanics. |

---

## 3. Vanilla Parity & Technical Minecraft

### Paper & Purpur Trade-Offs
Paper intentionally patches or alters Vanilla mechanics to preserve server MSPT:
- **Redstone & Physics:** Suppresses update cascades; modifies hopper item handling and entity collision boxes.
- **Mob Farms & Mob Caps:** Despawns mobs aggressively outside simulation distance; lobotomizes villager pathfinding.
- **Exploits:** Patches bedrock breaking, TNT duping, and sand duping by default (configurable, but underlying tick breaks persist).

### Fabric Parity Guarantee
Fabric does not alter game logic. When paired with **Lithium** (a server-side optimization mod), mathematical algorithms are rewritten without changing outputs:
- TNT duplication, bedrock breaking, and RNG manipulation function identically to Vanilla singleplayer.
- Fully compatible with **Carpet Mod** for deep server profiling, tick manipulation, and technical automation.

For diagnostic stack traces on either platform, refer to the [Spark Profiler Analysis Guide](./spark-profiler-guide.md).

---

## 4. Ecosystem & Extensibility

### Plugin vs. Mod Paradigm
- **Paper / Purpur:** Uses Bukkit-based plugins (e.g., EssentialsX, LuckPerms, WorldGuard). Plugins operate purely server-side—players join with an unmodded Vanilla client.
- **Fabric:** Uses Fabric mods. 
  - **Server-Only Mods:** Functions like plugins (e.g., Lithium, FerriteCore, Ledger, miniMOTD). Vanilla clients can join without installing anything.
  - **Content Mods:** Adds custom items, blocks, or mobs (e.g., Create, Cobblemon). Requires clients to install matching mods.

> **Warning on Hybrid Software:** Software like Cardboard or Arclight attempts to run Bukkit plugins on Fabric. These bridges introduce severe thread-safety bugs, broken event triggers, and unstable tick loops. Do not use hybrid wrappers on production environments.

---

## 6. Recommended Fabric Performance Stack

If choosing Fabric, install these core server-side optimization mods:

1. **Lithium:** Optimizes physics, chunk alignment, pathfinding, and entity collision logic without changing mechanics.
2. **FerriteCore:** Reduces Java memory footprint by optimizing data structures.
3. **Krypton:** Replaces Vanilla networking pipelines to reduce bandwidth usage and CPU overhead on tick threads.
4. **C2ME (Chunk Performance):** Adds multi-threaded chunk generation and I/O handling.
5. **Very Many Players (VMP):** Optimizes packet handling and tracking for dense player clusters.

---

## 7. Related Documentation

- [TPS & MSPT Optimization Guide](./tps-optimization-guide.md)
- [Spark Profiler Deep-Dive & Flamegraph Interpretation](./spark-profiler-guide.md)
- [Paper & Purpur Advanced Config Reference](./paper-purpur-optimization.md)
