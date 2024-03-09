<template>
  <v-menu v-model="dateMenu" :close-on-content-click="false">
    <template #activator="{ props }">
      <v-text-field v-model="endDateField" rounded="lg" variant="outlined" density="compact" :label="dataItem.label"
        readonly v-bind="props" :rules="rules"></v-text-field>
    </template>
    <v-date-picker :min="dataItem.min.callback()" :max="dataItem.max.callback()" v-model="dataItem.value" color="blue-darken-4"
       @update:model-value="dateMenu = false, handleInput()"></v-date-picker>
  </v-menu>
</template>

<script setup>
import { ref, computed, defineProps, defineEmits } from 'vue'
import dayjs from 'dayjs';
const props = defineProps({
  dataItem: Object,
  rules: Array,
})
const emit = defineEmits(['input'])
const dateMenu = ref(false)
const endDateField = computed(() =>
  dayjs(props.dataItem.value).format('DD/MM/YYYY')
)
function handleInput() {
  emit('input', props.dataItem.value)
}
</script>