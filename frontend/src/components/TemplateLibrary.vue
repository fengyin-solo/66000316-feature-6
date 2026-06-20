<template>
  <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
    <h3 class="text-sm font-bold text-slate-400 mb-3">模板库 ({{ templates.length }})</h3>
    <input v-model="search" placeholder="搜索模板..." class="w-full bg-slate-900 border border-slate-600 rounded-lg px-3 py-1.5 text-sm mb-3 focus:outline-none focus:border-cyan-500" />
    <div class="space-y-2 max-h-96 overflow-y-auto">
      <template v-if="search">
        <div v-for="t in filtered" :key="t.name"
          @click="store.applyTemplate(t)"
          :class="['cursor-pointer p-2 rounded-lg border transition-all', store.selectedTemplate === t.name ? 'border-cyan-500 bg-cyan-900/30' : 'border-slate-700 bg-slate-900 hover:border-slate-500']">
          <div class="flex items-center justify-between">
            <span class="text-sm font-bold text-slate-200">{{ t.name }}</span>
            <div class="flex items-center gap-1">
              <button @click.stop="store.toggleFavorite(t.name)" class="text-sm hover:scale-110 transition-transform">
                <span :class="store.isFavorite(t.name) ? 'text-yellow-400' : 'text-slate-600'">★</span>
              </button>
              <span class="text-xs px-1.5 py-0.5 rounded bg-slate-700 text-slate-400">{{ t.category }}</span>
            </div>
          </div>
          <div class="text-xs text-slate-500 mt-1">{{ t.description }}</div>
          <div class="text-xs font-mono text-cyan-500 mt-1 truncate">{{ t.pattern }}</div>
        </div>
        <div v-if="filtered.length === 0" class="text-sm text-slate-500 text-center py-4">无匹配模板</div>
      </template>

      <template v-else>
        <div v-if="favoriteList.length > 0" class="border border-yellow-600/40 rounded-lg bg-yellow-900/10 overflow-hidden">
          <div class="flex items-center justify-between px-3 py-2 bg-yellow-900/20 cursor-pointer" @click="toggleFavoritesCollapsed">
            <span class="text-sm font-bold text-yellow-400 flex items-center gap-1">
              <span>★</span> 收藏 ({{ favoriteList.length }})
            </span>
            <span class="text-slate-400 text-xs">{{ favoritesCollapsed ? '展开' : '收起' }}</span>
          </div>
          <div v-show="!favoritesCollapsed" class="p-2 space-y-1">
            <div v-for="t in favoriteList" :key="t.name"
              @click="store.applyTemplate(t)"
              :class="['cursor-pointer p-2 rounded-lg border transition-all', store.selectedTemplate === t.name ? 'border-cyan-500 bg-cyan-900/30' : 'border-slate-700 bg-slate-900 hover:border-slate-500']">
              <div class="flex items-center justify-between">
                <span class="text-sm font-bold text-slate-200">{{ t.name }}</span>
                <button @click.stop="store.toggleFavorite(t.name)" class="text-sm hover:scale-110 transition-transform text-yellow-400">
                  ★
                </button>
              </div>
              <div class="text-xs text-slate-500 mt-1">{{ t.description }}</div>
              <div class="text-xs font-mono text-cyan-500 mt-1 truncate">{{ t.pattern }}</div>
            </div>
          </div>
        </div>

        <div v-for="cat in categories" :key="cat" class="border border-slate-700 rounded-lg overflow-hidden">
          <div class="flex items-center justify-between px-3 py-2 bg-slate-700/50 cursor-pointer" @click="store.toggleCategory(cat)">
            <span class="text-sm font-bold text-slate-300">{{ cat }} ({{ getTemplatesByCategory(cat).length }})</span>
            <span class="text-slate-400 text-xs">{{ store.isCollapsed(cat) ? '展开' : '收起' }}</span>
          </div>
          <div v-show="!store.isCollapsed(cat)" class="p-2 space-y-1">
            <div v-for="t in getTemplatesByCategory(cat)" :key="t.name"
              @click="store.applyTemplate(t)"
              :class="['cursor-pointer p-2 rounded-lg border transition-all', store.selectedTemplate === t.name ? 'border-cyan-500 bg-cyan-900/30' : 'border-slate-700 bg-slate-900 hover:border-slate-500']">
              <div class="flex items-center justify-between">
                <span class="text-sm font-bold text-slate-200">{{ t.name }}</span>
                <button @click.stop="store.toggleFavorite(t.name)" class="text-sm hover:scale-110 transition-transform">
                  <span :class="store.isFavorite(t.name) ? 'text-yellow-400' : 'text-slate-600'">★</span>
                </button>
              </div>
              <div class="text-xs text-slate-500 mt-1">{{ t.description }}</div>
              <div class="text-xs font-mono text-cyan-500 mt-1 truncate">{{ t.pattern }}</div>
            </div>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useRegexStore, TEMPLATES } from '../store/regex'
import type { RegexTemplate } from '../types'

const store = useRegexStore()
const templates: RegexTemplate[] = TEMPLATES
const search = ref('')
const favoritesCollapsed = ref(false)

const filtered = computed(() => {
  if (!search.value) return []
  const q = search.value.toLowerCase()
  const result = templates.filter(t =>
    t.name.toLowerCase().includes(q) ||
    t.description.toLowerCase().includes(q) ||
    t.pattern.toLowerCase().includes(q)
  )
  result.sort((a, b) => {
    const aFav = store.isFavorite(a.name) ? 1 : 0
    const bFav = store.isFavorite(b.name) ? 1 : 0
    return bFav - aFav
  })
  return result
})

const categories = computed(() => {
  const cats = new Set<string>()
  templates.forEach(t => cats.add(t.category))
  return Array.from(cats)
})

const favoriteList = computed(() => {
  return templates.filter(t => store.isFavorite(t.name))
})

function getTemplatesByCategory(cat: string): RegexTemplate[] {
  return templates.filter(t => t.category === cat && !store.isFavorite(t.name))
}

function toggleFavoritesCollapsed() {
  favoritesCollapsed.value = !favoritesCollapsed.value
}
</script>
