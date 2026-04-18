<template>
  <div class="relative">
    <button
      class="flex items-center gap-2 cursor-pointer hover:bg-slate-100 px-3 py-2 rounded-lg transition-colors border-none bg-transparent font-[inherit]"
      @click="toggleDropdown"
      @blur="handleBlur"
    >
      <div class="w-8 h-8 rounded-full bg-gradient-to-br from-blue-400 to-blue-500 flex items-center justify-center text-white text-xs font-bold">
        {{ getInitials(currentUser.name) }}
      </div>
      <span class="text-sm font-medium text-slate-900">{{ currentUser.name }}</span>
      <svg
        class="text-slate-500 transition-transform duration-200"
        :class="{ 'rotate-180': isDropdownOpen }"
        width="16"
        height="16"
        viewBox="0 0 16 16"
        fill="none"
      >
        <path d="M4 6L8 10L12 6" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
      </svg>
    </button>

    <div v-if="isDropdownOpen" class="absolute right-0 top-full mt-2 w-64 bg-white rounded-xl shadow-lg border border-slate-200 overflow-hidden z-[1000]">

      <div class="flex items-center gap-3 px-4 py-4 bg-blue-50">
        <div class="w-12 h-12 rounded-full bg-gradient-to-br from-blue-400 to-blue-500 flex items-center justify-center text-white text-base font-bold shrink-0">
          {{ getInitials(currentUser.name) }}
        </div>
        <div class="flex-1 min-w-0">
          <div class="font-semibold text-slate-900 text-sm mb-0.5">{{ currentUser.name }}</div>
          <div class="text-xs text-slate-500 truncate">{{ currentUser.email }}</div>
        </div>
      </div>

      <div class="h-px bg-slate-200 my-1"></div>

      <button
        class="flex items-center gap-3 px-4 py-3 text-sm text-slate-700 hover:bg-blue-50 transition-colors cursor-pointer w-full border-none bg-transparent text-left font-[inherit]"
        @mousedown.prevent="showProfileDetails"
      >
        <svg class="text-slate-500 shrink-0" width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M9 9C10.6569 9 12 7.65685 12 6C12 4.34315 10.6569 3 9 3C7.34315 3 6 4.34315 6 6C6 7.65685 7.34315 9 9 9Z" stroke="currentColor" stroke-width="1.5"/>
          <path d="M15 15C15 12.7909 12.3137 11 9 11C5.68629 11 3 12.7909 3 15" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
        {{ t('profile.profileDetails') }}
      </button>

      <button
        class="flex items-center gap-3 px-4 py-3 text-sm text-slate-700 hover:bg-blue-50 transition-colors cursor-pointer w-full border-none bg-transparent text-left font-[inherit]"
        @mousedown.prevent="showTasks"
      >
        <svg class="text-slate-500 shrink-0" width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M15 3H3C2.44772 3 2 3.44772 2 4V14C2 14.5523 2.44772 15 3 15H15C15.5523 15 16 14.5523 16 14V4C16 3.44772 15.5523 3 15 3Z" stroke="currentColor" stroke-width="1.5"/>
          <path d="M6 7L8 9L12 5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        {{ t('profile.myTasks') }}
        <span v-if="pendingTaskCount > 0" class="ml-auto bg-blue-400 text-white text-xs font-bold px-1.5 py-0.5 rounded-full">{{ pendingTaskCount }}</span>
      </button>

      <div class="h-px bg-slate-200 my-1"></div>

      <button
        class="flex items-center gap-3 px-4 py-3 text-sm text-rose-600 hover:bg-rose-100 transition-colors cursor-pointer w-full border-none bg-transparent text-left font-[inherit]"
        @mousedown.prevent="handleLogout"
      >
        <svg class="shrink-0" width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M7 15H4C3.44772 15 3 14.5523 3 14V4C3 3.44772 3.44772 3 4 3H7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          <path d="M11 12L15 9M15 9L11 6M15 9H7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        {{ t('profile.logout') }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useAuth } from '../composables/useAuth'
import { useI18n } from '../composables/useI18n'

const { currentUser, logout, getInitials } = useAuth()
const { t } = useI18n()

const isDropdownOpen = ref(false)
const emit = defineEmits(['show-profile-details', 'show-tasks'])

const pendingTaskCount = computed(() => {
  return currentUser.value.tasks.filter(task => task.status === 'pending').length
})

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value
}

const handleBlur = () => {
  // Delay to allow mousedown events on dropdown items to fire first
  setTimeout(() => {
    isDropdownOpen.value = false
  }, 200)
}

const showProfileDetails = () => {
  isDropdownOpen.value = false
  emit('show-profile-details')
}

const showTasks = () => {
  isDropdownOpen.value = false
  emit('show-tasks')
}

const handleLogout = () => {
  isDropdownOpen.value = false
  logout()
}
</script>
