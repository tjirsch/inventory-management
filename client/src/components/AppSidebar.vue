<template>
  <div class="sidebar-root">
    <!-- Backdrop only exists in drawer mode (< 768px) -->
    <div
      v-if="isMobile && mobileOpen"
      class="sidebar-backdrop"
      aria-hidden="true"
      @click="$emit('close')"
    ></div>

    <aside
      id="app-sidebar"
      class="sidebar"
      :class="{ 'is-collapsed': isCollapsed, 'is-mobile': isMobile, 'is-open': mobileOpen }"
    >
      <div class="sidebar-brand">
        <span class="brand-mark" aria-hidden="true">{{ t('nav.companyName').charAt(0) }}</span>
        <div v-if="!isCollapsed" class="brand-text">
          <span class="brand-name">{{ t('nav.companyName') }}</span>
          <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
        </div>
      </div>

      <nav class="sidebar-nav" aria-label="Main">
        <div v-for="group in groups" :key="group.section" class="nav-group">
          <div v-if="!isCollapsed" class="sidebar-section-label">{{ t(group.labelKey) }}</div>
          <div v-else class="sidebar-section-divider" aria-hidden="true"></div>
          <ul class="nav-list">
            <li v-for="item in group.items" :key="item.to">
              <router-link
                :to="item.to"
                class="nav-item"
                :active-class="item.exact ? 'is-parent' : 'is-active'"
                exact-active-class="is-active"
                :title="isCollapsed ? t(item.labelKey) : null"
              >
                <svg
                  class="nav-icon"
                  xmlns="http://www.w3.org/2000/svg"
                  viewBox="0 0 24 24"
                  width="18"
                  height="18"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="1.75"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  aria-hidden="true"
                >
                  <path v-for="d in item.icon" :key="d" :d="d" />
                </svg>
                <span v-if="!isCollapsed" class="nav-label">{{ t(item.labelKey) }}</span>
                <span v-else class="sr-only">{{ t(item.labelKey) }}</span>
              </router-link>
            </li>
          </ul>
        </div>
      </nav>

      <div class="sidebar-footer">
        <button
          v-if="!isMobile"
          type="button"
          class="collapse-toggle"
          :aria-expanded="String(!isCollapsed)"
          :aria-label="isCollapsed ? t('nav.expandSidebar') : t('nav.collapseSidebar')"
          :title="isCollapsed ? t('nav.expandSidebar') : t('nav.collapseSidebar')"
          @click="toggleCollapsed"
        >
          <svg
            class="collapse-icon"
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 24 24"
            width="18"
            height="18"
            fill="none"
            stroke="currentColor"
            stroke-width="1.75"
            stroke-linecap="round"
            stroke-linejoin="round"
            aria-hidden="true"
          >
            <path d="m11 17-5-5 5-5" />
            <path d="m18 17-5-5 5-5" />
          </svg>
          <span v-if="!isCollapsed" class="nav-label">{{ t('nav.collapseSidebar') }}</span>
        </button>
        <button
          v-else
          type="button"
          class="collapse-toggle"
          :aria-label="t('nav.closeMenu')"
          @click="$emit('close')"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 24 24"
            width="18"
            height="18"
            fill="none"
            stroke="currentColor"
            stroke-width="1.75"
            stroke-linecap="round"
            stroke-linejoin="round"
            aria-hidden="true"
          >
            <path d="M18 6 6 18" />
            <path d="m6 6 12 12" />
          </svg>
          <span class="nav-label">{{ t('nav.closeMenu') }}</span>
        </button>
      </div>
    </aside>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted, onBeforeUnmount } from 'vue'
import { useRoute } from 'vue-router'
import { useI18n } from '../composables/useI18n'

const STORAGE_KEY = 'sidebar-collapsed'

