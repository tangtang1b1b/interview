<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn @click="handleAddList(tempData.name, tempData.age)" color="primary" class="q-mt-md">新增</q-btn>
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
                :color="btn.status === 'delete' ? 'red' : 'blue'"
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
    <div class="mask" v-show="modal">
      <div class="modal">
        <q-input v-model="dataUpdate.name" label="姓名" />
        <q-input v-model="dataUpdate.age" label="年齡" />
        <div class="btn">
          <q-btn @click="handleUpdate(dataUpdate.id, dataUpdate.name, dataUpdate.age)" color="primary" class="q-mt-md">修改</q-btn>
          <q-btn @click="modal = false" color="primary" class="q-mt-md">取消</q-btn>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from "axios";
import { QTableProps } from "quasar";
import { ref, onMounted } from "vue";
interface btnType {
  label: string;
  icon: string;
  status: string;
}

const axiosData = async () => {
  try {
    const res = await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a');
    blockData.value = res.data;
  } catch (error) {
    console.error(error);
  }
};
const modal = ref(false)
const blockData = ref([
  {
    name: "test",
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: "姓名",
    name: "name",
    field: "name",
    align: "left",
  },
  {
    label: "年齡",
    name: "age",
    field: "age",
    align: "left",
  },
]);
const tableButtons = ref([
  {
    label: "編輯",
    icon: "edit",
    status: "edit",
  },
  {
    label: "刪除",
    icon: "delete",
    status: "delete",
  },
]);

const dataUpdate = ref({
  id:"",
  name: "",
  age: ""
});

const tempData = ref({
  name: "",
  age: "",
});
function handleClickOption(btn, data) {
  if (btn.status === 'delete') {
    handleDelete(data.id);
  } else if (btn.status === 'edit') {
    modal.value = true
    dataUpdate.value = {id:data.id, name:data.name, age:data.age }
  }
}

const handleAddList = async (nameData, ageData) => {
  if(nameData.length === 0 || nameData === ''){
    alert('姓名不得空白')
  }
  if(ageData.length === 0 || ageData === ''){
    alert('年齡不得空白')
  }else if(isNaN(ageData)){
    alert('年齡限輸入數字')
  }
  const data = {
    name: nameData,
    age: ageData,
  }

  try {
    const res = await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', data);
    blockData.value.push(res.data);
    tempData.value = { name: '', age: '' };
    axiosData()
  } catch (error) {
    console.error( error);
  }
};

const handleDelete = async (idData) => {
  try {
    await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${idData}`);
    blockData.value = blockData.value.filter(item => item.id !== idData);
  } catch (error) {
    console.error(error);
  }
};

const handleUpdate = async (idData, nameData, ageData) => {
  const updatedItem = { id:idData, name:nameData, age:ageData };
  console.log(updatedItem)
  
  try {
    await axios.put(`https://dahua.metcfire.com.tw/api/CRUDTest/${idData}`, updatedItem);

    modal.value = false;
    await axiosData();
  } catch (error) {
    console.error(error);
  }
};

onMounted(() => {
  axiosData()
})

</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
.q-page .mask{
  position: fixed;
  width: 100vw;
  height: 100vh;
  top: 0;
  background-color: rgba(0,0,0,0.2);
  border-radius: 10px;
  border: solid 1px #ccc;
}
.q-page .mask .modal{
  position: absolute;
  padding: 20px;
  z-index: 2;
  top: 50%;
  left: 50%;
  transform: translate(-50%,-50%);
  width: 50%;
  aspect-ratio: 16/9;
  background-color: white;
  border-radius: 10px;
  border: solid 1px #ccc;
}
.q-page .mask .modal .btn{
  width: 100%;
  display: flex;
  justify-content: center;
  gap: 10px;
}
</style>
