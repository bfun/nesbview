<template>
  <lay-layout class="main">
    <lay-header>
    </lay-header>
    <lay-body>
      <lay-tab type="brief" tabPosition = "left" @change="tabChange">
        <lay-tab-item v-for="dp in dtaPorts" :title="dp" :id="dp">
          <lay-line theme="green" content-position="center" v-if="dtaPort.length > 0">{{ dtaPort }}</lay-line>
          <span class="code" v-for="c in codes" @click="btnClick" style="cursor: pointer; margin: 5px;">{{ c }}</span>
          <lay-line content-position="center" v-if="code.length > 0">{{ code }}</lay-line>
          <div v-for="(v,i) in reqDataSource">
            <lay-line theme="green" margin="50px" contentPosition="left">请求 {{ i }}: {{ v.SDta }}.{{v.SSvc}}.{{v.SFmt}} -> {{v.DDta}}.{{v.DSvc}}.{{v.DFmt}}</lay-line>
            <!-- lay-table :default-toolbar="false" :columns="columns" :data-source="v.Items"></lay-table -->
            <table>
              <thead><th>消费方标签</th><th>数据元素</th><th>服务方标签</th></thead>
              <tbody>
              <tr v-for="it in v.Items"><td>{{it.SvrTag}}</td><td>{{it.Elem}}</td><td>{{it.CltTag}}</td></tr>
              </tbody>
            </table>
          </div>
          <div v-for="(v,i) in resDataSource">
            <lay-line theme="green" margin="50px" contentPosition="left">响应 {{ i }}: {{ v.SDta }}.{{v.SSvc}}.{{v.SFmt}} -> {{v.DDta}}.{{v.DSvc}}.{{v.DFmt}}</lay-line>
            <lay-table :default-toolbar="false" :columns="columns" :data-source="v.Items"></lay-table>
          </div>
        </lay-tab-item>
      </lay-tab>
      <lay-space direction="vertical" fill>
      </lay-space>
    </lay-body>
    <lay-footer></lay-footer>
  </lay-layout>
</template>

<script lang="ts" setup>
import { ref,onMounted } from 'vue';
import axios from 'axios';

const dtaPorts = ref([]);
const dtaPort = ref('');
const dta = ref('');
const codes = ref([]);
const code = ref('');
const reqDataSource = ref([]);
const resDataSource = ref([]);

const columns = [
  {title:"消费方标签",width:"100px",key:"SvrTag"},
  {title:"数据元素",width:"100px",key:"Elem"},
  {title:"服务方标签",width:"100px",key:"CltTag"},
];

const getCodes = (dta) => {
  axios.get('http://28.4.199.2:8000/svcs/'+dta).then((response) => {
    codes.value = response.data;
  })
      .catch(error => {
        console.error('Error fetching data: ', error);
      });
}

const tabChange = (id :string)=>{
  dtaPort.value = ''
  dta.value = ''
  codes.value = []
  code.value = ''
  reqDataSource.value = []
  resDataSource.value = []
  let dtaName = id.split(":")[0].trim()
  getCodes(dtaName)
  dta.value = dtaName
  dtaPort.value = id
}

const btnClick = (event)=>{
  code.value = event.target.textContent
  reqDataSource.value = []
  resDataSource.value = []
  axios.get('http://28.4.199.2:8000/svc/'+dta.value+'/'+code.value).then((response) => {
    reqDataSource.value = response.data.RequestItems
    resDataSource.value = response.data.ResponseItems
  })
      .catch(error => {
        console.error('Error fetching data: ', error);
      });
}

onMounted(async () => {
  dtaPorts.value = []
  dtaPorts.value.push("DTA : 端口")
  await axios.get('http://28.4.199.2:8000/svrs').then((response) => {
    response.data.forEach((item) => {
      if(item.Name.includes("_SPUT")){
        return
      }
      if(item.Port!=='') {
        dtaPorts.value.push(item.Name + " : " + item.Port)
      }else{
        dtaPorts.value.push(item.Name)
      }
    });
  })
  .catch(error => {
    console.error('Error fetching data: ', error);
  });
});
</script>
<style>
span .code:hover{
  color: green;
}
@media screen and (min-width: 768px) {
  .main {
    width: 750px;
  }
}
@media screen and (min-width: 992px) {
  .main {
    width: 970px;
  }
}
</style>