# react-native

## 1. 你们在 RN 的项目中做了哪些事儿

首先我们由两部分，一部分是萌推商家端，这是一个纯 RN 的项目。
另一部分是萌推 C 端 APP，C 端 APP 是一个部分模块使用 RN 来做的。

然后我们做了一个 RN 统一化平台，一个是提供一些 RN 使用的 SDK（音视频、推送、埋点）给集团其他有需要的团队使用。二是 RN 的拆包和热更方案与热更发布平台。

拆包：业务包和基础包的拆分。
RN 打包：首先是基于 metro 的，rn 的打包是使用 metro，这个打包流程主要分为几步，首先通过 metro 从入口点构建所需要的所有模块，然后所有模块经过转换成目标平台可以理解的格式，转化完成就会被序列化，序列化就是将各个模块组合成一个 JavaScript 文件包。

主要的函数： `createModuleIdFactory`

```
function createModuleIdFactory() {
  const fileToIdMap = new Map();
  let nextId = 0;
  return path => {
    let id = fileToIdMap.get(path);

    if (typeof id !== "number") {
      id = nextId++;
      fileToIdMap.set(path, id);
    }

    return id;
  };
}

module.exports = createModuleIdFactory;
```

逻辑比较简单，就是 map 里没有查到这个模块则 ID 自增，然后将模块记录到 map 中。所以官方生成的 moduleId 的规则就是自增，这里要替换成我们配置逻辑，拆包就是要保证这个 ID 不能重复，但是这个 ID 只在打包时生成，如果我们单独打业务包，基础包，这个 ID 的连续性就会丢失，对于 ID 的处理，我们参考了`react-native-multibundler`，每个包邮十万位间隔区间的划分，基础包从 0 自增，业务包从 1000000（100w）开始自增，又或者可以通过模块的路径或者 uuid 去分配，避免碰撞，但是字符串会增大包的体积，没有使用物理路径。

我们做的就是把 createModuleIdFactory 修改的逻辑拆出来封装成 SDK。

## 2. RN 启动流程

- Native 编译并启动
- 创建 JS 虚拟机环境
- 创建 bridge，拥有独立的 context js 运行环境，并负责原生和 js 线程的通信（通过不同 bridge 加载 js 代码，可以存在相同的全局变量，不会冲突）
- 通过 bridge 获取 js 线程来解析 js 代码（可以是远程包和离线包）
- 运行 js 代码，根据参数创建 RootView

RCTBridge 是对 JavaScriptCore 中 Bridge 的封装，每个 Bridge 都是一个独立的 js 环境。Bridge 在 RN 中起到承上启下的作用，负责原生和 js 线程的通信。
