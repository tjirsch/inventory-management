<template>
  <div class="app-shell">
    <AppSidebar :mobile-open="mobileNavOpen" @close="mobileNavOpen = false" />

    <!-- min-width: 0 on .app-main lets wide tables scroll instead of stretching the grid -->
    <div class="app-main">
      <header class="topbar">
        <div class="topbar-left">
          <button
            type="button"
            class="menu-button"
            :aria-label="t('nav.openMenu')"
            aria-controls="app-sidebar"
            :aria-expanded="String(mobileNavOpen)"
            @click="mobileNavOpen = true"
          >
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <path d="M4 6h16" />
              <path d="M4 12h16" />
              <path d="M4 18h16" />
            </svg>
          </button>
          <span class="topbar-title">{{ pageTitle }}</span>
        </div>
        <div class="topbar-right">
          <LanguageSwitcher />
          <ProfileMenu
            @show-profile-details="showProfileDetails = true"
            @show-tasks="showTasks = true"
          />
        </div>
      </header>
      <FilterBar />
      <main class="content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import AppSidebar from './components/AppSidebar.vue'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    AppSidebar,
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])
    const mobileNavOpen = ref(false)

    // Topbar title derived from the current route, using the nav i18n keys
    const route = useRoute()
    const TITLE_KEYS = {
      '/': 'nav.overview',
      '/inventory': 'nav.inventory',
      '/orders': 'nav.orders',
      '/demand': 'nav.demandForecast',
      '/spending': 'nav.finance',
      '/reports': 'nav.reports'
    }
    const pageTitle = computed(() => {
      const key = TITLE_KEYS[route.path]
      return key ? t(key) : ''
    })

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      mobileNavOpen,
      pageTitle,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
:root {
  /* Surfaces */
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-surface-muted: #f1f5f9;
  --color-border: #e2e8f0;
  --color-border-strong: #cbd5e1;

  /* Text */
  --color-text: #0f172a;
  --color-text-muted: #64748b;
  --color-text-subtle: #94a3b8;

  /* Brand / interactive */
  --color-primary: #2563eb;
  --color-primary-hover: #1d4ed8;
  --color-primary-soft: #eff6ff;
  --color-focus-ring: rgb(37 99 235 / 0.2);

  /* Status (fill + soft background + darker text for legible badges) */
  --color-success: #16a34a;
  --color-success-soft: #f0fdf4;
  --color-success-text: #15803d;
  --color-info: #2563eb;
  --color-info-soft: #eff6ff;
  --color-info-text: #1d4ed8;
  --color-warning: #d97706;
  --color-warning-soft: #fffbeb;
  --color-warning-text: #b45309;
  --color-danger: #dc2626;
  --color-danger-soft: #fef2f2;
  --color-danger-text: #b91c1c;

  /* Overlay */
  --color-overlay: rgb(15 23 42 / 0.4);

  /* Spacing - 4px scale; use ONLY these */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
  --space-10: 40px;
  --space-12: 48px;

  /* Shape & depth */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-full: 9999px;
  --shadow-sm: 0 1px 2px rgb(15 23 42 / 0.05);
  --shadow-md: 0 4px 12px rgb(15 23 42 / 0.08);

  /* Type */
  --font-sans: Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  --text-xs: 12px;
  --text-sm: 13px;
  --text-base: 14px;
  --text-lg: 16px;
  --text-xl: 20px;
  --text-2xl: 24px;

  /* Layout */
  --sidebar-width: 240px;
  --sidebar-width-collapsed: 64px;
  --topbar-height: 56px;
  --content-max-width: 1400px;
  --control-height: 36px; /* buttons, inputs, selects */
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: var(--font-sans);
  font-size: var(--text-base);
  background: var(--color-bg);
  color: var(--color-text);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ---------- App shell ---------- */
.app-shell {
  display: grid;
  grid-template-columns: auto minmax(0, 1fr);
  min-height: 100vh;
  background: var(--color-bg);
}

.app-main {
  min-width: 0;
  display: flex;
  flex-direction: column;
}

/* z-index layering: filter bar 10 < topbar 20 (its dropdowns paint above the filter bar) < drawer backdrop 40 < drawer 50 < modals */
.topbar {
  position: sticky;
  top: 0;
  z-index: 20;
  height: var(--topbar-height);
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-4);
  padding: 0 var(--space-8);
  background: var(--color-surface);
  border-bottom: 1px solid var(--color-border);
}

.topbar-left,
.topbar-right {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  min-width: 0;
}

.topbar-right {
  gap: var(--space-4);
}

