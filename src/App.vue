<template>
  <div class="max-w-6xl mx-auto py-8 px-4">
    <h1 class="text-3xl font-bold mb-6">東京銭湯リスト</h1>

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

    <div v-if="groupedBlocks[currentBlock]" class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
      <div
        v-for="sento in filteredSentos"
        :key="sento.name"
        class="bg-white rounded shadow p-4"
      >
        <img
          :src="sento.thumbnail"
          alt="thumbnail"
          class="rounded w-full h-40 object-cover mb-4"
        />
        <h2 class="text-xl font-semibold">{{ sento.name }}</h2>
        <p class="text-sm text-gray-500 mb-2">
          {{ sento.furigana }} - {{ sento.location }}
        </p>
        <p class="text-sm mb-2">{{ sento.summary }}</p>
        <p class="text-sm">
          <i class="fas fa-map-marker-alt"></i> {{ sento.access }}
        </p>
        <p class="text-sm">
          <i class="fas fa-clock"></i> {{ sento.hours }}
        </p>
        <p class="text-sm">
          <i class="fas fa-calendar-minus"></i> {{ sento.holiday }}
        </p>
        <div class="mt-2 flex flex-wrap gap-1">
          <span
            v-for="tag in sento.tags"
            :key="tag"
            class="text-xs bg-blue-100 text-blue-800 px-2 py-1 rounded"
          >
            {{ tag }}
          </span>
        </div>
      </div>
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
  const result = {}
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
  data.value = sentoData
  groupByBlock()
})
</script>

<style scoped>
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css');
</style>
