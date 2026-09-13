<script setup>
import { useSlideContext } from '@slidev/client'

const { $page } = useSlideContext()
</script>

<template>
  <div v-if="$page > 1" class="slide-number">
    {{ $page }}
  </div>
</template>
