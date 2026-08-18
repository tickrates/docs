# Tickrates Technical Documentation

Tickrates is an open-source technical index and community hub for Minecraft server administrators, engineers, and developers. Our goal is to replace outdated forum posts, superstitious config advice, and unoptimized server defaults with precise, hardware-level diagnostic guides.

---

## Guides

* [Fixing Server TPS & High MSPT](./fix-server-tps.md): Systematic checklist for isolating bottlenecks and optimizing main-thread performance.
* [Reading Spark Profiler Reports](./read-spark-report.md): A guide to interpreting sampling data, method calls, and execution trees.
* [Analyzing Server Crash Logs](./read-crash-log.md): How to isolate thread deadlocks, NBT corruption, and heap exhaustion.
* [Fabric vs. Paper Software Comparison](./fabric-vs-paper.md): Technical breakdown of internal game loop modifications versus vanilla logic retention.
* [Memory Allocation & JVM Tuning](./server-ram-guide.md): Heap size recommendations, Garbage Collection behavior, and avoiding stop-the-world pauses.

---

## Contributing & Community

These docs are fully open-source. If you find outdated flags, inaccurate method descriptions, or better profiling techniques, submit a pull request on GitHub.

To discuss server architecture, share configuration files, or get profiling input from other operators, join our Discord community.

* [Join the Tickrates Discord](https://discord.gg/YOUR_INVITE_LINK)
