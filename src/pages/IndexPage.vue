<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" :rules="nameRules" />
        <q-input v-model="tempData.age" label="年齡" :rules="nameRules" />
        <q-btn color="primary" class="q-mt-md" @click="adding">新增</q-btn>
      </div>

      <q-table flat bordered ref="tableRef" :rows="blockData" :columns="(tableConfig as QTableProps['columns'])"
        row-key="id" hide-pagination separator="cell" :rows-per-page-options="[0]">
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
            <q-td v-for="col in props.cols" :key="col.name" :props="props" style="min-width: 120px">
              <div>{{ col.value }} </div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn @click="handleClickOption(btn, props.row)" v-for="(btn, index) in tableButtons" :key="index"
                size="sm" color="grey-6" round dense :icon="btn.icon" class="q-ml-md" padding="5px 5px">
                <q-tooltip transition-show="scale" transition-hide="scale" anchor="top middle" self="bottom middle"
                  :offset="[10, 10]">
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div class="full-width row flex-center items-center text-primary q-gutter-sm" style="font-size: 18px">
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
  <q-dialog v-model="dialogVisible">
    <q-card>
      <q-card-section>
        <div class="text-h6">編輯</div>
      </q-card-section>

      <q-card-section>
        <q-input v-model="after_edit.name" :rules="nameRules" label="Name" />
        <q-input v-model="after_edit.age" label="Age" type="number" :rules="ageRules" />
      </q-card-section>

      <q-card-actions>
        <q-btn flat label="Cancel" color="secondary" @click="closeEditDialog" />
        <q-btn flat label="Save" color="primary" @click="saveEdit" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, reactive, computed, onMounted } from 'vue';
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

const before_edit = reactive({
  name: 'test',
  age: null,
});

const after_edit = ref({
  name: '',
  age: null,
});

const tempData = ref({
  name: '',
  age: '',
});

const dialogVisible = ref(false);

onMounted(() => {
  getTable();
});

function adding() {
  const { name, age } = tempData.value;
  createTableData(tempData.value);
  blockData.value.push({
    name,
    age: parseInt(age, 10), // Ensure age is stored as a number
  });
}

function handleClickOption(btn, data) {
  // ...
  if (btn.status == "edit") {
    before_edit.name = data.name;
    before_edit.age = data.age;
    after_edit.value.name = before_edit.name;
    after_edit.value.age = before_edit.age;
    dialogVisible.value = true;
  }
  else if (btn.status == "delete") {
    const item = blockData.value.find(row => row.name === data.name && row.age === data.age);
    if (item) {
      deleteTableData(item.id)
      blockData.value = blockData.value.filter((row_data) => !(row_data["name"] === data["name"] && row_data["age"] === data["age"]));
    } else {
      console.log("something'wrong")
    }
    deleteTableData(item.id)
    blockData.value = blockData.value.filter((row_data) => !(row_data["name"] === data["name"] && row_data["age"] === data["age"]));
  }
  else {
    console.log("Something's wrong , deal later");
  }
}

function closeEditDialog() {
  dialogVisible.value = false;
};

function saveEdit() {
  console.log(after_edit)
  const editedRowIndex = blockData.value.findIndex(
    (row) => row.name === before_edit.name && row.age === before_edit.age
  );
  if (editedRowIndex !== -1) {
    console.log("inside")
    blockData.value[editedRowIndex].name = after_edit.value.name;
    blockData.value[editedRowIndex].age = after_edit.value.age;
  }
  closeEditDialog();
};

async function createTableData(new_data: any) {
  try {
    const res = await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', {
      name: new_data.name,
      age: new_data.age,
    }, {
      headers: {
        'Content-Type': 'application/json',
      },
    });
    if (res.status === 200) {
      console.log("success");
    } else {
      console.log("fail");
    }
  } catch (error) {
    console.log("something's wrong")
  }
}

async function getTable() {
  try {
    const res = await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a');
    blockData.value.push(...res.data);
    console.log(blockData)
  } catch (error) {
    console.log("something's wrong")
  }
}
async function deleteTableData(id: string) {
  try {
    const res = await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`);
    if (res.status === 200) {
      console.log("success");
    } else {
      console.log("fail");
    }
  } catch (error) {
    console.log("something's wrong")
  }
}
// async function updateTableData() {
//   try {
//     const res = await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest');
//     console.log(res);
//   } catch (error) {
//     console.log("something's wrong")
//   }
// }

const ageRules = computed(() => [
  val => !!val || '必填欄位',  // Checks if value exists (required field)
  val => (val >= 0 && val <= 150) || '不合理的年齡',  // Checks if age is between 0 and 150
]);

const nameRules = computed(() => [
  val => (val && val.length > 0 && val.length < 15) || '必填欄位，最多15字',  // Checks if the value is not empty and its length is between 1 and 15 characters
]);

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
