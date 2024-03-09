<template>
  <div class="pa-5">
    <v-card class="rounded-card pa-3 rounded-xl elevation-0" :title="title">
      <template v-slot:append>
        <div v-for="btn in itemsBtn" :key="btn.title">
          <v-menu v-if="btn.btnType == 'menu'">
            <template v-slot:activator="{ props }">
              <v-btn :class="btn.class" :color="btn.color" rounded="lg" :prepend-icon="btn.icon" :variant="btn.variant"
                v-bind="props" :disabled="btn.disabled" class="ms-3">
                {{ btn.title }}
              </v-btn>
            </template>
            <v-list>
              <v-list-item v-for="(child, index) in btn.children" :key="index" :value="index">
                <v-list-item-title @click="handleAction(child)">{{ child.title }}</v-list-item-title>
              </v-list-item>
            </v-list>
          </v-menu>
          <v-btn v-else-if="btn.btnType == 'btn'" :class="btn.class" :color="btn.color" rounded="lg"
            :prepend-icon="btn.icon" :variant="btn.variant" class="ms-3" @click="handleAction(btn)">
            {{ btn.title }}
          </v-btn>
        </div>
      </template>
      <v-data-table-server v-model="componentData.selected" :show-select="showSelect"
        v-model:items-per-page="componentData.limit" :headers="headers" :items-length="totalItems" :items="serverItems"
        :loading="loading" item-value="id" @update:options="loadItems" return-object>
        <template v-slot:item.actions="{ item }">
          <v-icon @click="handleAction({action: 'edit-item'})" size="small" class="me-2">
            mdi-pencil
          </v-icon>
          <v-icon @click="handleAction({ action: 'delete-item' })" size="small" class="me-2">
            mdi-delete
          </v-icon>
        </template></v-data-table-server>
    </v-card>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from 'vue'
const props = defineProps({
  itemsPerPage: Number,
  headers: Array,
  totalItems: Number,
  serverItems: Array,
  loading: Boolean,
  itemsBtn: Array,
  title: String,
  showSelect: Boolean,
})
const componentData = reactive({
  limit: props.itemsPerPage,
  selected: []
});
const emit = defineEmits(['loadItems', 'handleAction'])
function loadItems({ page, itemsPerPage, sortBy }) {
  let sort = JSON.parse(JSON.stringify(sortBy))
  emit('loadItems', { page, itemsPerPage, sortBy, sort })
}
watch(
  () => componentData.selected,
  val => {
    componentData.selected = val;
    emitsSelected(JSON.parse(JSON.stringify(val)));
  },
);
function emitsSelected(val) {
  emit('update-select-row', val);
}
function handleAction(val) {
  emit('handleAction', val)
}
</script>

<style lang="scss" scoped></style>