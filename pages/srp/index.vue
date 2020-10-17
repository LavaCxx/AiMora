<template>
  <div class="w-screen bg-gray-200 font-sans flex flex-col justify-center h-screen">
    <section class="mx-auto bg-white flex flex-col w-screen sm:w-screen md:w-4/5 lg:w-3/4 xl:w-2/3">
      <header class="text-center w-auto py-4 rounded">
        <h1 class="text-purple-500 text-4xl font-bold">剪刀石头布</h1>
      </header>
      <main class="flex flex-col">
        <h2 class="text-purple-500 pb-4 text-2xl text-center">得分</h2>
        <article class="flex">
          <aside class="w-1/2 flex flex-col text-center leading-10">
            <span class="text-4xl text-purple-400">{{humanScore}}</span>
            <span class="text-xl text-gray-900">你</span>
            <span class="text-xl text-gray-800">{{humanSelect|typeFilter}}</span>
          </aside>
          <div class="w-1 h-auto bg-black"></div>
          <aside class="w-1/2 flex flex-col text-center leading-10">
            <span class="text-4xl text-purple-400">{{AiScore}}</span>
            <span class="text-xl text-gray-900">AI</span>
            <span class="text-xl text-gray-800">{{AiSelect|typeFilter}}</span>
          </aside>
        </article>
        <footer class=" text-center">
          <h1 class="text-red-700 text-3xl mt-3">{{winner}}</h1>
          <article class="flex flex-row items-center justify-center">
            <button
              class="px-4 py-2 m-2 xl:m-10 xl:px-10 xl:py-4  text-white duration-500 bg-indigo-500 rounded hover:bg-indigo-600"
              @click="humanInput(1)"
            >剪刀</button>
            <button
              class="px-4 py-2 m-2 xl:m-10 xl:px-10 xl:py-4  text-white duration-500 bg-indigo-500 rounded hover:bg-indigo-600"
              @click="humanInput(2)"
            >石头</button>
            <button
              class="px-4 py-2 m-2 xl:m-10 xl:px-10 xl:py-4 text-white duration-500 bg-indigo-500 rounded hover:bg-indigo-600"
              @click="humanInput(3)"
            >布</button>
          </article>
        </footer>
      </main>
    </section>
  </div>
</template>

<script>
export default {
  head: {
    script: [ { src: '//unpkg.com/brain.js' } ]
  },
  data () {
    return {
      list: [],
      humanSelect: 0,
      AiSelect: 0,
      humanScore: 0,
      AiScore: 0,
      winner: '',
      conut: 0,
      maxLength:10
    }
  },
  methods: {

    async humanInput (data) {
      this.humanSelect = data
      await this.AiGuest()
      this.whoWin()
    },
    async AiGuest () {
      this.prepareData()
      const net = new brain.recurrent.LSTMTimeStep()
      net.train([ this.list ], { iterations: 100, log: true })
      let humanWillSelect=Math.round(net.run(this.list))
      console.log(humanWillSelect)
      this.updateList()
this.AiSelect = 1 <= humanWillSelect && humanWillSelect <= 3 ? (humanWillSelect % 3) + 1 : 1
    },
    whoWin(){
        if(this.humanSelect==this.AiSelect){
            this.winner="平手"
        }else if(
        (this.humanSelect === 1 && this.AiSelect === 3) ||
        (this.humanSelect === 3 && this.AiSelect === 2) ||
        (this.humanSelect === 2 && this.AiSelect === 1)
      ){
          this.winner='你'
          this.humanScore++
      }else{
          this.winner='Ai'
          this.AiScore++
      }
    },
    updateList(){
        if(this.list.length!==0){
            this.list.shift();
            this.list.push(this.humanSelect)
        }
    },
    prepareData () {
      if (this.list.length < 1) {
        for (let i = 1; i <= this.maxLength; i++) {
          this.list.push(Math.floor(Math.random() * 3) + 1)
        }
      }
    },
  },
  filters: {
    typeFilter (type) {
      switch (type) {
        case 1:
          return '剪刀'; break
        case 2:
          return '石头'; break
        case 3:
          return '布'
      }
    }
  }
}
</script>

<style>
</style>