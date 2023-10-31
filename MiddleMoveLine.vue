<template>
  <!--  重新优化了整体实现逻辑...一个盘 分成A/B 两份---
  需要先了解定位布局的[子绝父相]---子绝对定位/父相对定位（最常用的搭配），才能理解本组件中的布局
  另外，只有定位元素才能设置z-index---不设置就是类似入栈行为。先写的被压在下面。

  usage:嵌套即可

  <MiddleMoveLine :safe-distance="[50,50]" :percent-a="30">
    <template v-slot:paneA>
      <div style="background-color: #5ee012;width: 100%;height: 100%;font-size: 50px">我是第一个panA</div>
    </template>

    <template v-slot:paneB>

      <MiddleMoveLine direction="y" :safe-distance="[50,50]" :percent-a="60">
        <template v-slot:paneA>
          <div style="background-color: #c512e0;width: 100%;height: 100%;font-size: 50px">我是第二个panA</div>
        </template>

        <template v-slot:paneB>
          <div style="background-color: #c70b2c;width: 100%;height: 100%;font-size: 50px">我是第二个panB</div>

        </template>
      </MiddleMoveLine>
    </template>
  </MiddleMoveLine>
  -->
  <div class="line-pane-view" :class="{cursor}" ref="line-pane" @mouseup="mouseUp" @mousemove="mouseMove">
    <div :class="[direction==='x'?'pane-ax':'pane-ay']" :style="paneABstyle[0]">
      <slot name="paneA"></slot>
    </div>
    <div class="middle-line active" ref="mouse-ctrl" :style="middleLineStyle"
         :class="[direction==='x'?'middle-line-x':'middle-line-y']"
         @mousedown="mouseD">
    </div>
    <div :class="[direction==='x'?'pane-bx':'pane-by']" :style="paneABstyle[1]">
      <slot :style="{'z-index':zIndex}" name="paneB"></slot>
    </div>
  </div>
</template>

<script>

export default {
  name: "MiddleMoveLine",
  props: {//父组件传递过来的数据
    // 描述分割中线的方向、样子
    direction: {
      default: 'x' // x--中线在x轴即左右方向分割成 x 方向上的 A、B两份
    },
    safeDistance: {
      default: function () {
        return [30, 40]// A的安全距离30px,B的安全距离40px
      }
    },
    zIndex: {// 第二个元素的层级，因为往往第二个压住第一个，所以可能需要加这个改变层级
      default: 1
    },
    percentA: {
      default: 55// A占比55%，B占比45% = 1 - 50%
    },
  },
  data() {
    return {
      active: false,

      percentX: 20,// 20% 中线 x
      percentY: 10,// 中线为 y 时使用

      mouseX: 0,//鼠标在 line-pane-view 坐标中的位置
      mouseY: 0,
      parentWidth: 0,//父元素line-pane-view 的宽高
      parentHeight: 0
    };
  },
  mounted() {
    this.percentX = this.percentA
    this.percentY = 100 - this.percentA
  },
  computed: {
    paneABstyle() {// 通过样式改变A、B的长宽度
      // x--中线左右方向分割， 此时A、B 左右 关系，移动中线的时候应该改变A、B的宽度
      const x = this.percentX;
      const y = this.percentY;
      if (this.direction === 'x') {// 同时返回 A、B 的样式
        return [{['width']: x + '%'}, {['width']: 100 - x + '%'}]
      } else {
        return [{['height']: y + '%'}, {['height']: 100 - y + '%'},]
      }
    },
    middleLineStyle() {//中线也需要同步更改
      const left = this.percentX
      const top = this.percentY
      if (this.direction === 'x') {
        return {['left']: left + "%"}
      } else {
        return {['top']: top + "%"}
      }
    },
    cursor() {// 根据是否激活来设置鼠标的拖动样式
      return this.active ? (this.direction === 'x' ? 'row-resize' : 'col-resize') : ''
    },
  },
  methods: {
    mouseMove(e) {
      // 鼠标没有被按下 或者 没有被触发
      if (e.buttons === 0 || e.which === 0) {
        this.active = false
      }
      if (!this.active) return;
      // console.log("鼠标移动:", e,"e.currentTarget:",e.currentTarget);
      const parentRect = e.currentTarget.getBoundingClientRect();
      let x = e.clientX - parentRect.left;

      let y = e.clientY - parentRect.top;
      const w = parentRect.width;
      const h = parentRect.height;
      const safeDistance = this.safeDistance

      // 判断bu在安全距离内,例外处理：
      x = x < safeDistance[0] ? safeDistance[0] : x
      x = x > (w - safeDistance[1]) ? (w - safeDistance[1]) : x
      y = y < safeDistance[0] ? safeDistance[0] : y
      y = y > (h - safeDistance[1]) ? (h - safeDistance[1]) : y

      this.mouseX = x
      this.mouseY = y
      this.parentWidth = w;
      this.parentHeight = h;
      console.log("鼠标移动:this.mouseX,Y", this.mouseX, this.mouseY, "this.parentWidth,H:", this.parentWidth, this.parentHeight);
      console.log("鼠标移动:left,top:", (this.mouseX / this.parentWidth) * 100 + "%", (this.mouseY / this.parentHeight) * 100 + "%");

      this.percentX = (x / w) * 100;
      this.percentY = (y / h) * 100;
      //   可以向父组件发送事件，让父组件可以处理其他组件比例来适应
      this.$emit('change-view-size', {x, y, w, h})

    },
    mouseUp() {
      console.log("鼠标松开---整个面板的事件");
      if (this.active)
        this.active = false
    },
    // 中线的鼠标事件
    mouseD() {
      console.log("鼠标按下");
      if (!this.active)
        this.active = true
    },
  },
  watch: {
    // 暂时没有这个动态更改初始比例的需求...
    // percentA(newVal, oldVal) {// vue2中初次加载不会运行（不认为数据发生变化），运行中改变数据才能运行// vue3中可以在初始化的时候运行（具体配置看vue3官网）
    //   // console.log("percentA 发生变化了...", newVal, oldVal)
    //   // this.percentX = newVal
    //   // this.percentY = 100 - newVal
    // },
  }
}
;
</script>

