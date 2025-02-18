<template>
  <div>
    <div :style="{ height: '20px' }"></div>
    <a-form :model="form" class="form-container" @submit="handleSubmit">
      <a-form-item 
        v-if="enablePasswordCheck"
        field="password" 
        label="执行密码"
      >
        <a-input-password 
          v-model="form.password"
          placeholder="请输入执行密码"
          allow-clear
          @keypress.enter="handleSubmit"
        />
      </a-form-item>

      <a-form-item field="keyType" label="Key Type">
        <a-input v-model="form.keyType" placeholder="请输入Key Type..." />
      </a-form-item>
      <a-form-item field="uid" label="UID">
        <a-input v-model="form.uid" placeholder="请输入玩家UID..." />
      </a-form-item>
      <a-form-item field="command" label="Command">
        <a-input v-model="form.command" placeholder="请输入命令..." />
      </a-form-item>
      <a-form-item>
        <a-space>
          <a-button type="primary" shape="round" size="large" html-type="submit">提交</a-button>
          <a-button type="primary" shape="round" size="large" html-type="reset" @click="handleReset">重置</a-button>
        </a-space>
      </a-form-item>
      <a-form-item label="response">
        <a-input v-model="responseData" :disabled="true" />
      </a-form-item>
    </a-form>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref, computed } from 'vue';
import { Message } from '@arco-design/web-vue';
import axios, { AxiosError } from 'axios';

const env = import.meta.env;
const API_BASE_URL = env.VITE_DHWT_API_SERVER;
const enablePasswordCheck = computed(() => {
  const value = env.VITE_ENABLE_PASSWORD_CHECK;
  return value === 'true' || value === '1'; // 支持多种真值表示
});

if (import.meta.env.DEV) {
  console.log('Password Check Status:', enablePasswordCheck.value);
}

type AlertType = 'success' | 'info' | 'warning' | 'error';

const form = reactive({
  ...(enablePasswordCheck.value && { password: '' }), // 条件初始化
  keyType: 'PEM',
  uid: '',
  command: ''
});

const responseData = ref('');
const showMessage = ref(false);
const messageType = ref<AlertType>('info');
const message = ref('');

const handleReset = () => {
  localStorage.clear();
  if (enablePasswordCheck.value) {
    form.password = '';
  }
  form.keyType = "PEM";
  form.uid = "";
  form.command = "";
  responseData.value = "";
  showMessage.value = false;
  messageType.value = 'info';
  message.value = "";
};

const handleSubmit = async (data: { values: Record<string, any>; errors: Record<string, any> | undefined }, ev: Event) => {
  ev.preventDefault();

  try {
    if (enablePasswordCheck.value) {
      if (!form.password?.trim()) {
        Message.error('请输入执行密码');
        return;
      }
      const correctPassword = env.VITE_EXEC_PASSWORD || "dev_password";
      if (form.password !== correctPassword) {
        Message.error('密码错误，请检查后重试');
        return;
      }
    }

    const res = await axios.post(`${API_BASE_URL}/api/submit`, {
      keyType: form.keyType,
      uid: form.uid,
      command: form.command
    });

    const responseMessage = res.data.message;
    if (res.data.code === 0) {
      localStorage.setItem('uid', form.uid);
      console.log('UID stored:', localStorage.getItem('uid'));
    }

    responseData.value = JSON.stringify(res.data, null, 2);
    showMessage.value = true;
    messageType.value = 'success';
    message.value = `命令提交成功：${responseMessage}`;
    Message.success(`命令提交成功：${responseMessage}`);
  } catch (err: unknown) {
    let errorMessage = '请求失败';
    if (axios.isAxiosError(err)) {
      if (err.response) {
        errorMessage = err.response.status === 429 
          ? '请求速率过快，请稍后重试' 
          : (err.response.data as { error?: string }).error || err.response.statusText;
      } else {
        errorMessage = err.message;
      }
    }

    showMessage.value = true;
    messageType.value = 'error';
    message.value = errorMessage;
    Message.error(errorMessage);
    console.error(err);
  }
};
</script>

<style lang="less" scoped>
.form-container {
  margin-top: 20px;
  width: 600px;
  margin: auto;

  .arco-input-password {
    width: 100%;
    max-width: 300px;
    
    @media screen and (max-width: 768px) {
      max-width: 100%;
    }
  }
}

a-form-item {
  margin-bottom: 16px;
}

@media screen and (max-width: 768px) {
  .form-container {
    width: 100%;
    padding: 10px;
  }

  a-space {
    flex-direction: column;
    
    .arco-button {
      width: 100%;
      margin-top: 10px;
    }
  }
}
</style>
