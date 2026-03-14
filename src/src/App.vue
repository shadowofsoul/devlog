<template>
  <v-app>
    <v-app-bar color="primary" flat>
      <v-app-bar-title>Dev Log</v-app-bar-title>
    </v-app-bar>

    <v-main>
      <v-container class="py-8">
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
            <v-col cols="12" md="4">
              <v-text-field
                v-model="dateFrom"
                label="From Date"
                type="date"
                variant="outlined"
                density="comfortable"
                clearable
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="4">
              <v-text-field
                v-model="dateTo"
                label="To Date"
                type="date"
                variant="outlined"
                density="comfortable"
                clearable
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12">
              <v-btn
                v-if="selectedTags.length || dateFrom || dateTo"
                variant="text"
                color="secondary"
                @click="clearFilters"
              >
                Clear Filters
              </v-btn>
            </v-col>
          </v-row>
        </div>

        <v-timeline v-if="filteredLogs.length" side="end">
          <v-timeline-item
            v-for="log in filteredLogs"
            :key="log.id"
            :dot-color="getTagColor(log.tags[0])"
            size="small"
          >
            <template v-slot:opposite>
              <div class="text-caption text-grey">{{ formatDate(log.date) }}</div>
            </template>
            <v-card class="elevation-2" variant="elevated">
              <v-card-title class="text-h6">{{ log.title }}</v-card-title>
              <v-card-text>
                <div class="mb-3">{{ log.content }}</div>
                <div v-if="log.images && log.images.length" class="mb-3">
                  <v-img
                    v-for="(img, idx) in log.images"
                    :key="idx"
                    :src="img"
                    class="mb-2 rounded"
                    max-height="300"
                    cover
                  ></v-img>
                </div>
                <div class="d-flex flex-wrap gap-2">
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
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const logs = ref([])
const selectedTags = ref([])
const dateFrom = ref('')
const dateTo = ref('')

const allTags = computed(() => {
  const tags = new Set()
  logs.value.forEach(log => log.tags.forEach(tag => tags.add(tag)))
  return Array.from(tags).sort()
})

const filteredLogs = computed(() => {
  return logs.value.filter(log => {
    const matchesTags = selectedTags.value.length === 0 ||
      selectedTags.value.some(tag => log.tags.includes(tag))
    
    const logDate = new Date(log.date)
    const matchesFrom = !dateFrom.value || logDate >= new Date(dateFrom.value)
    const matchesTo = !dateTo.value || logDate <= new Date(dateTo.value)
    
    return matchesTags && matchesFrom && matchesTo
  }).sort((a, b) => new Date(b.date) - new Date(a.date))
})

function formatDate(dateStr) {
  return new Date(dateStr).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric'
  })
}

function getTagColor(tag) {
  const colors = {
    update: 'blue',
    launch: 'green',
    tutorial: 'orange',
    github: 'purple',
    vue: 'teal',
    code: 'amber'
  }
  return colors[tag] || 'grey'
}

function clearFilters() {
  selectedTags.value = []
  dateFrom.value = ''
  dateTo.value = ''
}

onMounted(async () => {
  try {
    const response = await fetch('/davidc.github.io/devlog.json')
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
</style>
