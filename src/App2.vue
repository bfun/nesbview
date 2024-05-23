<script setup>
import HelloWorld from './components/HelloWorld.vue'
import TheWelcome from './components/TheWelcome.vue'
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />
    </div>
  </header>

  <main>
    <TheWelcome />
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>

/*

<div class="layui-form-item">
<div class="layui-inline">
  <lay-input type="text" class="layui-input" placeholder="端口" :allow-clear="true" :maxlength="5" style="width:100px;" v-model="port"></lay-input>
</div>
<div class="layui-inline">
  <lay-input type="text" class="layui-input" placeholder="DTA" :allow-clear="true" :maxlength="16" style="width:150px;" v-model="dta"></lay-input>
</div>
<div class="layui-inline">
  <lay-input type="text" class="layui-input" placeholder="交易码" :allow-clear="true" :maxlength="32" style="width:250px;" v-model="code"></lay-input>
</div>
</div>
*/
