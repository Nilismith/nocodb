<script lang="ts" setup>
interface Props {
  record: Record<string, string>
  color?: string
  resize?: boolean
  selected?: boolean
  hover?: boolean
  position?: 'topRounded' | 'bottomRounded' | 'rounded' | 'none'
}

withDefaults(defineProps<Props>(), {
  resize: true,
  selected: false,
  hover: false,
  color: 'blue',
  position: 'rounded',
})

const emit = defineEmits(['resize-start'])
</script>

<template>
  <div
    v-if="(position === 'topRounded' || position === 'rounded') && resize"
    class="absolute flex items-center justify-center w-full -mt-4 h-7.1 hidden z-1 resize !group-hover:(flex rounded-lg)"
  >
    <NcButton
      :class="{
        '!flex border-1 rounded-lg border-brand-500': selected || hover,
      }"
      class="!group-hover:(border-brand-500) !border-1 cursor-ns-resize"
      size="xsmall"
      type="secondary"
      @mousedown.stop="emit('resize-start', 'left', $event, record)"
    >
      <component :is="iconMap.drag" class="text-gray-400"></component>
    </NcButton>
  </div>
  <div
    :class="{
      'rounded-t-lg': position === 'topRounded',
      'rounded-b-lg': position === 'bottomRounded',
      'rounded-lg': position === 'rounded',
      'rounded-none': position === 'none',
      'bg-maroon-50': color === 'maroon',
      'bg-blue-50': color === 'blue',
      'bg-green-50': color === 'green',
      'bg-yellow-50': color === 'yellow',
      'bg-pink-50': color === 'pink',
      'bg-purple-50': color === 'purple',
      'group-hover:(border-brand-500)': resize,
      '!border-brand-500 border-1': selected || hover,
    }"
    class="relative h-full ml-0.25 border-1 border-gray-50"
  >
    <div class="h-full absolute py-2">
      <div
        :class="{
          'bg-maroon-500': color === 'maroon',
          'bg-blue-500': color === 'blue',
          'bg-green-500': color === 'green',
          'bg-yellow-500': color === 'yellow',
          'bg-pink-500': color === 'pink',
          'bg-purple-500': color === 'purple',
        }"
        class="block h-full min-h-5 ml-1 w-1 rounded mr-2"
      ></div>
    </div>

    <div v-if="position === 'bottomRounded' || position === 'none'" class="ml-3">....</div>

    <div class="ml-3 pr-3 text-ellipsis overflow-hidden w-full h-8 absolute">
      <span class="text-sm text-gray-800">
        <slot />
      </span>
    </div>
    <div v-if="position === 'topRounded' || position === 'none'" class="h-full pb-7 flex items-end ml-3">....</div>
  </div>
  <div
    v-if="(position === 'bottomRounded' || position === 'rounded') && resize"
    class="absolute items-center justify-center w-full hidden h-7.1 -mt-4 !group-hover:(flex rounded-lg)"
  >
    <NcButton
      :class="{
        '!flex border-1 rounded-lg z-1 cursor-ns-resize	border-brand-500': selected || hover,
      }"
      class="!group-hover:(border-brand-500) !border-1"
      size="xsmall"
      type="secondary"
      @mousedown.stop="emit('resize-start', 'right', $event, record)"
    >
      <component :is="iconMap.drag" class="text-gray-400"></component>
    </NcButton>
  </div>
</template>

<style lang="scss" scoped></style>
