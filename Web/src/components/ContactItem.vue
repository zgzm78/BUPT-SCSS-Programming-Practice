<template>
  <div class="contact-item">
    <div class="contact-info">
      <div class="contact-name">{{ contact.username }}</div>
      <div class="contact-status">
        {{ contact.isOnline ? '在线' : '离线' }}
      </div>
    </div>
    <button 
      class="delete-button"
      @click="$emit('delete-contact', contact.username)"
    >
      删除
    </button>
  </div>
</template>

<script setup>
defineProps({
  contact: {
    type: Object,
    required: true
  }
})

defineEmits(['delete-contact'])
</script>

<style scoped>
.contact-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 0;
  border-bottom: 1px solid #eee;
}

.contact-info {
  flex: 1;
}

.contact-name {
  font-weight: 500;
  margin-bottom: 5px;
}

.contact-status {
  color: #07C160; /* 在线状态颜色 */
  font-size: 12px;
}

.delete-button {
  background-color: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 5px 10px;
  cursor: pointer;
}

.delete-button:hover {
  background-color: #d32f2f;
}
</style>