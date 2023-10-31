# Vue23-MiddleMoveLine
同时支持vue2、vue3的中线分割组件，可以实现拖动的效果

# usage|使用方法：

```css
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
```
