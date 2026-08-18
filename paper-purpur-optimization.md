# Paper & Purpur Configuration Reference

## 1. File Precedence & Hierarchy

Configuration values override parent settings down the engine chain:
`server.properties` -> `spigot.yml` -> `paper-global.yml` / `paper-world-defaults.yml` -> `purpur.yml`

Tuning values below target high-density multiplayer workloads without breaking core gameplay mechanics.

---

## 2. Chunk & World Saving Optimization

Dynamic chunk saving during main-thread execution causes major MSPT spikes.

### `paper-world-defaults.yml`
```yaml
chunks:
  auto-save-interval: 6000 # Save world data every 5 minutes (default 30s)
  max-auto-save-chunks-per-tick: 6 # Spreads disk I/O over multiple ticks
  delay-chunk-unloads-by: 10s # Prevents rapid load/unload thrashing from fast players
  prevent-moving-into-unloaded-chunks: true
```

---

## 3. Entity Activation & AI Throttling

Entities represent the single largest CPU bottleneck on modern Minecraft servers.

### `spigot.yml`
```yaml
world-settings:
  default:
    entity-activation-range:
      animals: 16
      monsters: 24
      raiders: 24
      misc: 8
      water: 16
      tick-inactive-villagers: false
    entity-tracking-range:
      players: 128
      animals: 48
      monsters: 48
      misc: 32
```

### `paper-world-defaults.yml`
```yaml
entities:
  behavior:
    tick-inactive-villagers: false
    spawner-mobs-check-ground: true
  spawning:
    despawn-ranges:
      monster:
        soft: 32
        hard: 96
      ambient:
        soft: 32
        hard: 64
    alt-item-despawn-rate:
      enabled: true
      items:
        cobblestone: 300
        netherrack: 300
        dirt: 300
```

### `purpur.yml`
```yaml
world-settings:
  default:
    mobs:
      villager:
        lobotomize:
          enabled: true
          check-interval: 100 # Strips pathfinding AI from enclosed villagers every 5s
        lobotomize-1x1: true
      zombie:
        aggressive-towards-villager-when-lagging: false
```

---

## 4. Redstone & Hopper Operations

### `paper-global.yml`
```yaml
hopper:
  disable-move-event: true # Disables InventoryMoveItemEvent unless required by specific plugins
  ignore-occluding-blocks: true
```

### `paper-world-defaults.yml`
```yaml
misc:
  redstone-implementation: ALTERNATE_CURRENT # Replaces vanilla redstone algorithm for 200%+ performance gain
```

### `purpur.yml`
```yaml
world-settings:
  default:
    blocks:
      hopper:
        cooldown:
          when-full: 8
        check-when-full: false
```

---

## 5. Collision & Physics Limits

### `paper-world-defaults.yml`
```yaml
collisions:
  max-entity-collisions: 2 # Default 8; drastically reduces tick load in mob farms
  only-players-collide: true
environment:
  optimize-explosions: true
```

---

## 6. Related Documentation

- [TPS & MSPT Optimization Guide](./tps-optimization-guide.md)
- [Spark Profiler Deep-Dive & Flamegraph Interpretation](./spark-profiler-guide.md)
- [JVM Flag Tuning for High-Memory Environments](./jvm-flags-guide.md)
