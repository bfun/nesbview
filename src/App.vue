<template>
  <div class="common-layout" style="width: 1200px;">
    <el-container>
      <el-header>
      </el-header>
      <el-main>
        <el-tabs tab-position="left" @tab-click="tabClick">
          <el-tab-pane v-for="item in dtaPorts" :key="item" :label="item">
            <el-divider content-position="center" v-if="dtaPort.length > 0">{{ dtaPort }}</el-divider>
            <el-button v-for="c in codes" :key="c" text @click="btnClick">{{ c }}</el-button>
            <el-divider content-position="center" v-if="code.length > 0">{{ code }}</el-divider>
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

const dtaPorts = ref([]);
const dtaPort = ref('');
const dta = ref('');
let codes = [];
const code = ref('');
const reqDataSource = ref([]);
const resDataSource = ref([]);

const getCodes = (dta) => {
  axios.get('http://28.4.199.2:8000/svcs/'+dta).then((response) => {
    codes = response.data;
  })
      .catch(error => {
        console.error('Error fetching data: ', error);
      });
}

const tabClick = (pane :TabsPaneContext, ev:Event)=>{
  dtaPort.value = ''
  dta.value = ''
  codes = []
  code.value = ''
  reqDataSource.value = []
  resDataSource.value = []
  let dtaName = pane.props.label.split(":")[0].trim()
  getCodes(dtaName)
  dta.value = dtaName
  dtaPort.value = pane.props.label
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