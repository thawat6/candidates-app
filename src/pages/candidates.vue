<template>
    <div>
        <box-header title="Candidates" icon="mdi-account-multiple" />
        <box-search :itemsSearch="itemsSearch" @loadItems="loadItems" />
        <box-table title="Candidates List" :itemsPerPage="itemsPerPage" :headers="headers" :totalItems="totalItems"
            :serverItems="serverItems" :loading="loading" @loadItems="loadItems" @update-select-row="selectedRow"
            :itemsBtn="itemsBtn" @handleAction="handleAction" />
        <v-dialog v-model="dialog" max-width="60%">
            <v-card title="Create Candidate" class="pa-3 rounded-lg">
                <box-form :itemsForm="itemsForm" @calAge="calAge" @handleAction="handleAction" /></v-card>
        </v-dialog>
        <export-excel ref="componentExport" :fields="fieldsExport" :name="'candidates_list'" />
        <modal-confirm ref="componentConfirm" :title="state.title" :content="state.content"
            @on-confirm="deleteItem()" />
        <v-notification ref="componentNotify" />
    </div>
</template>

<script setup>
import { ref, reactive, onMounted, computed } from 'vue';
import boxHeader from '@/components/widget/box.header.vue';
import dayjs from 'dayjs';
import { requestService } from '@/_services'

