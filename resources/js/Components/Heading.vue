<template>
  <div>
    <h2 class="text-2xl font-bold text-gray-800 mb-2">
      {{ title }}
    </h2>
    <p v-if="description" class="text-gray-600 text-sm leading-relaxed">
      {{ description }}
    </p>
  </div>
</template>

<script setup>
defineProps({
  title: {
    type: String,
    required: true
  },
  description: {
    type: String,
    default: ''
  }
})
</script>
