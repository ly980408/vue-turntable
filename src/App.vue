<template>
  <div id="app">
    <Turntable :prizeList="prizeList" :prizeIndex="prizeIndex" ref="turntable" @end="handleEnd">
      <!-- <template #pointer="{ start }">
        <div @click="start">start</div>
      </template> -->
    </Turntable>
  </div>
</template>

<script>
  import Turntable from './components/Turntable.vue'

  export default {
    name: 'App',
    components: {
      Turntable
    },
    data() {
      return {
        prizeList: [
          {
            name: '一等奖',
            img: ''
          },
          {
            name: '二等奖',
            img: ''
          },
          {
            name: '三等奖',
            img: ''
          },
          {
            name: '特等奖',
            img: ''
          },
          {
            name: '终极大奖',
            img: ''
          },
          {
            name: '谢谢参与',
            img: ''
          }
        ],
        prizeIndex: 3
      }
    },
    methods: {
      handleEnd() {
        console.log(`抽中了“${this.prizeList[this.prizeIndex].name}”！`)
      }
    }
  }
</script>

<style lang="scss">
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
