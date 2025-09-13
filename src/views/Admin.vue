<script setup>
import { computed, onMounted, ref } from 'vue';
import { useI18n } from 'vue-i18n'

import { useGlobalState } from '../store'
import { api } from '../api'

import SenderAccess from './admin/SenderAccess.vue'
import Statistics from "./admin/Statistics.vue"
import SendBox from './admin/SendBox.vue';
import Account from './admin/Account.vue';
import CreateAccount from './admin/CreateAccount.vue';
import AccountSettings from './admin/AccountSettings.vue';
import UserManagement from './admin/UserManagement.vue';
import UserSettings from './admin/UserSettings.vue';
import UserOauth2Settings from './admin/UserOauth2Settings.vue';
import Mails from './admin/Mails.vue';
import MailsUnknow from './admin/MailsUnknow.vue';
import About from './common/About.vue';
import Maintenance from './admin/Maintenance.vue';
import DatabaseManager from './admin/DatabaseManager.vue';
import Appearance from './common/Appearance.vue';
import Telegram from './admin/Telegram.vue';
import Webhook from './admin/Webhook.vue';
import MailWebhook from './admin/MailWebhook.vue';
import WorkerConfig from './admin/WorkerConfig.vue';

const {
  adminAuth, showAdminAuth, adminTab, loading,
  globalTabplacement, showAdminPage, userSettings
} = useGlobalState()
const message = useMessage()

const SendMail = defineAsyncComponent(() => {
  loading.value = true;
  return import('./admin/SendMail.vue')
    .finally(() => loading.value = false);
});

const authFunc = async () => {
  try {
    adminAuth.value = tmpAdminAuth.value;
    location.reload()
  } catch (error) {
    message.error(error.message || "error");
  }
}

const { t } = useI18n({
  messages: {
    en: {
      accessHeader: 'Admin Password',
      accessTip: 'Please enter the admin password',
      mails: 'Emails',
      sendMail: 'Send Mail',
      qucickSetup: 'Quick Setup',
      account: 'Account',
      account_create: 'Create Account',
      account_settings: 'Account Settings',
      user: 'User',
      user_management: 'User Management',
      user_settings: 'User Settings',
      userOauth2Settings: 'Oauth2 Settings',
      unknow: 'Mails with unknow receiver',
      senderAccess: 'Sender Access Control',
      sendBox: 'Send Box',
      telegram: 'Telegram Bot',
      webhookSettings: 'Webhook Settings',
      statistics: 'Statistics',
      maintenance: 'Maintenance',
      database: 'Database',
      workerconfig: 'Worker Config',
      appearance: 'Appearance',
      about: 'About',
      ok: 'OK',
      mailWebhook: 'Mail Webhook',
    },
    zh: {
      accessHeader: 'Admin 密码',
      accessTip: '请输入 Admin 密码',
      mails: '邮件',
      sendMail: '发送邮件',
      qucickSetup: '快速设置',
      account: '账号',
      account_create: '创建账号',
      account_settings: '账号设置',
      user: '用户',
      user_management: '用户管理',
      user_settings: '用户设置',
      userOauth2Settings: 'Oauth2 设置',
      unknow: '无收件人邮件',
      senderAccess: '发件权限控制',
      sendBox: '发件箱',
      telegram: '电报机器人',
      webhookSettings: 'Webhook 设置',
      statistics: '统计',
      maintenance: '维护',
      database: '数据库',
      workerconfig: 'Worker 配置',
      appearance: '外观',
      about: '关于',
      ok: '确定',
      mailWebhook: '邮件 Webhook',
    }
  }
});

const showAdminPasswordModal = computed(() => !showAdminPage.value || showAdminAuth.value)
const tmpAdminAuth = ref('')

// 弹窗样式配置
const authModalStyle = {
  width: '90%',
  maxWidth: '450px',
  minWidth: '320px',
}

const authModalContentStyle = {
  background: 'rgba(255, 255, 255, 0.25)',
  backdropFilter: 'blur(20px)',
  borderRadius: '20px',
  border: '1px solid rgba(255, 255, 255, 0.3)',
  boxShadow: '0 8px 32px rgba(0, 0, 0, 0.15), 0 2px 8px rgba(0, 0, 0, 0.1)',
  padding: '0',
}

const authModalHeaderStyle = {
  background: 'transparent',
  borderRadius: '20px 20px 0 0',
  borderBottom: '1px solid rgba(255, 255, 255, 0.2)',
  padding: '20px 24px 16px',
  margin: '0',
}

onMounted(async () => {
  // make sure user_id is fetched
  if (!userSettings.value.user_id) await api.getUserSettings(message);
})
</script>

