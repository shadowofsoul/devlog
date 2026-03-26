<template>
  <v-app>
    <v-app-bar color="primary" flat>
      <v-app-bar-title>David R. Comba Lareu DevLog</v-app-bar-title>

      <v-spacer></v-spacer>

      <v-btn icon href="https://github.com/shadowofsoul" target="_blank">
        <v-icon>mdi-github</v-icon>
      </v-btn>
      <v-btn icon href="https://www.instagram.com/ipsilondev" target="_blank">
        <v-icon>mdi-instagram</v-icon>
      </v-btn>
      <v-btn icon href="https://www.youtube.com/channel/UCaXrFwjnQYr6i2dxzU9ieAQ" target="_blank">
        <v-icon>mdi-youtube</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-container class="py-8">
        <v-row class="mb-6">
          <v-col cols="6" md="3">
            <v-card variant="tonal" color="primary" class="pa-4 text-center">
              <div class="text-h4 font-weight-bold">{{ filteredStats.ships }}</div>
              <div class="text-caption">Ships</div>
            </v-card>
          </v-col>
          <v-col cols="6" md="3">
            <v-card variant="tonal" color="secondary" class="pa-4 text-center">
              <div class="text-h4 font-weight-bold">{{ filteredStats.commits }}</div>
              <div class="text-caption">Commits</div>
            </v-card>
          </v-col>
          <v-col cols="6" md="3">
            <v-card variant="tonal" color="purple" class="pa-4 text-center">
              <div class="text-h4 font-weight-bold">{{ filteredStats.content }}</div>
              <div class="text-caption">Content</div>
            </v-card>
          </v-col>
        </v-row>

        <div class="activity-section mb-6">
          <div class="d-flex align-center mb-2">
            <div class="text-subtitle-1 mr-4">{{ currentYear }}</div>
            <v-btn
              v-if="selectedDate"
              variant="text"
              size="small"
              color="secondary"
              @click="selectedDate = null"
            >
              <v-icon start>mdi-close-circle</v-icon>
              Clear date
            </v-btn>
          </div>
          <div class="activity-grid">
            <div 
              v-for="(monthData, monthIndex) in yearActivity" 
              :key="monthIndex"
              class="month-column"
            >
              <div class="month-label text-caption mb-1">{{ months[monthIndex] }}</div>
              <div class="days-grid">
                <div 
                  v-for="(day, dayIndex) in monthData.days" 
                  :key="dayIndex"
                  class="activity-block"
                  :class="['level-' + day.level, { selected: isDateSelected(monthIndex, dayIndex) }]"
                  :style="{ backgroundColor: day.color || 'transparent' }"
                  @click="toggleDate(monthIndex, dayIndex)"
                ></div>
              </div>
            </div>
          </div>
        </div>

        <div class="filters mb-6">
          <v-row align="center">
            <v-col cols="12" md="4">
              <v-select
                v-model="selectedTags"
                :items="allTags"
                label="Filter by Tags"
                multiple
                chips
                clearable
                variant="outlined"
                density="comfortable"
              ></v-select>
            </v-col>
          </v-row>
          <v-row v-if="selectedTags.length || selectedDate">
            <v-col cols="12">
              <v-btn
                color="warning"
                variant="tonal"
                size="small"
                prepend-icon="mdi-filter-remove"
                @click="clearFilters"
              >
                Clear all filters
              </v-btn>
            </v-col>
          </v-row>
        </div>

        <v-timeline v-if="filteredLogs.selected.length || filteredLogs.rest.length" side="end">
          <v-timeline-item
            v-for="log in filteredLogs.selected"
            :key="'selected-' + log.id"
            :dot-color="getTagColor(log.tags[0])"
            size="small"
          >
            <template v-slot:opposite>
              <div class="text-caption text-grey">{{ formatDate(log.date) }}</div>
            </template>
            <v-card class="elevation-2" variant="elevated">
              <v-card-title class="text-h6 d-flex align-center">
                {{ log.title }}
                <v-btn
                  v-if="log.link"
                  icon
                  size="small"
                  variant="text"
                  :href="log.link"
                  target="_blank"
                  class="ml-2"
                >
                  <v-icon size="small">mdi-open-in-new</v-icon>
                </v-btn>
              </v-card-title>
              <v-card-text>
                <div class="mb-2">{{ log.content }}</div>
                <div v-if="log.images && log.images.length" class="mb-3">
                  <div class="d-flex flex-wrap gap-2">
                    <v-card
                      v-for="(img, idx) in log.images"
                      :key="idx"
                      width="80"
                      height="80"
                      class="rounded-lg cursor-pointer overflow-hidden"
                      @click="openImageDialog(log.images, idx)"
                    >
                      <v-img
                        :src="img"
                        width="80"
                        height="80"
                        cover
                      ></v-img>
                    </v-card>
                  </div>
                </div>
                <div class="d-flex flex-wrap align-center gap-2">
                  <div v-if="log.commits" class="text-caption text-grey mr-2">
                    <v-icon size="x-small" class="mr-1">mdi-source-commit</v-icon>
                    {{ log.commits }}
                  </div>
                  <v-chip
                    v-for="tag in log.tags"
                    :key="tag"
                    :color="getTagColor(tag)"
                    size="small"
                    variant="tonal"
                  >
                    {{ tag }}
                  </v-chip>
                </div>
              </v-card-text>
            </v-card>
          </v-timeline-item>
          
          <div v-if="filteredLogs.hasSelection && filteredLogs.rest.length" class="month-header my-4">
            <v-divider></v-divider>
            <div class="text-overline text-grey text-center my-2">
              More from {{ months[selectedDate.month] }} {{ currentYear }}
            </div>
            <v-divider></v-divider>
          </div>
          
          <v-timeline-item
            v-for="log in filteredLogs.rest"
            :key="'rest-' + log.id"
            :dot-color="getTagColor(log.tags[0])"
            size="small"
          >
            <template v-slot:opposite>
              <div class="text-caption text-grey">{{ formatDate(log.date) }}</div>
            </template>
            <v-card class="elevation-2" variant="elevated">
              <v-card-title class="text-h6 d-flex align-center">
                {{ log.title }}
                <v-btn
                  v-if="log.link"
                  icon
                  size="small"
                  variant="text"
                  :href="log.link"
                  target="_blank"
                  class="ml-2"
                >
                  <v-icon size="small">mdi-open-in-new</v-icon>
                </v-btn>
              </v-card-title>
              <v-card-text>
                <div class="mb-2">{{ log.content }}</div>
                <div v-if="log.images && log.images.length" class="mb-3">
                  <div class="d-flex flex-wrap gap-2">
                    <v-card
                      v-for="(img, idx) in log.images"
                      :key="idx"
                      width="80"
                      height="80"
                      class="rounded-lg cursor-pointer overflow-hidden"
                      @click="openImageDialog(log.images, idx)"
                    >
                      <v-img
                        :src="img"
                        width="80"
                        height="80"
                        cover
                      ></v-img>
                    </v-card>
                  </div>
                </div>
                <div class="d-flex flex-wrap align-center gap-2">
                  <div v-if="log.commits" class="text-caption text-grey mr-2">
                    <v-icon size="x-small" class="mr-1">mdi-source-commit</v-icon>
                    {{ log.commits }}
                  </div>
                  <v-chip
                    v-for="tag in log.tags"
                    :key="tag"
                    :color="getTagColor(tag)"
                    size="small"
                    variant="tonal"
                  >
                    {{ tag }}
                  </v-chip>
                </div>
              </v-card-text>
            </v-card>
          </v-timeline-item>
        </v-timeline>

        <v-alert v-else type="info" variant="tonal">
          No updates found matching your filters.
        </v-alert>

        <v-fab
          v-if="selectedTags.length || selectedDate"
          color="warning"
          icon="mdi-filter-remove"
          size="small"
          class="fab-filter"
          @click="clearFilters"
        ></v-fab>

        <v-dialog v-model="imageDialog" max-width="800">
          <v-carousel
            v-if="dialogImages.length"
            v-model="dialogSlide"
            hide-delimiters
            height="auto"
            class="rounded"
          >
            <v-carousel-item
              v-for="(img, idx) in dialogImages"
              :key="idx"
              :src="img"
              cover
            ></v-carousel-item>
          </v-carousel>
        </v-dialog>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const logs = ref([])
