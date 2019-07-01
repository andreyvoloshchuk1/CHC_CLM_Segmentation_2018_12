<template>
  <div id="content">
    <h1 class="title" v-html="t.title"></h1>

  </div>
</template>

<script>
  import mixins from '@/app-helper/mixins'

  export default {
    mixins: [...mixins.slide],
  }
</script>

<style lang="scss" scoped>
  $url: '../../assets/images/flow-1/slide-1_1';

  #content {
    //background-image: url($url + '/bg.jpg');
  }
</style>
