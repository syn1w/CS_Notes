# 一、介绍

有时候想要更清晰地表达自己的想法，需要简单画一些结构图等等。网上搜了一下，有 SVG 和 Graphviz DOT，之前看了一下 SVG，学了一点点，感觉有点绕，应该更适合前端的同学。  

Clang 生成的 CFG、Exploded Graph 都是 DOT 格式的，应该更适合自己学习，而且 DOT 可以很方便的转换为 SVG 格式，当然也可以很方便导出其他格式的图片。  

安装配置等等杂项参考附录 A 其他人的内容  



# 二、Hello World

```dot
digraph G {
    Hello -> World;
    hello -> DOT;
}
```

![hello](../../imgs/dot/hello.png)  



# 三、语法

终结符以**粗体**显示，字符串常量用单引号 `''` 引起来，括号 `()` 表示分组，方括号 `[]` 表示可选项，竖线 `|` 表示选择其一  

|   非终结符   |                             语法                             |
| :----------: | :----------------------------------------------------------: |
|   *graph*    | [ **strict** ] (**graph** \| **digraph**) [*ID*] **'{'** *stmt_list* **'}'** |
| *stmt_list*  |              [ *stmt* [ **';'** ] *stmt_list* ]              |
|    *stmt*    | *attr_stmt* \| *node_stmt* \|  *edge_stmt*  \| *ID* **'='** *ID*\| subgraph |
| *attr_stmt*  |       (**graph** \| **node** \| **edge**) *attr_list*        |
| *attr_list*  |         **'['** [ *a_list* ] **']'** [ *attr_list* ]         |
|   *a_list*   |     *ID* '=' *ID* [ (**';'** \| **','**) ] [ *a_list* ]      |
| *node_stmt*  |                   *node_id* [*attr_list*]                    |
|  *node_id*   |                       *ID* [ *port* ]                        |
|    *port*    | **':'** *ID* [ **':'** *compass_pt* ] \| **':'** *compass_pt* |
| *edge_stmt*  |     (*node_id* \| *subgraph*) *edgeRHS* [ *attr_list* ]      |
|  *edgeRHS*   |       *edgeop* (*node_id* \| *subgraph*) [ *edgeRHS* ]       |
|   *edgeop*   |                           -> \| --                           |
|  *subgraph*  |    [ **subgraph** [ *ID* ] ] **'{'** *stmt_list* **'}'**     |
| *compass_pt* | (**n** \| **ne** \| **e** \| **se** \| **s** \| **sw** \| **w** \| **nw** \| **c** \| _) |

关键字解释：  

- **strict**：严格的图修饰，禁止创建多个相同的边
- **graph**：无向图
- **digraph**：有向图
- **n, e, s, w**：分别表示北、东、南、西，最终指向
- **ne, se, sw, nw** 分别表示东北、东南、西南、西北
- **c, _**： 分别表示中部和任意方向



*ID* 是下面的其中一项：

- 字母（`[a-zA-Z\200-\377]`）、下划线（`_`）、数字(`[0-9]`)字符组成的字符串，但是不以数字开头
- 数字 `[-]?(.[0-9]+ | [0-9]+(.[0-9]*)?)`
- 双引号引起来的字符串 `"..."`
- HTML 字符串 `<...>`



*compass_pt* 具体指箭头从哪个位置指向哪个位置  

具体的例子：

```dot
digraph G {
    node [shape = record,height=.1];

    node0 [label = "<head> head|<body> body|<foot> foot", height=.5]
    node2 [shape = box label="mind"]

    node0:head:n -> node2:n [label = "n"]
    node0:head:ne -> node2:ne [label = "ne"]
    node0:head:e -> node2:e [label = "e"]
    node0:head:se -> node2:se [label = "se"]
    node0:head:s -> node2:s [label = "s"]
    node0:head:sw -> node2:sw [label = "sw"]
    node0:head:w -> node2:w [label = "w"]
    node0:head:nw -> node2:nw [label = "nw"]
    node0:head:c -> node2:c [label = "c"]
    node0:head:_ -> node2:_ [label = "_"]

    node0:body[style=filled color=lightblue]
}
```

![direction](../../imgs/dot/direction.png)



注释类似于 C/C++ 的注释，`//` 和 `/* */`  

`;` 和 `,` 都不是必须的，可以使用 whitespace 代替  

