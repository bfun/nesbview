<template>
  <div class="common-layout">
    <el-container>
      <el-header>
        <el-tabs :tab-position="left" style="height: 200px" class="demo-tabs">
          <el-tab-pane v-for="item in ports" :label="item">{{ item }}</el-tab-pane>
        </el-tabs>
        <el-row class="row-bg" justify="center">
          <el-col :span="6"><div class="grid-content">
            <el-select v-model="port" filterable clearable @change="portChange" placeholder="端口" style="width: 240px">
              <el-option v-for="item in ports" :key="item" :label="item" :value="item"/>
            </el-select></div>
          </el-col>
          <el-col :span="6"><div class="grid-content">
            <el-select v-model="dta" filterable clearable @change="dtaChange" placeholder="DTA" style="width: 240px">
              <el-option v-for="item in dtas" :key="item" :label="item" :value="item"/>
            </el-select></div>
          </el-col>
          <el-col :span="6"><div class="grid-content">
            <el-select v-model="code" filterable clearable @change="codeChange" placeholder="交易码" style="width: 240px">
              <el-option v-for="item in codes" :key="item" :label="item" :value="item"/>
            </el-select></div>
          </el-col>
        </el-row>
      </el-header>
      <el-main>
        <div v-for="(v,i) in reqDataSource">
          <el-divider content-position="left">请求 {{ i }}: {{ v.SDta }}.{{v.SSvc}}.{{v.SFmt}} -> {{v.DDta}}.{{v.DSvc}}.{{v.DFmt}}</el-divider>
          <el-table :data="v.Items" border style="width: 100%; display: flex; justify-content: center;">
            <el-table-column prop="SvrTag" label="消费方标签" width="180" />
            <el-table-column prop="Elem" label="数据元素" width="180" />
            <el-table-column prop="CltTag" label="服务方标签" width="180" />
          </el-table>
        </div>
        <div v-for="(v,i) in resDataSource">
          <el-divider content-position="left">响应 {{ i }}: {{ v.SDta }}.{{v.SSvc}}.{{v.SFmt}} -> {{v.DDta}}.{{v.DSvc}}.{{v.DFmt}}</el-divider>
          <el-table :data="v.Items" border style="width: 100%; display: flex; justify-content: center;">
            <el-table-column prop="SvrTag" label="消费方标签" width="180" />
            <el-table-column prop="Elem" label="数据元素" width="180" />
            <el-table-column prop="CltTag" label="服务方标签" width="180" />
          </el-table>
        </div>
      </el-main>
      <el-footer><!-- Footer --></el-footer>
    </el-container>
  </div>
</template>

<script setup>
import { ref,onMounted } from 'vue';
import axios from 'axios';

const initSvrs = [];
const initPorts = [];
const initDtas = [];
let initCodes = [];
const ports = ref([]);
const dtas = ref([]);
const codes = ref([]);
const port = ref('');
const dta = ref('');
const code = ref('');
const service = ref('')
const reqDataSource = ref([]);
const resDataSource = ref([]);

const portSearch = (val) => {
  port.value = val;
  if (val.trim() !== '') {
    ports.value = initPorts.filter(v => v.includes(val));
  }else{
    ports.value = initPorts;
  }
};
const portChange = (val) => {
  reqDataSource.value = []
  resDataSource.value = []
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
  reqDataSource.value = []
  resDataSource.value = []
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
};
const getCodes = (dta) => {
  axios.get('http://28.4.199.2:8000/svcs/'+dta).then((response) => {
    initCodes = response.data
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
  reqDataSource.value = []
  resDataSource.value = []
  if (val.trim()===''){
    return
  }
  if (dta.value.length === 0) {
    alert('请输入端口或DTA')
    return
  }
  axios.get('http://28.4.199.2:8000/svc/'+dta.value+'/'+val).then((response) => {
    service.value = response.data;
    reqDataSource.value = response.data.RequestItems
    resDataSource.value = response.data.ResponseItems
  })
      .catch(error => {
        console.error('Error fetching data: ', error);
      });
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
</script>
<style>
</style>