const dialog = ref(false);
const state = reactive({
    title: '',
    content: ''
});
const showConfirm = ref(false);
const componentExport = ref(null);
const componentConfirm = ref(null);
const componentNotify = ref(null);
const itemsPerPage = ref(10);
const headers = reactive([
    { title: 'Name', align: 'start', sortable: false, key: 'fullname' },
    { title: 'Position', key: 'position', align: 'start' },
    { title: 'Expect Salary', key: 'expect_salary', align: 'start' },
    { title: 'Address', key: 'address', align: 'start' },
    { title: 'Phone', key: 'phone', align: 'start' },
    { title: 'Email', key: 'email', align: 'start' },
    { title: 'Actions', key: 'actions', sortable: false, align: 'end' }
]);
const serverItems = ref([]);
const loading = ref(true);
const totalItems = ref(0);
const selectRow = ref([])
const lenSelectRow = ref(0)
const fieldsExport = reactive({
    'Name': {
        callback: (value) => {
            return value.name + ' ' + value.surname;
        },
    },
    'Position': 'position',
    'Expect Salary': 'expect_salary',
    'Address': 'address',
    'Tel': {
        callback: (value) => {
            return '\u200C' + value.tel;
        },
    },
    'Phone': {
        callback: (value) => {
            return '\u200C' + value.phone;
        },
    },
    'Email': 'email',
    'Living': 'living',
    'Birthday': 'birthday',
    'Age': {
        callback: (value) => {
            const ageInYears = findAge(value.birthday)
            return '\u200C' + ageInYears;
        },
    },
    'Birthday': 'birthday',
    'Race': 'race',
    'Nationality': 'nationality',
    'Religion': 'religion',
    'Identify card no.': 'id_card',
    'Expiration Date': 'expiration_date',
    'Height': 'height',
    'Weight': 'weight',
    'Military Status': 'military_status',
    'Marital Status': 'marital_status',
    'Gender': 'gender',
    'Father Name': 'father_name',
    'Father Age': 'father_age',
    'Father occupation': 'father_occupation',
    'Mother Name': 'mother_name',
    'Mother Age': 'mother_age',
    'Mother occupation': 'mother_occupation',
    'Name of wife / Husband': 'couple_name',
    'Working Place': 'couple_working_place',
    'Position': 'couple_position',
    'Number of children': 'number_children'
})
const itemsBtn = reactive([
    {
        title: 'Export',
        icon: 'mdi-file-export',
        outlined: true,
        color: 'blue-darken-4',
        checkSelect: true,
        action: 'export-csv',
        disabled: computed(() => selectRow.value.length > 0 ? false : true),
        class: 'text-none',
        variant: 'outlined',
        btnType: 'menu',
        children: [
            {
                title: 'Export Excel', checkSelect: true, action: 'export-excel', disabled: false
            },
            {
                title: 'Export .csv', checkSelect: true, action: 'export-csv', disabled: false
            }
        ]
    },
    {
        title: 'Export All',
        icon: 'mdi-file-export',
        outlined: true,
        color: 'blue-darken-4',
        checkSelect: true,
        action: 'export-csv',
        disabled: false,
        class: 'text-none',
        variant: 'outlined',
        btnType: 'menu',
        children: [
            {
                title: 'Export Excel', checkSelect: true, action: 'export-excel-all', disabled: false
            },
            {
                title: 'Export .csv', checkSelect: true, action: 'export-csv-all', disabled: false
            }
        ]
    },
    {
        title: 'Create New',
        icon: 'mdi-plus',
        outlined: false,
        color: 'blue-darken-4',
        checkSelect: false,
        action: 'modal-create',
        disabled: false,
        class: 'text-none',
        variant: 'flat',
        btnType: 'btn'
    }
]);
const itemsSearch = reactive([
    { widgetType: 'input', label: 'Fullname', value: '', col: 3, key: 'fullname' },
    { widgetType: 'select', label: 'Position', value: null, col: 3, options: ["Full Stack", "Frontend", "Backend"], key: 'position' },
    { widgetType: 'input-num', label: 'Salary Min', value: '', col: 3, key: 'salary_min' },
    { widgetType: 'input-num', label: 'Salary Max', value: '', col: 3, key: 'salary_max' },
    {
        widgetType: 'date',
        label: 'Start Apply',
        value: new Date(dayjs(new Date()).subtract(1, 'M')),
        col: 3,
        min: {
            callback: () => {
                return ''
            }
        },
        max: {
            callback: () => {
                let index = itemsSearch.findIndex(item => item.label == 'To Apply')
                return itemsSearch[index].value
            }
        },
        key: 'start_apply'
    },
    {
        widgetType: 'date',
        label: 'To Apply',
        value: new Date(),
        col: 3,
        min: {
            callback: () => {
                let index = itemsSearch.findIndex(item => item.label == 'Start Apply')
                return itemsSearch[index].value
            }
        },
        max: {
            callback: () => {
                return ''
            }
        },
        key: 'to_apply'
    },
]);
const itemsForm = reactive([
    {
        title: 'Personal information',
        contents: [
            {
                widgetType: 'input',
                label: 'Name',
                value: '',
                col: 3,
                rules: ['require'],
                key: 'name'
            },
            {
                widgetType: 'input',
                label: 'Surname',
                value: '',
                col: 3,
                rules: ['require'],
                key: 'surname'
            },
            {
                widgetType: 'select',
                label: 'Position Applied for',
                value: null,
                col: 3,
                options: ['Full Stack Develop', 'Frontend Develop', 'Backend Develop'],
                rules: ['require'],
                key: 'position'
            },
            {
                widgetType: 'input-num',
                label: 'Expect Salary',
                value: '',
                col: 3,
                rules: ['require'],
                key: 'expect_salary'
            },
            {
                widgetType: 'input',
                label: 'Address',
                value: '',
                col: 12,
                rules: ['require'],
                key: 'address'
            },
            {
                widgetType: 'input',
                label: 'Tel.',
                value: '',
                col: 3,
                rules: ['phone'],
                key: 'tel'
            },
            {
                widgetType: 'input',
                label: 'Mobile',
                value: '',
                col: 3,
                rules: ['require', 'phone'],
                key: 'mobile'
            },
            {
                widgetType: 'input',
                label: 'E-mail',
                value: '',
                col: 3,
                rules: ['require', 'email'],
                key: 'email'
            },
            {
                widgetType: 'select',
                label: 'Living',
                value: null,
                col: 3,
                options: ['Living with parent', 'Own home', 'Hire house', 'Hire flat/ Apartment'],
                key: 'living'
            },
            {
                widgetType: 'date',
                label: 'Birthday',
                value: new Date(dayjs(new Date()).subtract(20, 'y')),
                col: 3,
                rules: ['require'],
                key: 'birthday',
                min: {
                    callback: () => {
                        return ''
                    }
                },
                max: {
                    callback: () => {
                        return new Date()
                    }
                },
                action: 'calAge'
            },
            {
                widgetType: 'input',
                label: 'Age',
                value: '',
                col: 3,
                readonly: true,
                key: 'age',
            },
            {
                widgetType: 'input',
                label: 'Race',
                value: '',
                col: 3,
                key: 'race',
            },
            {
                widgetType: 'input',
                label: 'Nationality',
                value: '',
                col: 3,
                key: 'nationality',
            },
            {
                widgetType: 'input',
                label: 'Religion',
                value: '',
                col: 3,
                key: 'religion',
            },
            {
                widgetType: 'input',
                label: 'Identify card no.',
                value: '',
                col: 3,
                key: 'id_card',
            },
            {
                widgetType: 'date',
                label: 'Expiration date',
                value: new Date(),
                col: 3,
                rules: ['require'],
                key: 'expiration_date',
                min: {
                    callback: () => {
                        return ''
                    }
                },
                max: {
                    callback: () => {
                        return ''
                    }
                },
            },
            {
                widgetType: 'input',
                label: 'Height (cm.)',
                value: '',
                col: 3,
                key: 'height',
            },
            {
                widgetType: 'input',
                label: 'Weight (kgs.)',
                value: '',
                col: 3,
                key: 'weight',
            },
            {
                widgetType: 'select',
                label: 'Military status',
                value: null,
                col: 3,
                options: ['Exempted', 'Served', 'Not yet served'],
                key: 'military_status'
            },
            {
                widgetType: 'select',
                label: 'Marital status',
                value: null,
                col: 3,
                options: ['Single', 'Married', 'Widowed', 'Separated'],
                key: 'marital_status'
            },
            {
                widgetType: 'select',
                label: 'Gender',
                value: null,
                col: 3,
                options: ['Male', 'Female', 'LGBTQ+'],
                key: 'gender'
            },
        ]
    },
    {
        title: 'Family Information',
        contents: [
            {
                widgetType: 'input',
                label: 'Father’s name-surname',
                value: '',
                col: 5,
                key: 'father_name',
            },
            {
                widgetType: 'input',
                label: 'Age',
                value: '',
                col: 3,
                key: 'father_age',
            },
            {
                widgetType: 'input',
                label: 'Occupation',
                value: '',
                col: 4,
                key: 'father_occupation',
            },
            {
                widgetType: 'input',
                label: 'Mother’s name-surname',
                value: '',
                col: 5,
                key: 'mother_name',
            },
            {
                widgetType: 'input',
                label: 'Age',
                value: '',
                col: 3,
                key: 'mother_age',
            },
            {
                widgetType: 'input',
                label: 'Occupation',
                value: '',
                col: 4,
                key: 'mother_occupation',
            },
            {
                widgetType: 'input',
                label: 'Name of wife / Husband',
                value: '',
                col: 3,
                key: 'couple_name',
            },
            {
                widgetType: 'input',
                label: 'Working Place',
                value: '',
                col: 3,
                key: 'couple_working_place',
            },
            {
                widgetType: 'input',
                label: 'Position',
                value: '',
                col: 3,
                key: 'couple_position',
            },
            {
                widgetType: 'input',
                label: 'Number of children',
                value: '',
                col: 3,
                key: 'number_children',
            },
            {
                widgetType: 'table',
                itemsPerPage: 10,
                headers: [
                    { title: 'Name', key: 'fullname', sortable: false, align: 'start' },
                    { title: 'Position', key: 'position', sortable: false, align: 'start' },
                    { title: 'Position', key: 'position', sortable: false, align: 'start' },
                ],
                totalItems: 0,
                serverItems: [],
                itemsBtn: [
                    {
                        title: 'Remove',
                        icon: 'mdi-plus',
                        outlined: true,
                        color: 'danger',
                        checkSelect: true,
                        action: 'remove-family',
                        disabled: true,
                        class: 'text-none',
                        variant: 'flat',
                        btnType: 'btn'
                    },
                    {
                        title: 'Create New',
                        icon: 'mdi-plus',
                        outlined: false,
                        color: 'blue-darken-4',
                        checkSelect: false,
                        action: 'modal-create-family',
                        disabled: false,
                        class: 'text-none',
                        variant: 'flat',
                        btnType: 'btn'
                    }
                ],
                value: [],
                key: 'families',
            },
        ]
    }
]);

