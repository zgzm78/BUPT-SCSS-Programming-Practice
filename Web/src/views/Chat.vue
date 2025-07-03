<template>
  <div class="main-layout">
    <Sidebar />
    <div class="content-area">
      <h2>聊天</h2>
      <div class="chat-messages">
        <div
          v-for="(message, index) in chatMessages"
          :key="index"
          :class="{ 'self-message': message.isSelf, 'other-message': !message.isSelf }"
        >
          <div class="message-content">
            <!-- 判断消息类型 -->
            <span v-if="message.message.type === 'text'">{{ message.message.content }}</span>
            <!-- 图片消息 -->
            <img
              v-if="message.message.type === 'picture'"
              :src="message.message.content"
              alt="图片"
              @error="handleImageError(message)"
              @click="openImageModal(message.message.content)"
              class="chat-image"
            />
          </div>
          <div class="message-sender">
            {{ message.isSelf ? '我' : message.senderUsername }}
          </div>
        </div>
      </div>
      <div class="chat-input">
        <input
          type="text"
          v-model="newMessage"
          placeholder="输入消息..."
          @keyup.enter="sendMessage"
        />
        <input type="file" @change="handleFileSelect" accept="image/*" />
        <button @click="sendMessage">发送</button>
      </div>
      <!-- 图片模态框 -->
      <div v-if="isImageModalOpen" class="image-modal" @click="closeImageModal">
        <div class="image-modal-content" @click.stop>
          <img
            :src="currentImage"
            alt="大图"
            class="modal-image"
            :style="{ transform: `scale(${scale})` }"
            @wheel="handleMouseWheel"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, onBeforeUnmount } from 'vue';
import Sidebar from '@/components/Sidebar.vue';
import { sendMessage as apiSendMessage, getChatHistory } from '@/services/api';
import { useRoute } from 'vue-router';

const route = useRoute();
const chatMessages = ref([]);
const newMessage = ref('');
const selectedFile = ref(null);
const isImageModalOpen = ref(false);
const currentImage = ref('');
const scale = ref(1); // 初始缩放比例
const minScale = ref(0.2); // 最小缩放比例
const maxScale = ref(3); // 最大缩放比例

const fetchChatHistory = async (contactUsername) => {
  try {
    const response = await getChatHistory(contactUsername);
    if (response.data.status === 200) {
      const messages = response.data.data.messages.map((message) => {
        const isSelf = message.sender === 'your_username'; // 这里替换为当前用户的用户名
        return {
          isSelf,
          timestamp: message.timestamp,
          message: message.message,
          senderUsername: message.sender,
          receiverUsername: message.receiver
        };
      });
      // 根据时间戳排序
      messages.sort((a, b) => a.timestamp - b.timestamp);
      chatMessages.value = messages;
    } else {
      console.error('获取聊天记录失败:', response.data.message);
    }
  } catch (error) {
    console.error('获取聊天记录失败:', error);
  }
};

onMounted(() => {
  if (route.query.contactUsername) {
    fetchChatHistory(route.query.contactUsername);
  }
  window.addEventListener('keydown', handleKeyDown);
});

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeyDown);
});

watch(() => route.query.contactUsername, (newContactUsername) => {
  if (newContactUsername) {
    fetchChatHistory(newContactUsername);
  }
});

const handleFileSelect = (event) => {
  selectedFile.value = event.target.files[0];
};

const sendMessage = async () => {
  if (!newMessage.value.trim() && !selectedFile.value) return;

  try {
    let messageData;
    if (selectedFile.value) {
      const reader = new FileReader();
      reader.readAsDataURL(selectedFile.value);
      await new Promise((resolve) => {
        reader.onload = () => {
          messageData = {
            data: {
              friend_id: route.query.contactUsername,
              message: {
                type: 'picture',
                content: reader.result
              }
            }
          };
          resolve();
        };
      });
    } else {
      messageData = {
        data: {
          friend_id: route.query.contactUsername,
          message: {
            type: 'text',
            content: newMessage.value
          }
        }
      };
    }

    const response = await apiSendMessage(messageData);

    if (response.data.status === 200) {
      chatMessages.value.push({
        isSelf: true,
        timestamp: messageData.data.timestamp,
        message: messageData.data.message,
        senderUsername: messageData.data.senderUsername,
        receiverUsername: messageData.data.receiverUsername
      });

      newMessage.value = '';
      selectedFile.value = null;
    } else if (response.data.status === 404) {
      console.error('发送失败:', response.data.message);
    } else {
      console.error('发送失败:', response.data.message);
    }
  } catch (error) {
    console.error('API调用失败:', error);
  }
};

const handleImageError = (message) => {
  console.error('图片加载失败:', message);
  // 可以在这里添加更多处理逻辑，如显示默认图片
};

const openImageModal = (imageSrc) => {
  currentImage.value = imageSrc;
  scale.value = 1; // 打开模态框时重置缩放比例
  isImageModalOpen.value = true;
};

const closeImageModal = () => {
  isImageModalOpen.value = false;
};

const handleMouseWheel = (event) => {
  event.preventDefault();
  const delta = event.deltaY > 0 ? -0.1 : 0.1;
  const newScale = scale.value + delta;
  if (newScale >= minScale.value && newScale <= maxScale.value) {
    scale.value = newScale;
  }
};

const handleKeyDown = (event) => {
  if (isImageModalOpen.value) {
    if (event.key === '+') {
      const newScale = scale.value + 0.1;
      if (newScale <= maxScale.value) {
        scale.value = newScale;
      }
    } else if (event.key === '-') {
      const newScale = scale.value - 0.1;
      if (newScale >= minScale.value) {
        scale.value = newScale;
      }
    }
  }
};
</script>

<style scoped>
.main-layout {
  display: flex;
  height: 100vh;
}

.content-area {
  flex: 1;
  padding: 20px;
  display: flex;
  flex-direction: column;
  background-color: rgba(255, 255, 255, 0.44);
  border-radius: 8px;
  margin: 20px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: white;
  margin-bottom: 10px;
}

.self-message,
.other-message {
  margin-bottom: 10px;
}

.self-message {
  text-align: right;
}

.message-content {
  display: inline-block;
  padding: 8px 12px;
  border-radius: 4px;
  max-width: 70%;
}

.self-message .message-content {
  background-color: #07C160;
  color: white;
}

.other-message .message-content {
  background-color: #e0e0e0;
}

.message-sender {
  font-size: 12px;
  color: #888;
  margin-top: 3px;
}

.chat-input {
  display: flex;
}

.chat-input input {
  flex: 1;
  padding: 8px 10px;
  border: 1px solid #ddd;
  border-radius: 4px 0 0 4px;
}

.chat-input button {
  padding: 8px 15px;
  background-color: #07C160;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.chat-input button:hover {
  background-color: #06a453;
}

.chat-image {
  width: 150px;
  height: 150px;
  object-fit: cover;
  cursor: pointer;
}

.image-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.image-modal-content {
  position: relative;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: white;
  padding: 20px;
  border-radius: 8px;
}

.modal-image {
  max-width: 90%;
  max-height: 90%;
  background-color: transparent;
  transition: transform 0.3s ease;
}
</style>