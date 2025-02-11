# 最佳实践和建议

!!!注意，在“工作中” ，此文档页面不完整

本节提供一般建议和最佳惯例，以确保您的代码库对您的团队来说是健康和可读的。这些建议和最佳惯例都是可选的，如果遵循它们，将提高代码的可读性和整洁性。

- 您的房间应尽可能保持低等级，而将特定于游戏的功能委托给其他可组合结构。
- 可同步的数据结构应尽可能较小
    - 理想情况下，每个扩展 `Schema` 的类应该只有字段定义。
    - 可以实现自定义 getter 和 setter 方法，只要其中没有游戏逻辑即可。
- 应该由其他结构处理游戏逻辑，例如：
    - [命令模式](#the-command-pattern)
    - 这是一种[实体-组件系统](#entity-component-system-ecs)。

## 单元测试

> TODO：我们需要提供一个 `@colyseus/testing` 包，以轻松模拟 `Room` 类，并触发其生命周期事件，以及创建虚拟客户端。

## 设计模式

### 命令模式

**為什麼？**

- 模型 ([`@colyseus/schema`](https://github.com/colyseus/schema)) 应主要包含数据，而不包含复杂的游戏逻辑。
- 房间代码应尽可能少，并将动作转移到其他结构

**命令模式具有多项优势，例如：**

- 它将调用操作的类与知道如何执行操作的对象进行分离。
- 它允许通过提供队列系统来创建命令序列。
- 很容易实现扩展以添加新命令，无需更改现有代码即可完成。
- 严格控制调用命令的方式和时间。
- 提高代码可读性和单元测试可能性。

#### 用法

安装

``` npm install --save @colyseus/command ```

在房间实现中初始化\`调度程序`：

```typescript fct\_label="TypeScript" import { Room } from "colyseus"; import { Dispatcher } from "@colyseus/command";

import { OnJoinCommand } from "./OnJoinCommand";

class MyRoom extends Room<YourState> { dispatcher = new Dispatcher(this);

  onCreate() { this.setState(new YourState()); }

  onJoin(client, options) { this.dispatcher.dispatch(new OnJoinCommand(), { sessionId: client.sessionId }); }

  onDispose() { this.dispatcher.stop(); } } ```

```typescript fct\_label="JavaScript" const colyseus = require("colyseus"); const command = require("@colyseus/command");

const OnJoinCommand = require("./OnJoinCommand");

class MyRoom extends colyseus.Room {

  onCreate() { this.dispatcher = new command.Dispatcher(this); this.setState(new YourState()); }

  onJoin(client, options) { this.dispatcher.dispatch(new OnJoinCommand(), { sessionId: client.sessionId }); }

  onDispose() { this.dispatcher.stop(); } } ```

命令实现看起来是这样的：

```typescript fct\_label="TypeScript" // OnJoinCommand.ts import { Command } from "@colyseus/command";

export class OnJoinCommand extends Command<YourState, { sessionId: string }> {

  execute({ sessionId }) { this.state.players\[sessionId] = new Player(); }

} ```

```typescript fct\_label="JavaScript" // OnJoinCommand.js const command = require("@colyseus/command");

exports.OnJoinCommand = class OnJoinCommand extends command.Command {

  execute({ sessionId }) { this.state.players\[sessionId] = new Player(); }

} ```

#### 参阅更多

- 参见 [command definitions](https://github.com/colyseus/command/blob/master/test/scenarios/CardGameScenario.ts)
- [用法：](https://github.com/colyseus/command/blob/master/test/Test.ts)
- 参见 [implementation](https://github.com/colyseus/command/blob/master/src/index.ts)

### 实体组件系统 (ECS)

目前我们没有正式的 ECS（实体组件系统），尽管我们已经看到社区成员实现了他们自己的解决方案。

!!!警告“待测试性非常高” 某些作品[已经开始尝试将 ECSY 与 @colyseus/schema 相结合](http://github.com/endel/ecs)。
