<template>
  <table>
    <tbody>
      <tr>
        <td :colspan="datasource.children && datasource.children.length ? datasource.children.length * 2 : null">
          <div class="node" :id="datasource.id" @click.stop="handleClick(datasource)">
            <slot :node-data="datasource">
              <div class="title">
                <i class="fa fa-users symbol"></i>
                {{ datasource.name }}
              </div>
              <div class="content">{{ datasource.title }}</div>
            </slot>
          </div>
        </td>
      </tr>
      <template v-if="datasource.children && datasource.children.length">
        <tr class="lines">
          <td :colspan="datasource.children.length * 2">
            <div class="downLine"></div>
          </td>
        </tr>
        <tr class="lines">
          <td class="rightLine"></td>
          <template :key="index" v-for="(n, index) in (datasource.children.length - 1)">
            <td class="leftLine topLine"></td>
            <td class="rightLine topLine"></td>
          </template>
          <td class="leftLine"></td>
        </tr>
        <tr class="nodes">
          <td colspan="2" v-for="(child, index) in datasource.children" :key="index">
            <node :datasource="child" :handle-click="handleClick">
              <template :key="index" v-for="(slot, index) in $slots" :slot="slot" slot-scope="scope">
                <slot :name="slot" v-bind="scope" />
              </template>
            </node>
          </td>
        </tr>
      </template>
    </tbody>
  </table>
</template>

<script setup>
import { defineProps, ref } from 'vue';

const props = defineProps({
  datasource: {
    type: Object,
    default: () => ({}),
  },
  handleClick: {
    type: Function,
    default: () => {},
  },
});

const datasource = ref(props.datasource);

</script>

<style>
</style>
