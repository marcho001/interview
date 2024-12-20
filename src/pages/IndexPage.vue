<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input ref="nameInputEl" v-model="tempData.name" label="姓名" :rules="nameInputRules" lazy-rules />
        <q-input ref="ageInputEl" v-model="tempData.age" label="年齡" :rules="ageInputRules" lazy-rules />
        <q-btn v-if="mode === 'create'" color="primary" class="q-mt-md" :disable="isCreateLoading" @click="createUser">新增</q-btn>
        <div v-else>
          <q-btn color="primary" class="q-mt-md q-mr-md" :disable="isUpdateLoading" @click="updateUser">編輯</q-btn>
          <q-btn  class="q-mt-md" @click="handleCancel">取消</q-btn>

        </div>
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
              <div>{{ col.value }}</div>
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
                :disable="btn.status === 'delete' ? isDeleteLoading : false"
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
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps, useQuasar } from 'quasar';
import { onBeforeMount, ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
type UserItem = {
  id: string;
  name: string;
  age: number;
};
type UserList = UserItem[];

const $q = useQuasar();
// get user list
const blockData = ref<UserList>([]);
const getUsers = async () => {
  try {
    const { data } = await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a');
    blockData.value = data;
  } catch (error) {
    console.error(error);
  }
};
onBeforeMount(() => {
  getUsers()
})


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

const defaultData = {
  name: '',
  age: 0,
};
const tempData = ref<Omit<UserItem, 'id'>>({ ...defaultData });
const nameInputEl = ref()
const ageInputEl = ref()
const nameInputRules = [(val: string) => !!val || '輸入姓名']
const ageInputRules = [(val: string) => !Number.isNaN(+val) || '輸入數字', (val: string) => !!val || '輸入年齡']
const mode = ref('create');
const editingId = ref('');
function resetData() {
  tempData.value = { ...defaultData };
  nameInputEl.value?.resetValidation()
  ageInputEl.value?.resetValidation()
}
function validateInput() {
  nameInputEl.value?.validate()
  ageInputEl.value?.validate()
  return nameInputEl.value.hasError || ageInputEl.value.hasError
}

const isCreateLoading = ref(false);
async function createUser() {
  try {
    if (validateInput()) return
    isCreateLoading.value = true;
    await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', tempData.value);
    getUsers()
    resetData();
    $q.notify('新增成功');
  } catch (error) {
    console.error(error);
  } finally {
    isCreateLoading.value = false;
  }
};

const isUpdateLoading = ref(false);
async function updateUser() {
  try {
    if (validateInput()) return
    isUpdateLoading.value = true;
    await axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest', { ...tempData.value, id: editingId.value });
    blockData.value = blockData.value.map((item: UserItem) => {
      if (item.id === editingId.value) {
        return { ...tempData.value, id: editingId.value };
      }
      return item;
    });
    $q.notify('編輯成功');
    resetData();
    mode.value = 'create';
    editingId.value = '';
  } catch (error) {
    console.error(error);
  } finally {
    isUpdateLoading.value = false;
  }
};

const isDeleteLoading = ref(false);
async function deleteUser(userId: string) {
  try {
    isDeleteLoading.value = true;
    await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${userId}`);
    blockData.value = blockData.value.filter((item: UserItem) => item.id !== userId);
    $q.notify('刪除成功');
  } catch (error) {
    console.error(error);
  } finally {
    isDeleteLoading.value = false;
  }
};
function handleCancel() {
  mode.value = 'create';
  editingId.value = '';
  resetData();
}

function handleClickOption(btn: btnType, data: UserItem) {
  const { status } = btn
  if (status === 'edit') {
    mode.value = 'edit'
    const { id, ...rest } = data
    tempData.value = { ...rest }
    editingId.value = id
  } else if (status === 'delete') {
    $q.dialog({
      title: '提示',
      message: '是否確定刪除該筆資料？',
      ok: '確定',
      cancel: '取消',
    }).onOk(() => {
      deleteUser(data.id)
    })
  }
}


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
