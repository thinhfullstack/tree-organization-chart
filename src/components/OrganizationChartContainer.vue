<template>
  <div v-bind="{ scopedSlots: $scopedSlots }" class="orgchart-container" @onWheel="props.zoom && zoomHandler($event)"
    @mouseup="props.pan && state.panning && panEndHandler()">
    <div class="orgchart" :style="{ transform: state.transformVal, cursor: state.cursorVal }"
      @mousedown="props.pan && panStartHandler($event)" @mousemove="props.pan && state.panning && panHandler($event)">
      <OrganizationChartNode :datasource="props.datasource" :handle-click="handleClick">
        <div v-if="slots && Object.keys($scopedSlots).length > 0">
          <template v-for="(scope, slot) in Object.keys($scopedSlots)" :key="slot" slot="slot" slot-scope="scope">
            <slot :name="slot" v-bind="scope" />
          </template>
        </div>
      </OrganizationChartNode>
    </div>
  </div>
</template>
  
<script setup>
import $ from 'jquery';
import OrganizationChartNode from './OrganizationChartNode.vue'
import { reactive, defineEmits, defineProps } from 'vue';

const props = defineProps({
  datasource: {
    type: Object,
    required: true,
    default: () => {}
  },
  pan: {
    type: Boolean,
    required: false
  },
  zoom: {
    type: Boolean,
    required: false
  },
  zoomoutLimit: {
    type: Number,
    required: false,
    default: 0.5
  },
  zoominLimit: {
    type: Number,
    required: false,
    default: 7
  }
})

const state = reactive({
  cursorVal: 'default',
  panning: false,
  startX: 0,
  startY: 0,
  transformVal: ''
})

const emits = defineEmits(['node-click'])

const handleClick = (nodeData) => {
  emits('node-click', nodeData);
}

const panEndHandler = () => {
  state.panning = false
  state.cursorVal = 'default'
}

const panHandler = (e) => {
  let newX = 0
  let newY = 0
  if (!e.targetTouches) {
    newX = e.pageX - state.startX
    newY = e.pageY - state.startY
  } else if (e.targetTouches.length === 1) {
    newX = e.targetTouches[0].pageX - state.startX
    newY = e.targetTouches[0].pageY - state.startY
  } else if (e.targetTouches.length > 1) {
    return;
  }
  if (state.transformVal === '') {
    if (state.transformVal.indexOf('3d') === -1) {
      state.transformVal = 'matrix(1,0,0,1,' + newX + ',' + newY + ')'
    } else {
      state.transformVal = 'matrix3d(1,0,0,0,0,1,0,0,0,0,1,0,' + newX + ', ' + newY + ',0,1)'
    }
  } else {
    let matrix = state.transformVal.split(',')
    if (state.transformVal.indexOf('3d') === -1) {
      matrix[4] = newX
      matrix[5] = newY + ')'
    } else {
      matrix[12] = newX
      matrix[13] = newY
    }
    state.transformVal = matrix.join(',')
  }
}

const panStartHandler = (e) => {
  if ($(e.target).closest('.node').length) {
    state.panning = false
    return
  } else {
    state.cursorVal = 'move'
    state.panning = true
  }
  let lastX = 0
  let lastY = 0
  if (state.transformVal !== '') {
    let matrix = state.transformVal.split(',')
    if (state.transformVal.indexOf('3d') === -1) {
      lastX = parseInt(matrix[4])
      lastY = parseInt(matrix[5])
    } else {
      lastX = parseInt(matrix[12])
      lastY = parseInt(matrix[13])
    }
  }
  if (!e.targetTouches) { // pand on desktop
    state.startX = e.pageX - lastX
    state.startY = e.pageY - lastY
  } else if (e.targetTouches.length === 1) { // pan on mobile device
    state.startX = e.targetTouches[0].pageX - lastX
    state.startY = e.targetTouches[0].pageY - lastY
  } else if (e.targetTouches.length > 1) {
    return
  }
}

const setChartScale = (newScale) => {
  let matrix = ''
  let targetScale = 1
  if (state.transformVal === '') {
    state.transformVal = 'matrix(' + newScale + ', 0, 0, ' + newScale + ', 0, 0)'
  } else {
    matrix = state.transformVal.split(',')
    if (state.transformVal.indexOf('3d') === -1) {
      targetScale = Math.abs(window.parseFloat(matrix[3]) * newScale)
      if (targetScale > props.zoomoutLimit && targetScale < props.zoominLimit) {
        matrix[0] = 'matrix(' + targetScale
        matrix[3] = targetScale
        state.transformVal = matrix.join(',')
      }
    } else {
      targetScale = Math.abs(window.parseFloat(matrix[5]) * newScale)
      if (targetScale > props.zoomoutLimit && targetScale < props.zoominLimit) {
        matrix[0] = 'matrix3d(' + targetScale
        matrix[5] = targetScale
        state.transformVal = matrix.join(',')
      }
    }
  }
}

const zoomHandler = (e) => {
  let newScale = 1 + (e.deltaY > 0 ? -0.2 : 0.2)
  setChartScale(newScale)
}
</script>
  
