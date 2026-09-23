<template>
  <div class="bg-gray-800/60 rounded-xl p-3">
    <h3 class="text-blue-300 font-bold text-sm mb-2">星座目录</h3>

    <!-- group tabs -->
    <div class="flex flex-wrap gap-1 mb-2">
      <button v-for="g in tabs" :key="g.key"
        @click="activeGroup = g.key"
        :class="['px-2 py-0.5 rounded text-xs transition-colors',
          activeGroup === g.key
            ? 'bg-blue-600 text-white'
            : 'bg-gray-700 text-gray-300 hover:bg-gray-600']">
        {{ g.label }}
      </button>
    </div>

    <!-- sort & filter -->
    <div class="flex items-center gap-2 mb-1 text-xs text-gray-300">
      <label class="flex items-center gap-1 cursor-pointer select-none">
        <input type="checkbox" v-model="sortByMag" class="accent-blue-500" />
        按视星等
      </label>
      <span class="text-gray-500">·</span>
      <label class="flex items-center gap-1 whitespace-nowrap">
        亮于
        <select v-model.number="maxMag" class="bg-gray-900 rounded px-1 py-0.5 text-xs">
          <option :value="4">全部</option>
          <option :value="1">1.0</option>
          <option :value="1.5">1.5</option>
          <option :value="2">2.0</option>
          <option :value="2.5">2.5</option>
          <option :value="3">3.0</option>
        </select>
        等
      </label>
    </div>

    <!-- selection lives in another group hint -->
    <div v-if="selectedOutsideGroup"
      class="text-[11px] text-amber-300/90 bg-amber-500/10 rounded px-2 py-1 mb-1 flex items-center justify-between gap-1">
      <span>已选: {{ store.selectedStar!.nameCn }}（{{ selectedConstellation?.nameCn }}）</span>
      <button class="underline hover:text-amber-200" @click="jumpToSelectedGroup">前往</button>
    </div>

    <!-- groups -->
    <div v-if="visibleGroups.length === 0" class="py-6 text-center text-gray-400 text-xs leading-relaxed">
      <p class="text-gray-300 mb-1">没有符合条件的成员星</p>
      <p>当前分组中没有亮于 {{ maxMag.toFixed(1) }} 等的恒星，<br />试试放宽星等筛选或切换星座分组。</p>
    </div>

    <div v-else class="space-y-2 max-h-80 overflow-y-auto pr-1">
      <section v-for="group in visibleGroups" :key="group.const.name">
        <h4 class="text-xs text-blue-200/80 sticky top-0 bg-gray-800/90 py-0.5">
          {{ group.const.nameCn }}
          <span class="text-gray-500">{{ group.const.name }}</span>
          <span class="text-gray-500">（{{ group.stars.length }}）</span>
        </h4>
        <ul>
          <li v-for="star in group.stars" :key="star.name"
            @click="store.focusOn(star)"
            :class="['flex items-center justify-between gap-2 px-2 py-1 rounded mt-0.5 cursor-pointer text-xs',
              isSelected(star)
                ? 'bg-amber-500/25 ring-1 ring-amber-400/70'
                : 'hover:bg-gray-700/70']">
            <span class="min-w-0">
              <span :class="isSelected(star) ? 'text-amber-300 font-semibold' : 'text-gray-100'">
                {{ star.nameCn }}
              </span>
              <span class="text-gray-400 ml-1">{{ star.name }}</span>
            </span>
            <span class="shrink-0 text-gray-400 tabular-nums">
              <span :class="star.mag <= 1.5 ? 'text-sky-300' : ''">m{{ star.mag.toFixed(2) }}</span>
              <span class="ml-1 text-gray-500">{{ star.spectral }}</span>
            </span>
          </li>
        </ul>
      </section>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useSkyStore } from '../store/sky'
import type { Star, Constellation } from '../types'

const store = useSkyStore()

const activeGroup = ref<string>('all')
const sortByMag = ref(false)
const maxMag = ref(4)

const tabs = computed(() => [
  { key: 'all', label: '全部' },
  ...store.CONSTELLATIONS.map(c => ({ key: c.name, label: c.nameCn })),
])

function membersOf(c: Constellation): Star[] {
  return c.stars.map(i => store.STARS[i])
}

interface CatalogGroup {
  const: Constellation
  stars: Star[]
}

const visibleGroups = computed<CatalogGroup[]>(() => {
  const consts = activeGroup.value === 'all'
    ? store.CONSTELLATIONS
    : store.CONSTELLATIONS.filter(c => c.name === activeGroup.value)

  const groups: CatalogGroup[] = []
  for (const c of consts) {
    let stars = membersOf(c).filter(s => s.mag < maxMag.value)
    if (sortByMag.value) stars = [...stars].sort((a, b) => a.mag - b.mag)
    if (stars.length) groups.push({ const: c, stars })
  }
  return groups
})

function isSelected(star: Star): boolean {
  return store.selectedStar === star
}

const selectedConstellation = computed<Constellation | undefined>(() => {
  const sel = store.selectedStar
  if (!sel) return undefined
  return store.CONSTELLATIONS.find(c => membersOf(c).includes(sel))
})

const selectedOutsideGroup = computed(() => {
  const sel = store.selectedStar
  if (!sel || activeGroup.value === 'all') return false
  return selectedConstellation.value?.name !== activeGroup.value
})

function jumpToSelectedGroup() {
  if (selectedConstellation.value) activeGroup.value = selectedConstellation.value.name
}
</script>