const selectedTags = ref([])
const selectedDate = ref(null)
const imageDialog = ref(false)
const dialogImages = ref([])
const dialogSlide = ref(0)

const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']

const allTags = computed(() => {
  const tags = new Set(['apps', 'launch', 'code', 'update', 'content', '3d-print', 'design'])
  return Array.from(tags)
})

const currentYear = computed(() => {
  if (logs.value.length === 0) return new Date().getFullYear()
  const years = new Set()
  logs.value.forEach(log => years.add(new Date(log.date + 'T00:00:00').getFullYear()))
  return Math.max(...Array.from(years))
})

const yearActivity = computed(() => {
  const year = currentYear.value
  
  const activity = []
  const logsInYear = logs.value.filter(log => new Date(log.date + 'T00:00:00').getFullYear() === year)
  
  for (let month = 0; month < 12; month++) {
    const daysInMonth = new Date(year, month + 1, 0).getDate()
    const days = []
    
    for (let day = 1; day <= daysInMonth; day++) {
      const dayLogs = logsInYear.filter(log => {
        const d = new Date(log.date + 'T00:00:00')
        return d.getMonth() === month && d.getDate() === day
      })
      
      const hasLogs = dayLogs.length > 0
      let color = null
      let level = 0
      
      if (hasLogs) {
        const primaryTag = dayLogs[0].tags[0]
        const tagColors = {
          apps: '#2196F3',
          launch: '#4CAF50',
          code: '#FFC107',
          update: '#FF9800',
          content: '#9C27B0',
          '3d-print': '#00BCD4',
          design: '#E91E63'
        }
        color = tagColors[primaryTag] || '#666666'
        level = 3
      }
      
      days.push({ hasLogs, color, level })
    }
    activity.push({ days })
  }
  
  return activity
})

