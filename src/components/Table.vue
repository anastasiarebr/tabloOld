<template>
<div class="body">
<div class="titles">
      <div class="title">
        <span>Поступающий</span>
        <span>№ стола</span>
      </div>
      <div class="title">
        <span>Поступающий</span>
        <span>№ стола</span>
      </div>
</div>
  <div class="main">
    <div class="column">
      <div class="column-items" >
        <div class="column-item " v-for="(talon, index) in list.show" :key="index" :class="(indexInOrder(index) < 5 && indexInOrder(index) % 2 === 0) || (indexInOrder(index) > 4 && indexInOrder(index) % 2 !== 0) ? 'bgdblue' : ''">
          <div class="person">{{talon.talon}}</div>
          <div class="arrow">
              <svg width="91" height="65" viewBox="0 0 91 65" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M5.46961 37.7646C5.84196 37.8144 6.21878 37.8374 6.59581 37.8334L70.7945 37.8334L69.3946 38.37C68.0263 38.9038 66.7815 39.6302 65.7159 40.5165L47.713 55.3541C45.342 57.2195 44.9436 60.2204 46.7689 62.4643C48.8933 64.8554 52.9674 65.3746 55.8688 63.6237C56.1032 63.4822 56.326 63.3279 56.5354 63.1619L89.0904 36.3308C91.6346 34.2363 91.6368 30.8387 89.0955 28.7419C89.0939 28.7405 89.092 28.739 89.0904 28.7377L56.5354 1.90661C53.9892 -0.186042 49.8667 -0.181348 47.3274 1.91718C47.1276 2.08235 46.941 2.2581 46.7689 2.44323C44.9436 4.68714 45.342 7.68803 47.713 9.55346L65.6833 24.4179C66.6386 25.206 67.737 25.8669 68.9388 26.3765L70.8921 27.101L6.95412 27.101C3.62801 26.9992 0.709861 28.9141 0.0850081 31.6086C-0.490605 34.534 1.9201 37.2901 5.46961 37.7646Z" :fill="chooseColor(talon.status)"/>
              </svg>
          </div>
          <div class="table">{{talon.number_of_table}}</div>
        </div>
      </div>
    </div>
  </div>
  </div>
  <transition name="fade">
    <div v-if="addNewItem === 1" class="newItem">
      Абитуриент с талоном <span>{{newItem.talon}}</span> подойдите к столу <span>{{newItem.number_of_table}}</span>
  </div>
  </transition>
</template>

<script>
import { onMounted, ref, reactive, computed } from 'vue'
import { NEWCOLOR, REPEATCOLOR, OLDCOLOR } from './composables/Colors'
import { SHOW_TALON, DELAY_ADD_TALON, DELAY_UPDATE_LIST } from './composables/Time'

export default {
  setup () {
    // let sec = 0
    const showCount = ref(0)
    const axios = require('axios')
    const data = ref('')
    const newColor = NEWCOLOR
    const repeatColor = REPEATCOLOR
    const oldColor = OLDCOLOR
    const addNewItem = ref(0)
    const newItem = ref('')
    const audio = new Audio(require('../sound/sound.mp3'))
    const list = reactive({
      pre: [],
      forShow: [],
      show: {}
    })
    const url = 'https://api.sdo.mpgu.org/queue'
    // axios.defaults.headers.common[Authorization] = `Bearer ${token}`

    const lastKey = computed({
      get: () => Object.keys(data.value).reverse()[0]
    })

    const lastShowKey = computed({
      get: () => Object.keys(list.show).reverse()[0]
    })

    const showLength = computed({
      get: () => Object.keys(list.show).length
    })

    function indexInOrder (item) {
      return Object.keys(list.show).indexOf(item)
    }

    onMounted(() => {
      // setInterval(() => {
      //   console.log(sec)
      //   sec += 1
      // }, 1000)
      setInterval(() => {
        request()
      }, DELAY_UPDATE_LIST)
    })

    // console.log('from request')
    // https://tablo-83a92-default-rtdb.firebaseio.com/queue.json

    const username = 'adminsdotest'
    const password = 'Fjjj76654ghhd43'
    // const token = Buffer.from(`${username}:${password}`, 'utf8').toString('base64')

    function request () {
      axios.get(url, {
        auth: {
          username: username,
          password: password
        }
      })
        .then(response => {
          data.value = response.data
          let i = 0
          for (i in data.value) {
            if (!(data.value[i])) {
              delete data.value[i]
            }
          }
        })
        .then(() => {
          setTimeout(() => {
            updateList()
          }, 2000)
        })
    }

    function updateList () {
      if (data.value.length !== 0) {
        if (showCount.value === 0) {
          fillList(0)
        } else {
          deleteItems()
          fillList(+lastShowKey.value + 1)
        }
      }
    }

    function deleteItems () {
      console.log('from delete')
      let i = 0
      for (i in list.show) {
        if (!data.value[i]) {
          showCount.value--
          delete list.show[i]
        }
      }
    }

    function fillList (i) {
      if (i <= lastKey.value && showCount.value < 10) {
        if (data.value[i]) {
          setTimeout(() => {
            add(i, data.value[i]).then(() => {
              showCount.value++
              fillList(i + 1)
            })
          }, DELAY_ADD_TALON)
        } else {
          fillList(i + 1)
        }
      }
    }
    async function add (i, item) {
      audio.play()
      addNewItem.value = 1
      newItem.value = item
      await new Promise(resolve => {
        setTimeout(() => {
          pushItem(i, item)
          addNewItem.value = 0
          resolve('done')
        }, SHOW_TALON)
      })
    }

    async function pushItem (i, item) {
      await new Promise(resolve => {
        list.show[i] = (item)
        resolve('done')
      })
    }

    function chooseColor (status) {
      if (status === 0) {
        return '#EB5757'
      } else if (status === 1) {
        return '#1E9EE6'
      } else {
        return '#27AE60'
      }
    }

    return {
      list,
      addNewItem,
      newItem,
      newColor,
      repeatColor,
      oldColor,
      chooseColor,
      showLength,
      indexInOrder
    }
  }
}
</script>

<style lang="sass" scoped>
@import '@/sass/style'
</style>
