<script setup>
import { defineAsyncComponent, onMounted, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRoute } from 'vue-router'

import { useGlobalState } from '../store'
import { api } from '../api'
import { useIsMobile } from '../utils/composables'
import { FullscreenExitOutlined } from '@vicons/material'

import AddressBar from './index/AddressBar.vue';
import MailBox from '../components/MailBox.vue';
import SendBox from '../components/SendBox.vue';
import AutoReply from './index/AutoReply.vue';
import AccountSettings from './index/AccountSettings.vue';
import Appearance from './common/Appearance.vue';
import Webhook from './index/Webhook.vue';
import Attachment from './index/Attachment.vue';
import About from './common/About.vue';
import SimpleIndex from './index/SimpleIndex.vue';

const { loading, settings, openSettings, indexTab, globalTabplacement, useSimpleIndex } = useGlobalState()
const message = useMessage()
const route = useRoute()
const isMobile = useIsMobile()

const SendMail = defineAsyncComponent(() => {
  loading.value = true;
  return import('./index/SendMail.vue')
    .finally(() => loading.value = false);
});

const { t } = useI18n({
  messages: {
    en: {
      mailbox: 'Mail Box',
      sendbox: 'Send Box',
      sendmail: 'Send Mail',
      auto_reply: 'Auto Reply',
      accountSettings: 'Account Settings',
      appearance: 'Appearance',
      about: 'About',
      s3Attachment: 'S3 Attachment',
      saveToS3Success: 'save to s3 success',
      webhookSettings: 'Webhook Settings',
      query: 'Query',
      enterSimpleMode: 'Simple Mode',
    },
    zh: {
      mailbox: '收件箱',
      sendbox: '发件箱',
      sendmail: '发送邮件',
      auto_reply: '自动回复',
      accountSettings: '账户',
      appearance: '外观',
      about: '关于',
      s3Attachment: 'S3附件',
      saveToS3Success: '保存到s3成功',
      webhookSettings: 'Webhook 设置',
      query: '查询',
      enterSimpleMode: '极简模式',
    }
  }
});

const fetchMailData = async (limit, offset) => {
  if (mailIdQuery.value > 0) {
    const singleMail = await api.fetch(`/api/mail/${mailIdQuery.value}`);
    if (singleMail) return { results: [singleMail], count: 1 };
    return { results: [], count: 0 };
  }
  return await api.fetch(`/api/mails?limit=${limit}&offset=${offset}`);
};

const deleteMail = async (curMailId) => {
  await api.fetch(`/api/mails/${curMailId}`, { method: 'DELETE' });
};

const deleteSenboxMail = async (curMailId) => {
  await api.fetch(`/api/sendbox/${curMailId}`, { method: 'DELETE' });
};

const fetchSenboxData = async (limit, offset) => {
  return await api.fetch(`/api/sendbox?limit=${limit}&offset=${offset}`);
};

const saveToS3 = async (mail_id, filename, blob) => {
  try {
    const { url } = await api.fetch(`/api/attachment/put_url`, {
      method: 'POST',
      body: JSON.stringify({ key: `${mail_id}/${filename}` })
    });
    // upload to s3 by formdata
    const formData = new FormData();
    formData.append(filename, blob);
    await fetch(url, {
      method: 'PUT',
      body: formData
    });
    message.success(t('saveToS3Success'));
  } catch (error) {
    console.error(error);
    message.error(error.message || "save to s3 error");
  }
}

const mailBoxKey = ref("")
const mailIdQuery = ref("")
const showMailIdQuery = ref(false)

const queryMail = () => {
  mailBoxKey.value = Date.now();
}

watch(route, () => {
  if (!route.query.mail_id) {
    showMailIdQuery.value = false;
    mailIdQuery.value = "";
    queryMail();
  }
})

onMounted(() => {
  if (route.query.mail_id) {
    showMailIdQuery.value = true;
    mailIdQuery.value = route.query.mail_id;
    queryMail();
  }
})
</script>

