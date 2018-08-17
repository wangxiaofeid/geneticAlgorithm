<template>
  <div id="app">

    <div>
      <canvas ref="canvas" class="canvas" width="100" height="100"></canvas>
    </div>

    <input type="file" name="" id="" @change="changeFile">

    <div>
      <button @click="start">start</button>
      <button @click="stop">end</button>
    </div>

    <div>
      <p>运行时间：{{time/1000}}秒</p>
      <p>衍生次数：{{num}}</p>
    </div>

    <ul>
      <!-- <li v-for="(item, index) in bestList" :key="index">
        <img :src="item.imgSrc" alt="">
      </li> -->
      <li>
        <img :src="bestImg.imgSrc" alt="" style="width: 300px">
      </li>
      <li>
        <img :src="file" alt="" style="width: 300px">
      </li>
    </ul>
  </div>
</template>

<script>
import _ from "lodash";
//配置常量
const width = 100, height = 100, opcity = 1, chartNum = 200, bledNum = 100;

const R = 100;
const angle = 30;
const h = R * Math.cos(angle / 180 * Math.PI);
const r = R * Math.sin(angle / 180 * Math.PI);
const random = num => Math.round(Math.random() * num);
const exchange = (firstSamples, secondSamples, start, end) => {
  for (let i = start; i < end; i++) {
    const backup = firstSamples[i];
    firstSamples[i] = secondSamples[i];
    secondSamples[i] = backup;
  }
}
const exchangePoint = (arr) => {
  for (let i = 0; i < 15; i++) {
    const index = random(chartNum - 1);
    arr[index] = [random(width), random(width)]
  }
}

