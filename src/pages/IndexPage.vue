<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="adding">新增</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }} </div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
  <!-- <q-dialog v-model="dialogVisible">
    <q-card>
      <q-card-section>
        <div class="text-h6">Edit Details</div>
      </q-card-section>

      <q-card-section>
        <q-input v-model="form.name" label="Name" />
        <q-input v-model="form.age" label="Age" type="number" />
      </q-card-section>

      <q-card-actions>
        <q-btn flat label="Cancel" color="secondary" @click="closeEditDialog"/>
        <q-btn flat label="Save" color="primary" @click="saveEdit"/>
      </q-card-actions>
    </q-card>
  </q-dialog> -->
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const form = ref({
  name: 'test',
  age: null,
});

const tempData = ref({
  name: '',
  age: '',
});
const dialogVisible = ref(False)

function adding () {
  const { name, age } = tempData.value;

  blockData.value.push({
    name,
    age: parseInt(age, 10), // Ensure age is stored as a number
  });  
}

function handleClickOption(btn, data) {
  // ...
  if (btn.status == "edit"){
    form.name = data.name;
    form.age = data.age;
    // dialogVisible.value = true;
  }
  // else if (btn.status == "delete"){
  //   blockData.value = blockData.value.filter((row_data) => !(row_data["name"] === data["name"] && row_data["age"] === data["age"]));
  // }
  else {
    console.log("Something's wrong , deal later");
  }
}

// function closeEditDialog (){
//   dialogVisible.value = false;
// };

// function saveEdit (){
//   const editedRow = blockData.value.find((row) => row.name === form.name && row.age === form.age);
//   if (editedRow) {
//     editedRow.name = form.name;
//     editedRow.age = form.age;
//   }
//   closeEditDialog();
// };
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
