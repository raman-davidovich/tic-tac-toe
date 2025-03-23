<script setup lang="ts">
type FieldValue = '' | 'X' | 'O'

defineProps<{
  value: FieldValue
  isWinning: boolean
}>()

defineEmits<{
  (event: 'click'): void
}>()
</script>

<template>
  <button
    class="field"
    :class="{ 'winning-cell': isWinning }"
    :data-value="value"
    @click="$emit('click')"
  >
    {{ value }}
  </button>
</template>

<style scoped lang="scss">
.field {
  width: 100px;
  height: 100px;
  background-color: white;
  border: 2px solid #bdc3c7;
  border-radius: 8px;
  font-size: 2.5rem;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s;

  &:hover {
    background-color: #f8f9fa;
    transform: scale(1.05);
  }

  &:active {
    transform: scale(0.95);
  }

  &[data-value='X'] {
    color: #e74c3c;
  }

  &[data-value='O'] {
    color: #2ecc71;
  }

  &.winning-cell {
    background-color: #fcf8e3;
    border-color: #ffd700;
    animation: pulse 1s infinite;
  }
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}
</style>