<template>
  <div class="admin-container" v-if="userSettings.fetched">
    <!-- 管理员密码弹窗 -->
    <n-modal 
      v-model:show="showAdminPasswordModal" 
      :closable="false" 
      :closeOnEsc="false" 
      :maskClosable="false"
      preset="dialog" 
      :title="t('accessHeader')"
      class="admin-auth-modal"
      :style="authModalStyle"
      :content-style="authModalContentStyle"
      :header-style="authModalHeaderStyle"
    >
      <div class="auth-content">
        <p class="auth-tip">{{ t('accessTip') }}</p>
        <n-input 
          v-model:value="tmpAdminAuth" 
          type="password" 
          show-password-on="click" 
          class="auth-input"
          placeholder="请输入管理员密码..."
        />
      </div>
      <template #action>
        <n-button @click="authFunc" type="primary" :loading="loading" class="auth-button">
          {{ t('ok') }}
        </n-button>
      </template>
    </n-modal>

    <!-- 管理员主界面 -->
    <div v-if="showAdminPage" class="admin-tabs-container">
      <n-tabs type="card" v-model:value="adminTab" :placement="globalTabplacement" class="admin-main-tabs">
        <n-tab-pane name="qucickSetup" :tab="t('qucickSetup')">
          <div class="tab-content">
            <div class="sub-tabs-wrapper">
              <n-tabs type="bar" justify-content="center" animated class="admin-sub-tabs">
                <n-tab-pane name="database" :tab="t('database')">
                  <div class="sub-tab-content">
                    <DatabaseManager />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="account_settings" :tab="t('account_settings')">
                  <div class="sub-tab-content">
                    <AccountSettings />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="user_settings" :tab="t('user_settings')">
                  <div class="sub-tab-content">
                    <UserSettings />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="workerconfig" :tab="t('workerconfig')">
                  <div class="sub-tab-content">
                    <WorkerConfig />
                  </div>
                </n-tab-pane>
              </n-tabs>
            </div>
          </div>
        </n-tab-pane>
        <n-tab-pane name="account" :tab="t('account')">
          <div class="tab-content">
            <div class="sub-tabs-wrapper">
              <n-tabs type="bar" justify-content="center" animated class="admin-sub-tabs">
                <n-tab-pane name="account" :tab="t('account')">
                  <div class="sub-tab-content">
                    <Account />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="account_create" :tab="t('account_create')">
                  <div class="sub-tab-content">
                    <CreateAccount />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="account_settings" :tab="t('account_settings')">
                  <div class="sub-tab-content">
                    <AccountSettings />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="senderAccess" :tab="t('senderAccess')">
                  <div class="sub-tab-content">
                    <SenderAccess />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="webhook" :tab="t('webhookSettings')">
                  <div class="sub-tab-content">
                    <Webhook />
                  </div>
                </n-tab-pane>
              </n-tabs>
            </div>
          </div>
        </n-tab-pane>
        <n-tab-pane name="user" :tab="t('user')">
          <div class="tab-content">
            <div class="sub-tabs-wrapper">
              <n-tabs type="bar" justify-content="center" animated class="admin-sub-tabs">
                <n-tab-pane name="user_management" :tab="t('user_management')">
                  <div class="sub-tab-content">
                    <UserManagement />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="user_settings" :tab="t('user_settings')">
                  <div class="sub-tab-content">
                    <UserSettings />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="userOauth2Settings" :tab="t('userOauth2Settings')">
                  <div class="sub-tab-content">
                    <UserOauth2Settings />
                  </div>
                </n-tab-pane>
              </n-tabs>
            </div>
          </div>
        </n-tab-pane>
        <n-tab-pane name="mails" :tab="t('mails')">
          <div class="tab-content">
            <div class="sub-tabs-wrapper">
              <n-tabs type="bar" justify-content="center" animated class="admin-sub-tabs">
                <n-tab-pane name="mails" :tab="t('mails')">
                  <div class="sub-tab-content">
                    <Mails />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="unknow" :tab="t('unknow')">
                  <div class="sub-tab-content">
                    <MailsUnknow />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="sendBox" :tab="t('sendBox')">
                  <div class="sub-tab-content">
                    <SendBox />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="sendMail" :tab="t('sendMail')">
                  <div class="sub-tab-content">
                    <SendMail />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="mailWebhook" :tab="t('mailWebhook')">
                  <div class="sub-tab-content">
                    <MailWebhook />
                  </div>
                </n-tab-pane>
              </n-tabs>
            </div>
          </div>
        </n-tab-pane>
        <n-tab-pane name="telegram" :tab="t('telegram')">
          <div class="tab-content">
            <Telegram />
          </div>
        </n-tab-pane>
        <n-tab-pane name="statistics" :tab="t('statistics')">
          <div class="tab-content">
            <Statistics />
          </div>
        </n-tab-pane>
        <n-tab-pane name="maintenance" :tab="t('maintenance')">
          <div class="tab-content">
            <div class="sub-tabs-wrapper">
              <n-tabs type="bar" justify-content="center" animated class="admin-sub-tabs">
                <n-tab-pane name="database" :tab="t('database')">
                  <div class="sub-tab-content">
                    <DatabaseManager />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="workerconfig" :tab="t('workerconfig')">
                  <div class="sub-tab-content">
                    <WorkerConfig />
                  </div>
                </n-tab-pane>
                <n-tab-pane name="maintenance" :tab="t('maintenance')">
                  <div class="sub-tab-content">
                    <Maintenance />
                  </div>
                </n-tab-pane>
              </n-tabs>
            </div>
          </div>
        </n-tab-pane>
        <n-tab-pane name="appearance" :tab="t('appearance')">
          <div class="tab-content">
            <Appearance />
          </div>
        </n-tab-pane>
        <n-tab-pane name="about" :tab="t('about')">
          <div class="tab-content">
            <About />
          </div>
        </n-tab-pane>
      </n-tabs>
    </div>
  </div>
