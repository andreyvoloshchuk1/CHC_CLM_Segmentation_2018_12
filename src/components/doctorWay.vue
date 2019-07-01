<template>
  <section :class="doctorType">
    <transition-group name="fade" mode="out-in">
      <div class="question" v-for="(value, key) in data" :key="`question_${key}`" v-if="currentQuestion === key" :class="[{radio: value.type === 'radio', procent: value.type === 'procent'}, `${doctorType}_${key}`]">
        <div class="question__type" v-html="value.questionCounter"></div>

        <div class="question__number" v-html="key + 1 + '.'"></div>

        <div class="question__text" v-html="value.question"></div>

        <div class="alert" v-html="value.alert" v-if="isBigger"></div>

        <div class="question__descr" v-for="(item, index) in value.descr" v-html="item" v-if="value.descr" :class="`question__descr_${index}`"></div>

        <label v-for="(item, index) in value.input" class="question__radio" :class="`question__radio_${index}`" @click="isRadioChecked = true">
          <input type="radio" name="radio">
          <span></span>
          {{item}}
        </label>

        <div class="calc" v-if="value.placeholder && value.type !== 'radio'">
          <div class="calc__input" v-html="calcValue" @click="setActiveInput" :class="[{active: isActiveInput, calc__input_text: calcValueText !== 'Введите количество'}, {red: isBigger}]"></div>
          <div class="calc__text" v-html="value.inputName"></div>

          <div class="calc-keyboard">
            <div class="calc-keyboard__item" v-for="(value, key) in keyboard" v-html="value" @click="addNumber(key)"></div>
          </div>
        </div>

        <div class="calc-big" v-if="!value.placeholder">
          <div class="calc-big__item" v-for="(item, index) in calcValueTextArray" v-html="item" :class="[`calc-big__item_${index}`, {active: isActiveInputProcent[index]}, {calc__input_text: calcValueTextArray[index][0] !== 'В'}]" @click="setActiveInputArr(index)"></div>
          <div class="calc-big__value" v-for="(item, index) in value.inputsVal" v-html="item" :class="`calc-big__value_${index}`"></div>
          <div class="calc-keyboard">
            <div class="calc-keyboard__item" v-for="(value, key) in keyboard" v-html="value" @click="addNumberProcent(key)"></div>
          </div>
        </div>
      </div>
    </transition-group>

    <transition name="fade">
      <div class="finish" v-if="currentQuestion >= data.length" v-html="finishText"></div>
    </transition>

    <div class="stage" v-if="currentQuestion < data.length">
      <div class="stage__item" v-for="(value, key) in data" :class="{active: currentQuestion >= key}"></div>
    </div>

    <div class="back" @click="prevQuestion">
      <back-arrow />
    </div>

    <div class="next" @click="nextQuestion" :class="{active: isActiveNext}" v-if="currentQuestion < data.length">
      Далее
      <arrow-next />
    </div>
  </section>
</template>