<template>
  <div class="index-container">
    <div v-if="useSimpleIndex" class="simple-index-wrapper">
      <SimpleIndex />
    </div>
    <div v-else class="main-content">
      <div class="address-bar-wrapper">
        <AddressBar />
      </div>
      <div v-if="settings.address" class="tabs-container">
        <n-tabs type="card" v-model:value="indexTab" :placement="globalTabplacement" class="main-tabs">
          <template #prefix v-if="!isMobile">
            <n-button @click="useSimpleIndex = true" tertiary size="small" class="simple-mode-btn">
              <template #icon>
                <n-icon>
                  <FullscreenExitOutlined />
                </n-icon>
              </template>
              {{ t('enterSimpleMode') }}
            </n-button>
          </template>
          <n-tab-pane name="mailbox" :tab="t('mailbox')">
            <div class="tab-content">
              <div v-if="showMailIdQuery" class="query-section">
                <n-input-group class="query-input-group">
                  <n-input v-model:value="mailIdQuery" placeholder="输入邮件ID进行查询..." />
                  <n-button @click="queryMail" type="primary" tertiary class="query-btn">
                    {{ t('query') }}
                  </n-button>
                </n-input-group>
              </div>
              <div class="mailbox-wrapper">
                <MailBox :key="mailBoxKey" :showEMailTo="false" :showReply="true" :showSaveS3="openSettings.isS3Enabled"
                  :saveToS3="saveToS3" :enableUserDeleteEmail="openSettings.enableUserDeleteEmail"
                  :fetchMailData="fetchMailData" :deleteMail="deleteMail" />
              </div>
            </div>
          </n-tab-pane>
          <n-tab-pane name="sendbox" :tab="t('sendbox')">
            <div class="tab-content">
              <SendBox :fetchMailData="fetchSenboxData" :enableUserDeleteEmail="openSettings.enableUserDeleteEmail"
                :deleteMail="deleteSenboxMail" />
            </div>
          </n-tab-pane>
          <n-tab-pane name="sendmail" :tab="t('sendmail')">
            <div class="tab-content">
              <SendMail />
            </div>
          </n-tab-pane>
          <n-tab-pane name="accountSettings" :tab="t('accountSettings')">
            <div class="tab-content">
              <AccountSettings />
            </div>
          </n-tab-pane>
          <n-tab-pane name="appearance" :tab="t('appearance')">
            <div class="tab-content">
              <Appearance :showUseSimpleIndex="true" />
            </div>
          </n-tab-pane>
          <n-tab-pane v-if="openSettings.enableAutoReply" name="auto_reply" :tab="t('auto_reply')">
            <div class="tab-content">
              <AutoReply />
            </div>
          </n-tab-pane>
          <n-tab-pane v-if="openSettings.enableWebhook" name="webhook" :tab="t('webhookSettings')">
            <div class="tab-content">
              <Webhook />
            </div>
          </n-tab-pane>
          <n-tab-pane v-if="openSettings.isS3Enabled" name="s3_attachment" :tab="t('s3Attachment')">
            <div class="tab-content">
              <Attachment />
            </div>
          </n-tab-pane>
          <n-tab-pane v-if="openSettings.enableIndexAbout" name="about" :tab="t('about')">
            <div class="tab-content">
              <About />
            </div>
          </n-tab-pane>
        </n-tabs>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 主容器样式 */
.index-container {
  margin-top: 120px; /* 为悬浮菜单留出空间 */
  padding: 0 20px;
  max-width: 1200px;
  margin-left: auto;
  margin-right: auto;
  min-height: calc(100vh - 100px);
}