</template>

<style scoped>
/* 主容器样式 */
.admin-container {
  margin-top: 120px; /* 为悬浮菜单留出空间 */
  padding: 0 20px;
  max-width: 1200px;
  margin-left: auto;
  margin-right: auto;
  min-height: calc(100vh - 100px);
}

/* 管理员标签页容器 */
.admin-tabs-container {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 24px;
  box-shadow: 
    0 8px 32px rgba(0, 0, 0, 0.1),
    0 2px 8px rgba(0, 0, 0, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
  min-height: 600px;
}

.admin-tabs-container:hover {
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

/* 子标签页包装器 */
.sub-tabs-wrapper {
  background: rgba(255, 255, 255, 0.4);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  padding: 20px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
}

.sub-tabs-wrapper:hover {
  background: rgba(255, 255, 255, 0.5);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.08);
  transform: translateY(-1px);
}

/* 子标签页内容 */
.sub-tab-content {
  padding: 16px 0;
  animation: fadeInUp 0.3s ease-out;
}

/* 认证弹窗内容 */
.auth-content {
  padding: 20px 24px 24px;
  background: transparent;
}

.auth-tip {
  margin: 0 0 20px 0;
  color: rgba(0, 0, 0, 0.8);
  font-size: 14px;
  line-height: 1.6;
  text-align: center;
}

.auth-input {
  border-radius: 12px !important;
  background: rgba(255, 255, 255, 0.3) !important;
  border: 1px solid rgba(255, 255, 255, 0.3) !important;
  backdrop-filter: blur(10px) !important;
}

.auth-button {
  border-radius: 12px !important;
  padding: 8px 24px !important;
  font-weight: 600 !important;
  transition: all 0.3s ease !important;
}

.auth-button:hover {
  transform: translateY(-2px) !important;
  box-shadow: 0 4px 12px rgba(24, 160, 88, 0.3) !important;
}

/* 主标签页样式优化 */
:deep(.admin-main-tabs) {
  --n-tab-border-radius: 12px;
}

:deep(.admin-main-tabs .n-tabs-nav) {
  background: rgba(255, 255, 255, 0.1) !important;
  border-radius: 12px !important;
  padding: 4px !important;
  margin-bottom: 16px !important;
}

:deep(.admin-main-tabs .n-tabs-tab) {
  border-radius: 10px !important;
  margin: 0 2px !important;
  transition: all 0.3s ease !important;
}

:deep(.admin-main-tabs .n-tabs-tab:hover) {
  background: rgba(24, 160, 88, 0.1) !important;
  transform: translateY(-1px) !important;
}

:deep(.admin-main-tabs .n-tabs-tab.n-tabs-tab--active) {
  background: rgba(24, 160, 88, 0.15) !important;
  box-shadow: 0 2px 8px rgba(24, 160, 88, 0.2) !important;
}

/* 子标签页样式优化 */
:deep(.admin-sub-tabs .n-tabs-nav) {
  background: rgba(255, 255, 255, 0.2) !important;
  border-radius: 10px !important;
  padding: 4px !important;
  margin-bottom: 12px !important;
}

:deep(.admin-sub-tabs .n-tabs-tab) {
  border-radius: 8px !important;
  margin: 0 1px !important;
  transition: all 0.3s ease !important;
  font-size: 13px !important;
}

:deep(.admin-sub-tabs .n-tabs-tab:hover) {
  background: rgba(24, 160, 88, 0.1) !important;
  transform: translateY(-1px) !important;
}

:deep(.admin-sub-tabs .n-tabs-tab.n-tabs-tab--active) {
  background: rgba(24, 160, 88, 0.2) !important;
  box-shadow: 0 2px 6px rgba(24, 160, 88, 0.2) !important;
}

/* 分页样式 */
.n-pagination {
  margin-top: 16px;
  margin-bottom: 16px;
}

:deep(.n-pagination .n-pagination-item) {
  border-radius: 8px !important;
  transition: all 0.3s ease !important;
}

:deep(.n-pagination .n-pagination-item:hover) {
  transform: translateY(-1px) !important;
}

/* 弹窗遮罩层美化 */
:deep(.admin-auth-modal .n-modal-mask) {
  background: rgba(0, 0, 0, 0.4) !important;
  backdrop-filter: blur(4px) !important;
}

/* 弹窗动画 */
:deep(.admin-auth-modal .n-modal-container) {
  animation: modalFadeIn 0.3s cubic-bezier(0.4, 0, 0.2, 1) !important;
}

@keyframes modalFadeIn {
  from {
    opacity: 0;
    transform: scale(0.9) translateY(-20px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

/* 暗色模式适配 */
html.dark .admin-tabs-container {
  background: rgba(16, 16, 20, 0.8) !important;
  border-color: rgba(255, 255, 255, 0.1) !important;
  box-shadow: 
    0 8px 32px rgba(0, 0, 0, 0.3),
    0 2px 8px rgba(0, 0, 0, 0.2) !important;
}

html.dark .admin-tabs-container:hover {
  box-shadow: 
    0 12px 40px rgba(0, 0, 0, 0.4),
    0 4px 12px rgba(0, 0, 0, 0.3) !important;
}

html.dark .sub-tabs-wrapper {
  background: rgba(0, 0, 0, 0.3) !important;
  border-color: rgba(255, 255, 255, 0.1) !important;
}

html.dark .sub-tabs-wrapper:hover {
  background: rgba(0, 0, 0, 0.4) !important;
}

html.dark .auth-tip {
  color: rgba(255, 255, 255, 0.8);
}

html.dark .auth-input {
  background: rgba(0, 0, 0, 0.2) !important;
  border-color: rgba(255, 255, 255, 0.2) !important;
  color: rgba(255, 255, 255, 0.9) !important;
}

html.dark :deep(.admin-main-tabs .n-tabs-nav) {
  background: rgba(255, 255, 255, 0.05) !important;
}

html.dark :deep(.admin-main-tabs .n-tabs-tab:hover) {
  background: rgba(24, 160, 88, 0.15) !important;
}

html.dark :deep(.admin-main-tabs .n-tabs-tab.n-tabs-tab--active) {
  background: rgba(24, 160, 88, 0.2) !important;
}

html.dark :deep(.admin-sub-tabs .n-tabs-nav) {
  background: rgba(255, 255, 255, 0.05) !important;
}

html.dark :deep(.admin-sub-tabs .n-tabs-tab:hover) {
  background: rgba(24, 160, 88, 0.15) !important;
}

html.dark :deep(.admin-sub-tabs .n-tabs-tab.n-tabs-tab--active) {
  background: rgba(24, 160, 88, 0.25) !important;
}

/* 响应式设计 */
@media (max-width: 810px) {
  .admin-container {
    margin-top: 110px;
    padding: 0 16px;
  }
  
  .admin-tabs-container {
    border-radius: 16px;
    padding: 16px;
    min-height: 500px;
  }
  
  .sub-tabs-wrapper {
    border-radius: 12px;
    padding: 16px;
  }
  
  .tab-content,
  .sub-tab-content {
    padding: 12px 0;
  }
  
  .auth-content {
    padding: 16px 20px 20px;
  }
}

@media (max-width: 480px) {
  .admin-container {
    margin-top: 100px;
    padding: 0 12px;
  }
  
  .admin-tabs-container {
    border-radius: 12px;
    padding: 12px;
    min-height: 400px;
  }
  
  .sub-tabs-wrapper {
    border-radius: 10px;
    padding: 12px;
  }
  
  .auth-content {
    padding: 12px 16px 16px;
  }
  
  :deep(.admin-sub-tabs .n-tabs-tab) {
    font-size: 12px !important;
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