const filteredStats = computed(() => {
  const filtered = filteredLogs.value
  const all = [...filtered.selected, ...filtered.rest]
  return {
    ships: all.filter(log => log.tags.includes('launch')).length,
    commits: all.reduce((sum, log) => sum + (log.commits || 0), 0),
    content: all.filter(log => (log.tags.includes('3d-print') || log.tags.includes('content')) && !log.tags.includes('launch')).length
  }
})

const filteredLogs = computed(() => {
  let monthLogs = logs.value.filter(log => {
    const matchesTags = selectedTags.value.length === 0 ||
      selectedTags.value.some(tag => log.tags.includes(tag))
    
    const logDate = new Date(log.date + 'T00:00:00')
    const matchesYear = logDate.getFullYear() === currentYear.value
    const matchesMonth = !selectedDate.value || logDate.getMonth() === selectedDate.value.month
    
    return matchesTags && matchesYear && matchesMonth
  }).sort((a, b) => new Date(b.date + 'T00:00:00') - new Date(a.date + 'T00:00:00'))
  
  if (selectedDate.value) {
    const selectedDay = selectedDate.value.day + 1
    const selected = monthLogs.filter(log => new Date(log.date + 'T00:00:00').getDate() === selectedDay)
    const rest = monthLogs.filter(log => new Date(log.date + 'T00:00:00').getDate() !== selectedDay)
    return { selected, rest, hasSelection: true }
  }
  
  return { selected: [], rest: monthLogs, hasSelection: false }
})

function formatDate(dateStr) {
  return new Date(dateStr + 'T00:00:00').toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric'
  })
}

function getTagColor(tag) {
  const colors = {
    apps: 'blue',
    launch: 'green',
    code: 'amber',
    update: 'orange',
    content: 'purple',
    '3d-print': 'cyan',
    design: 'pink'
  }
  return colors[tag] || 'grey'
}

function isDateSelected(month, day) {
  return selectedDate.value && 
    selectedDate.value.month === month && 
    selectedDate.value.day === day
}

function toggleDate(month, day) {
  if (isDateSelected(month, day)) {
    selectedDate.value = null
  } else {
    selectedDate.value = { month, day }
  }
}

function clearFilters() {
  selectedTags.value = []
  selectedDate.value = null
}

function openImageDialog(images, index) {
  dialogImages.value = images
  dialogSlide.value = index
  imageDialog.value = true
}

onMounted(async () => {
  try {
    const response = await fetch('/devlog/devlog.json')
    logs.value = await response.json()
  } catch (error) {
    console.error('Failed to load dev log:', error)
  }
})
</script>

<style scoped>
.gap-2 {
  gap: 8px;
}

.month-chip {
  cursor: pointer;
}

.cursor-pointer {
  cursor: pointer;
}

.activity-section {
  padding: 12px;
  background: rgba(0,0,0,0.02);
  border-radius: 8px;
}

.activity-grid {
  display: flex;
  gap: 8px;
  overflow-x: auto;
  padding: 8px 0;
}

.month-column {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.month-label {
  font-size: 10px;
  color: #666;
}

.days-grid {
  display: grid;
  grid-template-columns: repeat(7, 12px);
  gap: 2px;
}

.activity-block {
  width: 12px;
  height: 12px;
  border-radius: 2px;
  cursor: pointer;
  transition: transform 0.1s;
}

.activity-block:hover {
  transform: scale(1.3);
}

.activity-block.selected {
  outline: 2px solid #fff;
  outline-offset: 1px;
  box-shadow: 0 0 4px rgba(0,0,0,0.5);
}

.activity-block.level-0 {
  background-color: #ebedf0;
}

.activity-block.level-1 {
  background-color: #9be9a8;
}

.activity-block.level-2 {
  background-color: #40c463;
}

.activity-block.level-3 {
  background-color: #30a14e;
}

.activity-block.level-4 {
  background-color: #216e39;
}

.fab-filter {
  position: fixed;
  bottom: 24px;
  right: 24px;
  z-index: 100;
}
</style>
