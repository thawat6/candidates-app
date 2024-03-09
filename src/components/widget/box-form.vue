<template>
  <div class="rounded-lg pa-5">
    <v-card v-for="(form, j) in itemsForm" :key="j" class="rounded-card pa-3 rounded-xl elevation-0 mb-3"
      :title="form.title">
      <v-form>
        <v-row class="d-flex align-center mt-3 mb-1">
          <v-col v-for="(item, i) in form.contents" :key="i" :cols="item.col" class="py-1">
            <v-text-field v-if="item.widgetType == 'input'" :label="item.label" rounded="lg" v-model="item.value"
              variant="outlined" density="compact" :rules="getRules(item.rules)" :readonly="item.readonly">
            </v-text-field>
            <v-select v-else-if="item.widgetType == 'select'" rounded="lg" v-model="item.value" :items="item.options"
              variant="outlined" density="compact" :label="item.label" :rules="getRules(item.rules)"></v-select>
            <v-datepicker v-else-if="item.widgetType == 'date'" :dataItem="item" :rules="getRules(item.rules)"
              @input="handleInput(item)"></v-datepicker>
            <v-currency v-else-if="item.widgetType == 'input-num'" v-model="item.value" :dataItem="item"
              :rules="getRules(item.rules)"></v-currency>
            <box-table v-else-if="item.widgetType == 'table'" :itemsPerPage="item.itemsPerPage" :headers="item.headers"
              :totalItems="item.totalItems" :serverItems="item.serverItems" :itemsBtn="item.itemsBtn"
              @handleAction="handleAction" />
          </v-col>
        </v-row>
      </v-form>
    </v-card>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'
const emit = defineEmits(['calAge', 'handleAction'])
const props = defineProps({
  itemsForm: Array
})
const requireRules = value => {
  if (value) return true
  return 'Requred'
}
const emailRules = value => {
  if (/.+@.+\..+/.test(value)) return true
  return 'E-mail must be valid.'
}
const phoneRules = value => {
  if (!isNaN(value) && /^0\d{0,9}$/.test(value) && value && value.length >= 9 && value.length <= 10) return true
  return 'Phone must be valid.'
}
function handleInput(val) {
  switch (val.action) {
    case 'calAge':
      emit('calAge', val)
      break;
  }
}
const handleAction = (val) => {
  emit('handleAction', val)
}
function getRules(val) {
  let rules = []
  if (val && val.length) {
    val.forEach(element => {
      if (element == 'require') {
        rules.push(requireRules)
      }
      if (element == 'email') {
        rules.push(emailRules)
      }
      if (element == 'phone') {
        rules.push(phoneRules)
      }
    });
  }

  return rules
}
</script>

<style lang="scss" scoped></style>