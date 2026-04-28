# cursor：AI编程助手
### 概括
Cursor 是一个基于人工智能的编程助手，提供智能代码补全、错误检测和自然语言处理功能，帮助开发者更高效地编写和优化代码。它支持多种编程语言，并与主流 IDE 集成，适用于初学者和经验丰富的开发者
### 下载
官网下载即可：https://www.cursor.com/

# Cursor的核心功能
### 特色功能
- cursor tab：代码自动补全与修改，与其他ai工具不同的、最具价值的是光标预测，也就是所谓的tab-tab-tab。

- 光标预测：在写代码过程中，cursor中的光标可以通过预测轻松推理出下一个要编辑修改的点

### 辅助编程
主界面右上角打开左右侧边栏
- Chat：基本的问答ai
- Composer：问答+给出指令并执行指令
- 选中部分代码：选中代码后使用快捷键Ctrl+L进入Chat；使用Ctrl+K进入Composer



### 修改使用的ai模型
设置的modols中修改或添加模型，gpt4、o1-mini等（推荐gpt4o和Claude-3.5-sonnet，前者综合问答能力强，后者在代码生成领域更优秀，二者的问答次数限量），在chat/composer的左下角切换模型

### Composer下的normal/agent模式
- normal：可以检索代码库、文档，创建、写入文件
- agent：除了创建文件，还可以自动提取相关的上下文、运行终端命令、按照语义搜索代码、执行文件操作，只支持claude模型
- 根据复杂度使用composer的两个模式，如果针对简单的功能，使用normal，速度快；如果是一个复杂功能，应将它拆解后一步一步喂给composer，这时使用agent

### 选中文件、文件夹、代码、文档、Web、git
在Chat/Composer下输入@或在资源管理器中拖拽进来
- 选中后，即可将所选作为上下文，可以将某个项目的接口文档链接/需求文档链接，作为一个私有ai知识库
- docs文档：设置--features--Docs--Add new doc，如果在索引的url后加上斜杠/，则会抓取这个索引下的所有子页面
- Web：使用Web后，会自动查询网络上与描述相符的信息
- git：可以添加两次历史git提交来对比差异
- notepad：用于针对chat与composer不同上下文的问题
- Codebase:cursor会扫描整个项目，查到与指令相关的文件或者代码块，排序，推理，生成。为了确保将整个项目扫描全，可以在设置--features--codebase indexing点击resync index重新扫描

### Cursor Rules
- 全局rules：在设置--General--Rules for AI为Cursor的全局ai配置，常用：中文
- 项目的cursor rules：可以在项目的根目录下创建.cusorrules文件，如设置项目简介、目录结构、代码规范、组件范式、命名规范、git提交规范、国际化规范




# cursor的使用技巧
。
。
。
。
。
。


