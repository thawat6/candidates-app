<template>
  <div class="pa-5">
    <v-card class="rounded-card pa-3 rounded-xl elevation-0" title="Master Parts changeover matrix">
      <template v-slot:append>
        <v-btn @click="exportData" color="blue-darken-4" rounded="lg" variant="outlined" class="text-none">
          Export
        </v-btn>
        <v-btn @click="dialogPart = true" color="blue-darken-4" rounded="lg" variant="flat" class="text-none ms-3">
          Add Part
        </v-btn>
      </template>
      <v-data-table :headers="headers" :items="matrix">
        <template v-slot:item="{ item }">
          <tr>
            <td>{{ item.part_name }}</td>
            <td v-for="(header, index) in headers.slice(1)" :key="index" class="text-center">
              <div v-if="header.value == 'edit'">
                <v-btn @click="setItemPart(item, headers)" color="blue-darken-4" rounded="lg" variant="outlined"
                  class="text-none" density="compact">
                  {{ !item.editing ? 'edit' : 'save' }}
                </v-btn>
              </div>
              <div v-else>
                <div v-if="header.title == item.part_name">
                  -
                </div>
                <div v-else-if="!item.editing">
                  {{ item[header.title] ? item[header.title] : '0' }}
                </div>
                <div v-else>
                  <v-text-field class="mt-5" type="number" rounded="lg" v-model.number="item[header.title]" variant="outlined"
                    density="compact">
                  </v-text-field>
                </div>
              </div>
            </td>
          </tr>
        </template>
        <template v-slot:bottom></template>
      </v-data-table>
    </v-card>
    <v-dialog v-model="dialogTime" max-width="400">
      <v-card class="rounded-lg pa-4">
        <div class="d-flex justify-end">
          <v-icon @click="dialogTime = false">mdi-close</v-icon>
        </div>
        <v-card-title class="justify-center mb-2">
          <h3 class="text-center" style="font-weight: 400">Update Changeover time</h3>
        </v-card-title>
        <v-text-field label="Changeover time" rounded="lg" v-model="componentData.part" variant="outlined"
          density="compact">
        </v-text-field>
        <v-card-actions class="d-flex justify-center">
          <v-btn class="text-none" color="blue-darken-4" rounded="lg" @click="dialogTime = false">
            ยกเลิก
          </v-btn>
          <v-btn :disabled="!componentData.part" :loading="loadingOut" class="text-none" color="blue-darken-4"
            rounded="lg" variant="flat" @click="addPart">
            ยืนยัน
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="dialogPart" max-width="400">
      <v-card class="rounded-lg pa-4">
        <div class="d-flex justify-end">
          <v-icon @click="dialogPart = false">mdi-close</v-icon>
        </div>
        <v-card-title class="justify-center mb-2">
          <h3 class="text-center" style="font-weight: 400">Add Part</h3>
        </v-card-title>
        <v-text-field label="Part" rounded="lg" v-model="componentData.part" variant="outlined" density="compact">
        </v-text-field>
        <v-card-actions class="d-flex justify-center">
          <v-btn class="text-none" color="blue-darken-4" rounded="lg" @click="dialogPart = false">
            ยกเลิก
          </v-btn>
          <v-btn :disabled="!componentData.part" :loading="loadingOut" class="text-none" color="blue-darken-4"
            rounded="lg" variant="flat" @click="addPart">
            ยืนยัน
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-notification ref="componentNotify" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { requestService } from '@/_services'

const headers = ref([]);
const matrix = ref([]);
const jsonData = ref([]);
const componentNotify = ref(null);
const dialogPart = ref(false)
const loadingOut = ref(false)
const componentData = reactive({
  part: ''
});


const setItemPart = (itemPart, headers) => {
  
  if (itemPart.editing) {
    let dateUpdate = []
    jsonData.value.forEach(entry => {
      if (entry.part_name == itemPart.part_name) {
        dateUpdate.push({
          matrix_id: entry.id,
          changeover_time: itemPart[entry.target_part_name]
        })
      }
    });
    const data = dateUpdate
    requestService.PutFetch('/matrix/more/', data)
      .then(response => response.json())
      .then((data) => {
        if (data) {
          generateHeaders()
          itemPart.editing = false;
          if (componentNotify.value && componentNotify.value.addNotification) {
            componentNotify.value.addNotification({ message: 'Update success.', typeAlert: 'success' });
          }
        }
      })
  } else {
    itemPart.editing = true;
  }
}

const addPart = () => {
  if (componentData.part) {
    loadingOut.value = true
    const data = {
      part_name: componentData.part
    }
    requestService.PostFetch('/parts-and-matrix/', data)
      .then(response => response.json())
      .then((data) => {
        if (data) {
          generateHeaders()
          dialogPart.value = false
          if (componentNotify.value && componentNotify.value.addNotification) {
            componentNotify.value.addNotification({ message: 'Add new part success.', typeAlert: 'success' });
          }
          componentData.part = ''
          loadingOut.value = false
        }
      })
  }
}
const exportData = () => {
  requestService.GetFetch('/export/matrix/')
    .then(response => response.blob())
    .then((data) => {
      const blob = data
      const url = window.URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.setAttribute('download', 'master_parts_changeover.csv');
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    })
}
const generateHeaders = () => {
  requestService.GetFetch('/matrix/')
    .then(response => response.json())
    .then((data) => {
      jsonData.value = data
      const partNames = [...new Set(jsonData.value.map(entry => entry.part_name))];
      const targetPartNames = [...new Set(jsonData.value.map(entry => entry.target_part_name))];

      const newHeaders = [
        { title: '', value: 'part_name' },
        ...targetPartNames.map(targetPartName => ({
          title: targetPartName,
          align: 'center',
          sortable: false,
          value: targetPartName
        }))
      ];

      headers.value = newHeaders;
      headers.value.push({
        title: 'Edit time',
        align: 'center',
        sortable: false,
        value: 'edit'
      })

      const newData = partNames.map(partName => {
        const item = { part_name: partName };
        targetPartNames.forEach(targetPartName => {
          const entry = jsonData.value.find(entry => entry.part_name === partName && entry.target_part_name === targetPartName);
          if (entry) {
            item[targetPartName] = entry.changeover_time;
          } else {
            item[targetPartName] = '-';
          }
        });
        return item;
      });
      matrix.value = newData;
    })
};
const getChangeoverTime = (partItem, headerText) => {
  const header = headers.value.find(header => header.title === headerText);
  if (!header) return '-';
  if (header.value == 'edit') return 'edit';
  if (partItem.part_name == headerText) return '-';

  const entry = matrix.value.find(entry => entry.part_name === partItem.part_name && entry.target_part_id === header.value);
  return entry ? (entry.changeover_time || '0') : '0';
};

onMounted(() => {
  generateHeaders();
});
</script>