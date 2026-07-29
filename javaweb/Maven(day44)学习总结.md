# Maven 学习总结（Day 44）

> Maven 是一个用于**管理和构建 Java 项目**的工具。  
> 核心能力：统一项目结构、管理依赖、完成项目构建。

## 目录

- [一、Maven 概述](#一maven-概述)
- [二、POM 项目对象模型](#二pom-项目对象模型)
- [三、下载安装与配置](#三下载安装与配置)
- [四、IDEA 集成 Maven](#四idea-集成-maven)
- [五、依赖管理](#五依赖管理)
- [六、项目构建](#六项目构建)
- [七、继承与聚合](#七继承与聚合)
- [八、重点速记](#八重点速记)

---

## 一、Maven 概述

### 1. Maven 是什么

Maven 是用于**管理和构建 Java 项目**的工具。

### 2. Maven 的主要作用

#### 2.1 统一项目结构

典型 Maven 项目结构如下：

```text
项目名称/
├─ src/
│  ├─ main/                         # 主程序目录
│  │  ├─ java/                      # Java 源文件
│  │  ├─ resources/                 # 配置文件或资源文件
│  │  └─ webapp/                    # Web 项目特有目录
│  │     └─ WEB-INF/
│  │        └─ web.xml              # Web 项目配置文件
│  └─ test/                         # 测试程序目录
│     ├─ java/                      # 测试 Java 源文件
│     └─ resources/                 # 测试配置或资源文件
└─ pom.xml                          # Maven 项目核心配置文件
```

`pom.xml` 用于定义：

- 项目的基本信息
- 项目的依赖信息
- 项目的构建信息
#### 2.2 管理依赖

Maven 可以：
- 自动下载项目所需的 JAR 包
- 自动下载依赖所需要的其他依赖
- 按规则处理依赖版本冲突

#### 2.3 项目构建

构建是一个完整过程，由**生命周期**和**阶段**描述。每个阶段对应相应命令，可以通过一条 Maven 命令连续完成多个构建阶段。

---

## 二、POM 项目对象模型

POM（Project Object Model）即**项目对象模型**，Maven 通过 `pom.xml` 描述项目。

POM 主要负责两部分内容：

1. **项目构建**：定义生命周期、构建阶段、打包方式等。
2. **依赖管理**：通过依赖坐标和仓库管理项目所需的 JAR 包。

### Maven 仓库

| 仓库类型 | 说明 |
|---|---|
| 本地仓库 | 本地磁盘中的一个文件夹，用于缓存已下载的依赖 |
| 中央仓库 | 由 Maven 官方维护的公共仓库 |
| 远程仓库（私服） | 通常由公司自行搭建；学习阶段也可使用阿里云 Maven 镜像 |

依赖查找通常遵循：

```text
本地仓库 → 远程仓库/私服 → 中央仓库
```

---

## 三、下载安装与配置

### 1. 下载与安装

- 从 Maven 官网下载安装包。
- 安装路径尽量不要包含中文或空格。
- 建议将开发工具统一放在固定目录中，便于管理。

### 2. 配置本地仓库

编辑 Maven 安装目录中的：

```text
conf/settings.xml
```

通过 `<localRepository>` 设置本地仓库路径。若不配置，默认位置通常是：

```text
用户主目录/.m2/repository
```

### 3. 配置远程仓库镜像

在 `conf/settings.xml` 的 `<mirrors>` 中添加 `<mirror>`，可配置远程镜像仓库。

### 4. 配置环境变量

1. 新建环境变量：

   ```text
   MAVEN_HOME=Maven 安装目录
   ```

2. 在 `Path` 中添加：

   ```text
   %MAVEN_HOME%\bin
   ```

3. 可使用以下命令验证：

   ```bash
   mvn -v
   ```

---

## 四、IDEA 集成 Maven

在 IntelliJ IDEA 中依次进入：

```text
File → Settings → Build, Execution, Deployment
     → Build Tools → Maven
```

重点检查以下配置：

- **Maven home path**：Maven 安装目录
- **User settings file**：`settings.xml` 文件
- **Local repository**：本地仓库目录

---

## 五、依赖管理

### 1. 依赖坐标

依赖坐标是 JAR 包的唯一标识，核心由三部分组成：

| 坐标 | 含义 |
|---|---|
| `groupId` | 公司或组织名称 |
| `artifactId` | 项目或模块名称 |
| `version` | 版本号 |

版本常见类型：

- `SNAPSHOT`：快照版本，通常表示仍在开发中
- `RELEASE`：正式发行版本

依赖配置示例：

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.10</version>
    </dependency>
</dependencies>
```

修改 `pom.xml` 后，记得在 IDEA 中**刷新 Maven 项目**。

### 2. JAR 包下载失败的处理

可按以下顺序排查：

1. 检查 Maven、`settings.xml`、本地仓库和镜像配置。
2. 删除本地仓库中的 `.lastUpdated` 文件。
3. 重新刷新 Maven 项目并下载依赖。

### 3. 依赖范围（scope）

通过 `<scope>` 标签设置依赖的使用范围：

| scope 值 | 主程序 | 测试程序 | 参与打包/运行 | 示例 |
|---|:---:|:---:|:---:|---|
| `compile`（默认） | ✅ | ✅ | ✅ | log4j |
| `test` | ❌ | ✅ | ❌ | junit |
| `provided` | ✅ | ✅ | ❌ | servlet-api |
| `runtime` | ❌ | ✅ | ✅ | JDBC 驱动 |
| `import` | — | — | — | 用于引入 `dependencyManagement`，通常配合 BOM 使用 |

### 4. 依赖传递

- **直接依赖**：当前项目在 `pom.xml` 中直接声明的依赖。
- **间接依赖**：由直接依赖传递进来的依赖。
- **依赖树**：用于展示直接依赖和间接依赖之间的层级关系。

#### 排除依赖

当不希望某个间接依赖被传递进项目时，可以使用 `<exclusions>`：

```xml
<dependency>
    <!-- 当前项目需要的直接依赖坐标 -->
    <exclusions>
        <!-- 排除依赖时不需要指定版本号 -->
        <exclusion>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

#### 可选依赖

设置 `<optional>true</optional>` 后，该依赖不会继续传递给引用当前项目的外部项目：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.10</version>
    <optional>true</optional>
</dependency>
```

### 5. 依赖冲突

依赖冲突是指同一个 JAR 包以不同版本出现在依赖树中。

Maven 确定最终版本的主要规则：

1. **路径优先**：层级不同时，离当前项目越近（层级越浅）的依赖优先。
2. **声明优先**：路径深度相同时，先声明的依赖通常优先。
3. **直接声明可覆盖传递版本**：可在当前项目中直接声明目标版本，明确控制最终使用的版本。

> 建议：复杂项目可通过 `<dependencyManagement>` 统一锁定依赖版本。

---

## 六、项目构建

### 1. Maven 生命周期

#### Clean 生命周期

| 阶段 | 作用 |
|---|---|
| `clean` | 清理构建结果，删除 `target` 文件夹 |

#### Default 生命周期

| 阶段 | 作用 |
|---|---|
| `compile` | 编译项目源代码 |
| `test` | 执行测试用例 |
| `package` | 将项目打包 |
| `install` | 将构建产物安装到本地仓库 |
| `deploy` | 将构建产物部署到远程仓库或私服 |

同一个生命周期中，执行后面的阶段时，前面的阶段会按顺序先执行。例如执行：

```bash
mvn package
```

会依次完成编译、测试、打包等前置阶段。

常用组合命令：

```bash
mvn clean package
```

该命令会先清理旧构建结果，再完成项目打包。

### 2. 打包方式

通过 `pom.xml` 中的 `<packaging>` 设置：

| 打包方式 | 说明 |
|---|---|
| `jar` | 默认方式，常用于普通 Java 项目 |
| `war` | 常用于传统 Web 项目 |
| `pom` | 常用于父工程或聚合工程 |

### 3. 触发构建阶段

- 在 IDEA 的 Maven 面板中进行图形化操作。
- 在命令行中执行 Maven 命令，例如 `mvn clean package`。

---

## 七、继承与聚合

### 1. 继承

**作用：简化依赖管理，统一子工程配置。**

父工程中常见的配置方式：

- `<dependencies>`：父工程声明的依赖会被子工程继承。
- `<dependencyManagement>`：只统一管理依赖版本，子工程仍需声明要使用的依赖，但可以不再重复指定版本号。
- `<properties>`：统一声明版本或其他属性，并通过 `${属性名}` 引用。

示例：

```xml
<properties>
    <junit.version>4.13.2</junit.version>
</properties>
```

引用属性：

```xml
<version>${junit.version}</version>
```

### 2. 聚合

**作用：简化多模块项目的构建。**

对聚合工程执行构建操作时，可以同步构建其包含的所有子工程。

### 3. 继承与聚合的注意事项

- 使用前提通常是**分模块开发**。
- 继承和聚合经常一起使用。
- 父工程一般不存放业务代码，可删除 `src` 目录。
- 父工程的打包方式通常设置为：

  ```xml
  <packaging>pom</packaging>
  ```

---

## 八、重点速记

> [!TIP]
> **Maven 三大核心作用：统一结构、依赖管理、项目构建。**

- `pom.xml`：Maven 项目的核心配置文件。
- Maven 坐标：`groupId + artifactId + version`。
- 修改依赖后：记得刷新 Maven 项目。
- 下载失败：检查配置 → 删除 `.lastUpdated` → 重新刷新。
- `scope` 决定依赖在哪些阶段可用、是否参与打包。
- 依赖冲突优先记住：**路径近者优先；路径相同时声明在前者优先**。
- `mvn clean package`：清理后重新打包。
- `dependencyManagement`：统一管理版本，不会自动引入依赖。
- 继承用于统一配置，聚合用于批量构建。

---

> 本文根据思维导图 **《Maven(day44)》** 整理，并对原有知识点进行了结构化归纳与文字规范化。
