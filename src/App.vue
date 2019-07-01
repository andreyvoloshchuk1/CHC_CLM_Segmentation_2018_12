<template>
  <div id="app" :class="[currentSlide.id, clmName]" v-touch:swipe="swipeHandler">
    <router-view/>
    <div class="home" @click="reload">
      <home />
    </div>

    <!-- Components will rendered only in development -->
    <DevElements v-if="isDev"/>
  </div>
</template>

<script>
  import '@/style/main.scss';
  import mixins from '@/app-helper/mixins'
  import home from "@/assets/media/images/slide-main/Menu.svg"

  export default {
    mixins: [ ...mixins.app ],

    methods: {
      reload(){
        window.location.reload()
      }
    },

    components: {
      home
    }
  }
</script>

<style lang="scss">
  .home {
    position: absolute;
    bottom: 41px;
    left: 37px;
  }
</style>
