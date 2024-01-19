<template>
  <div class="container" ref="dragSelectDom">
    <div class="box" :class="{'active': activeList.includes(index)&&item!==0, 'hidden': item===0}" v-for="(item, index) in dataList">{{ item }}</div>
    <div class="selectArea" v-show="showDrag" :style="{
      left: `${left}px`,
      top: `${top}px`,
      width: `${width}px`,
      height: `${height}px`,
    }"></div>
  </div>
</template>

<script setup>
import { defineComponent, ref } from 'vue';
import {shuffle} from 'lodash-es'

// display
const dragSelectDom = ref();
const activeList = ref([]);
const dataList = ref([])

const datas = []
for(let i = 0; i < 50; i++) {
  const value = Math.floor(Math.random() * 9) +1;
  datas.push(value);
  datas.push(10-value);
}

dataList.value = shuffle(datas)
// select func
// 鏆傛椂鍙€冭檻鍨傜洿鏂瑰悜婊氬姩
const left = ref(0),
      top = ref(0),
      width = ref(0),
      height = ref(0),
      showDrag = ref(false);
// calc box attribute
const calcBox = (arr, dom, baseScrollTop) => {
  const length = dom.children.length;
  for(let i = 0; i < length; i++) {
    let child = dom?.children[i];
    let domData = child.getBoundingClientRect();
    let obj = {
      index: i,
      dom: child,
      position: [
        domData.x,
        domData.y + baseScrollTop,
        domData.width,
        domData.height,
      ]
    }
    arr.push(obj);
  }
}

const updateRect = (x, y, w, h) => {
  left.value = x;
  top.value = y;
  width.value = w;
  height.value = h;
}

// 鏍规嵁鐩掑瓙鍥涚淮纭畾鏄惁閫変腑
const calcSelect = (left, top, width, height, boxList) => {
  const arr = new Set();
  for(let i = 0; i < boxList.length; i++) {
    const p = boxList[i].position;
    if (
      (left <= p[0] && left + width > p[0]) ||
      (left > p[0] && left < p[0] + p[2] && width > 0)
    ) {
      if (
        (top <= p[1] && top + height > p[1]) ||
        (top > p[1] && top < p[1] + p[3] && height > 0)
      ) {
        arr.add(boxList[i].index);
      } else {
        arr.delete(boxList[i].index);
      }
    } else {
      arr.delete(boxList[i].index);
    }
  }
  return Array.from(arr)
}

const dragStart = (e) => {
  let baseL = e.clientX,
      baseT = e.clientY,
      baseScrollTop = document.children[0].scrollTop, // html鏍硅妭鐐癸紝闇€鏍规嵁瀹為檯鍦烘櫙鏇挎崲
      boxList = [];
  calcBox(boxList, dragSelectDom.value, baseScrollTop);
  activeList.value = [];
  document.onmousemove = (ev) => {
    showDrag.value = true;
    const xOffset = ev.clientX - baseL;
    const yOffset = ev.clientY - baseT;
    const width = Math.abs(xOffset);
    const height = Math.abs(yOffset);
    const left = xOffset < 0 ? ev.clientX : baseL;
    const top = yOffset < 0 ? ev.clientY : baseT;
    const scrollTop = document.children[0].scrollTop;
    updateRect(left, top + scrollTop, width, height);
    activeList.value = [];
    activeList.value = calcSelect(left, top + scrollTop, width, height, boxList)
  }
  document.onmouseup = () => {
    showDrag.value = false;
    document.onmousemove = null;
    document.onmouseup = null;
    console.log(activeList.value)
    if(activeList.value.length>1){
      const sum = activeList.value.reduce((a,b)=>a+dataList.value[b],0)
      if(sum===10){
        activeList.value.forEach(item=>{
          dataList.value[item]=0
        })
      }
    }
    activeList.value=[]
  }
}
document.addEventListener("mousedown", dragStart);
</script>

<style>
.container {
  display: flex;
  flex-wrap: wrap;
  row-gap: 4px;
  column-gap: 4px;
  width: 600px;
}
.box {
  width: 50px;
  height: 50px;
  background: rgba(0,0,0,0.08);
  user-select: none;
  border: 1px solid transparent;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
.box.active {
  border: 1px solid #66ccff;
  font-size: 20px;
  font-weight: bold;
}
.selectArea {
  position: absolute;
  left: 0;
  top: 0;
  border: 1px solid #66ccff;
  pointer-events: none;
}

.hidden{
  visibility: hidden;
}
</style>
      