<template lang="pug">
.SelectField(
  ref="rootEl"
  :data-inline="props.inline"
  :data-inactive="props.inactive"
  :data-opened="opened"
  @mousedown="switchOption"
  @contextmenu.stop.prevent
  @blur="onBlur")
  .body
    .label {{translate(props.label)}}
    SelectInput(
      :label="props.optLabel"
      :value="props.value"
      :opts="props.opts"
      :noneOpt="props.noneOpt"
      :color="props.color"
      :icon="props.icon"
      :folded="folded"
      :opened="opened"
      @update:value="select")
  .note(v-if="props.note") {{props.note}}
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import { translate } from 'src/dict'
import SelectInput from './select-input.vue'

type InputObjOpt = {
  value: string | number
  tooltip?: string
  color?: string
  icon?: string
}
type InputOption = string | number | InputObjOpt

interface SelectFieldProps {
  value: InputOption | InputOption[]
  label: string
  inactive?: boolean
  inline?: boolean
  optLabel?: string
  opts: readonly InputOption[]
  color?: string
  icon?: string
  noneOpt?: string | number
  note?: string
  folded?: boolean
}

const emit = defineEmits(['update:value'])
const props = defineProps<SelectFieldProps>()
const opened = ref(false)
const rootEl = ref<HTMLElement | null>(null)

function switchOption(e: DOMEvent<MouseEvent>): void {
  if (props.inactive || !props.opts || Array.isArray(props.value)) return
  if (props.folded) {
    opened.value = !opened.value
    if (rootEl.value) {
      rootEl.value.tabIndex = 0
      rootEl.value.focus()
    }
    return
  }
  let i = props.value !== undefined ? props.opts.indexOf(props.value) : -1
  if (i === -1) i = props.opts.findIndex(o => (o as InputObjOpt).value === props.value)
  if (i === -1) return
  if (e.button === 0) i++
  if (e.button === 2) i--
  if (i >= props.opts.length) i = 0
  if (i < 0) i = props.opts.length - 1

  let selected = props.opts[i]
  if (selected && (selected as InputObjOpt).value) {
    emit('update:value', (selected as InputObjOpt).value)
  } else {
    emit('update:value', selected)
  }
}

function select(option: string): void {
  emit('update:value', option)
  opened.value = false
  if (rootEl.value) rootEl.value.tabIndex = -1
}

function onBlur(): void {
  opened.value = false
  if (rootEl.value) rootEl.value.tabIndex = -1
}
</script>