// Icon geometry: Lucide-style outline paths (24x24 viewBox)
const NAV_ITEMS = [
  {
    to: '/',
    labelKey: 'nav.overview',
    section: 'operations',
    exact: true,
    icon: ['M3 3h7v9H3z', 'M14 3h7v5h-7z', 'M14 12h7v9h-7z', 'M3 16h7v5H3z']
  },
  {
    to: '/inventory',
    labelKey: 'nav.inventory',
    section: 'operations',
    icon: [
      'M16.5 9.4 7.5 4.21',
      'M21 8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z',
      'M3.27 6.96 12 12.01l8.73-5.05',
      'M12 22.08V12'
    ]
  },
  {
    to: '/orders',
    labelKey: 'nav.orders',
    section: 'operations',
    icon: [
      'M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2',
      'M9 2h6v4H9z',
      'M12 11h4',
      'M12 16h4',
      'M8 11h.01',
      'M8 16h.01'
    ]
  },
  {
    to: '/demand',
    labelKey: 'nav.demandForecast',
    section: 'operations',
    icon: ['M22 7 13.5 15.5 8.5 10.5 2 17', 'M16 7h6v6']
  },
  {
    to: '/spending',
    labelKey: 'nav.finance',
    section: 'finance',
    icon: [
      'M12 21a9 9 0 1 0 0-18 9 9 0 0 0 0 18z',
      'M15 9.5a2.5 2 0 0 0-3-1.5 2 2 0 0 0 0 4 2 2 0 0 1 0 4 2.5 2 0 0 1-3-1.5',
      'M12 6v2',
      'M12 16v2'
    ]
  },
  {
    to: '/reports',
    labelKey: 'nav.reports',
    section: 'finance',
    icon: [
      'M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z',
      'M14 2v6h6',
      'M8 13h8',
      'M8 17h8'
    ]
  }
]

const SECTIONS = [
  { section: 'operations', labelKey: 'nav.sectionOperations' },
  { section: 'finance', labelKey: 'nav.sectionFinance' }
]

export default {
  name: 'AppSidebar',
  props: {
    mobileOpen: { type: Boolean, default: false }
  },
  emits: ['close'],
  setup(props, { emit }) {
    const { t } = useI18n()
    const route = useRoute()

    const groups = SECTIONS.map(s => ({
      ...s,
      items: NAV_ITEMS.filter(item => item.section === s.section)
    }))

    // Breakpoints: < 768 drawer, 768-1023 collapsed default, >= 1024 expanded default
    const mobileQuery = window.matchMedia('(max-width: 767px)')
    const tabletQuery = window.matchMedia('(max-width: 1023px)')
    const isMobile = ref(mobileQuery.matches)
    const isTablet = ref(tabletQuery.matches)

    // null = user has not chosen; fall back to viewport default
    const userPref = ref(null)
    try {
      const stored = localStorage.getItem(STORAGE_KEY)
      if (stored === 'true' || stored === 'false') userPref.value = stored === 'true'
    } catch (e) {
      // localStorage unavailable; use viewport default
    }

    // Drawer always shows the full labelled sidebar
    const isCollapsed = computed(() => {
      if (isMobile.value) return false
      return userPref.value === null ? isTablet.value : userPref.value
    })

    const toggleCollapsed = () => {
      userPref.value = !isCollapsed.value
      try {
        localStorage.setItem(STORAGE_KEY, String(userPref.value))
      } catch (e) {
        // ignore persistence failure
      }
    }

    const onMobileChange = (e) => { isMobile.value = e.matches }
    const onTabletChange = (e) => { isTablet.value = e.matches }
    const onKeydown = (e) => {
      if (e.key === 'Escape' && isMobile.value && props.mobileOpen) emit('close')
    }

    onMounted(() => {
      mobileQuery.addEventListener('change', onMobileChange)
      tabletQuery.addEventListener('change', onTabletChange)
      window.addEventListener('keydown', onKeydown)
    })
    onBeforeUnmount(() => {
      mobileQuery.removeEventListener('change', onMobileChange)
      tabletQuery.removeEventListener('change', onTabletChange)
      window.removeEventListener('keydown', onKeydown)
    })

    // Close the drawer on navigation
    watch(() => route.path, () => {
      if (props.mobileOpen) emit('close')
    })

    // Leaving mobile width with the drawer open must reset parent state
    watch(isMobile, (mobile) => {
      if (!mobile && props.mobileOpen) emit('close')
    })

    return { t, groups, isMobile, isCollapsed, toggleCollapsed }
  }
}
</script>

