<script setup lang="ts">
import { computed, ref } from "vue";
import { useStore } from "vuex";
import Dropdown from "primevue/dropdown";
import { useI18n } from "vue-i18n";

const props = defineProps({
  modelValue: String,
  itemId: String,
});

const emit = defineEmits(["update:modelValue"]);

const model = computed({
  get() {
    return props.modelValue;
  },

  set(value) {
    return emit("update:modelValue", value);
  },
});

const store = useStore();
const { t } = useI18n();

function getAccountName() {
  const ret = store.state.wallet.privateAccounts.find(
    (a) => a.addr == props.modelValue
  );
  if (!ret) return props.modelValue;
  return `${ret.name}: ${props.modelValue}`;
}
</script>
<template>
  <Dropdown
    v-model="model"
    filter
    :options="store.state.wallet.privateAccounts"
    optionLabel="name"
    optionValue="addr"
    :placeholder="$t('account.select_account')"
    class="w-full md:w-14rem"
    :itemid="props.itemId"
  >
    <template #value="slotProps">
      <div v-if="slotProps.value" class="flex align-items-center">
        <div>{{ getAccountName() }}</div>
      </div>
      <span v-else>
        {{ slotProps.placeholder }}
      </span>
    </template>
    <template #option="slotProps">
      <div v-if="slotProps.option" class="flex align-items-center">
        <div>{{ slotProps.option.name }} : {{ slotProps.option.addr }}</div>
      </div>
    </template>
  </Dropdown>
</template>
