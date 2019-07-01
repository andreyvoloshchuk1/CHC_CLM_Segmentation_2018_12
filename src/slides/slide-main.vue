<template>
  <div id="content">
    <transition name="fade">
      <section class="main" v-if="!choosenDoctor" >
        <button class="main-choose" v-for="(value, key) in t.mainBtns" v-html="value" :class="`main-choose_${key}`" @click="choose(key)"></button>
      </section>
    </transition>
    <transition-group name="fade" mode="out-in">
      <section-way v-for="(value, key) in t.doctors" :key="`section_${key}`" :doctorType="key" v-if="choosenDoctor === key" :data="t.doctors[key]" @prev="prev" />
    </transition-group>
  </div>
</template>

<script>
  import mixins from '@/app-helper/mixins';
  import sectionWay from "@/components/doctorWay";
  import textData from "@/data/ru/slide-main";

  export default {
    mixins: [...mixins.slide],

    data(){
      return {
        choosenDoctor: ""
      }
    },

    methods: {
      choose(key){
        this.choosenDoctor = Object.keys(textData.content.doctors)[key];
      },

      prev(){
        this.choosenDoctor = ""
      }
    },

    components: {
      sectionWay
    }
  }
</script>

<style lang="scss" scoped>
  $url: '../assets/media/images/slide-main';

  #content {
    background-image: url($url + '/bg.jpg');
  }

  section{
    width: 100%;
    height: 100%;

    position: absolute;
    top: 0;
    left: 0;
  }

  .main-choose {
    width: 330px;
    height: 60px;

    border-radius: 50px;
    background: #F8F8F8;
    box-shadow: 0px 4px 7px rgba(0, 0, 0, 0.2);

    @include f(300);
    font-size: 28px;

    position: absolute;
    transition: .8s linear all;

    &_0 {
      top: 197px;
      left: 255px;
    }
    &_1 {
      top: 197px;
      left: 695px;
    }
    &_2 {
      top: 337px;
      left: 95px;
    }
    &_3 {
      top: 337px;
      left: 475px;
    }
    &_4 {
      top: 337px;
      left: 855px;
    }
    &:active {
      box-shadow: inset 0px 4px 7px rgba(0, 0, 0, 0.2);
    }
  }


</style>