<style scoped lang="scss">
.line-pane-view {
  width: 100%;
  height: 100%;
  position: relative; /*子绝父相---由于孩子都是绝对的，父亲必须有定位才能困住*/
  user-select: none;
}

.middle-line {
  /* 设置元素的定位方式为绝对定位，相对于最近的定位上下文进行定位 */
  position: absolute;
  /* 设置盒模型为 border-box，元素的宽度和高度包括内容、内边距和边框 */
  //-moz-box-sizing: border-box; /* 适用于 Firefox 浏览器 */
  //-webkit-box-sizing: border-box; /* 适用于 WebKit 内核浏览器（如 Chrome、Safari） */
  box-sizing: border-box; /* 标准写法，适用于大多数现代浏览器 */
  background-color: #0D0D0D;
  /* 设置元素的不透明度为 0.1，取值范围为 0（完全透明）到 1（完全不透明）之间 */
  opacity: .1;
  /* 设置元素的层叠顺序，用于控制元素在堆叠顺序中的显示优先级 */
  z-index: 2;
  /* 设置背景剪切方式为 padding-box，背景仅绘制在内边距区域 */
  -moz-background-clip: padding; /* 适用于 Firefox 浏览器 */
  -webkit-background-clip: padding; /* 适用于 WebKit 内核浏览器（如 Chrome、Safari） */
  background-clip: padding-box; /* 标准写法，适用于大多数现代浏览器 */
}


/*x--中线左右方向分割 */
.middle-line-x {
  width: 11px;
  height: 100%;
  margin-left: -5px;
  border-left: 5px solid rgba(255, 255, 255, 0);
  border-right: 5px solid rgba(255, 255, 255, 0);
  cursor: col-resize;
}

.middle-line-y {
  height: 11px;
  margin: -5px 0;
  border-top: 5px solid rgba(255, 255, 255, 0);
  border-bottom: 5px solid rgba(255, 255, 255, 0);
  cursor: row-resize;
  width: 100%;
}

// A 代表上、左  | B 代表下、右
.pane-ay {
  position: absolute;
  top: 0; /*上 --- 中线为 y 时*/
  width: 100%;
}

.pane-by {
  position: absolute;
  bottom: 0; /*下 --- 中线为 y 时*/
  width: 100%;
  padding-top: 3px;
}

.pane-ax {
  position: absolute;
  left: 0; /*左 --- 中线为 x 时*/
  height: 100%;
  padding-right: 3px;
}

.pane-bx {
  position: absolute;
  right: 0; /*右 --- 中线为 x 时*/
  height: 100%;
  padding-left: 3px;
}

.active:hover {
  background-color: mediumvioletred;

}

.middle-move-line.active {
  background-color: #575555;
}
</style>