<script>
  import mixins from '@/app-helper/mixins';
  import arrowNext from "@/assets/media/images/slide-main/arrow.svg";
  import backArrow from "@/assets/media/images/slide-main/back.svg";

  export default {
    name: "doctorWay",

    data(){
      return {
        currentQuestion: 0,
        finishText: "Опрос завершен!",
        calcValueText: "Введите количество",
        calcValueTextArray: {
          0: "Введите<br> количество",
          1: "Введите<br> количество",
          2: "Введите<br> процент",
          3: "Введите<br> процент"
        },
        isActiveInput: false,
        isActiveInputProcent: [false, false, false, false],
        isActiveNextStage: false,
        isRadioChecked: false,
        isBigger: false,
        activeInput: Number,
        answers: [],


        keyboard: ["1", "2", "3", "4", "5", "6", "7", "8", "9", "0", ""]
      }
    },

    computed: {
      calcValue(){
        return this.calcValueText;
      },

      isActiveNext(){
        if (((this.calcValueText !== "") && (this.calcValueText !== "Введите количество"))){
          this.isActiveNextStage = true;
        }
        else if (this.isRadioChecked){
          this.isActiveNextStage = true;
        }
        else {
          this.isActiveNextStage = false;
        }

        if ((this.calcValueTextArray[0] !== "Введите<br> количество") && (this.calcValueTextArray[0] !== '')){
          this.isActiveNextStage = true;
        }
        return this.isActiveNextStage;
      },

      _calcValueTextArray(){
        return this.calcValueTextArray
      },


    },

    methods: {
      nextQuestion(){
        if (!this.isActiveNextStage){
          return
        }
        if (this.data[this.currentQuestion].alert){
          if (this.answers[this.answers.length - 1] < parseInt(this.calcValueText)){
            this.isBigger = true;
            return
          }
        }

        if(this.isRadioChecked){
          for (let i = 0; i < document.querySelectorAll("input[name='radio']").length; i++){
            if (document.querySelectorAll("input[name='radio']")[i].checked){
              this.answers.push(document.querySelectorAll("label")[i].innerText)
            }
          }
        }
        else if (this.doctorType === "fd" && this.currentQuestion === 1 && (parseInt(this.calcValueTextArray[2]) >= 30)){
          this.answers[0] = "fd-pd";
          this.data[2] = {
            questionCounter: "fd-pd1",
            question: "Если говорить про всех Ваших пациентов, скольким Вы назначили препараты-пробиотики за последнюю неделю?",
            placeholder: "Введите количество",
            inputName: "Назначений препаратов-пробиотиков за последнюю неделю",
          };
          this.data[3] = {
            questionCounter: "fd-pd2",
            question: "Если говорить про всех Ваших пациентов, скольким Вы назначили Энтерожермину<sup>®</sup> за последнюю неделю?",
            placeholder: "Введите количество",
            inputName: "Назначений Энтерожермины<sup>®</sup> за последнюю неделю",
            alert: "<span class='bold'>ПРОВЕРЬТЕ!</span><br> Значение в <span>FD-PD2</span> должно быть МЕНЬШЕ<br> чем в <span>FD-PD1</span>"
          };
          this.data[4] = {
            questionCounter: "fd-pd3",
            question: "Если говорить про всех Ваших пациентов, скольким Вы назначили препараты от кашля за последнюю неделю?",
            placeholder: "Введите количество",
            inputName: "Назначений препаратов от кашля за последнюю неделю",
          };
          this.data[5] = {
            questionCounter: "fd-pd4",
            question: "Если говорить про всех Ваших пациентов, скольким Вы назначили Лазолван<sup>®</sup> за последнюю неделю?",
            placeholder: "Введите количество",
            inputName: "Назначений Лазолвана<sup>®</sup>за последнюю неделю",
            alert: "<span class='bold'>ПРОВЕРЬТЕ!</span><br> Значение в <span>FD-PD4</span> должно быть МЕНЬШЕ<br> чем в <span>FD-PD3</span>"
          };

          this.answers.push(parseInt(this.calcValueTextArray[0]));
          this.answers.push(parseInt(this.calcValueTextArray[1]));
        }
        else if (this.doctorType === "fd" && this.currentQuestion === 1 && (parseInt(this.calcValueTextArray[2]) < 30)){
          this.answers[0] = "fd-gp";
          this.data[2] = {
            questionCounter: "FD_GP1",
            question: "Если говорить про всех Ваших пациентов, скольким Вы назначили гепатопротекторы за последнюю неделю?",
            placeholder: "Введите количество",
            inputName: "Назначений гепатопротекторов за последнюю неделю",
          };
          this.data[3] = {
            questionCounter: "FD_GP2",
            question: "Если говорить про всех Ваших пациентов, скольким Вы назначили Эссенциале<sup>®</sup> Форте Н за последнюю неделю?",
            placeholder: "Введите количество",
            inputName: "Назначений Эссенциале<sup>®</sup> Форте Н за последнюю неделю",
            alert: "<span class='bold'>ПРОВЕРЬТЕ!</span><br> Значение в <span>FD-GP2</span> должно быть МЕНЬШЕ чем в <span>FD-GP1</span>"
          };
          this.data[4] = {
            questionCounter: "FD_GP3",
            question: "Если говорить про всех Ваших пациентов, скольким Вы назначили препараты-пробиотики за последнюю неделю?",
            placeholder: "Введите количество",
            inputName: "Назначений препаратов-пробиотиков за последнюю неделю",
          };
          this.data[5] = {
            questionCounter: "FD_GP4",
            question: "Если говорить про всех Ваших пациентов, скольким Вы назначили Энтерожермину<sup>®</sup> за последнюю неделю?",
            placeholder: "Введите количество",
            inputName: "Назначений Энтерожермину<sup>®</sup> за последнюю неделю",
            alert: "<span class='bold'>ПРОВЕРЬТЕ!</span><br> Значение в <span>FD-GP4</span> должно быть МЕНЬШЕ<br> чем в <span>FD-GP3</span>"
          };

          this.answers.push(parseInt(this.calcValueTextArray[0]));
          this.answers.push(parseInt(this.calcValueTextArray[1]));
        }
        else {
          this.answers.push(this.calcValueText);
        }
        console.log(this.answers);
        this.calcValueText = "Введите количество";
        this.currentQuestion++;
        this.isRadioChecked = false;
        this.isBigger = false;
        this.calcValueTextArray = {
          0: "Введите<br> количество",
          1: "Введите<br> количество",
          2: "Введите<br> процент",
          3: "Введите<br> процент"
        }

        if (this.currentQuestion >= this.data.length){
          let self = this;
          function AddData(ID, Answer){
            try {
              window.parent.addData(ID, Answer);
            } catch (e) {
              console.log('QuestionId: '+ID+' / Answer: '+Answer + "/ Type" + typeof Answer);
            }
          }

          this.answers.forEach(function(item, i){
            if (i === 0){
              AddData("doctorType", item)
            }
            else if (self.answers[0] === "ge" && i === 1){
              AddData("ge" + i, item)
            }
            else {
              AddData(self.answers[0] + i, item + "");
            }
          });
        }
      },

      prevQuestion(){
        if (this.currentQuestion === 0){
          this.$emit("prev")
        }
        else if (this.doctorType === "fd" && this.currentQuestion === 2){
          this.currentQuestion--;
          this.activeInput = Number;
          this.answers.pop();
          this.answers.pop();
        }
        else {
          this.currentQuestion--;
          this.activeInput = Number;
          this.answers.pop();
        }

      },

      setActiveInput(){
        this.calcValueText = "";
        this.isActiveInput = true;
      },

      setActiveInputArr(key){
        if (!this.calcValueTextArray[0].includes('Введите') && this.calcValueTextArray[0] !== ""){
          return
        }
        this.calcValueTextArray = {
          0: "Введите<br> количество",
          1: "Введите<br> количество",
          2: "Введите<br> процент",
          3: "Введите<br> процент"
        };
        this.isActiveInputProcent = [false, false, false, false];

        for (let i = 0; i < Object.keys(this.calcValueTextArray).length; i++){

          if (this.calcValueTextArray[i] === '' && i < 2){
            this.calcValueTextArray[i] = "Введите<br> количество"
          }
          else if (this.calcValueTextArray[i] === '' && i >= 2){
            this.calcValueTextArray[i] = "Введите<br> процент"
          }
        }
        this.calcValueTextArray[key] = ""
        this.$set(this.isActiveInputProcent, key, true);
      },

      addNumber(key){
        if (this.isActiveInputProcent === [false, false, false, false]){
          return
        }
        this.isActiveInput = false;
        if (this.calcValueText === "Введите количество"){
          this.calcValueText = "";
        }

        if (key === 10){
          this.calcValueText = this.calcValueText.slice(0, -1);

          if (this.calcValueText === ""){
            this.isActiveInput = true;
          }
          return
        }
        else if (key === 9){
          if (this.calcValueText === ""){

            this.calcValueText += 0;
            return
          }
          else if (this.calcValueText === "0"){
            this.isActiveInput = false;
            return
          }
          else {
            this.calcValueText += 0;
            return
          }
        }
        else {
          if (this.calcValueText === "0"){
            this.calcValueText = ""
          }
          this.calcValueText += key + 1;
        }
      },

      addNumberProcent(key){
        let allPatients = this.answers[this.answers.length - 1];
        for (let i = 0; i <= this.isActiveInputProcent.length; i++){
          if (this.isActiveInputProcent[i] === true){
            this.activeInput = i;
            this.isActiveInputProcent[i] = false;
          }
        }
        if (this.activeInput === Number){
          return
        }

        if(key === 9){
          key = 0;
        }
        else {
          key++
        }

        if (this.activeInput === 0 || this.activeInput === 1){
          if ((parseInt(this.calcValueTextArray[this.activeInput] + key)) > allPatients && (key != 11)){
            return
          }
        }
        else if (this.activeInput === 2 || this.activeInput === 3){
          if (this.calcValueTextArray[this.activeInput] + key > 100 && (key != 11)){
            return
          }
        }

        if (this.activeInput === undefined){
          return
        }

        else if (key === 11){
          this.calcValueTextArray[this.activeInput] = this.calcValueTextArray[this.activeInput].slice(0, -1);
          if (this.calcValueTextArray[this.activeInput] === ""){
            this.isActiveInputProcent[this.activeInput] = true
          }

        }
        else if (key === 0){
          if (this.calcValueTextArray[this.activeInput] === ""){
            this.calcValueTextArray[this.activeInput] = "0";
          }
          else if (this.calcValueTextArray[this.activeInput] === "0"){
            return
          }
          else {
            this.calcValueTextArray[this.activeInput] += 0;
          }
        }
        else {
          if (this.calcValueTextArray[this.activeInput] === "0"){
            this.calcValueTextArray[this.activeInput] = key + ""
          }
          else {
            this.calcValueTextArray[this.activeInput] += key;
          }
        }

        if(this.activeInput === 0){
          if (this.calcValueTextArray[this.activeInput] === ""){
            this.calcValueTextArray = {
              0: "",
              1: "Введите<br> количество",
              2: "Введите<br> процент",
              3: "Введите<br> процент"
            }
          }
          else{
            this.calcValueTextArray[1] = parseInt(allPatients) - parseInt(this.calcValueTextArray[this.activeInput]);
            this.calcValueTextArray[2] = Math.round(this.calcValueTextArray[this.activeInput] / allPatients * 100);
            this.calcValueTextArray[3] = Math.round(this.calcValueTextArray[1] / allPatients * 100);
          }

        }
        else if(this.activeInput === 1){
          if (this.calcValueTextArray[this.activeInput] === ""){
            this.calcValueTextArray = {
              0: "Введите<br> количество",
              1:  "",
              2: "Введите<br> процент",
              3: "Введите<br> процент"
            }
          }
          else{
            this.calcValueTextArray[0] = parseInt(allPatients) - parseInt(this.calcValueTextArray[this.activeInput]);
            this.calcValueTextArray[2] = Math.round(this.calcValueTextArray[this.activeInput] / allPatients * 100);
            this.calcValueTextArray[3] = Math.round(this.calcValueTextArray[0] / allPatients * 100);
          }
        }
        else if(this.activeInput === 2){
          if (this.calcValueTextArray[this.activeInput] === ""){
            this.calcValueTextArray = {
              0: "Введите<br> количество",
              1: "Введите<br> количество",
              2: "",
              3: "Введите<br> процент"
            }
          }
          else{
            this.calcValueTextArray[0] = (parseInt(allPatients) / 100 * parseInt(this.calcValueTextArray[this.activeInput])).toFixed(0);
            this.calcValueTextArray[1] = (parseInt(allPatients) / 100 * parseInt((100 - this.calcValueTextArray[this.activeInput]))).toFixed(0);
            this.calcValueTextArray[3] = 100 - Math.round(this.calcValueTextArray[this.activeInput]);
          }
        }

        else if (this.activeInput === 3){
          if (this.calcValueTextArray[this.activeInput] === ""){
            this.calcValueTextArray = {
              0: "Введите<br> количество",
              1: "Введите<br> количество",
              2: "Введите<br> процент",
              3: ""
            }
          }
          else{
            this.calcValueTextArray[0] = (parseInt(allPatients) / 100 * parseInt((100 - this.calcValueTextArray[this.activeInput]))).toFixed(0);
            this.calcValueTextArray[1] = (parseInt(allPatients) / 100 * parseInt(this.calcValueTextArray[this.activeInput])).toFixed(0);
            this.calcValueTextArray[2] = 100 - Math.round(this.calcValueTextArray[this.activeInput]);
          }
        }
      }
    },

    mounted(){
      this.answers.push(this.doctorType);

    },

    props: {
      doctorType: {
        type: String
      },
      data: {
        type: Array
      }
    },

    components: {
      arrowNext,
      backArrow
    }
  }