/* 极简模式包装器 */
.simple-index-wrapper {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 24px;
  box-shadow: 
    0 8px 32px rgba(0, 0, 0, 0.1),
    0 2px 8px rgba(0, 0, 0, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
}

.simple-index-wrapper:hover {
  box-shadow: 
    0 12px 40px rgba(0, 0, 0, 0.15),
    0 4px 12px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

/* 主内容区域 */
.main-content {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* 地址栏包装器 */
.address-bar-wrapper {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  padding: 20px;
  box-shadow: 
    0 4px 20px rgba(0, 0, 0, 0.08),
    0 1px 4px rgba(0, 0, 0, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
}

.address-bar-wrapper:hover {
  box-shadow: 
    0 6px 24px rgba(0, 0, 0, 0.12),
    0 2px 6px rgba(0, 0, 0, 0.06);
  transform: translateY(-1px);
}

/* 标签页容器 */
.tabs-container {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 24px;
  box-shadow: 
    0 8px 32px rgba(0, 0, 0, 0.1),
    0 2px 8px rgba(0, 0, 0, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
  min-height: 500px;
}

.tabs-container:hover {
  box-shadow: 
    0 12px 40px rgba(0, 0, 0, 0.15),
    0 4px 12px rgba(0, 0, 0, 0.08);
}

/* 标签页内容 */
.tab-content {
  padding: 16px 0;
  animation: fadeInUp 0.3s ease-out;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 查询区域 */
.query-section {
  margin-bottom: 20px;
  padding: 16px;
  background: rgba(24, 160, 88, 0.05);
  border-radius: 12px;
  border: 1px solid rgba(24, 160, 88, 0.1);
}

.query-input-group {
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.query-btn {
  border-radius: 0 10px 10px 0 !important;
  transition: all 0.3s ease;
}

.query-btn:hover {
  background: rgba(24, 160, 88, 0.1) !important;
  transform: translateX(-2px);
}

/* 邮箱包装器 */
.mailbox-wrapper {
  border-radius: 12px;
  overflow: hidden;
}

/* 极简模式按钮 */
.simple-mode-btn {
  border-radius: 10px !important;
  transition: all 0.3s ease !important;
  background: rgba(255, 255, 255, 0.1) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
}

.simple-mode-btn:hover {
  background: rgba(24, 160, 88, 0.1) !important;
  border-color: rgba(24, 160, 88, 0.2) !important;
  transform: translateY(-2px) !important;
  box-shadow: 0 4px 12px rgba(24, 160, 88, 0.2) !important;
}

/* 标签页样式优化 */
:deep(.n-tabs) {
  --n-tab-border-radius: 12px;
}

:deep(.n-tab-pane) {
  border-radius: 12px;
}

:deep(.n-tabs .n-tabs-nav) {
  background: rgba(255, 255, 255, 0.1) !important;
  border-radius: 12px !important;
  padding: 4px !important;
  margin-bottom: 16px !important;
}

:deep(.n-tabs .n-tabs-tab) {
  border-radius: 10px !important;
  margin: 0 2px !important;
  transition: all 0.3s ease !important;
}

:deep(.n-tabs .n-tabs-tab:hover) {
  background: rgba(24, 160, 88, 0.1) !important;
  transform: translateY(-1px) !important;
}

:deep(.n-tabs .n-tabs-tab.n-tabs-tab--active) {
  background: rgba(24, 160, 88, 0.15) !important;
  box-shadow: 0 2px 8px rgba(24, 160, 88, 0.2) !important;
}

/* 暗色模式适配 */
html.dark .simple-index-wrapper,
html.dark .address-bar-wrapper,
html.dark .tabs-container {
  background: rgba(16, 16, 20, 0.8) !important;
  border-color: rgba(255, 255, 255, 0.1) !important;
  box-shadow: 
    0 8px 32px rgba(0, 0, 0, 0.3),
    0 2px 8px rgba(0, 0, 0, 0.2) !important;
}

html.dark .simple-index-wrapper:hover,
html.dark .address-bar-wrapper:hover,
html.dark .tabs-container:hover {
  box-shadow: 
    0 12px 40px rgba(0, 0, 0, 0.4),
    0 4px 12px rgba(0, 0, 0, 0.3) !important;
}

html.dark .query-section {
  background: rgba(24, 160, 88, 0.1) !important;
  border-color: rgba(24, 160, 88, 0.2) !important;
}

html.dark .simple-mode-btn {
  background: rgba(255, 255, 255, 0.05) !important;
  border-color: rgba(255, 255, 255, 0.1) !important;
}

html.dark .simple-mode-btn:hover {
  background: rgba(24, 160, 88, 0.15) !important;
  border-color: rgba(24, 160, 88, 0.3) !important;
}

html.dark :deep(.n-tabs .n-tabs-nav) {
  background: rgba(255, 255, 255, 0.05) !important;
}

html.dark :deep(.n-tabs .n-tabs-tab:hover) {
  background: rgba(24, 160, 88, 0.15) !important;
}

html.dark :deep(.n-tabs .n-tabs-tab.n-tabs-tab--active) {
  background: rgba(24, 160, 88, 0.2) !important;
}

/* 响应式设计 */
@media (max-width: 810px) {
  .index-container {
    margin-top: 110px;
    padding: 0 16px;
  }
  
  .simple-index-wrapper,
  .address-bar-wrapper,
  .tabs-container {
    border-radius: 16px;
    padding: 16px;
  }
  
  .main-content {
    gap: 16px;
  }
  
  .tab-content {
    padding: 12px 0;
  }
}

@media (max-width: 480px) {
  .index-container {
    margin-top: 100px;
    padding: 0 12px;
  }
  
  .simple-index-wrapper,
  .address-bar-wrapper,
  .tabs-container {
    border-radius: 12px;
    padding: 12px;
  }
  
  .main-content {
    gap: 12px;
  }
  
  .query-section {
    padding: 12px;
    margin-bottom: 16px;
  }
}

/* 滚动条美化 */
:deep(*::-webkit-scrollbar) {
  width: 6px;
  height: 6px;
}

:deep(*::-webkit-scrollbar-track) {
  background: rgba(0, 0, 0, 0.05);
  border-radius: 3px;
}

:deep(*::-webkit-scrollbar-thumb) {
  background: rgba(24, 160, 88, 0.3);
  border-radius: 3px;
  transition: background 0.3s ease;
}

:deep(*::-webkit-scrollbar-thumb:hover) {
  background: rgba(24, 160, 88, 0.5);
}

html.dark :deep(*::-webkit-scrollbar-track) {
  background: rgba(255, 255, 255, 0.05);
}

html.dark :deep(*::-webkit-scrollbar-thumb) {
  background: rgba(24, 160, 88, 0.4);
}

html.dark :deep(*::-webkit-scrollbar-thumb:hover) {
  background: rgba(24, 160, 88, 0.6);
}
</style>