const FakeAPI = {
    async fetch({ page, itemsPerPage, sortBy }) {
        return new Promise(resolve => {
            setTimeout(() => {
                const start = (page - 1) * itemsPerPage;
                const end = start + itemsPerPage;
                const items = desserts.slice();

                if (sortBy.length) {
                    const sortKey = sortBy[0].key;
                    const sortOrder = sortBy[0].order;
                    items.sort((a, b) => {
                        const aValue = a[sortKey];
                        const bValue = b[sortKey];
                        return sortOrder === 'desc' ? bValue - aValue : aValue - bValue;
                    });
                }

                const paginated = items.slice(start, end);

                resolve({ items: paginated, total: items.length });
            }, 500);
        });
    }
};

const findAge = (val) => {
    const today = new Date();
    const today2 = today.toISOString();
    const formattedDate = today2.substring(0, 10) + 'T00:00:00+07:00';
    const timestamp1 = new Date(formattedDate).getTime();
    const timestamp2 = new Date(val.value).getTime();
    const ageInMilliseconds = timestamp1 - timestamp2;
    const ageInSeconds = ageInMilliseconds / 1000;
    const ageInMinutes = ageInSeconds / 60;
    const ageInHours = ageInMinutes / 60;
    const ageInDays = ageInHours / 24;
    const ageInYears = ageInDays / 365;
    return ageInYears
}

