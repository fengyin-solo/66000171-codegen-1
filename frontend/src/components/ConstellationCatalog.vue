<template>
  <div class="text-xs">
    <h4 class="text-gray-400 mb-2">星座目录</h4>

    <!-- Group tabs -->
    <div class="grid grid-cols-3 gap-1 mb-2">
      <button v-for="(c, i) in store.CONSTELLATIONS" :key="c.name"
        @click="activeGroup = i"
        class="rounded px-1 py-1.5 transition-colors"
        :class="i === activeGroup ? 'bg-blue-600 text-white' : 'bg-gray-800 text-gray-300 hover:bg-gray-700'">
        {{ c.nameCn }}
      </button>
    </div>

    <!-- Sort & filter -->
    <div class="flex gap-1 mb-2">
      <select v-model="sortMode" class="flex-1 min-w-0 bg-gray-800 rounded px-2 py-1.5 text-gray-200">
        <option value="default">默认顺序</option>
        <option value="asc">星等 亮→暗</option>
        <option value="desc">星等 暗→亮</option>
      </select>
      <select v-model="magLimit" class="flex-1 min-w-0 bg-gray-800 rounded px-2 py-1.5 text-gray-200">
        <option :value="null">星等不限</option>
        <option :value="1.5">亮于 1.5 等</option>
        <option :value="2.0">亮于 2.0 等</option>
        <option :value="2.5">亮于 2.5 等</option>
        <option :value="3.0">亮于 3.0 等</option>
      </select>
    </div>

    <!-- Group header -->
    <div class="text-gray-500 mb-1 px-0.5">
      {{ group.nameCn }} {{ group.name }} · {{ members.length }}/{{ group.stars.length }} 颗
    </div>

    <!-- Member list -->
    <div v-if="members.length" class="space-y-1">
      <div v-for="s in members" :key="s.name"
        @click="store.focusOnStar(s)"
        class="flex items-center justify-between gap-2 rounded px-2 py-1.5 cursor-pointer transition-colors"
        :class="s === store.selectedStar
          ? 'bg-amber-500/20 ring-1 ring-amber-400'
          : 'bg-gray-800 hover:bg-gray-700'">
        <div class="min-w-0">
          <span class="text-gray-100">{{ s.nameCn }}</span>
          <span class="text-gray-400 ml-1.5">{{ s.name }}</span>
        </div>
        <div class="flex items-center gap-1.5 shrink-0 text-gray-300">
          <span>{{ s.mag.toFixed(2) }} 等</span>
          <span class="inline-flex items-center gap-1">
            <i class="w-2 h-2 rounded-full inline-block" :style="{ background: store.spectralColor(s.spectral) }"></i>
            {{ s.spectral }} 型
          </span>
        </div>
      </div>
    </div>

    <!-- Empty state -->
    <div v-else class="bg-gray-800/60 rounded px-3 py-4 text-center text-gray-400">
      <p>{{ group.nameCn }}当前没有符合筛选条件的成员星</p>
      <p class="mt-1 text-gray-500">试试放宽星等上限，或切换到其他星座分组</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { useSkyStore } from '../store/sky'

const store = useSkyStore()

const activeGroup = ref(0)
const sortMode = ref<'default' | 'asc' | 'desc'>('default')
const magLimit = ref<number | null>(null)

const group = computed(() => store.CONSTELLATIONS[activeGroup.value])

const members = computed(() => {
  let list = group.value.stars.map(i => store.STARS[i])
  if (magLimit.value !== null) list = list.filter(s => s.mag <= magLimit.value!)
  if (sortMode.value === 'asc') list = [...list].sort((a, b) => a.mag - b.mag)
  else if (sortMode.value === 'desc') list = [...list].sort((a, b) => b.mag - a.mag)
  return list
})

// Keep the panel in sync when a star is picked on the canvas or via search:
// jump to the group that contains the newly selected star.
watch(() => store.selectedStar, (s) => {
  if (!s) return
  const idx = store.CONSTELLATIONS.findIndex(c => c.stars.some(i => store.STARS[i] === s))
  if (idx >= 0 && idx !== activeGroup.value) activeGroup.value = idx
})
</script>
