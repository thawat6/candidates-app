<template>
  <div>
    <v-text-field v-model="formattedValue" :label="dataItem.label" ref="inputRef" type="text" rounded="lg"
      variant="outlined" density="compact" :rules="rules"></v-text-field>
  </div>
</template>

<script>
import { useCurrencyInput } from 'vue-currency-input';
import { watch } from 'vue';

export default {
  name: 'CurrencyInput',
  props: {
    dataItem: Object,
    options: Object,
    rules: Array,
  },

  setup(props) {
    const { inputRef, formattedValue, setValue } = useCurrencyInput({
      currency: 'USD',
      currencyDisplay: 'hidden',
      precision: 2,
      hideCurrencySymbolOnFocus: true,
      hideGroupingSeparatorOnFocus: false,
      hideNegligibleDecimalDigitsOnFocus: true,
      autoDecimalDigits: false,
      useGrouping: true,
      accountingSign: false,
    });
    watch(
      () => props.dataItem.value,
      (value) => {
        setValue(value);
      }
    );
    return { inputRef, formattedValue };
  },
};
</script>
