
基于D3.js的交互式加权评分花瓣图可视化，允许用户调整不同数据指标的权重，并观察这些变化如何影响不同实体的总体评分。这个可视化工具的灵感来自OECD Better Life Index。

此项目灵感来源于[OECD Better Life Index](https://www.oecdbetterlifeindex.org/)。

[![Demo](https://img.shields.io/badge/Demo-weighted--flower--chart-purple)](https://tianze-hou.github.io/weighted-flower-chart/)

## 快速开始
若要将你自己的数据接入本项目，可按如下简单步骤进行操作：

1. **准备数据**：确保你的数据结构与现有示例一致。数据需包含一个`entities`数组，其中每个对象应具有`name`和`scores`属性。`scores`需包含类别以及对应的分数。例如：

```javascript
const data = {
    "entities": [
        { "name": "国家 A", "scores": { "类别 1": 80, "类别 2": 70,... } },
        { "name": "国家 B", "scores": { "类别 1": 90, "类别 2": 60,... } },
        // 更多国家
    ]
};
```

2. **替换数据**：在项目的 JavaScript 文件`script.js`开头部分找到“const data = {... }”部分，将其替换为你的数据对象。要确保所有对象的“scores”包含相同的类别，这样滑块和可视化才能正常工作。

3. **调整类别**：如果你的数据包含与演示数据不同的类别数量，代码会依据类别数量自动生成滑块和花瓣，无需手动调整其他部分。

### 注意
- 由于代码能够自动适应不同的项目、类别和数量，**你可以使用任何数据**，而不一定只能是国家的评价——只要你保证**每一行的数据类别**是相同的，在示例数据中即即“**从哪几个几个角度评价每个国家**”必须相同，否则程序将默认忽略那些只出现在部分行的数据类别。
- 以下是一个可行的例子（数据由大语言模型生成，不代表作者观点，下同）：
```javascript
const data = {
    "entities": [
        { "name": "《战争与和平》", "scores": { "文学价值": 90, "思想深度": 85, "人物塑造": 88 } },
        { "name": "《巴黎圣母院》", "scores": { "文学价值": 88, "思想深度": 82, "人物塑造": 85 } },
        { "name": "《红与黑》", "scores": { "文学价值": 85, "思想深度": 80, "人物塑造": 83 } },
        // 更多世界名著
    ]
};
```

- 以下这个例子看起来能够被正常读取，但实际上程序会按照第一行的数据类别依次读取后面每一行的数据，比如第三行后两项“心理描写”与“社会反映”在图表中也会被当成第一行的最后两项“情节张力”与“历史感”。
```javascript
const data = {
    "entities": [
        { "name": "《战争与和平》", "scores": { "文学价值": 90, "思想深度": 85, "人物塑造": 88, "情节张力": 85, "历史感": 90 } },
        { "name": "《巴黎圣母院》", "scores": { "文学价值": 88, "思想深度": 82, "人物塑造": 85, "情节张力": 80, "艺术美感": 88 } },
        { "name": "《红与黑》", "scores": { "文学价值": 85, "思想深度": 80, "人物塑造": 83, "心理描写": 85, "社会反映": 88 } },
        { "name": "《傲慢与偏见》", "scores": { "文学价值": 82, "思想深度": 78, "人物塑造": 85, "爱情描写": 90, "社会风俗": 85 } },
        // 更多世界名著
    ]
};
```

- 由于上述原理，下面这个例子某种程度上讲也可以"正常运行"，但这是一种不规范的做法，仅用于辅助理解代码读取数据的方式：
```javascript
const data = {
    "entities": [
        { "name": "《战争与和平》", "scores": { "文学价值": 90, "思想深度": 85, "人物塑造": 88, "情节张力": 85, "历史感": 90 } },
        { "name": "《巴黎圣母院》", "scores": { "a": 88, "b": 82, "c": 85, "44444": 85, "2147483647": 90 } },
        { "name": "《红与黑》", "scores": { "whatever": 85, "iLikeIceCream": 80, "supercalifragilisticexpialidocious": 83, "hello_world": 85, "JonKleinbergIsStillRebelKing": 90 } },
        // 更多世界名著
    ]
};
```

## 实现原理
本项目使用 D3.js 实现，通过以下几个关键步骤来完成图表绘制和更新：

**一、数据解析与比例尺设置**：
 - **X 轴**：使用 d3.scaleBand 将国家名称映射到图表的水平位置，保证每个国家都有足够的空间展示其数据。
 - **Y 轴**：通过 d3.scaleLinear 根据所有评分的最小值和最大值确定比例尺，以适应不同的数据范围。
 - **花瓣长度**：另一个线性比例尺 petalLengthScale 将评分转换为花瓣的长度。

**二、绘制基本元素**：
 - **X 轴与网格线**：在 SVG 中绘制 X 轴，添加水平网格线。
 - **国家分组**：为每个国家创建一个分组`<g>`元素，包含加权分数线、花瓣和中心点。

**三、加权总分计算与绘制**：
 - **权重调整**：滑块允许用户动态调整各类别的权重。
 - **加权总分**：每个国家的加权总分根据当前权重计算，绘制柱状图。

**四、花瓣图绘制**：
 - **角度计算**：根据类别数量计算花瓣夹角，使花瓣均匀分布在360度的一圈上。
 - **花瓣绘制**：每个类别的评分通过线条长度和颜色表示，花瓣从加权总分的位置辐射出去。

**五、交互与更新**：
 - **滑块控制**：每个类别对应一个滑块，用户调整滑块值时会触发 updateWeights 函数，重新计算权重并实时更新图表。
 - **动态更新**：通过 D3 的数据绑定和更新机制，图表元素（如加权线条和花瓣）会根据新的权重和评分实时重新绘制（看起来是在移动），确保交互流畅。

## 免责声明

本项目及文档使用的示例数据均为大语言模型随机生成，仅用于演示功能，不代表任何现实中的国家或书籍评分或排名，亦不代表作者观点。

## 许可证

此项目基于MIT许可证开源 - 详情参见[LICENSE](LICENSE)文件。

## 鸣谢

- 灵感源自[OECD Better Life Index](https://www.oecdbetterlifeindex.org/)。
- 使用D3.js构建。
