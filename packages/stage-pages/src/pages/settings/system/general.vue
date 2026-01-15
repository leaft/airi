<script setup lang="ts">
import { all } from '@proj-airi/i18n'
import { useSettings } from '@proj-airi/stage-ui/stores/settings'
import { FieldCheckbox, FieldSelect, useTheme } from '@proj-airi/ui'
import { computed, onMounted, ref, watchEffect } from 'vue'
import { useI18n } from 'vue-i18n'

const props = withDefaults(defineProps<{
  needscontrolsIslandIconSizeSetting?: boolean
}>(), {
  needscontrolsIslandIconSizeSetting: import.meta.env.RUNTIME_ENVIRONMENT === 'electron',
})

const showControlsIsland = computed(() => props.needscontrolsIslandIconSizeSetting)

const settings = useSettings()

const { t } = useI18n()
const { isDark: dark } = useTheme()

const languages = computed(() => {
  return Object.entries(all).map(([value, label]) => ({ value, label }))
})

const showTamagotchiDisplay = computed(() => import.meta.env.RUNTIME_ENVIRONMENT === 'electron')
const displayOptions = ref<Array<{ value: string, label: string }>>([
  { value: 'primary', label: t('settings.tamagotchi.display.primary') },
])

onMounted(async () => {
  try {
    if (!showTamagotchiDisplay.value)
      return

    // dynamic import of the tamagotchi renderer composable that wraps electron.screen
    // Use relative path since @proj-airi/stage-tamagotchi alias is not configured in vite/electron-vite config
    const mod = await import('../../../../../../apps/stage-tamagotchi/src/renderer/composables/electron-vueuse')
    // composable returns a ref of displays
    const allDisplays = mod.useElectronAllDisplays()

    watchEffect(() => {
      const displays = allDisplays.value || []
      displayOptions.value = displays.map(d => ({
        value: String(d.id ?? `${d.bounds?.x}_${d.bounds?.y}`),
        label: d.id != null ? `Display ${d.id} — ${d.size?.width}×${d.size?.height}` : `${d.size?.width}×${d.size?.height}`,
      }))
      if (!displayOptions.value.length) {
        displayOptions.value = [{ value: 'primary', label: t('settings.tamagotchi.display.primary') }]
      }
    })
  }
  catch (e) {
  // silently ignore on non-electron runtimes
    console.error(e)
  }
})
</script>

<template>
  <div rounded-lg bg-neutral-50 p-4 dark:bg-neutral-800 flex="~ col gap-4">
    <FieldCheckbox
      v-model="dark"
      v-motion
      mb-2
      :initial="{ opacity: 0, y: 10 }"
      :enter="{ opacity: 1, y: 0 }"
      :duration="250 + (2 * 10)"
      :delay="2 * 50"
      :label="t('settings.theme.title')"
      :description="t('settings.theme.description')"
    />

    <!-- Language Setting -->
    <FieldSelect
      v-model="settings.language"
      v-motion
      :initial="{ opacity: 0, y: 10 }"
      :enter="{ opacity: 1, y: 0 }"
      :duration="250 + (3 * 10)"
      :delay="3 * 50"
      transition="all ease-in-out duration-250"
      :label="t('settings.language.title')"
      :description="t('settings.language.description')"
      :options="languages as Array<{ value: string; label: string }>"
    />

    <!-- Controls Island Icon Size -->
    <FieldSelect
      v-if="showControlsIsland"
      v-model="settings.controlsIslandIconSize"
      v-motion
      :initial="{ opacity: 0, y: 10 }"
      :enter="{ opacity: 1, y: 0 }"
      :duration="250 + (4 * 10)"
      :delay="4 * 50"
      transition="all ease-in-out duration-250"
      :label="t('settings.controls-island.icon-size.title')"
      :description="t('settings.controls-island.icon-size.description')"
      :options="[
        { value: 'auto', label: t('settings.controls-island.icon-size.auto') },
        { value: 'large', label: t('settings.controls-island.icon-size.large') },
        { value: 'small', label: t('settings.controls-island.icon-size.small') },
      ]"
    />

    <!-- Tamagotchi Display (Electron only) -->
    <FieldSelect
      v-if="showTamagotchiDisplay"
      v-model="settings.tamagotchiDisplay"
      v-motion
      :initial="{ opacity: 0, y: 10 }"
      :enter="{ opacity: 1, y: 0 }"
      :duration="250 + (5 * 10)"
      :delay="5 * 50"
      transition="all ease-in-out duration-250"
      :label="t('settings.tamagotchi.display.title')"
      :description="t('settings.tamagotchi.display.description')"
      :options="displayOptions"
    />

    <div
      v-motion
      text="neutral-200/50 dark:neutral-600/20" pointer-events-none
      fixed top="[65dvh]" right--15 z--1
      :initial="{ scale: 0.9, opacity: 0, rotate: 30 }"
      :enter="{ scale: 1, opacity: 1, rotate: 0 }"
      :duration="250"
      flex items-center justify-center
    >
      <div text="60" i-solar:emoji-funny-square-bold-duotone />
    </div>
  </div>
</template>

<route lang="yaml">
meta:
  layout: settings
  stageTransition:
    name: slide
</route>