<style>
.orgchart-container {
  position: relative;
  display: inline-block;
  height: 420px;
  width: calc(100% - 24px);
  border: 2px dashed #aaa;
  border-radius: 5px;
  overflow: auto;
  text-align: center;
}

.orgchart {
  box-sizing: border-box;
  display: inline-block;
  min-height: 202px;
  min-width: 202px;
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  background-image: linear-gradient(90deg,
      rgba(200, 0, 0, 0.15) 10%,
      rgba(0, 0, 0, 0) 10%),
    linear-gradient(rgba(200, 0, 0, 0.15) 10%, rgba(0, 0, 0, 0) 10%);
  background-size: 10px 10px;
  border: 1px dashed rgba(0, 0, 0, 0);
  padding: 20px;
}

.orgchart .hidden,
.orgchart~.hidden {
  display: none;
}

.orgchart.b2t {
  transform: rotate(180deg);
}

.orgchart.l2r {
  position: absolute;
  transform: rotate(-90deg) rotateY(180deg);
  transform-origin: left top;
}

.orgchart .verticalNodes ul {
  list-style: none;
  margin: 0;
  padding-left: 18px;
  text-align: left;
}

.orgchart .verticalNodes ul:first-child {
  margin-top: 2px;
}

.orgchart .verticalNodes>td::before {
  content: "";
  border: 1px solid rgba(217, 83, 79, 0.8);
}

.orgchart .verticalNodes>td>ul>li:first-child::before {
  box-sizing: border-box;
  top: -4px;
  height: 30px;
  width: calc(50% - 2px);
  border-width: 2px 0 0 2px;
}

.orgchart .verticalNodes ul>li {
  position: relative;
}

.orgchart .verticalNodes ul>li::before,
.orgchart .verticalNodes ul>li::after {
  box-sizing: border-box;
  content: "";
  position: absolute;
  left: -6px;
  border-color: rgba(217, 83, 79, 0.8);
  border-style: solid;
  border-width: 0 0 2px 2px;
}

.orgchart .verticalNodes ul>li::before {
  top: -4px;
  height: 30px;
  width: 11px;
}

.orgchart .verticalNodes ul>li::after {
  top: 1px;
  height: 100%;
}

.orgchart .verticalNodes ul>li:first-child::after {
  box-sizing: border-box;
  top: 24px;
  width: 11px;
  border-width: 2px 0 0 2px;
}

.orgchart .verticalNodes ul>li:last-child::after {
  box-sizing: border-box;
  border-width: 2px 0 0;
}

.orgchart.r2l {
  position: absolute;
  transform: rotate(90deg);
  transform-origin: left top;
}

.orgchart>.spinner {
  font-size: 100px;
  margin-top: 30px;
  color: rgba(68, 157, 68, 0.8);
}

.orgchart table {
  border-spacing: 0;
  border-collapse: separate;
}

.orgchart>table:first-child {
  margin: 20px auto;
}

.orgchart td {
  text-align: center;
  vertical-align: top;
  padding: 0;
}

.orgchart .lines:nth-child(3) td {
  box-sizing: border-box;
  height: 20px;
}

.orgchart .lines .topLine {
  border-top: 2px solid rgba(217, 83, 79, 0.8);
}

.orgchart .lines .rightLine {
  border-right: 1px solid rgba(217, 83, 79, 0.8);
  float: none;
  border-radius: 0;
}

.orgchart .lines .leftLine {
  border-left: 1px solid rgba(217, 83, 79, 0.8);
  float: none;
  border-radius: 0;
}

.orgchart .lines .downLine {
  background-color: rgba(217, 83, 79, 0.8);
  margin: 0 auto;
  height: 20px;
  width: 2px;
  float: none;
}

/* node styling */
.orgchart .node {
  box-sizing: border-box;
  display: inline-block;
  position: relative;
  margin: 0;
  padding: 3px;
  border: 2px dashed transparent;
  text-align: center;
  width: 130px;
}

.orgchart.l2r .node,
.orgchart.r2l .node {
  width: 50px;
  height: 130px;
}

.orgchart .node>.spinner {
  position: absolute;
  top: calc(50% - 15px);
  left: calc(50% - 15px);
  vertical-align: middle;
  font-size: 30px;
  color: rgba(68, 157, 68, 0.8);
}

.orgchart .node:hover {
  background-color: rgba(238, 217, 54, 0.5);
  transition: 0.5s;
  cursor: default;
  z-index: 20;
}

.orgchart .node.focused {
  background-color: rgba(238, 217, 54, 0.5);
}

.orgchart .ghost-node {
  position: fixed;
  left: -10000px;
  top: -10000px;
}

.orgchart .ghost-node rect {
  fill: #ffffff;
  stroke: #bf0000;
}

.orgchart .node.allowedDrop {
  border-color: rgba(68, 157, 68, 0.9);
}

.orgchart .node .title {
  text-align: center;
  font-size: 12px;
  font-weight: bold;
  height: 20px;
  line-height: 20px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  background-color: rgba(217, 83, 79, 0.8);
  color: #fff;
  border-radius: 4px 4px 0 0;
}

