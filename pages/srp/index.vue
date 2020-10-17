<template>
  <div class="w-screen bg-gray-200 font-sans flex justify-center items-center sm:flex-col md:flex-row xl:flex-row h-screen">
    <section class="mx-10 bg-white flex flex-col w-screen sm:w-3/4 md:w-2/7 lg:w-1/4 xl:w-1/4">
      <header class="text-center w-auto py-4 rounded px-5">
        <h1 class="text-purple-500 text-4xl font-bold shadow">剪刀石头布</h1>
      </header>
      <main class="flex flex-col">
        <h2 class="text-purple-500 pb-4 text-2xl text-center">得分</h2>
        <article class="flex flex-row">
          <aside class="w-1/2 flex flex-col text-center leading-10 py-1">
            <span class="text-4xl text-purple-400 py-1">{{humanScore}}</span>
            <span class="text-xl text-gray-900 py-1">你</span>
            <span class="text-xl text-gray-800 py-1">{{(humanSelect|typeFilter)||'XX'}}</span>
          </aside>
          <div class="w-1 h-auto bg-black"></div>
          <aside class="w-1/2 flex flex-col text-center leading-10">
            <span class="text-4xl text-purple-400 py-1">{{AiScore}}</span>
            <span class="text-xl text-gray-900 py-1">AI</span>
            <span class="text-xl text-gray-800 py-1">{{(AiSelect|typeFilter)||'XX'}}</span>
          </aside>
        </article>
        <footer class="text-center">
          <h1
            v-if="winner=='你'"
            class="text-green-500 text-3xl mt-3"
          >{{winner}}</h1>
          <h1
            v-if="winner=='Ai'"
            class="text-red-500 text-3xl mt-3"
          >{{winner}}</h1>
          <h1
            v-if="winner=='平手'"
            class="text-orange-500 text-3xl mt-3"
          >{{winner}}</h1>
          <article class="flex flex-row items-center justify-center">
            <button
              class="px-4 py-2 m-2 xl:m-10 xl:px-6 xl:py-4  text-white duration-500 bg-indigo-500 rounded hover:bg-indigo-600"
              @click="humanInput(1)"
            >剪刀</button>
            <button
              class="px-4 py-2 m-2 xl:m-10 xl:px-6 xl:py-4  text-white duration-500 bg-indigo-500 rounded hover:bg-indigo-600"
              @click="humanInput(2)"
            >石头</button>
            <button
              class="px-4 py-2 m-2 xl:m-10 xl:px-6 xl:py-4 text-white duration-500 bg-indigo-500 rounded hover:bg-indigo-600"
              @click="humanInput(3)"
            >布</button>
          </article>
        </footer>
      </main>
    </section>
    <section class="xl:mx-10 sm:mx-0 text-center sm:my-10  w-screen sm:w-3/4 md:w-2/7 lg:w-1/4 xl:w-1/4">
      <table>
          <tr class="bg-gray-500">
            <td class="xl:px-5 xl:py-1 sm:px-5">局数</td>
            <td class="xl:px-5 xl:py-1 sm:px-5">你</td>
            <td class="xl:px-5 xl:py-1 sm:px-5">Ai</td>
            <td class="xl:px-5 xl:py-1 sm:px-5">Ai认为你会出</td>
            <td class="xl:px-5 xl:py-1 sm:px-5">胜方</td>
          </tr>
        </thead>
        <tbody class="divide-y-2 divide-gray-800">
          <tr v-for="item in totalList" :key="item.index">
            <td>{{item.index}}</td>
            <td>{{item.human|typeFilter}}</td>
            <td>{{item.Ai|typeFilter}}</td>
            <td>{{item.AiGuest|typeFilter}}</td>
            <td>{{item.winner}}</td>
          </tr>
        </tbody>
      </table>
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
      totalList: [],
      humanSelect: 0,
      AiSelect: 0,
      humanWillSelect:0,
      humanScore: 0,
      AiScore: 0,
      winner: '',
      conut: 0,
      maxLength: 12
    }
  },
  methods: {

    async humanInput (data) {
      this.humanSelect = data
      await this.AiGuest()
      this.conut++
      this.whoWin()
    },
    async AiGuest () {
      this.prepareData()
      const net = new brain.recurrent.LSTMTimeStep()
      net.train([ this.list ], { iterations: 100, log: true })
      this.humanWillSelect = Math.round(net.run(this.list))
      console.log(this.humanWillSelect)
      this.updateList()
      this.AiSelect = 1 <= this.humanWillSelect && this.humanWillSelect <= 3 ? (this.humanWillSelect % 3) + 1 : 1
    },
    whoWin () {
      if (this.humanSelect == this.AiSelect) {
        this.winner = "平手"
      } else if (
        (this.humanSelect === 1 && this.AiSelect === 3) ||
        (this.humanSelect === 3 && this.AiSelect === 2) ||
        (this.humanSelect === 2 && this.AiSelect === 1)
      ) {
        this.winner = '你'
        this.humanScore++
      } else {
        this.winner = 'Ai'
        this.AiScore++
      }
      this.totalList.push({
        index:this.conut,
        human:this.humanSelect,
        Ai:this.AiSelect,
        AiGuest:this.humanWillSelect,
        winner:this.winner
      })
    },
    updateList () {
      if (this.list.length !== 0) {
        this.list.shift()
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