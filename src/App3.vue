<template>
  <lay-layout class="example">
    <lay-header>
      <lay-form class="form-row">
        <lay-form-item>
          <lay-select id="portSelect" v-model="port" @search="portSearch" @change="portChange" allow-clear :show-search="true" placeholder="端口">
            <lay-select-option v-for="(item,index) in ports" :key="item" :value="item">
              {{ item }}
            </lay-select-option>
          </lay-select>
        </lay-form-item>
        <lay-form-item>
          <lay-select id="dtaSelect" v-model="dta" @search="dtaSearch" @change="dtaChange" allow-clear :show-search="true" placeholder="DTA">
            <lay-select-option v-for="(item,index) in dtas" :key="item" :value="item">
              {{ item }}
            </lay-select-option>
          </lay-select>
        </lay-form-item>
        <lay-form-item>
          <lay-select id="codeSelect" v-model="code" @search="codeSearch" @change="codeChange" allow-clear :show-search="true" placeholder="交易码">
            <lay-select-option v-for="(item,index) in codes" :key="item" :value="item">
              {{ item }}
            </lay-select-option>
          </lay-select>
        </lay-form-item>
      </lay-form>
    </lay-header>
    <lay-body>
      <lay-space direction="vertical" fill>
        <div v-for="(v,i) in reqDataSource">
          <lay-line theme="green" margin="50px" contentPosition="left">请求 {{ i }}: {{ v.SDta }}.{{v.SSvc}}.{{v.SFmt}} -> {{v.DDta}}.{{v.DSvc}}.{{v.DFmt}}</lay-line>
          <lay-table :default-toolbar="false" :columns="columns" :data-source="v.Items"></lay-table>
        </div>
        <div v-for="(v,i) in resDataSource">
          <lay-line theme="green" margin="50px" contentPosition="left">响应 {{ i }}: {{ v.SDta }}.{{v.SSvc}}.{{v.SFmt}} -> {{v.DDta}}.{{v.DSvc}}.{{v.DFmt}}</lay-line>
          <lay-table :default-toolbar="false" :columns="columns" :data-source="v.Items"></lay-table>
        </div>
      </lay-space>
    </lay-body>
    <lay-footer></lay-footer>
  </lay-layout>
</template>

<script setup>
import { ref,onMounted } from 'vue';
import axios from 'axios';

const initSvrs = [];
const initPorts = [];
const initDtas = [];
const initCodes = [];
const ports = ref([]);
const dtas = ref([]);
const codes = ref([]);
const port = ref('');
const dta = ref('');
const code = ref('');
const service = ref('')

const portSearch = (val) => {
  port.value = val;
  if (val.trim() !== '') {
    ports.value = initPorts.filter(v => v.includes(val));
  }else{
    ports.value = initPorts;
  }
};
const portChange = (val) => {
  if (val.trim() === '') {
    return;
  }
  for(const i of initSvrs){
    if (i.Port === val) {
      dta.value = i.Name;
      getCodes(i.Name)
      return
    }
  }
  reqDataSource.value = []
  resDataSource.value = []
};
const dtaSearch = (val) => {
  dta.value = val;
  if (val.trim() !== '') {
    dtas.value = initDtas.filter(v => v.includes(val.toUpperCase()));
  }else{
    dtas.value = initDtas;
  }
};
const dtaChange = (val) => {
  if (val.trim() === '') {
    return;
  }
  for(const i of initSvrs) {
    if (i.Name === val.toUpperCase()) {
      port.value = i.Port;
      getCodes(i.Name)
      return
    }
  }
  reqDataSource.value = []
  resDataSource.value = []
};
const getCodes = (dta) => {
  axios.get('http://28.4.199.2:8000/svcs/'+dta).then((response) => {
    codes.value = response.data;
  })
      .catch(error => {
        console.error('Error fetching data: ', error);
      });
}
const codeSearch = (val) => {
  if (dta.value.length === 0) {
    alert('请输入端口或DTA')
    return
  }
  if (val.trim() !== '') {
    codes.value = initCodes.filter(v => v.includes(val));
  }else{
    codes.value = initCodes;
  }
};
const codeChange = (val) => {
  if (val.trim()===''){
    return
  }
  if (dta.value.length === 0) {
    alert('请输入端口或DTA')
    return
  }
  axios.get('http://28.4.199.2:8000/svc/'+dta.value+'/'+val).then((response) => {
    service.value = response.data;
    // service.value.Request.forEach((f)=>{mapper(f.Dta,f.Svc,f.Fmt)})
    reqDataSource.value = response.data.RequestItems
    resDataSource.value = response.data.ResponseItems
  })
      .catch(error => {
        console.error('Error fetching data: ', error);
      });
  reqDataSource.value = []
  resDataSource.value = []
}
onMounted(async () => {
  await axios.get('http://28.4.199.2:8000/svrs').then((response) => {
    response.data.forEach((item) => {
      initSvrs.push(item);
      if(item.Port!=='') {
        initPorts.push(item.Port);
        ports.value.push(item.Port);
      }
      initDtas.push(item.Name);
      dtas.value.push(item.Name);
    });
  })
  .catch(error => {
    console.error('Error fetching data: ', error);
  });
});

const columns = [
  {title:"消费方标签",width:"100px",key:"SvrTag"},
  {title:"数据元素",width:"100px",key:"Elem"},
  {title:"服务方标签",width:"100px",key:"CltTag"},
];
const dataSource = ref([]);
const reqDataSource = ref([]);
const resDataSource = ref([]);
const mapper = (dta,svc,fmt) => {
  axios.get('http://28.4.199.2:8000/fmt/'+dta+'/'+svc+'/'+fmt).then((response) => {
    for (let k in response.data) {
      const f = response.data[k]
      const items = []
      for(let tag in f){
        let a = {"svrTag": tag, "dataElem": f[tag]};
        items.push(a)
      }
      dataSource.value.push({fmt:k,items:items})
    }
  })
      .catch(error => {
        console.error('Error fetching data: ', error);
      });
}
</script>
<style>
.form-row {
  display: flex; /* 使用 Flexbox 布局 */
  flex-wrap: wrap; /* 允许换行 */
  justify-content: space-evenly;
  /* space-between; 项目之间的间距平均分布 */
  /* 或者使用 justify-content: flex-start; 让项目紧挨着左边排列 */
}
@media screen and (min-width: 768px) {
  .form-row {
    width: 750px;
  }
}
@media screen and (min-width: 992px) {
  .form-row {
    width: 970px;
  }
}
</style>