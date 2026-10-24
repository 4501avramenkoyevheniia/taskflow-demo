<script setup>
import { ref, computed } from 'vue'
import AppTabbar from '../components/AppTabbar.vue'
import TaskCard from '../components/TaskCard.vue'
import TaskCounter from '../components/TaskCounter.vue'
import {
  taskState,
  setTaskStatus,
  setTaskPriority
} from '../store/tasks'
import '../css/task-components.css'

const selectedTaskId = ref(null)

const totalTasks = computed(() => taskState.tasks.length)

const completedTasks = computed(() =>
  taskState.tasks.filter((task) => task.status === 'done').length
)

const progressTasks = computed(() =>
  taskState.tasks.filter((task) => task.status === 'progress').length
)

const plannedTasks = computed(() =>
  taskState.tasks.filter((task) => task.status === 'todo').length
)

const completedPercent = computed(() =>
  totalTasks.value === 0
    ? 0
    : Math.round(completedTasks.value / totalTasks.value * 100)
)

const activeTasks = computed(() => {
  const order = { high: 0, medium: 1, low: 2 }

  return taskState.tasks
    .filter((task) => task.status !== 'done')
    .sort((a, b) => order[a.priority] - order[b.priority])
    .slice(0, 3)
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
  <f7-page name="home" class="tf3-page">
    <f7-navbar title="TaskFlow">
      <f7-nav-right>
        <f7-link href="/about/">Про застосунок</f7-link>
      </f7-nav-right>
    </f7-navbar>

    <div class="tf3-container">
      <section class="tf3-hero">
        <div class="tf3-eyebrow">TASKFLOW / DASHBOARD</div>
        <h1>Панель керування</h1>
        <p>Контролюйте виконання завдань та їхні пріоритети.</p>
      </section>

      <section class="tf3-stat-grid">
        <TaskCounter :count="totalTasks" label="Усього завдань" />
        <TaskCounter :count="progressTasks" label="У роботі" />
        <TaskCounter :count="completedTasks" label="Виконано" />
        <TaskCounter :count="plannedTasks" label="Заплановано" />
      </section>

      <section class="tf3-section">
        <div class="tf3-stat">
          <span class="tf3-stat-label">
            Відсоток виконаних завдань
          </span>
          <strong>{{ completedPercent }}%</strong>
          <progress
            :value="completedPercent"
            max="100"
            aria-label="Відсоток виконаних завдань"
          ></progress>
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
            <div class="tf3-eyebrow">PRIORITY TASKS</div>
            <h2>Актуальні завдання</h2>
          </div>

          <f7-button href="/tasks/" outline small>
            Переглянути всі
          </f7-button>
        </div>

        <div v-if="activeTasks.length" class="tf3-task-grid">
          <TaskCard
            v-for="task in activeTasks"
            :key="task.id"
            :task="task"
            @status-change="handleStatusChange"
            @priority-change="handlePriorityChange"
            @task-select="handleTaskSelect"
          />
        </div>

        <div v-else class="tf3-empty">
          Наразі всі завдання виконані.
        </div>
      </section>
    </div>

    <AppTabbar active="home" />
  </f7-page>
</template>

<style scoped>
progress {
  width: 100%;
  height: 16px;
  accent-color: #35b4e5;
}

.tf3-selected-title {
  margin: 0;
  color: #fff;
  font-size: 20px;
}
</style>