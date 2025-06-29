<template>
  <div class="message-board">
    <div class="header">
      <h2>留言板</h2>
      <p class="subtitle">在这里分享你的想法和建议</p>
    </div>
    
    <!-- 添加留言表单 -->
    <div class="add-message-form">
      <div class="form-group">
        <input 
          v-model="newMessage.author" 
          type="text" 
          placeholder="请输入你的昵称"
          class="input-field"
        />
      </div>
      <div class="form-group">
        <textarea 
          v-model="newMessage.content" 
          placeholder="请输入留言内容..."
          class="textarea-field"
          rows="3"
        ></textarea>
      </div>
      <button @click="addMessage" class="submit-btn" :disabled="!canSubmit">
        发布留言
      </button>
    </div>

    <!-- 留言列表 -->
    <div class="messages-list">
      <div class="messages-header">
        <h3>留言列表 ({{ messages.length }})</h3>
      </div>
      
      <div v-if="messages.length === 0" class="empty-state">
        <div class="empty-icon">💬</div>
        <p>暂无留言，快来发布第一条留言吧！</p>
      </div>
      
      <div v-else class="message-items">
        <div 
          v-for="(message, index) in messages" 
          :key="message.id"
          class="message-item"
        >
          <div class="message-header">
            <span class="author">{{ message.author }}</span>
            <span class="time">{{ formatTime(message.timestamp) }}</span>
            <button @click="deleteMessage(index)" class="delete-btn">删除</button>
          </div>
          <div class="message-content">
            {{ message.content }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup name="StartManagerTest">
import { ref, computed, onMounted } from 'vue'

// 响应式数据
const messages = ref([])
const newMessage = ref({
  author: '',
  content: ''
})

// 计算属性
const canSubmit = computed(() => {
  return newMessage.value.author.trim() && newMessage.value.content.trim()
})

// 方法
const addMessage = () => {
  if (!canSubmit.value) return
  
  const message = {
    id: Date.now(),
    author: newMessage.value.author.trim(),
    content: newMessage.value.content.trim(),
    timestamp: new Date()
  }
  
  messages.value.unshift(message)
  
  // 清空表单
  newMessage.value.author = ''
  newMessage.value.content = ''
  
  // 保存到本地存储
  saveMessages()
}

const deleteMessage = (index) => {
  messages.value.splice(index, 1)
  saveMessages()
}



const formatTime = (timestamp) => {
  const date = new Date(timestamp)
  return date.toLocaleString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  })
}

const saveMessages = () => {
  localStorage.setItem('commandplayground-messages', JSON.stringify(messages.value))
}

const loadMessages = () => {
  const saved = localStorage.getItem('commandplayground-messages')
  if (saved) {
    try {
      messages.value = JSON.parse(saved)
    } catch (e) {
      console.error('Failed to load messages:', e)
    }
  }
}

// 生命周期
onMounted(() => {
  loadMessages()
})
</script>

<style lang="less" scoped>
.message-board {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.header {
  text-align: center;
  margin-bottom: 30px;
  
  h2 {
    color: #2c3e50;
    margin: 0 0 8px 0;
    font-size: 28px;
    font-weight: 600;
  }
  
  .subtitle {
    color: #7f8c8d;
    margin: 0;
    font-size: 14px;
  }
}

.add-message-form {
  background: #f8f9fa;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 30px;
  border: 1px solid #e9ecef;
  
  .form-group {
    margin-bottom: 16px;
  }
  
  .input-field {
    width: 100%;
    padding: 12px 16px;
    border: 1px solid #ddd;
    border-radius: 8px;
    font-size: 14px;
    transition: border-color 0.3s;
    box-sizing: border-box;
    
    &:focus {
      outline: none;
      border-color: #007bff;
      box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.1);
    }
  }
  
  .textarea-field {
    width: 100%;
    padding: 12px 16px;
    border: 1px solid #ddd;
    border-radius: 8px;
    font-size: 14px;
    resize: vertical;
    min-height: 80px;
    font-family: inherit;
    transition: border-color 0.3s;
    box-sizing: border-box;
    
    &:focus {
      outline: none;
      border-color: #007bff;
      box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.1);
    }
  }
  
  .submit-btn {
    background: #007bff;
    color: white;
    border: none;
    padding: 12px 24px;
    border-radius: 8px;
    font-size: 14px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.3s;
    
    &:hover:not(:disabled) {
      background: #0056b3;
      transform: translateY(-1px);
    }
    
    &:disabled {
      background: #6c757d;
      cursor: not-allowed;
      transform: none;
    }
  }
}

.messages-list {
  .messages-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    
    h3 {
      color: #2c3e50;
      margin: 0;
      font-size: 20px;
      font-weight: 600;
    }
    

  }
  
  .empty-state {
    text-align: center;
    padding: 60px 20px;
    color: #6c757d;
    
    .empty-icon {
      font-size: 48px;
      margin-bottom: 16px;
    }
    
    p {
      margin: 0;
      font-size: 16px;
    }
  }
  
  .message-items {
    .message-item {
      background: white;
      border: 1px solid #e9ecef;
      border-radius: 12px;
      padding: 20px;
      margin-bottom: 16px;
      transition: all 0.3s;
      
      &:hover {
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        transform: translateY(-2px);
      }
      
      .message-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 12px;
        
        .author {
          font-weight: 600;
          color: #007bff;
          font-size: 14px;
        }
        
        .time {
          color: #6c757d;
          font-size: 12px;
        }
        
        .delete-btn {
          background: #dc3545;
          color: white;
          border: none;
          padding: 4px 8px;
          border-radius: 4px;
          font-size: 11px;
          cursor: pointer;
          transition: background 0.3s;
          
          &:hover {
            background: #c82333;
          }
        }
      }
      
      .message-content {
        color: #2c3e50;
        line-height: 1.6;
        font-size: 14px;
        white-space: pre-wrap;
        word-break: break-word;
      }
    }
  }
}

@media (max-width: 768px) {
  .message-board {
    padding: 16px;
  }
  
  .add-message-form {
    padding: 20px;
  }
  
  .messages-header {
    flex-direction: column;
    gap: 12px;
    align-items: flex-start !important;
  }
  
  .message-item .message-header {
    flex-direction: column;
    align-items: flex-start !important;
    gap: 8px;
  }
}
</style>