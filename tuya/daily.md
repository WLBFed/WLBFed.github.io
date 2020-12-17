# Daily Work

1. 从Taro源码到跨三端同构的预研

## 2020.12.14 - 2020.12.18
1. 声光报警小夜灯交付
2. 项目模板问题整理
3. 管理平台server端优化
   1. 接口整理与单测
   2. code review项目优化


2020.12.14
1. wifi声光报警小夜灯公版交付，发布测试。 [wiki](https://wiki.tuya-inc.com:7799/pages/viewpage.action?pageId=49369529)
2. wifi声光报警小夜灯-代码review与优化
3. [流体渐变技术评估需求](https://wiki.tuya-inc.com:7799/pages/viewpage.action?pageId=58825540)
   1. [GCanvas](https://github.com/alibaba/GCanvas/tree/%40bugfix/react-native-ios/GCanvas/docs)-RN API测试一遍，评估有无风险。
   2. 折木之前接过，询问一下看一下有没有问题。
4. Dashboard项目需求交付给米修。


2020.12.15
1. wifi声光报警小夜灯公版问题修复
   1. 项目模板问题整理
2. 管理平台server端优化
   1. 接口整理与单测
   2. code review项目优化


项目模板问题：

1. 代码层面 3.0问题，4.0无问题
   1. withProvider与withUIConfig两个高阶不太友好，需要配置的地方不明确，提供了withProvider和composeLayout能力有点重合。
   2. 模板的ts项目any太多不如直接js，基本所有入参都是any，这块在4.0基本完善掉。
   3. 新建项目用的照明/3.0，用的dragon是在不知道怎么配置这个rules、formaters。

```
	
export default {
  // 开启节流
  openThrottle: true,
  // updateValidTime: 'syncs', // 同步更新 dp 数据
  /**
   * 下发数据时，检测当前值，若当前值已经是下发的数据，则过滤掉
   */
  checkCurrent: true,

  // 下发规则
  rules: [
    {
      type: 'NEED',
      conditionType: 'OR',
      condition: [sceneCode],
      effect: { [powerCode]: true },
    },
    {
      type: 'NEED',
      conditionType: 'OR',
      condition: [brightCode, temperatureCode],
      effect: { [workModeCode]: WORKMODE.WHITE },
    },
    {
      type: 'NEED',
      conditionType: 'AND',
      condition: [colourCode],
      effect: { [workModeCode]: WORKMODE.COLOUR },
    },
    {
      type: 'NEED',
      conditionType: 'AND',
      condition: [sceneCode],
      effect: { [workModeCode]: WORKMODE.SCENE },
    },
  ],
  // 场景转化插件
  formaters: [
    new SceneFormater(sceneCode),
    {
      uuid: countdownCode,
      parse(value: number) {
        return Math.floor(value / 60);
      },
      format(value: number) {
        return value * 60;
      },
    },
  ],
  // dp 转化规则
  dpMap: {
    [colourCode]: [
      {
        name: 'hue',
        length: 4,
        default: 0,
      },
      {
        name: 'saturation',
        length: 4,
        default: 1000,
      },
      {
        name: 'value',
        length: 4,
        default: 1000,
      },
    ],
    [controlCode]: controllMap,
    [musicCode]: controllMap,
  },
};
```

2. git流：版本并入husky、lint-staged，避免一些无意识的crash。
   遇到问题：空解构crash
```
"husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
"lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "prettier --write '**/*.{js,jsx,ts,tsx}'",
      "eslint --fix '**/*.{js,jsx,ts,tsx}'",
      "git add"
    ]
  },
```

3. 流程问题
	1. 需要更明确用哪个模板开发，是否要给面板和公版单独分类。（先选公版|面板）
	```
	? 选择模板品类 (Use arrow keys)
		❯ 电工
		安防传感
		其他
		其他
		照明
	```
	2. 公版测试与上线流程不太明确
	3. 打包的时候先要去git手动关闭MR，在填入libra表单？ 
      	1. ci掉，通过指定的commit msg自动发布？合掉MR自动发布？