`\` 类似与 C/C++，表示换行  





# 四、属性

所有的属性可以见[文档](https://graphviz.gitlab.io/_pages/doc/info/attrs.html)  

接下来学习使用常用的属性  

|    属性名     |                             说明                             |
| :-----------: | :----------------------------------------------------------: |
|  **charset**  |                     编码，一般设置 UTF-8                     |
| **fontname**  | 字体名称，这个在中文的情况需要设置，否则导出图片的时候会乱码 |
| **fontcolor** |                           字体颜色                           |
| **fontsize**  |                    字体大小，用于文本内容                    |
| **fillcolor** |         用于填充节点或者群组 (cluster) 的背景颜色。          |
|   **size**    |                     图形的最大宽度和高度                     |
|   **label**   |                       图形上的文本标记                       |
|  **margin**   |                        设置图形的边距                        |
|    **pad**    | 指定将绘制区域扩展到绘制图形所需的最小区域的长度（以英寸为单位） |
|   **style**   |                    设置图形组件的样式信息                    |
|  **rankdir**  | 设置图形布局的排列方向 (全局只有一个生效). "TB", "LR", "BT", "RL", 分别对应于从上到下，从左到右，从下到上和从右到左绘制的有向图 |
|  **ranksep**  |                以英寸为单位提供所需的排列间隔                |
|   **ratio**   |                     设置生成图片的纵横比                     |

`style` 可以选择的值和效果：

节点 `style`：

| style 值  |                             效果                             |
| :-------: | :----------------------------------------------------------: |
|   solid   | ![solid](https://graphviz.gitlab.io/_pages/doc/info/n_solid.png) |
|  dashed   | ![dashed](https://graphviz.gitlab.io/_pages/doc/info/n_dashed.png) |
|  dotted   | ![dotted](https://graphviz.gitlab.io/_pages/doc/info/n_dotted.png) |
|   bold    |  ![](https://graphviz.gitlab.io/_pages/doc/info/n_bold.png)  |
|  rounded  | ![rounded](https://graphviz.gitlab.io/_pages/doc/info/n_rounded.png) |
| diagonals | ![](https://graphviz.gitlab.io/_pages/doc/info/n_diagonals.png) |
|  filled   | ![filled](https://graphviz.gitlab.io/_pages/doc/info/n_filled.png) |
|  striped  | ![striped](https://graphviz.gitlab.io/_pages/doc/info/n_striped.png) |
|  wedged   | ![wedged](https://graphviz.gitlab.io/_pages/doc/info/n_wedged.png) |



边 `style`：

| style 值 |                             效果                             |
| :------: | :----------------------------------------------------------: |
|  solid   | ![solid](https://graphviz.gitlab.io/_pages/doc/info/e_solid.png) |
|  dashed  | ![dashed](https://graphviz.gitlab.io/_pages/doc/info/e_dashed.png) |
|  dotted  | ![style](https://graphviz.gitlab.io/_pages/doc/info/e_dashed.png) |
|   bold   | ![bold](https://graphviz.gitlab.io/_pages/doc/info/e_bold.png) |



`cluster style`：

| style 值 |                             效果                             |
| :------: | :----------------------------------------------------------: |
|  solid   | ![solid](https://graphviz.gitlab.io/_pages/doc/info/c_solid.png) |
|  dashed  | ![dashed](https://graphviz.gitlab.io/_pages/doc/info/c_dashed.png) |
|  dotted  | ![dotted](https://graphviz.gitlab.io/_pages/doc/info/c_dotted.png) |
|   bold   | ![bold](https://graphviz.gitlab.io/_pages/doc/info/c_bold.png) |
|   ...    |                             ...                              |



## 1. 节点

默认的节点属性是 `shape=ellipse,width=.75,height=.5` 并且 `lable` 是节点名  

一个图中可能有非常多的 node 和 edge，可以事先声明一个公共的属性，比如 

```dot
digraph G {
    node [shape=box color=blue];
    edge [color=red];
}
```

节点常用的属性：

|      属性名       |                             说明                             |
| :---------------: | :----------------------------------------------------------: |
|     **shape**     | 具体见[这里](https://graphviz.gitlab.io/_pages/doc/info/shapes.html)，常用有 `box, circle, ellipse,plaintext,square` |
| **width, height** | 图形的宽度和高度，如果设置了 **fixedsize** 为 true，则宽和高为最终的长度 |
|   **fixedsize**   |    如果为 false，节点的大小由其文本内容所需要的最小值决定    |
|     **rank**      | 子图中节点上的排列等级约束。最小等级是最顶部或最左侧，最大等级是最底部或最右侧。 |



## 2. 边






# 附录 A 参考资料

[DOT语言 wiki](https://zh.wikipedia.org/zh-hans/DOT语言)

https://graphviz.gitlab.io/doc/info/lang.html

https://www.graphviz.org/pdf/dotguide.pdf

https://www.cnblogs.com/shuqin/p/11897207.html

https://github.com/uolcano/blog/issues/13

