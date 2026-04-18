<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen" class="fixed inset-0 bg-slate-900/50 flex items-center justify-center z-[2000] p-4" @click="close">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-3xl max-h-[90vh] flex flex-col overflow-hidden" @click.stop>

          <div class="flex items-center justify-between px-6 py-5 border-b border-slate-200">
            <h3 class="text-lg font-semibold text-slate-900">{{ t('tasks.title') }}</h3>
            <button class="text-slate-400 hover:text-slate-600 hover:bg-slate-100 p-2 rounded-lg transition-colors" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="flex-1 overflow-y-auto px-6 py-5">

            <!-- Add Task Form -->
            <div class="bg-blue-50 rounded-xl p-5 mb-6">
              <div class="flex gap-3 mb-3">
                <div class="flex flex-col gap-1.5 flex-1">
                  <label for="task-title" class="text-sm font-semibold text-slate-600">{{ t('tasks.taskTitle') }}</label>
                  <input
                    id="task-title"
                    v-model="newTask.title"
                    type="text"
                    :placeholder="t('tasks.taskTitlePlaceholder')"
                    class="flex-1 border border-slate-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-400 focus:border-transparent"
                    @keyup.enter="handleAddTask"
                  />
                </div>
              </div>

              <div class="flex gap-3 items-end">
                <div class="flex flex-col gap-1.5 flex-1">
                  <label for="task-priority" class="text-sm font-semibold text-slate-600">{{ t('tasks.priority') }}</label>
                  <select
                    id="task-priority"
                    v-model="newTask.priority"
                    class="border border-slate-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-400 focus:border-transparent bg-white cursor-pointer"
                  >
                    <option value="high">{{ t('priority.high') }}</option>
                    <option value="medium">{{ t('priority.medium') }}</option>
                    <option value="low">{{ t('priority.low') }}</option>
                  </select>
                </div>

                <div class="flex flex-col gap-1.5 flex-1">
                  <label for="task-due-date" class="text-sm font-semibold text-slate-600">{{ t('tasks.dueDate') }}</label>
                  <input
                    id="task-due-date"
                    v-model="newTask.dueDate"
                    type="date"
                    class="border border-slate-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-400 focus:border-transparent"
                  />
                </div>

                <button
                  @click="handleAddTask"
                  class="bg-blue-400 hover:bg-blue-500 text-white text-sm font-semibold px-4 py-2 rounded-lg transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
                  :disabled="!newTask.title.trim() || !newTask.dueDate"
                >
                  {{ t('tasks.addTask') }}
                </button>
              </div>
            </div>

            <div class="h-px bg-slate-200 my-5"></div>

            <!-- Tasks List -->
            <div v-if="sortedTasks.length === 0" class="text-center py-12 text-slate-500 text-base italic">
              {{ t('tasks.noTasks') }}
            </div>

            <div v-else class="flex flex-col gap-3">
              <div
                v-for="task in sortedTasks"
                :key="task.id"
                class="bg-white border-2 border-slate-200 rounded-xl px-5 py-4 transition-all hover:border-slate-300 hover:shadow-sm"
                :class="{
                  'border-l-4 border-l-rose-400': task.priority === 'high',
                  'border-l-4 border-l-amber-400': task.priority === 'medium',
                  'border-l-4 border-l-blue-500': task.priority === 'low',
                  'opacity-60': task.status === 'completed'
                }"
              >
                <div class="flex justify-between items-start mb-3 gap-4">
                  <div class="flex items-center gap-3 flex-1">
                    <input
                      type="checkbox"
                      :checked="task.status === 'completed'"
                      @change="$emit('toggle-task', task.id)"
                      class="accent-blue-400 w-4 h-4 shrink-0 cursor-pointer"
                    />
                    <span
                      class="flex-1 cursor-pointer select-none text-slate-900 text-base font-semibold leading-snug"
                      :class="{ 'line-through text-slate-400': task.status === 'completed' }"
                      @click="$emit('toggle-task', task.id)"
                    >{{ task.title }}</span>
                  </div>
                  <button
                    @click="$emit('delete-task', task.id)"
                    class="w-7 h-7 bg-rose-1000 hover:bg-red-600 text-white border-none rounded-md text-xl leading-none cursor-pointer transition-all flex items-center justify-center p-0 shrink-0 hover:scale-110"
                    title="Delete task"
                  >
                    &times;
                  </button>
                </div>

                <div class="flex items-center gap-3">
                  <span
                    class="inline-flex items-center text-xs font-semibold px-2.5 py-0.5 rounded-full uppercase tracking-wide"
                    :class="{
                      'bg-rose-200 text-rose-700': task.priority === 'high',
                      'bg-amber-200 text-amber-700': task.priority === 'medium',
                      'bg-slate-100 text-slate-700': task.priority === 'low'
                    }"
                  >
                    {{ translatePriority(task.priority) }}
                  </span>

                  <div class="flex items-center gap-1.5 text-sm text-slate-500">
                    <svg width="14" height="14" viewBox="0 0 14 14" fill="none" class="text-slate-400">
                      <rect x="2" y="3" width="10" height="9" rx="1" stroke="currentColor" stroke-width="1.2"/>
                      <path d="M4.5 1.5V4.5M9.5 1.5V4.5M2 6H12" stroke="currentColor" stroke-width="1.2" stroke-linecap="round"/>
                    </svg>
                    {{ formatDueDate(task.dueDate) }}
                  </div>

                  <span
                    class="ml-auto inline-flex items-center text-xs font-semibold px-2.5 py-0.5 rounded-full"
                    :class="{
                      'bg-rose-200 text-rose-700': getStatusClass(task.dueDate, task.status) === 'overdue',
                      'bg-amber-200 text-amber-700': getStatusClass(task.dueDate, task.status) === 'urgent',
                      'bg-blue-100 text-blue-800': getStatusClass(task.dueDate, task.status) === 'upcoming',
                      'bg-emerald-200 text-emerald-700': getStatusClass(task.dueDate, task.status) === 'completed'
                    }"
                  >
                    {{ getStatusText(task.dueDate, task.status) }}
                  </span>
                </div>
              </div>
            </div>
          </div>

          <div class="px-6 py-4 border-t border-slate-200 flex justify-end gap-3">
            <button class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-sm font-semibold px-4 py-2 rounded-lg transition-colors" @click="close">{{ t('profileDetails.close') }}</button>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script>
