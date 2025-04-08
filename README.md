# XSS-Detector: Burp Suite XSS漏洞检测扩展

[![JAVA](https://img.shields.io/badge/Java-11%2B-brightgreen.svg)](https://www.oracle.com/java/)
[![Burp Suite](https://img.shields.io/badge/Burp%20Suite-2021+-orange.svg)](https://portswigger.net/burp)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)




## 架构概览

XSS-Detector是一款面向Burp Suite的扩展插件，专注于检测Web应用中的跨站脚本(XSS)漏洞。本插件采用多线程并发设计，通过精确的请求-响应-漏洞映射关系，提升测试效率和结果准确性。



### 核心技术栈

- **Java并发框架**：采用`ExecutorService`与`Semaphore`进行请求并发控制
- **Burp Extender API**：集成Burp Suite功能，处理HTTP请求/响应
- **线程安全数据结构**：使用`AtomicInteger`和`ConcurrentHashMap`确保数据一致性
- **基于UUID的唯一标识系统**：实现请求与漏洞记录的精确关联
- **Swing UI组件**：双面板界面设计，清晰呈现测试结果

## 界面使用指南
[![UI-1](https://github.com/Deep0/XSS-Detector/blob/main/UI-1.png?raw=true)](UI1)
[![UI-2](https://github.com/Deep0/XSS-Detector/blob/main/UI-2.png?raw=true)](UI2)
### 1. 配置面板

位于插件顶部的标签页，提供核心配置选项：

#### 1.1 Payload配置

* **功能**：自定义XSS测试载荷列表
* **操作**：在文本区域中每行输入一个payload
* **语法**：支持标准HTML、JavaScript的XSS向量
* **示例**：`<script>alert(1)</script>`, `"><img src=x onerror=alert(1)>`
* **最佳实践**：根据目标应用类型选择适合的payload，避免过于复杂的向量

#### 1.2 范围控制

* **白名单配置**：
  * **格式**：每行一个正则表达式
  * **匹配规则**：URL与任一正则匹配则进行测试
  * **示例**：`^https://example\.com/.*`

* **黑名单配置**：
  * **格式**：每行一个正则表达式
  * **匹配规则**：URL与任一正则匹配则跳过测试
  * **优先级**：黑名单规则优先于白名单规则

* **Burp作用域集成**：
  * 勾选"使用Burp作用域"可直接采用Burp已配置的目标范围
  * 与白名单可同时使用，形成"或"的关系

#### 1.3 并发设置

* **最大并发任务**：调整同时发送的HTTP请求数
* **调优建议**：
  * 高性能服务器：设置为10-20
  * 普通应用：设置为5-10
  * 脆弱系统：设置为1-3

#### 1.4 操作按钮

* **保存配置**：将当前配置保存到文件系统，支持跨会话使用
* **加载配置**：从文件系统读取已保存的配置
* **清除数据**：重置插件状态，清空所有记录和结果

### 2. 请求列表（左侧表格）

显示被拦截和处理的HTTP请求：

* **列说明**：
  * **URL**：请求的目标地址
  * **方法**：HTTP方法（GET、POST等）
  * **参数**：包含的参数列表
  * **状态码**：HTTP响应状态码
  * **测试状态**：当前请求的测试进度与结果

* **交互操作**：
  * **单击选择**：显示请求/响应详情，并在右侧显示相关漏洞
  * **右键菜单**：提供额外操作（如重发请求、复制URL等）
  * **列排序**：点击列标题进行升序/降序排序

* **状态指示器**：
  * **待测试**：请求尚未进行XSS测试
  * **测试中...**：正在进行XSS测试
  * **无漏洞**：测试完成，未发现漏洞
  * **!!Vulnerable!!**：发现潜在XSS漏洞

### 3. 发现问题列表（右侧表格）

显示与选中请求相关的潜在XSS漏洞：

* **列说明**：
  * **参数**：存在潜在漏洞的参数名
  * **Payload**：触发漏洞的测试载荷
  * **数据包长度**：响应包大小
  * **存在**：漏洞确认状态（✓表示确认存在）
  * **响应码**：对应响应的HTTP状态码

* **颜色指示**：
  * **橘黄色行**：表示高置信度的漏洞（响应中包含完整payload）
  * **普通行**：表示低置信度的可能漏洞或测试记录

* **交互操作**：
  * **单击选择**：在下方显示包含漏洞的请求和响应详情
  * **双击操作**：快速跳转到响应中payload位置

### 4. 请求/响应查看器（底部面板）

展示选中项的HTTP请求和响应内容：

* **请求查看器（左侧）**：
  * 显示原始HTTP请求，包括参数和载荷
  * 支持语法高亮，便于分析
  * 支持文本查找功能（Ctrl+F）

* **响应查看器（右侧）**：
  * 显示完整HTTP响应，包括头部和正文
  * 自动高亮显示payload位置
  * 支持多种查看模式（原始/渲染/十六进制）

* **辅助分析功能**：
  * **高亮匹配**：自动标记响应中与payload匹配的部分
  * **上下文分析**：显示payload周围的HTML/JS上下文

## 工作流程指南

1. **初始配置**：
   * 设置合适的payload列表
   * 配置目标范围（白名单/黑名单）
   * 调整并发参数

2. **获取请求**：
   * 通过Burp代理浏览目标应用
   * 或从Burp其他工具（如Spider）导入请求

3. **执行测试**：
   * 点击请求列表中的项目开始测试
   * 观察测试状态变化和进度

4. **分析结果**：
   * 查看发现问题列表中的潜在漏洞
   * 通过请求/响应查看器验证漏洞的真实性
   * 特别关注橘黄色高亮的高置信度漏洞

5. **导出报告**：
   * 使用右键菜单将发现的漏洞导出为HTML/CSV格式
   * 或将漏洞发送到Burp的Issue活动面板

## 高级技巧

* **批量测试**：选中多个请求同时进行测试，提高效率
* **自定义策略**：针对不同类型的应用调整payload列表
* **筛选结果**：使用表格过滤功能快速定位特定类型的漏洞
* **会话管理**：对需要认证的应用，确保测试请求包含有效Cookie

## 技术实现细节

### 并发测试框架

采用Semaphore控制并发请求数量，避免目标系统过载：

```java
private Semaphore requestSemaphore;  // 并发请求控制器
private AtomicInteger activeTaskCount = new AtomicInteger(0);  // 原子任务计数器
private ExecutorService threadPool;  // 线程池管理器
```

通过`maxConcurrentTasks`参数控制并发度，用户可根据目标系统特性调整并发数量，平衡测试速度与系统负载。

### XSS检测机制

插件通过参数注入自定义payload，然后分析响应内容判断是否存在潜在XSS漏洞：

```java
// 注入XSS payload并检查响应
if (responseBody.contains(payload)) {
    // 潜在的XSS漏洞
    String issue = "可能的XSS漏洞 - 参数: " + paramName + ", Payload: [" + payload + "]";
    vulnerableRecordsMap.put(issueKey, record);
}
```

插件支持自定义payload列表，可根据不同场景需求调整测试向量。测试结果通过表格清晰展示，支持按参数和payload分类。

### 数据关联机制

采用基于UUID的关联系统，确保请求、响应和漏洞记录间的精确映射：

```java
private Map<String, RequestResponseRecord> vulnerableRecordsMap = new HashMap<>();
private Map<Integer, String> rowToVulnIdMap = new ConcurrentHashMap<>();
```

每个漏洞记录通过唯一ID前缀与描述性键值关联，实现高效查找和精确显示。此设计解决了多线程环境下的数据关联问题，确保UI展示与后台数据一致。

## 系统需求

- JDK 11+
- Burp Suite Professional/Community 2020.12+
- 最小内存: 2GB (推荐4GB)

## 构建与安装

使用Gradle构建系统：

```bash
./gradlew build
```

生成的JAR文件可直接在Burp Suite的Extender面板中加载。

---

*XSS-Detector 通过并发测试和精确的数据关联机制，为Web应用安全测试提供可靠的XSS漏洞检测能力。* 
