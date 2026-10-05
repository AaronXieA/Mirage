# Mirage 26.x 版本移植说明

原项目面向 Minecraft 1.21.11（Fabric）。本次移植使其适配 Minecraft 26.2 / 26.3（2026 年新版本号体系，游戏已去混淆）。

## 构建工具链变化

| 项目 | 原值（1.21.11） | 新值（26.2 / 26.3） |
|------|-----------------|----------------------|
| Minecraft | 1.21.11 | 26.2 或 26.3 |
| Fabric Loader | 0.18.4 | 0.19.5 |
| Fabric API | 0.141.3+1.21.11 | 0.161.0+26.2 / 0.161.0+26.3 |
| Loom 插件 | `net.fabricmc.fabric-loom-remap` 1.15-SNAPSHOT | `net.fabricmc.fabric-loom` 1.18.2 |
| Gradle | 9.3.0 | 9.7.1 |
| Java | 21 | 25（26.x 服务器要求 Java 25） |
| `mappings` 声明 | `loom.officialMojangMappings()` | 已删除（26.x 原版不再混淆，Loom 无需映射） |
| 依赖配置 | `modImplementation` | `implementation`（同理，不再重映射） |
| mixin `compatibilityLevel` | JAVA_21 | JAVA_25 |

## 源码适配点

1. **`ChunkPos.asLong(int,int)` → `ChunkPos.pack(int,int)`**（26.2 起改名，`MirrorApplyTask`）
2. **`RegionFileStorage.regionCache` 元素类型变化**：
   - 26.2：`Long2ObjectLinkedOpenHashMap<RegionFile>`
   - 26.3：`Long2ObjectLinkedOpenHashMap<Optional<RegionFile>>`
   - 由于泛型擦除二者签名相同，编译无差别但运行时不同。`ChunkUnloader.clearRegionCache` 现按元素实际类型（`instanceof`）同时兼容两个版本，`RegionFileStorageAccessor` 的声明对运行时无影响。
3. 其余 Mixin 目标（`ServerLevel.noSave`、`ServerChunkCache.chunkMap`、`ChunkMap.getVisibleChunkIfPresent`/`saveAllChunks`、`SimpleRegionStorage.worker`、`IOWorker.storage`）在 26.2 与 26.3 中均存在，已逐一用 `javap` 校验，并在 26.2 / 26.3 真实服务端上完成启动验证（Mixin 注入成功、`/mirage status` 正常、优雅关服）。

## 如何构建

当前源码树默认配置为 **26.2**：

```bash
./gradlew build          # 产出 build/libs/mirage-1.2.0+mc26.2.jar
```

构建 26.3 版本需切换两处（`gradle.properties` 的 `minecraft_version`/`fabric_api_version` 与 `src/main/resources/fabric.mod.json` 的 `"minecraft": "~26.3"`）后：

```bash
./gradlew build -Pmod_version=1.2.0+mc26.3
```

编译与运行均需本机 JDK 25（服务器 26.x 运行时同样要求 Java 25）。

## 产物

| 文件 | 适用版本 |
|------|----------|
| `mirage-1.2.0+mc26.2.jar` | Minecraft 26.2 |
| `mirage-1.2.0+mc26.3.jar` | Minecraft 26.3 |

两份 jar 均要求：Fabric Loader ≥ 0.19.5、Fabric API、Java ≥ 25。
