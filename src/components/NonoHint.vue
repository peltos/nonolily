<template>
  <span class="nono-hint flex justify-center items-center transition-all"> {{ hint }} </span>
</template>

<script setup>

const props = defineProps({
  hint: {
    type: Number,
    default: 1
  }
});

</script>

<style lang="scss"></style>