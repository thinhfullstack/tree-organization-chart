<script setup>
// import OrganizationChart from 'vue-organization-chart'
import 'vue-organization-chart/dist/orgchart.css'
import OrganizationChart from './components/OrganizationChartContainer.vue'
import { reactive } from 'vue';
 
const state = reactive({
  list: {
    'id': '1',
    'name': 'Khôi Phạm',
    'title': 'tech manager',
    'children': [
      { 'id': '2', 'name': 'Ngô An', 'title': 'junior engineer'},
      { 'id': '3', 'name': 'Đình Mạnh', 'title': 'junior engineer',
        'children': [
          { 'id': '4', 'name': 'Văn Thịnh', 'title': 'fresher developer' },
          { 'id': '5', 'name': 'Minh Đức', 'title': 'fresher developer',
            'children': [
              { 'id': '6', 'name': 'Nguyễn Lượng', 'title': 'fresher developer' },
              { 'id': '7', 'name': 'Thanh Hải', 'title': 'fresher developer' }
            ]
            }
          ]
        },
      { 'id': '8', 'name': 'Hoàng Long', 'title': 'senior BrSE' },
      { 'id': '9', 'name': 'Đức Võ', 'title': 'junior engineer' }
    ]
  }
})

const selectNode = (nodeData) => {
  alert('node ' + nodeData.name + ' is selected')
}

</script>

<template>
  <OrganizationChart :datasource="state.list" @node-click="selectNode" >
    <!-- <template slot-scope="{ nodeData }">
      <b @click="selectNode(nodeData)">{{ nodeData.name }}</b>
    </template> -->
  </OrganizationChart>
</template>

<style scoped>

</style>
