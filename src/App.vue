<template>
  <div class="max-w-6xl mx-auto py-8 px-4">
    <!-- <h1 class="text-3xl font-bold mb-6">東京銭湯リスト</h1> -->
    <div class="flex items-center justify-between mb-6">
      <button @click="currentTab = 'list'" class="text-2xl">
        <i class="fas fa-list"></i>
      </button>
      <button @click="currentTab = 'home'">
        <h1 class="text-3xl font-bold text-center">東京銭湯リスト</h1>
      </button>
      <button @click="currentTab = 'myList'; currentBlock = '全て'" class="text-2xl">
        <i class="fas fa-user"></i>
      </button>
    </div>


    <template v-if="currentTab !== 'myList'">
      <div class="mb-4">
        <button
          v-for="(block, name) in groupedBlocks"
          :key="name"
          @click="currentBlock = name; currentArea = ''"
          :class="[
            'px-4 py-2 mr-2 mb-2 rounded font-semibold',
            currentBlock === name
              ? `bg-${block.color}-600 text-white`
              : `border border-${block.color}-600 text-${block.color}-600 bg-white`
          ]"
        >
          {{ name }} ({{ block.sentos.length }})
        </button>
      </div>
  
      <div class="mb-6" v-if="areaTabs.length > 0">
        <button
          v-for="area in areaTabs"
          :key="area"
          @click="currentArea = area"
          :class="[
            'px-3 py-1 mr-2 mb-2 rounded text-sm',
            currentArea === area
              ? 'bg-gray-800 text-white'
              : 'bg-white text-gray-800 border border-gray-400'
          ]"
        >
          {{ area }} ({{ areaCounts[area] || 0 }})
        </button>
      </div>
    </template>
    <template v-else>
      <hr>
      <h2 class="text-center my-5">制覇済み: {{ filteredSentos.filter(isBookmarked).length }}件</h2>
    </template>

    
    <div v-if="groupedBlocks[currentBlock]" class="grid grid-cols-2 gap-3 gap-y-6 lg:grid-cols-3 lg:gap-6">
      <template v-for="sento in filteredSentos" :key="sento.href">
        <div v-if="currentTab !== 'myList' || isBookmarked(sento)" class="bg-gray-100 rounded shadow overflow-hidden pb-[50px] relative">
          <img
            :src="sento.thumbnail"
            alt="thumbnail"
            class="w-full h-auto object-cover"
          />
          <div class="p-3 lg:p-4">
            <h2 class="text-center text-l font-semibold">{{ sento.name }} [{{ sento.location }}]</h2>
            <div class="justify-center gap-3 flex mt-3">
              <i v-if="checkIcon(sento,'サウナ')" class="fa-solid fa-fire-flame-curved text-red-500"></i>
              <i v-if="checkIcon(sento,'水風呂')" class="fa-solid fa-droplet text-blue text-blue-500"></i>
            </div>
            <div class="absolute left-1/2 bottom-[10px] transform -translate-x-1/2 w-[80%] mt-4 flex justify-between text-gray-600 text-xl">
                <!-- Link Icon -->
                <a :href="sento.href" target="_blank" rel="noopener noreferrer">
                    <i class="fas fa-link text-black"></i>
                </a>
                <!-- Star Icon (e.g. for rating or favorite) -->
                <button @click="toggleFavorite(sento)">
                    <i
                        :class="[
                            'fas',
                            isFavorite(sento) ? 'fa-star text-yellow-400' : 'fa-star text-gray-400'
                        ]"
                    ></i>
                </button>
                <!-- Bookmark Icon (e.g. for myList) -->
                <button @click="toggleBookmark(sento)">
                    <i
                        :class="[
                            'fas',
                            isBookmarked(sento) ? 'fa-bookmark text-blue-400' : 'fa-bookmark text-gray-400'
                        ]"
                    ></i>
                </button>
            </div>
            <!-- <p class="text-sm mb-2">{{ sento.summary }}</p> -->
            <!-- <p class="text-xs mb-2">
              <i class="fas fa-map-marker-alt"></i> {{ sento.access }}
            </p>
            <p class="text-xs mb-2">
              <i class="fas fa-clock"></i> {{ sento.hours }}
            </p>
            <p class="text-xs">
              <i class="fas fa-calendar-minus"></i> {{ sento.holiday }}
            </p> -->
          </div>
          <!-- <div class="mt-2 flex flex-wrap gap-1">
            <span
              v-for="tag in sento.tags"
              :key="tag"
              class="text-xs bg-blue-100 text-blue-800 px-2 py-1 rounded"
            >
              {{ tag }}
            </span>
          </div> -->
        </div>
      </template>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import sentoData from './const/tokyo-sento.json'

