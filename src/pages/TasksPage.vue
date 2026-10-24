<script setup>
import { ref, computed } from 'vue'
import AppTabbar from '../components/AppTabbar.vue'
import TaskCard from '../components/TaskCard.vue'
import {
  taskState,
  setTaskStatus,
  setTaskPriority
} from '../store/tasks'
import '../css/task-components.css'

const selectedStatus = ref('all')
const selectedPriority = ref('all')
const sortOrder = ref('default')
const selectedTaskId = ref(null)

const statusOptions = [
  { value: 'all', label: 'Усі' },
  { value: 'todo', label: 'Заплановано' },
  { value: 'progress', label: 'У роботі' },
  { value: 'done', label: 'Виконано' },
]

const priorityOptions = [
  { value: 'all', label: 'Усі' },
  { value: 'low', label: 'Низький' },
  { value: 'medium', label: 'Середній' },
  { value: 'high', label: 'Високий' },
]

const filteredTasks = computed(() => {
  const result = taskState.tasks.filter((task) => (
    (selectedStatus.value === 'all' ||
      task.status === selectedStatus.value) &&
    (selectedPriority.value === 'all' ||
      task.priority === selectedPriority.value)
  ))

  const order = { low: 1, medium: 2, high: 3 }

  if (sortOrder.value === 'high') {
    result.sort((a, b) => order[b.priority] - order[a.priority])
  }

  if (sortOrder.value === 'low') {
    result.sort((a, b) => order[a.priority] - order[b.priority])
  }

  return result
})

const selectedTask = computed(() =>
  taskState.tasks.find((task) => task.id === selectedTaskId.value)
)

function handleStatusChange(payload) {
  setTaskStatus(payload)
}

function handlePriorityChange(payload) {
  setTaskPriority(payload)
}

function handleTaskSelect(payload) {
  selectedTaskId.value = payload.id
}
</script>

<template>
  <f7-page name="tasks" class="tf3-page">
    <f7-navbar title="Мої завдання" />

    <div class="tf3-container">
      <section class="tf3-hero">
        <div class="tf3-eyebrow">TASKFLOW / TASK MANAGER</div>
        <h1>Мої завдання</h1>
        <p>Керуйте завданнями, статусами та пріоритетами.</p>
      </section>

      <section class="tf3-filters">
        <div class="tf3-filter-group">
          <div class="tf3-filter-title">Фільтр за статусом</div>

          <div class="tf3-filter-buttons">
            <button
              v-for="option in statusOptions"
              :key="option.value"
              type="button"
              class="tf3-filter-button"
              :class="{ 'is-active': selectedStatus === option.value }"
              :aria-pressed="selectedStatus === option.value"
              @click="selectedStatus = option.value"
            >
              {{ option.label }}
            </button>
          </div>
        </div>

        <div class="tf3-filter-group">
          <div class="tf3-filter-title">Фільтр за пріоритетом</div>

          <div class="tf3-filter-buttons">
            <button
              v-for="option in priorityOptions"
              :key="option.value"
              type="button"
              class="tf3-filter-button"
              :class="{ 'is-active': selectedPriority === option.value }"
              :aria-pressed="selectedPriority === option.value"
              @click="selectedPriority = option.value"
            >
              {{ option.label }}
            </button>
          </div>
        </div>

        <div class="tf3-filter-group">
          <label class="tf3-filter-title" for="task-sort">
            Сортування за пріоритетом
          </label>

          <select
            id="task-sort"
            v-model="sortOrder"
            class="tf3-sort"
          >
            <option value="default">Початковий порядок</option>
            <option value="high">Спочатку високий</option>
            <option value="low">Спочатку низький</option>
          </select>
        </div>
      </section>

      <section
        v-if="selectedTask"
        class="tf3-section tf3-stat"
        aria-live="polite"
      >
        <span class="tf3-stat-label">Обране завдання</span>
        <h2 class="tf3-selected-title">{{ selectedTask.title }}</h2>
        <p>{{ selectedTask.description }}</p>

        <button
          type="button"
          class="tf3-action"
          @click="selectedTaskId = null"
        >
          Скасувати вибір
        </button>
      </section>

      <section class="tf3-section">
        <div class="tf3-section-heading">
          <div>
            <div class="tf3-eyebrow">TASK LIST</div>
            <h2>Список завдань</h2>
          </div>

          <div class="tf3-counter">
            Знайдено: {{ filteredTasks.length }}
          </div>
        </div>

        <div v-if="filteredTasks.length" class="tf3-task-grid">
          <TaskCard
            v-for="task in filteredTasks"
            :key="task.id"
            :task="task"
            @status-change="handleStatusChange"
            @priority-change="handlePriorityChange"
            @task-select="handleTaskSelect"
          />
        </div>

        <div v-else class="tf3-empty">
          За обраними фільтрами завдань не знайдено.
        </div>
      </section>
    </div>

    <AppTabbar active="tasks" />
  </f7-page>
</template>

<style scoped>
.tf3-sort {
  box-sizing: border-box;
  width: 100%;
  padding: 12px;
  border: 1px solid #334b67;
  border-radius: 9px;
  background: #1e3047;
  color: #fff;
  font: inherit;
}

.tf3-selected-title {
  margin: 0;
  color: #fff;
  font-size: 20px;
}
</style>