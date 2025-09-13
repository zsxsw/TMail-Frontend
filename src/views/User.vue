<script setup>
import { useI18n } from 'vue-i18n'

import { useGlobalState } from '../store'

import AddressMangement from './user/AddressManagement.vue';
import UserSettingsPage from './user/UserSettings.vue';
import UserBar from './user/UserBar.vue';
import BindAddress from './user/BindAddress.vue';
import UserMailBox from './user/UserMailBox.vue';

const {
    userTab, globalTabplacement, userSettings
} = useGlobalState()

const { t } = useI18n({
    messages: {
        en: {
            address_management: 'Address Management',
            user_mail_box_tab: 'Mail Box',
            user_settings: 'User Settings',
            bind_address: 'Bind Mail Address',
        },
        zh: {
            address_management: '地址管理',
            user_mail_box_tab: '收件箱',
            user_settings: '用户设置',
            bind_address: '绑定邮箱地址',
        }
    }
});

</script>

<template>
    <div class="user-container">
        <div class="user-bar-wrapper">
            <UserBar />
        </div>
        <div v-if="userSettings.user_email" class="tabs-container">
            <n-tabs type="card" v-model:value="userTab" :placement="globalTabplacement" class="user-tabs">
                <n-tab-pane name="address_management" :tab="t('address_management')">
                    <div class="tab-content">
                        <AddressMangement />
                    </div>
                </n-tab-pane>
                <n-tab-pane name="user_mail_box_tab" :tab="t('user_mail_box_tab')">
                    <div class="tab-content">
                        <UserMailBox />
                    </div>
                </n-tab-pane>
                <n-tab-pane name="user_settings" :tab="t('user_settings')">
                    <div class="tab-content">
                        <UserSettingsPage />
                    </div>
                </n-tab-pane>
                <n-tab-pane name="bind_address" :tab="t('bind_address')">
                    <div class="tab-content">
                        <BindAddress />
                    </div>
                </n-tab-pane>
            </n-tabs>
        </div>
    </div>
</template>

<style scoped>
/* 主容器样式 */
.user-container {
    margin-top: 120px; /* 为悬浮菜单留出空间 */
    padding: 0 20px;
    max-width: 1200px;
    margin-left: auto;
    margin-right: auto;
    min-height: calc(100vh - 100px);
}

/* 用户栏包装器 */
.user-bar-wrapper {
    background: rgba(255, 255, 255, 0.8);
    backdrop-filter: blur(10px);
    border-radius: 16px;
    padding: 20px;
    margin-bottom: 20px;
    box-shadow: 
        0 4px 20px rgba(0, 0, 0, 0.08),
        0 1px 4px rgba(0, 0, 0, 0.04);
    border: 1px solid rgba(255, 255, 255, 0.2);
    transition: all 0.3s ease;
}

.user-bar-wrapper:hover {
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

/* 标签页样式优化 */
:deep(.user-tabs) {
    --n-tab-border-radius: 12px;
}

:deep(.user-tabs .n-tabs-nav) {
    background: rgba(255, 255, 255, 0.1) !important;
    border-radius: 12px !important;
    padding: 4px !important;
    margin-bottom: 16px !important;
}

:deep(.user-tabs .n-tabs-tab) {
    border-radius: 10px !important;
    margin: 0 2px !important;
    transition: all 0.3s ease !important;
}

:deep(.user-tabs .n-tabs-tab:hover) {
    background: rgba(24, 160, 88, 0.1) !important;
    transform: translateY(-1px) !important;
}

:deep(.user-tabs .n-tabs-tab.n-tabs-tab--active) {
    background: rgba(24, 160, 88, 0.15) !important;
    box-shadow: 0 2px 8px rgba(24, 160, 88, 0.2) !important;
}

/* 暗色模式适配 */
html.dark .user-bar-wrapper,
html.dark .tabs-container {
    background: rgba(16, 16, 20, 0.8) !important;
    border-color: rgba(255, 255, 255, 0.1) !important;
    box-shadow: 
        0 8px 32px rgba(0, 0, 0, 0.3),
        0 2px 8px rgba(0, 0, 0, 0.2) !important;
}

html.dark .user-bar-wrapper:hover,
html.dark .tabs-container:hover {
    box-shadow: 
        0 12px 40px rgba(0, 0, 0, 0.4),
        0 4px 12px rgba(0, 0, 0, 0.3) !important;
}

html.dark :deep(.user-tabs .n-tabs-nav) {
    background: rgba(255, 255, 255, 0.05) !important;
}

html.dark :deep(.user-tabs .n-tabs-tab:hover) {
    background: rgba(24, 160, 88, 0.15) !important;
}

html.dark :deep(.user-tabs .n-tabs-tab.n-tabs-tab--active) {
    background: rgba(24, 160, 88, 0.2) !important;
}

/* 响应式设计 */
@media (max-width: 810px) {
    .user-container {
        margin-top: 110px;
        padding: 0 16px;
    }
    
    .user-bar-wrapper,
    .tabs-container {
        border-radius: 16px;
        padding: 16px;
    }
    
    .user-bar-wrapper {
        margin-bottom: 16px;
    }
    
    .tab-content {
        padding: 12px 0;
    }
}

@media (max-width: 480px) {
    .user-container {
        margin-top: 100px;
        padding: 0 12px;
    }
    
    .user-bar-wrapper,
    .tabs-container {
        border-radius: 12px;
        padding: 12px;
    }
    
    .user-bar-wrapper {
        margin-bottom: 12px;
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