.topbar-title {
  font-size: var(--text-lg);
  font-weight: 600;
  color: var(--color-text);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.menu-button {
  display: none;
  align-items: center;
  justify-content: center;
  width: var(--space-10);
  height: var(--space-10);
  flex-shrink: 0;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  background: var(--color-surface);
  color: var(--color-text-muted);
  cursor: pointer;
}

.menu-button:hover {
  background: var(--color-surface-muted);
  color: var(--color-text);
}

.menu-button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.content {
  flex: 1;
  width: 100%;
  max-width: var(--content-max-width);
  margin: 0 auto;
  padding: var(--space-8);
}

/* Breakpoint 768px matches the sidebar drawer switch in AppSidebar.vue */
@media (max-width: 767px) {
  .app-shell {
    grid-template-columns: minmax(0, 1fr);
  }

  .menu-button {
    display: inline-flex;
  }

  .topbar {
    padding: 0 var(--space-4);
    gap: var(--space-2);
  }

  .topbar-right {
    flex-shrink: 0;
    gap: var(--space-2);
  }

  .content {
    padding: var(--space-4);
  }
}

/* Language switcher + profile menu need ~280px; drop the title on very narrow phones */
@media (max-width: 479px) {
  .topbar-title {
    display: none;
  }
}

/* ---------- Page header ---------- */
.page-header {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto;
  align-items: end;
  column-gap: var(--space-4);
  margin-bottom: var(--space-6);
}

.page-header > * {
  grid-column: 1;
}

.page-header h2 {
  font-size: var(--text-2xl);
  font-weight: 600;
  color: var(--color-text);
  margin-bottom: var(--space-1);
  letter-spacing: -0.02em;
}

.page-header p {
  color: var(--color-text-muted);
  font-size: var(--text-sm);
}

/* Optional right-aligned actions on the title row */
.page-header > .page-header-actions {
  grid-column: 2;
  grid-row: 1 / span 2;
  display: flex;
  align-items: center;
  gap: var(--space-2);
}

/* ---------- Grids and stat cards ---------- */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: var(--space-6);
  margin-bottom: var(--space-6);
}

.stat-card {
  background: var(--color-surface);
  padding: var(--space-5) var(--space-6);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-sm);
  transition: box-shadow 0.15s ease, border-color 0.15s ease;
}

.stat-card:hover {
  border-color: var(--color-border-strong);
  box-shadow: var(--shadow-md);
}

.stat-label {
  color: var(--color-text-muted);
  font-size: var(--text-xs);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: var(--space-2);
}

.stat-value {
  font-size: var(--text-2xl);
  font-weight: 600;
  color: var(--color-text);
  letter-spacing: -0.02em;
  font-variant-numeric: tabular-nums;
}

.stat-card.warning .stat-value {
  color: var(--color-warning);
}

.stat-card.success .stat-value {
  color: var(--color-success);
}

.stat-card.danger .stat-value {
  color: var(--color-danger);
}

.stat-card.info .stat-value {
  color: var(--color-info);
}

/* ---------- Cards ---------- */
.card {
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  padding: var(--space-6);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-sm);
  margin-bottom: var(--space-6);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: var(--space-4);
  margin-bottom: var(--space-4);
  padding-bottom: var(--space-4);
  border-bottom: 1px solid var(--color-border);
}

.card-title {
  font-size: var(--text-lg);
  font-weight: 600;
  color: var(--color-text);
  letter-spacing: -0.01em;
}

/* ---------- Tables ---------- */
.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--color-surface-muted);
}

th {
  text-align: left;
  padding: var(--space-3) var(--space-4);
  font-weight: 600;
  color: var(--color-text-muted);
  font-size: var(--text-xs);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  white-space: nowrap;
}

td {
  padding: var(--space-3) var(--space-4);
  border-bottom: 1px solid var(--color-border);
  color: var(--color-text);
  font-size: var(--text-sm);
  font-variant-numeric: tabular-nums;
}

tbody tr:last-child td {
  border-bottom: none;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: var(--color-surface-muted);
}

/* ---------- Badges ---------- */
.badge {
  display: inline-block;
  padding: 2px var(--space-2);
  border-radius: var(--radius-full);
  font-size: var(--text-xs);
  font-weight: 500;
  line-height: 1.5;
  white-space: nowrap;
  background: var(--color-surface-muted);
  color: var(--color-text-muted);
}

.badge.success,
.badge.increasing {
  background: var(--color-success-soft);
  color: var(--color-success-text);
}

.badge.warning,
.badge.medium {
  background: var(--color-warning-soft);
  color: var(--color-warning-text);
}

.badge.danger,
.badge.decreasing,
.badge.high {
  background: var(--color-danger-soft);
  color: var(--color-danger-text);
}

.badge.info,
.badge.low {
  background: var(--color-info-soft);
  color: var(--color-info-text);
}

.badge.stable {
  background: var(--color-surface-muted);
  color: var(--color-text-muted);
}

/* ---------- Buttons ---------- */
.btn,
.btn-primary,
.btn-secondary {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
  height: var(--control-height);
  padding: 0 var(--space-4);
  border-radius: var(--radius-md);
  border: 1px solid var(--color-border);
  background: var(--color-surface);
  color: var(--color-text);
  font-family: inherit;
  font-size: var(--text-sm);
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.15s ease, border-color 0.15s ease;
}

.btn:hover,
.btn-secondary:hover {
  background: var(--color-surface-muted);
}

.btn-primary {
  background: var(--color-primary);
  border-color: var(--color-primary);
  color: var(--color-surface);
}

.btn-primary:hover {
  background: var(--color-primary-hover);
  border-color: var(--color-primary-hover);
}

.btn:disabled,
.btn-primary:disabled,
.btn-secondary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn:focus-visible,
.btn-primary:focus-visible,
.btn-secondary:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

/* ---------- States ---------- */
.loading {
  text-align: center;
  padding: var(--space-12);
  color: var(--color-text-muted);
  font-size: var(--text-base);
}

.error {
  background: var(--color-danger-soft);
  border: 1px solid var(--color-danger);
  color: var(--color-danger-text);
  padding: var(--space-4);
  border-radius: var(--radius-md);
  margin: var(--space-4) 0;
  font-size: var(--text-base);
}
</style>
