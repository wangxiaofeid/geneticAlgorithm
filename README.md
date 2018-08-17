# genetic

> A Vue.js project

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).

遗传算法（Genetic Algorithm）是模拟达尔文生物进化论的自然选择和遗传学机理的生物进化过程的计算模型

1. 随机生成200个三角形（基因），数据形式为[x1, y1, x2, y2, x3, y3, color]，坐标为偏移量，范围在-20到20之间
2. 首次生成100个样本（个体-种群），数据形式为[[x, y], ...]，坐标为三角形对应的基础坐标
3. 根据样本绘图，拿到像素点数据跟原图进行像素对比，得到种子分数
4. 淘汰分数较高的样本
5. 在当前50个样本的基础上通过交叉、变异生成50个新样本增加到样本库
6. 重复3操作
