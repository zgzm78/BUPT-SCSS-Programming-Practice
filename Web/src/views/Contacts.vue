<template>
  <div class="main-layout">
    <Sidebar />
    <div class="content-area">
      <h2>通讯录</h2>
      <div v-if="error" class="error-message">{{ error }}</div>
      
      <div class="add-contact-form">
        <input
          type="text"
          v-model="newContactUsername"
          placeholder="输入好友用户名"
        />
        <button @click="handleAddContact">添加好友</button>
      </div>
      
      <div v-if="loading">加载中...</div>
      <ContactList 
        v-else
        :contacts="contacts"
        @delete-contact="handleDeleteContact"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Sidebar from '@/components/Sidebar.vue';
import ContactList from '@/components/ContactList.vue';
import { getContacts, addContact, deleteContact } from '@/services/api';

const contacts = ref([]);
const newContactUsername = ref('');
const error = ref('');
const loading = ref(false);

onMounted(() => {
  fetchContacts();
});

const fetchContacts = async () => {
  loading.value = true;
  try {
    const response = await getContacts();
    if (response.data.status === 200) {
      contacts.value = response.data.data.contacts;
    } else {
      error.value = response.data.message || '获取通讯录失败';
    }
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

const handleAddContact = async () => {
  if (!newContactUsername.value) return;
  
  try {
    const response = await addContact({ friend_id: newContactUsername.value });
    if (response.data.status === 200) {
      newContactUsername.value = '';
      fetchContacts();
    } else if (response.data.status === 404) {
      error.value = '好友不存在';
    } else if (response.data.status === 409) {
      error.value = '已提交好友';
    } else {
      error.value = response.data.message || '添加联系人失败';
    }
  } catch (err) {
    error.value = err.message;
  }
};

const handleDeleteContact = async (username) => {
  try {
    const response = await deleteContact({ friend_id: username });
    if (response.data.status === 200) {
      fetchContacts();
    } else if (response.data.status === 404) {
      error.value = '好友不存在';
    } else {
      error.value = response.data.message || '删除联系人失败';
    }
  } catch (err) {
    error.value = err.message;
  }
};
</script>

<style scoped>
.add-contact-form {
  display: flex;
  margin-bottom: 20px;
}

.add-contact-form input {
  flex: 1;
  padding: 8px 10px;
  border: 1px solid #ddd;
  border-radius: 4px 0 0 4px;
}

.add-contact-form button {
  padding: 8px 15px;
  background-color: #07C160;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
}

.error-message {
  color: #f44336;
  margin-bottom: 15px;
}
</style>