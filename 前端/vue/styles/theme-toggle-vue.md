---
created: 2026-07-10
updated: "created: 2026-07-10"
---
## CSS
```css
/* Color Mode transition */
::view-transition-old(root),
::view-transition-new(root) {
  animation: none;
  mix-blend-mode: normal;
}

::view-transition-old(root) {
  z-index: 2147483646;
}

::view-transition-new(root) {
  z-index: 1;
}

.dark::view-transition-old(root) {
  z-index: 1;
}

.dark::view-transition-new(root) {
  z-index: 2147483646;
}

```

## Store useThemeStore
```ts
export type Theme = 'light' | 'dark'

export interface ThemeState {
  theme: Theme
}

export const STORAGE_KEY = import.meta.env.VITE_STORAGE_KEY

function getInitialTheme(): Theme {
  const stored = localStorage.getItem(STORAGE_KEY)
  if (stored === 'light' || stored === 'dark')
    return stored
  return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'
}

function applyTheme(t: Theme) {
  const html = document.documentElement
  html.setAttribute('data-theme', t)
  html.classList.toggle('dark', t === 'dark')
}

const isAppearanceTransition
  = typeof document !== 'undefined'
    // @ts-expect-error: Transition API
    && document.startViewTransition
    && !window.matchMedia('(prefers-reduced-motion: reduce)').matches

export const useThemeStore = defineStore('theme', {
  state: (): ThemeState => ({
    theme: getInitialTheme(),
  }),
  actions: {
    toggleTheme(event: MouseEvent) {
      const isDark = this.theme === 'dark'

      if (!isAppearanceTransition) {
        this.theme = isDark ? 'light' : 'dark'
        return
      }

      const x = event.clientX
      const y = event.clientY
      // 返回其参数平方和的平方根。
      const endRadius = Math.hypot(
        Math.max(x, window.innerWidth - x),
        Math.max(y, window.innerHeight - y),
      )

      // startViewTransition 开始新的单页（SPA）视图过渡并返回一个表示它的 transition 对象
      const transition = document.startViewTransition(() => {
        const newTheme = isDark ? 'light' : 'dark'
        this.theme = newTheme
        applyTheme(newTheme)
      })

      transition.ready.then(() => {
        const clipPath = [
          `circle(0px at ${x}px ${y}px)`,
          `circle(${endRadius}px at ${x}px ${y}px)`,
        ]

        document.documentElement.animate(
          {
            clipPath: isDark ? clipPath.toReversed() : clipPath,
          },
          {
            duration: 10000,
            easing: 'ease-in',
            pseudoElement: isDark
              ? '::view-transition-old(root)'
              : '::view-transition-new(root)',
          },
        )
      })
    },
  },
  persist: {
    key: STORAGE_KEY,
  },
})

```

## Hooks useTheme
```ts
import { useThemeStore } from '@/stores/theme'

export function useTheme() {
  const themeStore = useThemeStore()

  return {
    theme: computed(() => themeStore.theme),
    isDark: computed(() => themeStore.theme === 'dark'),
    toggleTheme: themeStore.toggleTheme,
  }
}

```

