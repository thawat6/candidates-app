<script setup>
import { ref, reactive, defineProps, defineExpose } from 'vue';
import { requestService } from '@/_services'

const props = defineProps({
    name: String,
    fields: Object,
});

const data_export_all = ref([]);
const file_name = ref(props.name);
const file_type = ref(props.name);

const getDataFromModule = async (url, sendFile) => {
    if (sendFile) {
        file_type.value = await sendFile
    }
    await requestService.GetFetch(url)
        .then(response => response.json())
        .then((data) => {
            data_export_all.value = data;
            document.getElementById("button_export_excel").click();
        })
}
const exportData = async (dataItems, sendFile) => {
    if (sendFile) {
        file_type.value = await sendFile
    }
    if (dataItems) {
        data_export_all.value = await dataItems;
        document.getElementById("button_export_excel").click();
    }
}

defineExpose({
    getDataFromModule,
    exportData
});
</script>

<template>
    <download-excel v-if="file_type == '.csv'" v-show="false" type="csv" id="button_export_excel" :data="data_export_all"
        :name="file_name + '.csv'" :fields="props.fields">
    </download-excel>
    <download-excel v-else v-show="false" id="button_export_excel" :data="data_export_all"
        :name="file_name" :fields="props.fields">
    </download-excel>
</template>
