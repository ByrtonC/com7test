<script setup lang="ts">
import axios from "axios";
import { onMounted, ref, watch } from "vue";
import AppBar from "./components/AppBar.vue";

type ProductDataType = {
  id: number;
  active: true;
  color: string;
  color_hex: string;
  image_url: string;
  image: string[];
  model_name: string;
  name: string;
  price: number;
  price_deposit: number;
  priority: number;
  size: string;
  variant_product: [];
};

type ProductModelType = {
  color: string;
  color_hex: string;
  data: ProductDataType[];
};

type DataProductType = {
  name: string;
  model: ProductModelType[];
};

const idRef = ref<number | null>(null);
const dataSource = ref<DataProductType[]>([]);

const GetData = async () => {
  try {
    const result = await axios.get(`https://interview.com7.in/api/pre-order`);
    dataSource.value = result.data.data.main_product.sort(
      (a: DataProductType, b: DataProductType) => {
        return b.model[0].data[0].price - a.model[0].data[0].price;
      }
    );
    console.log(dataSource.value);

    if (dataSource.value.length === 0) return;
    idRef.value = dataSource.value[0].model[0].data[0].id;
  } catch {
    //
  }
};

const getValueData = () => {
  let name: any = null;
  let img: any = null;
  let color: any = null;
  dataSource.value.forEach((product) => {
    product.model.forEach((model) => {
      model.data.forEach((data) => {
        if (data.id === idRef.value) {
          img = data.image_url;
          name = product.name;
          color = model.color;
        }
      });
    });
  });
  return { img, name, color };
};

const getColorData = () => {
  let list: ProductModelType[] = [];
  dataSource.value.forEach((product) => {
    product.model.forEach((model) => {
      model.data.forEach((data) => {
        if (data.id === idRef.value) {
          list = product.model;
        }
      });
    });
  });
  return list;
};

const getSizeData = () => {
  let list: ProductDataType[] = [];
  dataSource.value.forEach((product) => {
    product.model.forEach((model) => {
      model.data.forEach((data) => {
        if (data.id === idRef.value) {
          list = model.data;
        }
      });
    });
  });
  return list;
};

const onSelectProductName = (name: string) => {
  if (getValueData().name === name) return;
  const find = dataSource.value.find((product) => product.name === name);
  if (!find) return;
  idRef.value = find.model[0].data[0].id;
};

const onSelectColor = (color: string) => {
  if (getValueData().color === color) return;
  const find = dataSource.value.find(
    (product) => product.name === getValueData().name
  );
  if (!find) return;
  const findColor = find.model.find((model) => model.color === color);
  if (!findColor) return;
  idRef.value = findColor.data[0].id;
};

const onSelectSize = (id: number) => {
  idRef.value = id;
};

const getProductType = ref(1);

const modalVisible = ref(false);

const onSubmit = () => {
  try {
    axios.post(`https://interview.com7.in/api/pre-order`, { id: idRef.value });
  } finally {
    modalVisible.value = true;
  }
};

onMounted(() => {
  GetData();
});
</script>

