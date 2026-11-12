# Java 21 派生版

## 定位与版本

基于官方 [TA4J 0.28.0](https://github.com/ta4j/ta4j/releases/tag/0.28.0)，使用独立 `jdk21` 分支。`master` 继续同步上游 Java 25；本分支不宣称是官方 Java 21 发行版。

Maven 版本为 `0.28.0-zhao-jdk21.1`，避免与官方构件同名覆盖。未发布到 Maven Central，需要从源码安装。

源码通过 [Java 21 分支](https://github.com/haers/zhao-ta4j/tree/jdk21)提供。获取源码后，在该分支执行下方构建命令，再由消费工程通过 Maven 引入派生版版本号。

```sh
git clone --branch jdk21 https://github.com/haers/zhao-ta4j.git
cd zhao-ta4j
```

## 改动范围

- 编译 release、Enforcer、CI、README 和基线一致性测试统一为 Java 21。
- MassIndex、RWIHigh、RWILow、MACDV、VolatilityNormalizedMACD、ChaikinOscillator 六处构造函数改用构造委托；MACDV 的复杂依赖构造放在私有不可变记录中。
- 保持原有帮助指标的创建次序、共享实例、依赖注册、数值算法与缓存契约，不新增公共 API。
- 复用六个已有测试类中的依赖图、数值、暖机与缓存回归，避免重复新增同义测试。

## 构建与接入

要求 JDK 21+、Maven 3.9+；先确认 `java -version` 与 `mvn -version` 使用目标 JDK。

```sh
# 仓库完整门禁，包含默认非演示测试和质量检查
scripts/run-full-build-quiet.sh
# 安装派生版构件（包含测试，不跳过质量门禁）
mvn -B -ntp install
```

消费端只依赖核心库：

```xml
<dependency>
  <groupId>org.ta4j</groupId>
  <artifactId>ta4j-core</artifactId>
  <version>0.28.0-zhao-jdk21.1</version>
</dependency>
```

消费工程可将该核心库作为普通 Maven 依赖使用，并按自身需求定义行情输入、指标暖机和回测边界。TA4J 提供指标、规则、策略与回测组件，交易平台连接及订单执行由消费工程实现。默认完整门禁沿用上游测试标签范围，覆盖范围以构建脚本和测试标签配置为准。

## 验证记录

2026-11-03，在 Zulu Java 21.0.10 下完成：

| 检查 | 结果 |
| --- | --- |
| 六项指标定向回归 | Tests run 94 / Failures 0 / Errors 0 / Skipped 0 |
| 原生全仓门禁 | Tests run 11823 / Failures 0 / Errors 0 / Skipped 0；core 10894、examples 929 |
| SpotBugs | core 与 examples 均通过 |
| JaCoCo core | 行覆盖 89.00%，分支覆盖 76.59% |
| 字节码 | core 1483 个类最大 major 65；core 测试类路径共 32 个 jar 的 Java21 有效类均不超过 65，排除仅更高 JDK 生效的多版本条目 |
| 独立消费 | 仅 core jar 与 slf4j-api、commons-math3、gson 三个正式依赖，真实 Java21 下编译并运行固定 K 线程序，SMA(3)=5 |

以上为固定派生源码在所列日期的验证结果。运行完整门禁脚本时会输出当前环境的日志位置，复核时以新生成的结果为准。消费方接入后的依赖组合、数据与交易流程需要在其所属工程中验证。

## 后续同步

上游更新先同步 `master`，再评估是否向 `jdk21` 引入。每次引入需要重查 Java 新语法/API/依赖字节码要求，并重新执行完整门禁；不保证未来版本始终只需修改这六处构造函数。