const data = ref([])
const groupedBlocks = ref({})
const currentBlock = ref('中央ブロック')
const currentArea = ref('')
const currentTab = ref('list')

const myList = ref([]);
const myFavoriteList = ref([]);

const blocks = [
  {
    name: '中央ブロック',
    color: 'purple',
    areas: ['千代田区', '中央区', '台東区', '文京区', '港区', '荒川区']
  },
  {
    name: '城東ブロック',
    color: 'pink',
    areas: ['墨田区', '江東区', '足立区', '葛飾区', '江戸川区']
  },
  {
    name: '城西ブロック',
    color: 'yellow',
    areas: ['新宿区', '渋谷区', '中野区', '杉並区']
  },
  {
    name: '城南ブロック',
    color: 'blue',
    areas: ['品川区', '大田区', '世田谷区', '目黒区']
  },
  {
    name: '城北ブロック',
    color: 'red',
    areas: ['豊島区', '北区', '板橋区', '練馬区']
  },
  {
    name: '三多摩ブロック',
    color: 'green',
    areas: []
  }
]

const groupByBlock = () => {
    const result = {
        全て: {
            color: 'gray', // or whatever color you want
            sentos: [...data.value] // include all sentos
        }
    }

    blocks.forEach(block => {
        result[block.name] = {
            color: block.color,
            sentos: []
        }
    })

    data.value.forEach(sento => {
        const found = blocks.find(block => block.areas.includes(sento.location))
        const blockName = found ? found.name : '三多摩ブロック'
        result[blockName].sentos.push(sento)
    })

    groupedBlocks.value = result
}

const isBookmarked = (sento) =>{
  return myList.value.includes(sento.href);
}

const isFavorite = (sento) =>{
  return myFavoriteList.value.includes(sento.href);
}

const toggleBookmark = (sento) => {
    const index = myList.value.indexOf(sento.href)

    if (index === -1) {
        // Not in list, so add it
        myList.value.push(sento.href)
    } else {
        // Already in list, confirm and remove
        const confirmed = window.confirm('この銭湯をお気に入りから削除しますか？')
        if (confirmed) {
          myList.value.splice(index, 1)
        }
    }

    // Save to localStorage every time
    localStorage.setItem('myList', JSON.stringify(myList.value))
}

const toggleFavorite = (sento) => {
    const index = myFavoriteList.value.indexOf(sento.href)

    if (index === -1) {
        // Not in list, so add it
        myFavoriteList.value.push(sento.href);
        toggleBookmark(sento);
    } else {
        // Already in list, confirm and remove
        const confirmed = window.confirm('お気に入りから削除しますか？')
        if (confirmed) {
          myFavoriteList.value.splice(index, 1)

        }
    }

    // Save to localStorage every time
    localStorage.setItem('myFavoriteList', JSON.stringify(myFavoriteList.value))
}

const checkIcon = (sento,property) => {
  if(sento.tags){
    return sento.tags.includes(property)
  }
}


const areaTabs = computed(() => {
  const list = groupedBlocks.value[currentBlock.value]?.sentos.map(s => s.location)
  return list ? [...new Set(list)] : []
})

const areaCounts = computed(() => {
  const counts = {}
  const sentos = groupedBlocks.value[currentBlock.value]?.sentos || []
  sentos.forEach(sento => {
    if (!counts[sento.location]) counts[sento.location] = 0
    counts[sento.location]++
  })
  return counts
})

const filteredSentos = computed(() => {
  const sentos = groupedBlocks.value[currentBlock.value]?.sentos || []
  return currentArea.value
    ? sentos.filter(s => s.location === currentArea.value)
    : sentos
})

onMounted(() => {
  const saved = localStorage.getItem('myList')
  if (saved) {
    myList.value = JSON.parse(saved)
  }

  const favoriteSaved = localStorage.getItem('myFavoriteList')
  if (favoriteSaved) {
    myFavoriteList.value = JSON.parse(favoriteSaved)
  }

  data.value = sentoData
  groupByBlock()
})
</script>
