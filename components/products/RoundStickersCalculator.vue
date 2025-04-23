<script setup>
import { ref, computed } from "vue";
import settings from "~/assets/settings.json";

const printPrices = settings.print_price;
const cuttingPrices = settings.cutting_price;
const sheet = settings.sheet;

const tirazh = ref(1000);
const diameter = ref(50);
const materialKey = ref("paper_sticker");
const laminationKey = ref("soft_touch");

const getPrintPricePerSheet = (sheets) => {
  return (
    printPrices.find((p) => sheets <= p.to)?.price ?? printPrices.at(-1).price
  );
};

const getCuttingPrice = (itemsPerSheet) => {
  return (
    cuttingPrices.find((p) => itemsPerSheet <= p.to)?.price ??
    cuttingPrices.at(-1).price
  );
};

const result = computed(() => {
  const sizeWithMargin = Number(diameter.value) + sheet.margin * 2;

  const itemsPerRow = Math.floor(sheet.width / sizeWithMargin);
  const itemsPerColumn = Math.floor(sheet.height / sizeWithMargin);
  const itemsPerSheet = itemsPerRow * itemsPerColumn;

  if (!itemsPerSheet || itemsPerSheet === 0) return { total: 0 };

  const sheetsNeeded = Math.ceil(tirazh.value / itemsPerSheet);

  const printPrice = getPrintPricePerSheet(sheetsNeeded);
  const materialPrice = settings.materials[materialKey.value];
  const laminationPrice = settings.lamination[laminationKey.value];

  const unitPrice = printPrice + materialPrice + laminationPrice;
  const subtotal = unitPrice * sheetsNeeded;

  const cuttingUnitPrice = getCuttingPrice(itemsPerSheet);
  const cuttingTotal = cuttingUnitPrice * sheetsNeeded;

  return {
    itemsPerSheet,
    sheetsNeeded,
    unitPrice,
    subtotal,
    cuttingUnitPrice,
    cuttingTotal,
    total: subtotal + cuttingTotal,
  };
});
</script>

<template>
  <div class="max-w-2xl mx-auto p-6 space-y-6">
    <h1 class="text-2xl font-bold">Калькулятор круглых наклеек</h1>

    <div class="space-y-4">
      <label class="block">
        Тираж:
        <input
          v-model.number="tirazh"
          type="number"
          min="1"
          class="mt-1 border px-2 py-1 w-full"
        />
      </label>

      <label class="block">
        Диаметр (мм):
        <input
          v-model.number="diameter"
          type="number"
          min="1"
          class="mt-1 border px-2 py-1 w-full"
        />
      </label>

      <label class="block">
        Материал:
        <select v-model="materialKey" class="mt-1 border px-2 py-1 w-full">
          <option
            v-for="(price, key) in settings.materials"
            :value="key"
            :key="key"
          >
            {{ key }} — {{ price }}₽
          </option>
        </select>
      </label>

      <label class="block">
        Ламинация:
        <select v-model="laminationKey" class="mt-1 border px-2 py-1 w-full">
          <option
            v-for="(price, key) in settings.lamination"
            :value="key"
            :key="key"
          >
            {{ key }} — {{ price }}₽
          </option>
        </select>
      </label>
    </div>

    <div class="border-t pt-6">
      <h2 class="text-xl font-semibold mb-2">Расчёт:</h2>
      <ul class="space-y-1">
        <li>
          Изделий на листе: <strong>{{ result.itemsPerSheet }}</strong>
        </li>
        <li>
          Необходимо листов: <strong>{{ result.sheetsNeeded }}</strong>
        </li>
        <li>
          Стоимость за лист (печать + материал + ламинация):
          <strong>{{ result.unitPrice }}₽</strong>
        </li>
        <li>
          Сумма за печать и материалы: <strong>{{ result.subtotal }}₽</strong>
        </li>
        <li>
          Резка: {{ result.cuttingUnitPrice }}₽ × {{ result.sheetsNeeded }} =
          <strong>{{ result.cuttingTotal }}₽</strong>
        </li>
        <li class="text-lg font-bold mt-2">
          Итого: <span class="text-green-600">{{ result.total }}₽</span>
        </li>
      </ul>
    </div>
  </div>
</template>