<style scoped>
/* The wrapper must not become its own grid item in .app-shell: in drawer mode its
   children are position:fixed, so it would render as an empty auto row that pushes
   .app-main down on short pages. display:contents makes .sidebar the grid item,
   which is also what lets sticky + align-self below take effect on desktop. */
.sidebar-root {
  display: contents;
}

.sidebar {
  position: sticky;
  top: 0;
  align-self: start;
  height: 100vh;
  width: var(--sidebar-width);
  display: flex;
  flex-direction: column;
  background: var(--color-surface);
  border-right: 1px solid var(--color-border);
  overflow-y: auto;
  overflow-x: hidden;
  transition: width 0.2s ease;
}

.sidebar.is-collapsed {
  width: var(--sidebar-width-collapsed);
}

/* Drawer mode. z-index layering: topbar 20 < backdrop 40 < drawer 50 < modals */
.sidebar.is-mobile {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 50;
  width: var(--sidebar-width);
  box-shadow: var(--shadow-md);
  transform: translateX(-100%);
  visibility: hidden;
  transition: transform 0.2s ease, visibility 0.2s;
}

.sidebar.is-mobile.is-open {
  transform: translateX(0);
  visibility: visible;
}

.sidebar-backdrop {
  position: fixed;
  inset: 0;
  z-index: 40;
  background: var(--color-overlay);
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  height: var(--topbar-height);
  flex-shrink: 0;
  padding: 0 var(--space-4);
  border-bottom: 1px solid var(--color-border);
}

.sidebar.is-collapsed .sidebar-brand {
  justify-content: center;
  padding: 0;
}

.brand-mark {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: var(--space-8);
  height: var(--space-8);
  border-radius: var(--radius-md);
  background: var(--color-primary);
  color: var(--color-surface);
  font-size: var(--text-lg);
  font-weight: 600;
}

.brand-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.brand-name {
  font-size: var(--text-base);
  font-weight: 600;
  color: var(--color-text);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.brand-subtitle {
  font-size: var(--text-xs);
  color: var(--color-text-muted);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.sidebar-nav {
  flex: 1;
  padding: var(--space-2) var(--space-3);
}

.sidebar.is-collapsed .sidebar-nav {
  padding: var(--space-2);
}

.sidebar-section-label {
  margin: var(--space-6) 0 var(--space-2);
  padding: 0 var(--space-3);
  font-size: var(--text-xs);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--color-text-subtle);
}

.nav-group:first-child .sidebar-section-label {
  margin-top: var(--space-4);
}

.sidebar-section-divider {
  margin: var(--space-3) var(--space-2);
  border-top: 1px solid var(--color-border);
}

.nav-group:first-child .sidebar-section-divider {
  display: none;
}

.nav-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
}

.nav-item,
.collapse-toggle {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  width: 100%;
  height: var(--space-10);
  padding: var(--space-2) var(--space-3);
  border: none;
  border-radius: var(--radius-md);
  background: transparent;
  color: var(--color-text-muted);
  font-family: inherit;
  font-size: var(--text-sm);
  font-weight: 500;
  text-decoration: none;
  white-space: nowrap;
  cursor: pointer;
  transition: background-color 0.15s ease, color 0.15s ease;
}

.sidebar.is-collapsed .nav-item,
.sidebar.is-collapsed .collapse-toggle {
  justify-content: center;
  padding: var(--space-2);
}

.nav-item:hover,
.collapse-toggle:hover {
  background: var(--color-surface-muted);
  color: var(--color-text);
}

.nav-item.is-active,
.nav-item.is-active:hover {
  background: var(--color-primary-soft);
  color: var(--color-primary);
}

.nav-item:focus-visible,
.collapse-toggle:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.nav-icon,
.collapse-icon {
  flex-shrink: 0;
}

.sidebar.is-collapsed .collapse-icon {
  transform: rotate(180deg);
}

.sidebar-footer {
  flex-shrink: 0;
  padding: var(--space-3);
  border-top: 1px solid var(--color-border);
}

.sidebar.is-collapsed .sidebar-footer {
  padding: var(--space-2);
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
}
</style>
