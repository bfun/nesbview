<template>
  <div class="common-layout">
    <el-container>
      <el-header>
      </el-header>
      <el-main>
        <el-tabs tab-position="left" @tab-click="tabClick">
          <el-tab-pane v-for="item in dtaPorts" :label="item">
            <div v-for="(code,codeIndex) in codes">
                <el-button>{{ code }}</el-button>
                <el-divider direction="vertical" v-if="codeIndex !== codes.length - 1" />
            </div>
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
          </el-tab-pane>
        </el-tabs>
      </el-main>
      <el-footer><!-- Footer --></el-footer>
    </el-container>
  </div>
</template>

<script lang="ts" setup>
import { ref,onMounted } from 'vue';
import type { TabsPaneContext } from 'element-plus';
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
const dtaPorts = ref([]);
const tabContent = ref('')

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
const tabClick = (pane :TabsPaneContext, ev:Event)=>{
  let dtaName = (pane.props.label).split(":")[0].trim()
  getCodes(dtaName)
}

onMounted(async () => {
  await axios.get('http://28.4.199.2:8000/svrs').then((response) => {
    response.data.forEach((item) => {
      if(item.Name.includes("_SPUT")){
        return
      }
      initSvrs.push(item);
      if(item.Port!=='') {
        initPorts.push(item.Port);
        ports.value.push(item.Port);
        dtaPorts.value.push(item.Name + " : " + item.Port)
      }else{
        dtaPorts.value.push(item.Name)
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
/* 默认样式 */
.common-layout {
  width: 100%; /* 组件宽度默认为100% */
}

/* 当屏幕宽度小于768px时应用的样式 */
@media (max-width: 768px) {
  .common-layout {
    width: 90%; /* 在小屏幕上减少宽度 */
  }
}
</style>