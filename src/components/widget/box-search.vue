<template>
  <div class="rounded-lg pa-5">
    <v-card class="rounded-card pa-8 rounded-xl elevation-0">
      <v-form @submit.prevent="search">
        <v-row class="d-flex align-center">
          <v-col cols="10">
            <v-row class="align-center">
              <v-col v-for="(item, i) in itemsSearch" :key="i" :cols="item.col" class="py-0">
                <v-text-field v-if="item.widgetType == 'input'" :label="item.label" rounded="lg" v-model="item.value"
                  variant="outlined" density="compact">
                </v-text-field>
                <v-select v-else-if="item.widgetType == 'select'" rounded="lg" v-model="item.value" :items="item.options"
                  variant="outlined" density="compact" :label="item.label" clearable></v-select>
                <v-datepicker v-else-if="item.widgetType == 'date'" :dataItem="item"></v-datepicker>
                <v-currency v-else-if="item.widgetType == 'input-num'" v-model="item.value" :dataItem="item"></v-currency>
              </v-col>
            </v-row>
          </v-col>
          <v-divider vertical class="mx-4"></v-divider>
          <v-col cols="auto">
            <v-btn @click="search()" class="rounded-lg px-12" color="grey-darken-3" max-width="50%" type="submit"
              elevation="0"><v-icon>mdi-magnify</v-icon>ค้นหา</v-btn>
          </v-col>
        </v-row>
      </v-form>
    </v-card>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'
const emit = defineEmits(['loadItems'])
const props = defineProps({
  itemsSearch: Array
})
function search() {
  let itemsSearch = props.itemsSearch
  const dataObj = {};
  itemsSearch.forEach(item => {
    if (item.widgetType == 'date') {
      // item.value = 
    }
    dataObj[item.key] = item.value;
  });
  const queryString = itemsSearch.filter(item => item.value)
    .map(item => `${item.key}=${item.value}`)
    .join('&');
  emit('loadItems', { queryString, dataObj })
}
</script>

<style lang="scss" scoped></style>