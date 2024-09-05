<template>
  <div>
    <input :value="modelValue" @input="handleInput">
  </div>
</template>

<script setup>
import { toRefs,defineProps,defineEmits } from 'vue';

const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])
const handleInput = (e) =>{
  emit('update:modelValue',e.target.value)
} 
console.log(props.modelValue);


const {modelValue} = toRefs(props)
</script>

<style lang="scss" scoped>

</style>