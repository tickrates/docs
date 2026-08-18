# Minecraft JVM & Garbage Collection (GC) Optimization Guide

## 1. How the JVM Interacts with Minecraft

Minecraft runs on the **Java Virtual Machine (JVM)**, which manages system memory automatically using a process called **Garbage Collection (GC)**. 

* **The Allocation Pattern:** Minecraft generates massive amounts of short-lived data per second (block updates, packet handling, entity velocity). 
* **The Heap:** The JVM divides memory into the **Young Generation** (new data) and the **Old Generation** (persistent data like loaded worlds and registries).
* **The GC Problem:** When the JVM cleans memory, it triggers a pause. Unoptimized default JVM settings cause these pauses to last hundreds of milliseconds, resulting in visible server freezes ("Stop-The-World" lag).

---

## 2. Standard JVM Startup Parameters

Every Minecraft startup script relies on core flags to govern memory allocation and execution.

| Flag | Purpose | Recommended Usage |
| :--- | :--- | :--- |
| `-Xms` | Sets initial/minimum allocated RAM heap. | **Must match `-Xmx`** to prevent live memory resizing lag. |
| `-Xmx` | Sets absolute maximum RAM heap limit. | Set to server target (leave 1–2 GB system headroom). |
| `-XX:+UseG1GC` | Enables Garbage-First Collector (best for most servers). | Standard collector for modern Minecraft (Java 17/21+). |
| `-XX:+AlwaysPreTouch` | Pre-allocates allocated RAM on boot. | Prevents OS memory allocation pauses during gameplay. |
| `-XX:+DisableExplicitGC` | Blocks plugins from manually calling `System.gc()`. | Prevents poorly coded plugins from triggering full server freezes. |

---

## 3. Production GC Flag Sets

### A. Aikar’s Flags (Standard G1GC)
Developed by the PaperMC team, **Aikar’s Flags** tune Java’s G1 Garbage Collector specifically for Minecraft’s high-allocation runtime. They split memory processing into micro-tasks to keep pause times imperceptible (<20ms).

#### Optimized Flag Set (For <= 12 GB Heap)
```bash
java -Xms8G -Xmx8G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -jar server.jar --nogui