.orgchart.b2t .node .title {
  transform: rotate(-180deg);
  transform-origin: center bottom;
}

.orgchart.l2r .node .title {
  transform: rotate(-90deg) translate(-40px, -40px) rotateY(180deg);
  transform-origin: bottom center;
  width: 120px;
}

.orgchart.r2l .node .title {
  transform: rotate(-90deg) translate(-40px, -40px);
  transform-origin: bottom center;
  width: 120px;
}

.orgchart .node .title .symbol {
  float: left;
  margin-top: 4px;
  margin-left: 2px;
}

.orgchart .node .content {
  box-sizing: border-box;
  width: 100%;
  height: 20px;
  font-size: 11px;
  line-height: 18px;
  border: 1px solid rgba(217, 83, 79, 0.8);
  border-radius: 0 0 4px 4px;
  text-align: center;
  background-color: #fff;
  color: #333;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.orgchart.b2t .node .content {
  transform: rotate(180deg);
  transform-origin: center top;
}

.orgchart.l2r .node .content {
  transform: rotate(-90deg) translate(-40px, -40px) rotateY(180deg);
  transform-origin: top center;
  width: 120px;
}

.orgchart.r2l .node .content {
  transform: rotate(-90deg) translate(-40px, -40px);
  transform-origin: top center;
  width: 120px;
}

.orgchart .node .edge {
  font-size: 15px;
  position: absolute;
  color: rgba(68, 157, 68, 0.5);
  cursor: default;
  transition: 0.2s;
}

.orgchart.noncollapsable .node .edge {
  display: none;
}

.orgchart .edge:hover {
  color: #449d44;
  cursor: pointer;
}

.orgchart .node .verticalEdge {
  width: calc(100% - 10px);
  width: -webkit-calc(100% - 10px);
  width: -moz-calc(100% - 10px);
  left: 5px;
}

.orgchart .node .topEdge {
  top: -4px;
}

.orgchart .node .bottomEdge {
  bottom: -4px;
}

.orgchart .node .horizontalEdge {
  width: 15px;
  height: calc(100% - 10px);
  height: -webkit-calc(100% - 10px);
  height: -moz-calc(100% - 10px);
  top: 5px;
}

.orgchart .node .rightEdge {
  right: -4px;
}

.orgchart .node .leftEdge {
  left: -4px;
}

.orgchart .node .horizontalEdge::before {
  position: absolute;
  top: calc(50% - 7px);
}

.orgchart .node .rightEdge::before {
  right: 3px;
}

.orgchart .node .leftEdge::before {
  left: 3px;
}

.orgchart .node .toggleBtn {
  position: absolute;
  left: 5px;
  bottom: -2px;
  color: rgba(68, 157, 68, 0.6);
}

.orgchart .node .toggleBtn:hover {
  color: rgba(68, 157, 68, 0.8);
}

.oc-export-btn {
  display: inline-block;
  position: absolute;
  right: 5px;
  top: 5px;
  padding: 6px 12px;
  margin-bottom: 0;
  font-size: 14px;
  font-weight: 400;
  line-height: 1.42857143;
  text-align: center;
  white-space: nowrap;
  vertical-align: middle;
  touch-action: manipulation;
  cursor: pointer;
  user-select: none;
  color: #fff;
  background-color: #5cb85c;
  border: 1px solid transparent;
  border-color: #4cae4c;
  border-radius: 4px;
}

.oc-export-btn[disabled] {
  cursor: not-allowed;
  box-shadow: none;
  opacity: 0.3;
}

.oc-export-btn:hover,
.oc-export-btn:focus,
.oc-export-btn:active {
  background-color: #449d44;
  border-color: #347a34;
}

.orgchart~.mask {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 999;
  text-align: center;
  background-color: rgba(0, 0, 0, 0.3);
}

.orgchart~.mask .spinner {
  position: absolute;
  top: calc(50% - 54px);
  left: calc(50% - 54px);
  color: rgba(255, 255, 255, 0.8);
  font-size: 108px;
}

.orgchart .node {
  transition: transform 0.3s, opacity 0.3s;
}

.orgchart .slide-down {
  opacity: 0;
  transform: translateY(40px);
}

.orgchart.l2r .node.slide-down,
.orgchart.r2l .node.slide-down {
  transform: translateY(130px);
}

.orgchart .slide-up {
  opacity: 0;
  transform: translateY(-40px);
}

.orgchart.l2r .node.slide-up,
.orgchart.r2l .node.slide-up {
  transform: translateY(-130px);
}

.orgchart .slide-right {
  opacity: 0;
  transform: translateX(130px);
}

.orgchart.l2r .node.slide-right,
.orgchart.r2l .node.slide-right {
  transform: translateX(40px);
}

.orgchart .slide-left {
  opacity: 0;
  transform: translateX(-130px);
}

.orgchart.l2r .node.slide-left,
.orgchart.r2l .node.slide-left {
  transform: translateX(-40px);
}
</style>
  