<template>
  <AppBar />
  <div
    v-show="modalVisible"
    class="inset-0 flex items-center justify-center bg-gray-700 bg-opacity-50"
    style="position: fixed; z-index: 9999"
  >
    <div
      class="max-w-2xl p-6 bg-white rounded-md shadow-xl grid justify-items-center md:grid-cols-1"
    >
      <div class="text-3xl">การสั่งซื้อล่วงหน้าสำเร็จ</div>
      <div class="mt-3">การชำระเงินของคุณสำเร็จแล้ว!</div>
      <div>ตอนนี้เราจะส่งอีเมลยืนยันคำสั่งซื้อสำเร็จให้คุณ</div>
      <button
        class="rounded-md bg-green-600 text-white text-2xl p-2 mt-4"
        @click="modalVisible = false"
      >
        ยืนยัน
      </button>
    </div>
  </div>
  <div class="grid md:grid-cols-2 p-4 mt-24">
    <div class="grid justify-items-end">
      <img :src="getValueData().img" style="width: 30%; position: fixed" />
    </div>
    <div class="flex md:order-2 text-2xl">
      <div style="width: 100%">
        <div class="text-3xl">ซื้อ iPhone 13</div>
        <div class="text-2xl my-4">รุ่น</div>
        <div
          v-for="item in dataSource"
          :key="item.name"
          style="
            border-radius: 8px;
            width: 100%;
            padding: 16px;
            cursor: pointer;
          "
          :style="{
            border:
              getValueData().name === item.name
                ? 'solid lightgreen 1px'
                : 'solid lightgrey 1px',
          }"
          @click="onSelectProductName(item.name)"
          class="mt-2 flex justify-between"
        >
          <div class="text-md font-bold">{{ item.name }}</div>
          <div class="text-md">
            ฿{{ Intl.NumberFormat().format(item.model[0].data[0].price) }}
          </div>
        </div>
        <div class="text-2xl my-4 mt-10">สี</div>
        <div class="grid md:grid-cols-2 mx-auto gap-1">
          <div
            v-for="item in getColorData()"
            :key="`${item.color}-${item.color_hex}`"
            style="
              border-radius: 8px;
              width: 100%;
              padding: 16px;
              cursor: pointer;
              border: solid red 1px;
            "
            :style="{
              border:
                getValueData().color === item.color
                  ? 'solid lightgreen 1px'
                  : 'solid lightgrey 1px',
            }"
            class="grid justify-items-center"
            @click="onSelectColor(item.color)"
          >
            <div
              style="border-radius: 100px; width: 40px; height: 40px"
              :style="{ backgroundColor: `#${item.color_hex}` }"
            />
            <div>
              {{ item.color }}
            </div>
          </div>
        </div>
        <div class="text-2xl my-4 mt-10">ขนาด</div>
        <div class="grid md:grid-cols-2 mx-auto gap-1">
          <div
            v-for="item in getSizeData()"
            :key="item.id"
            style="
              border-radius: 8px;
              width: 100%;
              padding: 16px;
              cursor: pointer;
              border: solid red 1px;
            "
            :style="{
              border:
                idRef === item.id
                  ? 'solid lightgreen 1px'
                  : 'solid lightgrey 1px',
            }"
            class="grid justify-items-center"
            @click="onSelectSize(item.id)"
          >
            <div>
              {{ item.size }}
            </div>
            <div class="text-xs">
              ฿{{ Intl.NumberFormat().format(item.price) }}
            </div>
          </div>
        </div>
        <div class="text-2xl my-4 mt-10">คุณต้องการรับสินค้าด้วยวิธีใด</div>
        <div class="grid md:grid-cols-1 mx-auto gap-1">
          <div
            style="
              border-radius: 8px;
              width: 100%;
              padding: 16px;
              cursor: pointer;
              border: solid red 1px;
            "
            :style="{
              border:
                getProductType === 1
                  ? 'solid lightgreen 1px'
                  : 'solid lightgrey 1px',
            }"
            class="grid"
            @click="getProductType = 1"
          >
            <div>บริการรับสินค้าหน้าร้าน</div>
            <div class="text-xs">จองเริ่มต้นเพียง ฿3,000.00 เท่านั้น</div>
          </div>
          <div
            style="
              border-radius: 8px;
              width: 100%;
              padding: 16px;
              cursor: pointer;
              border: solid red 1px;
            "
            :style="{
              border:
                getProductType === 2
                  ? 'solid lightgreen 1px'
                  : 'solid lightgrey 1px',
            }"
            class="grid"
            @click="getProductType = 2"
          >
            <div>บริการรับสินค้าหน้าร้าน</div>
            <div class="text-xs">จองเริ่มต้นเพียง ฿3,000.00 เท่านั้น</div>
          </div>
        </div>
        <div class="grid md:grid-cols-1 mx-auto gap-10 divide-y">
          <div></div>
          <button
            class="rounded-md bg-green-600 text-white p-4"
            @click="onSubmit"
          >
            ยืนยันการสั่งซื้อล่วงหน้า
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