export default {
  name: 'App',
  data () {
    return {
      chartData: [],   // 图形
      bledList: [],    // 种子
      bestList: [],    // 最优解列表
      num: 0,          // 衍生次数
      time: 0,         // 运行时长
      bestImg: {},
      si: null,
      file: '',
      fileColor: [],
      ctx: null
    }
  },
  mounted() {
    this.ctx = this.$refs.canvas.getContext("2d");
  },
  methods: {
    start () {
      this.timeStart = (new Date()).getTime();
      this.si = setInterval(() => {
        this.num += 1;
        this.toEliminate();
      }, 1000);
    },

    stop () {
      this.time += (new Date()).getTime() - this.timeStart;
      this.si && clearInterval(this.si) && (this.si = null);
    },

    // 获取100个随机三角形
    createRandomGraph(colorArray = []) {
      const chartData = [];
      for (let i = 0; i < chartNum; i++) {
        const item = [];
        for (let j = 0; j < 6; j++) {
          item.push(random(20) - 10);
        }
        const startIndex = (random(width * height) - 1) * 4;
        item.push(colorArray.slice(startIndex, startIndex + 3));
        chartData.push(item);
      }
      this.chartData = chartData;
    },

    // 首次生成100个样本
    firstCreateSamples() {
      const bledList = [];
      for (let i = 0; i < bledNum; i++) {
        const item = {
          point: []
        };
        for (let j = 0; j < chartNum; j++) {
          item.point.push([random(width), random(width)])
        }
        bledList.push(item);
      }
      this.bledList = bledList;
    },

    // 优胜劣汰
    toEliminate() {
      const self = this;
      const { bledList, chartData } = self;
      // 生成图片数据
      for (let i = 0; i < bledList.length; i++) {
        if (bledList.imgSrc)
          break;
        const { fileColor, imgSrc } = this.getFileColor(bledList[i].point);
        bledList[i].fileColor = fileColor;
        bledList[i].imgSrc = imgSrc;
        // 相似度分值计算
        bledList[i].matchingScore = this.getMatchingScore(fileColor);
      }
      let nextList = _.sortBy(bledList, 'matchingScore');
      nextList.length = bledNum/2;
      this.bledList = nextList;
      // this.bestList.push(nextList[0]);
      this.bestImg = nextList[0];
      this.createSamples();
    },

    // 生成衍生样本
    createSamples() {
      const { bledList } = this;
      const sampleList = [];
      for (let i = 0; i < bledNum/4; i++) {
        // 交叉变异生成两个样本
        // 选取两个样本来衍生
        const firstSamplesIndex = random(bledNum/2 - 1);
        let secondSamplesIndex = 0;
        do {
          secondSamplesIndex = random(bledNum/2 - 1);
        } while (secondSamplesIndex == firstSamplesIndex);
        let firstSamples = _.cloneDeep(bledList[firstSamplesIndex].point), 
            secondSamples = _.cloneDeep(bledList[secondSamplesIndex].point);

        // 选取两段用于交换
        const firstSection = random(5);
        let secondSection = 0;
        do {
          secondSection = random(6);
        } while (secondSection == firstSection);

        let point = [0];
        for (let j = 0; j < 5; j++) {
          point.push(random(chartNum - 2));
        }
        point.push(chartNum - 1);
        point = _.sortBy(point);

        exchange(firstSamples, secondSamples, point[firstSection], point[firstSection + 1]);
        exchange(firstSamples, secondSamples, point[secondSection], point[secondSection + 1]);

        // 变异
        exchangePoint(firstSamples);
        exchangePoint(secondSamples);
        sampleList.push({
          point: firstSamples
        }, {
          point: secondSamples
        });
      }
      this.bledList = bledList.concat(sampleList);
    },

    // 上传图片
    changeFile(e) {
      const self = this;
      const reader = new FileReader();
      reader.readAsDataURL(e.target.files[0]);
      reader.onload =  async function() {
        self.file = this.result;
        self.fileColor = await self.img2Colors(self.file);
        self.firstCreateSamples();
      }
    },

    getFileColor(point) {
      const { ctx, chartData } = this;
      ctx.fillStyle="#fff";
      ctx.fillRect(0, 0, width, height);
      for (let i = 0; i < point.length; i++) {
        const item = point[i];
        const chart = chartData[i];
        ctx.beginPath();
        ctx.moveTo(item[0] + chart[0], item[1] + chart[1]);
        ctx.lineTo(item[0] + chart[2], item[1] + chart[3]);
        ctx.lineTo(item[0] + chart[4], item[1] + chart[5]);
        ctx.closePath();
        ctx.fillStyle = `rgba(${chart[6][0]},${chart[6][1]},${chart[6][2]}, ${opcity})`;
        ctx.fill();
      }
      return {
        imgSrc: this.$refs.canvas.toDataURL(),
        fileColor: this.canvas2Colors()
      }
    },
    // 计算相似度分值
    getMatchingScore(color) {
      const { fileColor } = this;
      let score = 0;
      for (let i = 0; i < color.length; i++) {
        score += this.distanceOf(color[i], fileColor[i]);
      }
      return score;
    },
    // 比较两颜色距离
    distanceOf(hsv1, hsv2) {
      const x1 = r * hsv1[2] * hsv1[1] * Math.cos(hsv1[0] / 180 * Math.PI);
      const y1 = r * hsv1[2] * hsv1[1] * Math.sin(hsv1[0] / 180 * Math.PI);
      const z1 = h * (1 - hsv1[2]);
      const x2 = r * hsv2[2] * hsv2[1] * Math.cos(hsv2[0] / 180 * Math.PI);
      const y2 = r * hsv2[2] * hsv2[1] * Math.sin(hsv2[0] / 180 * Math.PI);
      const z2 = h * (1 - hsv2[2]);
      const dx = x1 - x2;
      const dy = y1 - y2;
      const dz = z1 - z2;
      return Math.sqrt(dx * dx + dy * dy + dz * dz);
    },

    // 把当前画布上像素转化为hsv
    canvas2Colors() {
      const colors = this.ctx.getImageData(0, 0, width, height);
      const backColors = [];
      for (var i=0;i<colors.data.length;i+=4)
      {
        backColors.push(this.rgb2hsv(colors.data[i], colors.data[i + 1], colors.data[i + 2]))
      }
      return backColors
    },

    img2Colors(imgSrc) {
      const self = this;
      return new Promise((resolove, reject) => {
        const img = new Image();
        img.src = imgSrc;
        img.onload = function() {
          self.ctx.drawImage(img, 0, 0, width, height);
          self.createRandomGraph(self.ctx.getImageData(0, 0, width, height).data);
          resolove(self.canvas2Colors());
        };
      })
    },

    rgb2hsv(r,g,b) {
      const hsv_red = Number(r) / 255; 
      const hsv_green = Number(g) / 255; 
      const hsv_blue = Number(b) / 255;
      const hsv_max = Math.max(hsv_red, hsv_green, hsv_blue), hsv_min = Math.min(hsv_red, hsv_green, hsv_blue);
      let hsv_h, hsv_s, hsv_v = hsv_max;

      let hsv_d = hsv_max - hsv_min;
      hsv_s = hsv_max == 0 ? 0 : hsv_d / hsv_max;

      if(hsv_max == hsv_min)
          hsv_h = 0; 
      else
      {
        switch(hsv_max)
        {
          case hsv_red: hsv_h = (hsv_green - hsv_blue) / hsv_d + (hsv_green < hsv_blue ? 6 : 0); break;
          case hsv_green: hsv_h = (hsv_blue - hsv_red) / hsv_d + 2; break;
          case hsv_blue: hsv_h = (hsv_red - hsv_green) / hsv_d + 4; break;
        }
        hsv_h /= 6;
      }
      return [hsv_h.toFixed(4), hsv_s.toFixed(4), hsv_v.toFixed(4)];
    }
  }
}
</script>

<style scoped>
ul, li{
  margin: 0;
  padding: 0;
}
li {
  display: inline-block;
}
.canvas{
  width: 300px;
  height: 300px;
}
</style>