import { ref, computed } from 'vue'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'TasksModal',
  props: {
    isOpen: {
      type: Boolean,
      required: true
    },
    tasks: {
      type: Array,
      default: () => []
    }
  },
  emits: ['close', 'add-task', 'delete-task', 'toggle-task'],
  setup(props, { emit }) {
    const { t, currentLocale } = useI18n()
    const newTask = ref({
      title: '',
      priority: 'medium',
      dueDate: ''
    })

    const sortedTasks = computed(() => {
      // Don't sort - just return tasks in their current order (newest first)
      return [...props.tasks]
    })

    const close = () => {
      emit('close')
    }

    const handleAddTask = () => {
      if (newTask.value.title.trim() && newTask.value.dueDate) {
        emit('add-task', {
          title: newTask.value.title.trim(),
          priority: newTask.value.priority,
          dueDate: newTask.value.dueDate
        })
        newTask.value = {
          title: '',
          priority: 'medium',
          dueDate: ''
        }
      }
    }

    const formatDueDate = (dateString) => {
      const date = new Date(dateString)
      const today = new Date()
      today.setHours(0, 0, 0, 0)
      const dueDate = new Date(date)
      dueDate.setHours(0, 0, 0, 0)

      const diffTime = dueDate - today
      const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))

      const isJapanese = currentLocale.value === 'ja'

      if (diffDays === 0) return isJapanese ? '今日' : 'today'
      if (diffDays === 1) return isJapanese ? '明日' : 'tomorrow'
      if (diffDays === -1) return isJapanese ? '昨日' : 'yesterday'
      if (diffDays < 0) return isJapanese ? `${Math.abs(diffDays)}日前` : `${Math.abs(diffDays)} days ago`
      if (diffDays < 7) return isJapanese ? `${diffDays}日後` : `in ${diffDays} days`

      const locale = isJapanese ? 'ja-JP' : 'en-US'
      return date.toLocaleDateString(locale, {
        month: 'short',
        day: 'numeric',
        year: date.getFullYear() !== today.getFullYear() ? 'numeric' : undefined
      })
    }

    const getStatusClass = (dueDate, status) => {
      if (status === 'completed') return 'completed'

      const today = new Date()
      today.setHours(0, 0, 0, 0)
      const due = new Date(dueDate)
      due.setHours(0, 0, 0, 0)

      const diffTime = due - today
      const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))

      if (diffDays < 0) return 'overdue'
      if (diffDays <= 1) return 'urgent'
      return 'upcoming'
    }

    const getStatusText = (dueDate, status) => {
      const isJapanese = currentLocale.value === 'ja'

      if (status === 'completed') return isJapanese ? '完了' : 'Completed'

      const statusClass = getStatusClass(dueDate, status)
      if (statusClass === 'overdue') return isJapanese ? '期限超過' : 'Overdue'
      if (statusClass === 'urgent') return isJapanese ? 'もうすぐ期限' : 'Due Soon'
      return isJapanese ? '予定' : 'Upcoming'
    }

    const translatePriority = (priority) => {
      const priorityMap = {
        'high': t('priority.high'),
        'medium': t('priority.medium'),
        'low': t('priority.low')
      }
      return priorityMap[priority] || priority
    }

    return {
      t,
      newTask,
      sortedTasks,
      close,
      handleAddTask,
      formatDueDate,
      getStatusClass,
      getStatusText,
      translatePriority
    }
  }
}
</script>