</script>

<style lang="scss" scoped>

  .next {
    width: 270px;
    height: 60px;
    box-sizing: border-box;
    padding: 0 39px 0 50px;

    background: #4F4F4F;
    box-shadow: 0px 4px 7px rgba(0, 0, 0, 0.3);
    border-radius: 50px 0px 0px 50px;

    position: absolute;
    right: 0;
    bottom: 41px;

    @include f(300, #fff);
    font-size: 28px;

    display: flex;
    justify-content: space-around;
    align-items: center;

    &.active {
      background: #4F4F4F;
      box-shadow: inset 0px 4px 10px rgba(0, 0, 0, 0.5);
      border-radius: 50px 0px 0px 50px;

    }
  }

  .question {
    position: absolute;
    top: 0;
    left: 0;

    width: 100%;
    height: 100%;
  }

  .question__type {
    @include f(200, #4F4F4F);
    font-size: 36px;
    text-transform: uppercase;

    position: absolute;
    top: 23px;
    right: 32px;
  }

  .stage {
    position: absolute;
    left: 50%;
    bottom: 40px;
    transform: translateX(-50%);
    &__item {
      width: 12px;
      height: 12px;

      border-radius: 50%;
      background-color: #F2F2F2;

      display: inline-block;
      margin-right: 30px;

      &.active {
        background-color: #BABABA;
      }
    }
  }

  .back {
    position: absolute;
    top: 30px;
    left: 40px;
  }

  .question__number {
    @include f(300);
    font-size: 100px;


    position: absolute;
    top: 65px;
    left: 113px;
  }

  .question__text {
    @include f();
    font-size: 32px;

    position: absolute;
    top: 82px;
    left: 213px;
  }

  .question__descr{
    @include f(300, #161616, italic);
    font-size: 21px;
    line-height: 27px;

    position: absolute;
    left: 213px;

    &_0 {
      top: 175px;
    }

    &_1 {
      @include f();
      font-size: 24px;
      top: 215px;
    }
  }

  .radio {
    .question__text {
      top: 226px;
    }

    .question__descr_0 {
      top: 82px;
      @include f(300, #161616, italic);
      font-size: 21px;
      line-height: 27px;
    }
    .question__descr_1 {
      top: 120px;
      @include f(300, #161616, italic);
      font-size: 21px;
      line-height: 27px;
    }

    .question__radio {
      @include f(300);
      font-size: 24px;

      position: absolute;
      left: 296px;

      display: flex;
      justify-content: space-between;
      align-items: center;

      &_0 {
        top: 338px;
      }
      &_1 {
        top: 418px;
      }

      input {
        opacity: 0;
        visibility: hidden;
      }

      span {
        width: 40px;
        height: 40px;
        display: inline-block;
        margin-right: 27px;
        position: relative;

        border-radius: 50%;
        background: #F8F8F8;
        border: 1px solid #4F4F4F;
        box-sizing: border-box;
        box-shadow: inset 0px 4px 7px rgba(0, 0, 0, 0.25);
      }

      input:checked~span:after {
        content: '';
        width: 24px;
        height: 24px;

        background-color: #4F4F4F;
        border-radius: 50%;

        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
      }
    }
  }

  .calc__input {
    position: absolute;
    width: 400px;
    height: 60px;
    left: 217px;
    top: 311px;

    background: #F8F8F8;
    border: 1px solid #8D8D8D;
    box-sizing: border-box;
    box-shadow: inset 0px 4px 7px rgba(0, 0, 0, 0.15);
    border-radius: 50px;

    @include f(300);
    font-size: 24px;
    color: #8D8D8D;

    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;

    &.red {
      border: 1px solid red;
    }

    &.active {
      background-image: url(../assets/media/images/slide-main/palochka.gif);
      background-position: center center;
      background-repeat: no-repeat;
    }

    &.calc__input_text {
      @include f(300);
      font-size: 42px;
      color: #161616;
    }
  }

  .calc-keyboard {
    width: 477px;

    position: absolute;
    top: 304px;
    left: 679px;

    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;

    &__item {
      width: 75px;
      height: 75px;
      margin-right: 20px;
      margin-bottom: 20px;

      background: #F8F8F8;
      border: 1px solid #4F4F4F;
      box-sizing: border-box;
      box-shadow: 0px 4px 7px rgba(0, 0, 0, 0.2);
      border-radius: 50px;

      display: flex;
      justify-content: center;
      align-items: center;

      @include f(300, #4F4F4F);
      font-size: 36px;

      &:last-child {
        width: 130px;
        height: 60px;

        background: #F8F8F8;
        box-shadow: 0px 4px 7px rgba(0, 0, 0, 0.2);
        border-radius: 30px;

        &:after {
          content: url(../assets/media/images/slide-main/del.png);
          padding-top: 6px;
          padding-right: 4px;
        }
      }

      &:active {
        background: #E8E8E8;
        border: 1px solid #4F4F4F;
        box-sizing: border-box;
        box-shadow: inset 0px 4px 7px rgba(0, 0, 0, 0.2);
        border-radius: 50px;
      }
    }
  }

  .calc__text {
    @include f(300);
    font-size: 21px;
    text-align: center;

    position: absolute;
    top: 390px;
    left: 266px;
  }

  /deep/.alert {
    @include f(300, #CA0000, italic);
    font-size: 21px;

    position: absolute;
    left: 217px;
    top: 495px;

    span {
      @include f(normal, #CA0000, italic);
    }
    .bold {
      @include f(500, #CA0000, italic);
    }
  }

  .finish {
    @include f(300);
    font-size: 48px;

    position: absolute;
    top: 240px;
    left: 50%;
    transform: translateX(-50%);
  }

  .calc-big__item {
    position: absolute;
    width: 160px;
    height: 60px;

    background: #F8F8F8;
    border: 1px solid #8D8D8D;
    box-sizing: border-box;
    box-shadow: inset 0px 4px 7px rgba(0, 0, 0, 0.15);
    border-radius: 50px;
    overflow: hidden;

    display: flex;
    align-items: center;
    justify-content: center;

    @include f(300, #8D8D8D);
    font-size: 18px;
    line-height: 19px;
    text-align: center;

    &.active {
      background-image: url(../assets/media/images/slide-main/palochka.gif);
      background-repeat: no-repeat;
      background-position: center center;
    }

    &.calc__input_text {
      @include f(300);
      font-size: 42px;
      color: #161616;
    }

    &_0 {
      left: 283px;
      top: 300px;
    }

    &_1 {
      left: 473px;
      top: 300px;
    }

    &_2 {
      left: 283px;
      top: 395px;
    }

    &_3 {
      left: 473.05px;
      top: 395px;
    }
  }

  .fd_1 {
    .question__descr_1 {
      top: 86px;
    }
    .question__text {
      top: 123px;
    }
    .question__descr_0 {
      top: 213px;
    }
  }

  .calc-big__value {
    position: absolute;

    &_0 {
      @include f(200, #161616, italic);
      font-size: 18px;
      top: 319px;
      left: 216px;
    }

    &_1 {
      @include f(200, #161616, italic);
      font-size: 36px;
      left: 232px;
      top: 405px;
    }

    &_2 {
      @include f(300, #161616, italic);
      font-size: 21px;
      text-align: center;

      top: 470px;
      left: 307px;
    }
    &_3 {
      @include f(300, #161616, italic);
      font-size: 21px;
      text-align: center;

      top: 470px;
      left: 481px;
    }
  }

  .ge_2 {
    .calc__text {left: 248px;}
  }
  .ge_3, .gyn_0, .pd_0, .pd_3, .gp_2, .fd_2{
    .calc__text {
      left: 236px;
    }
  }
  .gyn_2, .gyn_3 {
    .calc__text {
      left: 313px;
    }
  }

  .pd_2, .gp_1 , .fd_4{
    .calc__text {
      left: 256px;
    }
  }

  .fd_0, .fd_5 {
    .calc__text {
      left: 302px;
    }
  }
</style>