const calAge = (val) => {
    const ageInYears = findAge(val)
    itemsForm.forEach(element => {
        let fieldAge = element.contents.find(item => item.key == 'age');
        if (fieldAge) {
            fieldAge.value = Math.floor(ageInYears);
        }
    });
};

const loadItems = (e) => {
    loading.value = true;
    requestService.GetFetch('/posts')
        .then(response => response.json())
        .then((data) => {
            serverItems.value = data;
            // totalItems.value = total;
            loading.value = false;
        })
};

const selectedRow = (val) => {
    selectRow.value = val
};
const deleteItem = (val) => {
    if (componentNotify.value && componentNotify.value.addNotification) {
        componentNotify.value.addNotification({ message: 'Delete item success.', typeAlert: 'success' });
    }
    const path = `/posts/${val.id}`

    requestService.DeleteFetch(path).then(() => { });
};

const handleAction = (val) => {
    const url = `/posts`;
    switch (val.action) {
        case 'modal-create':
            dialog.value = true;
            break;
        case 'edit-item':
            dialog.value = true;
            break;
        case 'delete-item':
            state.title = 'Delete this item'
            state.content = 'Are you sure you want to delete this item?'
            if (componentConfirm.value && componentConfirm.value.showModal)
                componentConfirm.value.showModal();
            break;
        case 'export-excel-all':
            if (componentExport.value && componentExport.value.getDataFromModule)
                componentExport.value.getDataFromModule(url, '.xls');
            break;
        case 'export-csv-all':
            if (componentExport.value && componentExport.value.getDataFromModule)
                componentExport.value.getDataFromModule(url, '.csv');
            break;
        case 'export-excel':
            if (componentExport.value && componentExport.value.exportData)
                componentExport.value.exportData(selectRow.value, '.xls');
            break;
        case 'export-csv':
            if (componentExport.value && componentExport.value.exportData)
                componentExport.value.exportData(selectRow.value, '.csv');
            break;
    }
};

onMounted(() => {
    loadItems({ page: 1, itemsPerPage: itemsPerPage.value, sortBy: [] });
});
</script>