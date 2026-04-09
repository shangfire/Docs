# 背景介绍

自从2023年以来由ChatGPT引发的AI浪潮席卷全球，各种AI产品层出不穷，用户体验也日益丰富，本文将介绍一些常见的AI工具，并给出一些使用建议。

# AI工具类型

### Web端

- [ChatGPT](https://chatgpt.com/)

- [Gemini](https://gemini.google.com/app)

- [DeepSeek](https://chat.deepseek.com/)

- [豆包](https://www.doubao.com/chat/)

### 移动端

- Icons

![](https://raw.githubusercontent.com/shangfire/img/main/img/20260409221312.png)

### 桌面端（cli）

- ClawdeCode

![](https://raw.githubusercontent.com/shangfire/img/main/img/20260409221503.png)

- OpenCode

![](https://raw.githubusercontent.com/shangfire/img/main/img/20260409221552.png)

### 桌面端（app）

- OpenCode

![](https://raw.githubusercontent.com/shangfire/img/main/img/20260409223706.png)

- OpenWork

![](https://raw.githubusercontent.com/shangfire/img/main/img/20260409224821.png)

### Web端（app）

- OpenClaw

![](https://raw.githubusercontent.com/shangfire/img/main/img/20260409225344.png)


### IDE插件

- Coplilot

![](https://raw.githubusercontent.com/shangfire/img/main/img/20260409225553.png)

### 独立IDE

- Cursor

- Antigravity

- Kiro

- Trae

### 效果演示

1. 使用OpenWork或者OpenClaw生成一个三维模型
提示词：在我的E:\test文件夹中新建一个三维模型，两个球体略微相交，置于平面上

2. 使用OpenCode从0开始生成一个完整的项目
提示词：在我的E:\test文件夹中新建一个vs studio 2026的C++项目，用于演示ZMQ的通信方式。注意：我本地没有ZMQ库，你要负责在项目中安装和配置ZMQ库。

3. 使用Copilot Agent修改OpenCode完成的项目
提示词：为项目里所有的头文件添加符合doxygen规范的注释

### LLM基础知识

在讲解Agent为什么能做到这些事情之前，我们需要先了解一下LLM的基础知识，LLM全称是Large Language Model，中文叫做大语言模型，它是一种基于深度学习的自然语言处理模型，它能够理解人类的语言，并生成人类能够理解的语言。关于LLM的基础知识，由以下几个视频来讲解：

1. 什么是神经网络
<iframe src="https://player.bilibili.com/player.html?isOutside=true&aid=114240644060353&bvid=BV1MNoRYEEVM&cid=29118171120&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="500px"></iframe>

2. 什么是Rnn
<iframe src="//player.bilibili.com/player.html?isOutside=true&aid=114240644060353&bvid=BV1MNoRYEEVM&cid=29118171120&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="500px"></iframe>

3. 什么是Transformer
<iframe src="//player.bilibili.com/player.html?isOutside=true&aid=114328522984185&bvid=BV1C3dqYxE3q&cid=29401417977&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"></iframe>

总结：
- 神经网络是一种模拟人脑神经元结构的计算模型，通过堆接受训练数据求解激活函数参数，从而在后续的使用中接受真实的输入并输出预测结果。当参数足够多、训练数据足够大时，神经网络的效果看起来就像是具有了智能。
- Rnn是一种特殊的神经网络，它可以处理序列数据，比如文本。
- Transformer是一种特殊的Rnn，它可以并行处理序列数据，并生成新的序列数据。

### Agent工作原理

LLM刚推出的时候非常惊艳，人们可以向它提问各种问题并得到解答，但是随着时间的推移，人们不再满足与LLM只是能够回答问题，还希望LLM能够帮助人们完成各种任务，比如写代码、生成图片、生成视频、执行脚本等等。为了实现这个目标，人们做了以下的努力来改进LLM：

1. 中间件
为了让LLM能够提供上述的服务，需要一个中间件来完成与LLM的交互，这个中间件叫做Agent，它能够将用户的问题加工、打包成转化为LLM能够理解的问题，并将LLM的回答转化为用户能够理解的结果。

2. 输出控制
为了让agent能够理解LLM的答案，首先需要规范LLM的答案。比如，agent规定LLM回答问题时必须使用json格式来回答，必须具有特定字段，比如result、success、status、message、data等等。这样，agent就能够理解LLM的答案了。

3. MCP（LLM的手）
为了让LLM能够做事，我们需要预先定义一些方法，并告诉LLM我们有若干工具方法，调用的方式是怎样的，并询问LLM，为了完成用户的需求，需不需要调用这些方法，如果需要，应该如何调用，调用后如何处理结果。这些方法叫做MCP，全称是Model Context Protocol，中文叫做方法调用协议。
一个典型的MCP代码如下：
```ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({ name: "SystemInfo", version: "1.0.0" });

server.tool("get_system_status", 
  { platform: z.string().describe("操作系统平台") },
  async ({ platform }) => ({
    content: [{ 
      type: "text", 
      text: platform.includes("mac") ? "Intel Mac 2019" : "Windows Dev Environment" 
    }]
  })
);

// 绑定传输层
const transport = new StdioServerTransport();
await server.connect(transport);
```
关于该MCP的使用，与LLM的对话流程可能如下：
- 用户向LLM提问，比如：获取我的系统信息
- agent接收到用户提问后，转发给LLM该问题，并告诉LLM自己有这么一个名为SystemInfo的工具，可以获操作系统信息，并询问LLM是否需要调用该工具
- LLM回答：需要调用该工具，并告诉agent，调用该工具需要传入一个参数，参数名为platform，参数类型为string，参数描述为操作系统平台
- agent接收到LLM的回答后，调用SystemInfo工具，并传入参数platform，参数值为mac
- agent接收到SystemInfo工具的返回结果后，将结果转化为用户能够理解的结果，并返回给用户

4. Skills（LLM的指导建议者）
有时候关于复杂场景，可能需要告诉LLM一些指导建议，它本质上是一份包含特定领域知识、操作规程和工具使用偏好的说明书。通过这种方式，我们可以像写文档一样，赋予 LLM 处理复杂业务流程的能力。

一个典型的 Skills 代码（即 .md 文件内容）如下：
```md
# Skill: UVCDebugger
# Description: 专门用于处理 UVC 摄像头开发中的跨平台差异验证与调试

## 1. 核心流程
当用户遇到摄像头无法打开或黑屏时，请按以下步骤操作：
1. 调用 `get_system_status` 确认系统环境（Win/Mac）。
2. 调用 `list_devices` 获取设备路径。
3. 如果是 Windows，优先检查 Media Foundation 权限；如果是 Mac，检查权限描述文件。

## 2. 工具调用规范
- 在调用 `capture_frame` 时，必须传入 `width` 和 `height` 参数。
- 如果返回码为 -1，请调用 `get_ffmpeg_error` 翻译错误信息。

## 3. 注意事项
- 优先处理灰度图（Grayscale）转换逻辑的报错。
- 所有的日志信息需按格式输出，方便开发者对比系统差异。
```
每次调用LLM时，agent会自动将Skills的内容插入到LLM的输入中，这样LLM就能够理解Skills的内容了。

MCP和skills有很多网站可以获取：
https://clawhub.ai/
https://mcp.directory/

5. AgentLoop
在实际使用中，agent针对我们的问题，内部会像deepseek一样，反复拆解，反复自我验证，以确保任务切实有效的执行完成
```ts
async function agentLoop(userPrompt: string, skillDoc: string) {
  let context = `技能文档：${skillDoc}\n用户需求：${userPrompt}`;
  
  while (true) {
    // 1. 决策阶段：LLM 阅读上下文（包括 Skill）决定下一步
    const decision = await llm.chat([
      { role: "system", content: "你是一个拥有 MCP 工具的 Agent，请参考技能文档操作。" },
      { role: "user", content: context }
    ]);

    // 2. 终止条件：LLM 认为任务已完成
    if (decision.type === "final_answer") {
      console.log("任务完成:", decision.content);
      break;
    }

    // 3. 执行阶段：根据 LLM 意图，真正通过 MCP 调用底层方法
    if (decision.type === "call_tool") {
      const toolResult = await mcpServer.execute(decision.toolName, decision.args);
      
      // 4. 反馈阶段：将执行结果喂回上下文，进入下一轮 Loop
      context += `\n工具执行结果 [${decision.toolName}]: ${JSON.stringify(toolResult)}`;
    }
  }
}
```

### 让Agent深入到软件工程
现在的Agent在工作区基本都有Agent.md文档用于指导、缓存、记录Agent的工作。我们在开发时，通过让Agent生成各种设计文档，一方面用于规范性引导，一方面也是用来缓存之前的工作成果，让Agent能够记忆之前的工作内容，从而在后续的工作中，能够更好的完成工作。

### Agent工具推荐
- OpenCode
- Coplilot