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

## 3. RN JSI

目前，React Native 新架构仍处于开发阶段，网上的资料比较少。提到新架构，会出现 Fabric、TurboModules、CodeGen、JSI 等术语，让人眼花缭乱

1. Fabric: RN 的 UI 重构
2. CodeGen: 代码生成器，与 React Native Modules 相关，规范其接口
3. TurboModules：与 React Native Modules 相关，基于 JSI
4. JSI：JavaScript 与 Java/ObjeC/C++ 相互调用的一种机制

JavaScript 引擎能够执行 JavaScript 代码，以 JavaScriptCore 为例。JavaScriptCore 提供了一系列 C/C++ API 供上层调用。其中有一个方法名为 evaluateScript，接收一个字符串。这里的字符串就是 JavaScript 代码。

以 React Native Bundle 为例，我们将其从磁盘中读出取来，变成内存中的一段字符串，正是通过传入 evaluateScript，使 JavaScriptCore 执行代码。

除了 evaluateScript 接口之外，JavaScriptCore 还提供一种 API：向 JavaScriptCore 注册用 C 编写的回调，将 C 函数映射为一个 JavaScript 函数。

一旦注册成功后，当 JavaScript 代码中调用这个函数，对应的 C 函数就会被触发。这将 JavaScript 世界与原生世界打通了。

各种动态化技术都是基于此实现，React Native 也不例外。

向 JavaScriptCore 注册用 C 编写的回调的 API，JSI 是基于这些 API 封装出来的框架。有了这个框架，我们就能更方便地在 JavaScript 世界和 Native 世界之间建立映射，而不必每次都从底层 API 搞起。
JSI 的另一优势是它抹平了 JavaScript 引擎的差异。使用 JSI，我们不必关心底层是 Hermes 引擎还是 JavaScriptCore 引擎，JSI 底层都消化了。因此只需要基于 JSI 的接口编写即可。

## 4. Turbo Module

引入了 Turbo Module，是对 Native Module 的全面升级。

初始化速度：React Native 首次启动时会先加载全部 Native Modules（不管是否真正用到），这会拖慢启动速度。
接口一致性：Native Module 所提供的接口，在 Native 侧和 JavaScript 侧各有一份，两者必须严格一致。但一致依赖缺少保障机制。
单例：Native Module 采用单例设计，生命周期过长。
多余运行时操作：Native Module 收集在运行时扫描，这一步其实没有必要放在运行时。
反射：Native Module 中的方法通过运行时反射调用。这个反射可以被优化。

- TurboCxxModule：将原有的 CxxModule 转换为 TurboModule。兼容老架构用的。
- JavaTurboModule：在 React Native 新架构中，Android 平台下基于 Java 的 Native Module 都基于此类。可以理解成将 Java 映射到 JavaScript。
- ObjCTurboModule：类似地，在 iOS 平台下将 ObjC Native Module 映射到 JavaScript。

我们说到 Turbo Module 是一个基类，通过它能分化出不同种类。Java Turbo Module 就是其分化出的一种。

Turbo Module 的功能是将 C++ 模块映射到 JavaScript 模块。

Java Turbo Module 再次基础上，实现了将 Java 模块映射到 JavaScript 模块。
具体来说：Java ⇌ C++ ⇌ JavaScript。
