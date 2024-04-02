<script setup lang="ts">
import { ref, watch, onMounted } from 'vue';

const digits = ref<string[]>([]);
const secret = ref(1);
const readOnly = ref(false);
const numberPin = ref(4);
const inputFields = ref<HTMLInputElement[]>([]);

const onInput = (index: number) => {
  if (digits.value[index].length > 1) {
    digits.value[index] = digits.value[index].slice(0, 1);
  }
};

const onPaste = (event: any) => {
  event.preventDefault();
  const pasteData = event.clipboardData?.getData('text');
  if (pasteData && pasteData.length <= numberPin.value && /^\d+$/.test(pasteData)) {
    const pasteDigits = pasteData.split('');
    digits.value = pasteDigits.concat(Array(numberPin.value - pasteDigits.length).fill(''));
  }
};

const onKeyDown = (event: KeyboardEvent, index: number) => {
  if (event.key === 'Backspace') {
    if (index > 0 && digits.value[index] === '') {
      inputFields.value[index - 1].focus();
      digits.value[index - 1] = '';
    }
  } else if (/^\d$/.test(event.key)) {
    digits.value[index] = event.key;
    if (index < digits.value.length - 1) {
      inputFields.value[index + 1].focus();
    }
  }
};

watch(digits, () => {
  if (digits.value.every(digit => digit !== '')) {
    console.log("All input filled");
  }
});

watch(numberPin, (newVal) => {
  if (digits.value.length > newVal) {
    digits.value.splice(newVal, digits.value.length - newVal)
  } else if (digits.value.length < newVal) {

    for (let index = 0; index < newVal - digits.value.length; index++) {
      digits.value.push('')
    }
  }
});

onMounted(() => {
  for (let index = 0; index < numberPin.value; index++) {
    digits.value.push('')
  }
});
</script>

<template>
  <div class="space-y-4">
    <div class="flex flex-row">
      <span>Mark secret: </span>
      <input type="radio" id="yes" name="yes" value="1" v-model="secret" class="mx-5" />
      <label for="yes">Yes</label><br />
      <input type="radio" id="no" name="no" value="0" v-model="secret" class="mx-5" />
      <label for="no">No</label><br />
    </div>


    <span>Number of pin: </span>
    <input type="number" v-model="numberPin" class="border border-gray-30 mx-5 px-2" />

    <div class="flex flex-row space-x-2 my-4">
      <template v-for="(item, index) in digits" :key="index">
        <input v-model="digits[index]"
          class="w-12 h-12 text-center border border-gray-300 rounded-md focus:outline-none focus:border-blue-500"
          :type="secret == 1 ? 'password' : 'text'" maxlength="1" @input="onInput(index)"
          @keydown="onKeyDown($event, index)" :autofocus="index === 0" :readonly="readOnly" @paste="onPaste"
          :class="{ 'bg-gray-200': secret && digits[index] !== '' }" ref="inputFields" />
      </template>
    </div>
  </div>
</template>

<style></style>
