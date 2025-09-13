<script setup>
import { ref, h, computed, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useHead } from '@unhead/vue'
import { useRoute, useRouter, RouterLink } from 'vue-router'
// 自定义响应式hook，810px断点
const useIsTablet = () => {
    const isTablet = ref(false)
    
    const updateIsTablet = () => {
        isTablet.value = window.innerWidth <= 810
    }
    
    onMounted(() => {
        updateIsTablet()
        window.addEventListener('resize', updateIsTablet)
    })
    
    return isTablet
}
import {
    DarkModeFilled, LightModeFilled, MenuFilled,
    AdminPanelSettingsFilled
} from '@vicons/material'
import { GithubAlt, Language, User, Home } from '@vicons/fa'

import { useGlobalState } from '../store'
import { api } from '../api'
import { getRouterPathWithLang } from '../utils'

const message = useMessage()
const notification = useNotification()

const {
    toggleDark, isDark, isTelegram, showAdminPage,
    showAuth, auth, loading, openSettings, userSettings
} = useGlobalState()
const route = useRoute()
const router = useRouter()
const isTablet = useIsTablet()

const showMobileMenu = ref(false)
const menuValue = computed(() => {
    if (route.path.includes("user")) return "user";
    if (route.path.includes("admin")) return "admin";
    return "home";
});

const authFunc = async () => {
    try {
        location.reload()
    } catch (error) {
        message.error(error.message || "error");
    }
}

const changeLocale = async (lang) => {
    if (lang == 'zh') {
        await router.push(route.fullPath.replace('/en', ''));
    } else {
        await router.push(`/${lang}${route.fullPath}`);
    }
}

const { locale, t } = useI18n({
    messages: {
        en: {
            title: 'LiuShen TMail',
            dark: 'Dark',
            light: 'Light',
            accessHeader: 'Access Password',
            accessTip: 'Please enter the correct access password',
            home: 'Home',
            menu: 'Menu',
            user: 'User',
            ok: 'OK',
        },
        zh: {
            title: 'LiuShen TMail',
            dark: '暗色',
            light: '亮色',
            accessHeader: '访问密码',
            accessTip: '请输入站点访问密码',
            home: '主页',
            menu: '菜单',
            user: '用户',
            ok: '确定',
        }
    }
});

const version = import.meta.env.PACKAGE_VERSION ? `v${import.meta.env.PACKAGE_VERSION}` : "";

const menuOptions = computed(() => [
    {
        label: () => h(NButton,
            {
                text: true,
                size: "small",
                type: menuValue.value == "home" ? "primary" : "default",
                style: "width: 100%",
                onClick: async () => {
                    await router.push(getRouterPathWithLang('/', locale.value));
                    showMobileMenu.value = false;
                }
            },
            {
                default: () => t('home'),
                icon: () => h(NIcon, { component: Home })
            }),
        key: "home"
    },
    {
        label: () => h(
            NButton,
            {
                text: true,
                size: "small",
                type: menuValue.value == "user" ? "primary" : "default",
                style: "width: 100%",
                onClick: async () => {
                    await router.push(getRouterPathWithLang("/user", locale.value));
                    showMobileMenu.value = false;
                }
            },
            {
                default: () => t('user'),
                icon: () => h(NIcon, { component: User }),
            }
        ),
        key: "user",
        show: !isTelegram.value
    },
    {
        label: () => h(
            NButton,
            {
                text: true,
                size: "small",
                type: menuValue.value == "admin" ? "primary" : "default",
                style: "width: 100%",
                onClick: async () => {
                    loading.value = true;
                    await router.push(getRouterPathWithLang('/admin', locale.value));
                    loading.value = false;
                    showMobileMenu.value = false;
                }
            },
            {
                default: () => "管理",
                icon: () => h(NIcon, { component: AdminPanelSettingsFilled }),
            }
        ),
        show: showAdminPage.value,
        key: "admin"
    },
    {
        label: () => h(
            NButton,
            {
                text: true,
                size: "small",
                style: "width: 100%",
                onClick: () => { toggleDark(); showMobileMenu.value = false; }
            },
            {
                default: () => isDark.value ? t('light') : t('dark'),
                icon: () => h(
                    NIcon, { component: isDark.value ? LightModeFilled : DarkModeFilled }
                )
            }
        ),
        key: "theme"
    },
    {
        label: () => h(
            NButton,
            {
                text: true,
                size: "small",
                style: "width: 100%",
                onClick: async () => {
                    locale.value == 'zh' ? await changeLocale('en') : await changeLocale('zh');
                    showMobileMenu.value = false;
                }
            },
            {
                default: () => locale.value == 'zh' ? "EN" : "中文",
                icon: () => h(
                    NIcon, { component: Language }
                )
            }
        ),
        key: "lang"
    },
    // {
    //     label: () => h(
    //         NButton,
    //         {
    //             text: true,
    //             size: "small",
    //             style: "width: 100%",
    //             tag: "a",
    //             target: "_blank",
    //             href: "https://github.com/dreamhunter2333/cloudflare_temp_email",
    //         },
    //         {
    //             default: () => version || "Github",
    //             icon: () => h(NIcon, { component: GithubAlt })
    //         }
    //     ),
    //     show: openSettings.value?.showGithub,
    //     key: "github"
    // }
]);

useHead({
    title: () => openSettings.value.title || t('title'),
    meta: [
        { name: "description", content: openSettings.value.description || t('title') },
    ]
});

const logoClickCount = ref(0);
const logoClick = async () => {
    if (route.path.includes("admin")) {
        logoClickCount.value = 0;
        return;
    }
    if (logoClickCount.value >= 5) {
        logoClickCount.value = 0;
        message.info("Change to admin Page");
        loading.value = true;
        await router.push(getRouterPathWithLang('/admin', locale.value));
        loading.value = false;
    } else {
        logoClickCount.value++;
    }
    if (logoClickCount.value > 0) {
        message.info(`Click ${5 - logoClickCount.value + 1} times to enter the admin page`);
    }
}

// 标题点击回到首页
const titleClick = async () => {
    await router.push(getRouterPathWithLang('/', locale.value));
}

onMounted(async () => {
    await api.getOpenSettings(message, notification);
    // make sure user_id is fetched
    if (!userSettings.value.user_id) await api.getUserSettings(message);
});
</script>

<template>
    <div class="floating-header-wrapper">
        <!-- 悬浮头部容器 -->
        <div class="floating-header">
            <div class="header-content">
                <!-- 左侧Logo和标题 -->
                <div class="header-left">
                    <div class="logo-container" @click="logoClick">
                        <n-avatar class="logo-avatar" src="/logo.png" />
                    </div>
                    <div class="header-title-container" @click="titleClick">
                        <h3 class="header-title">{{ openSettings.title || t('title') }}</h3>
                    </div>
                </div>
                
                <!-- 右侧菜单 -->
                <div class="header-right">
                    <n-space v-if="!isTablet" class="desktop-menu">
                        <n-menu mode="horizontal" :options="menuOptions" responsive />
                    </n-space>
                    <n-button v-else :text="true" @click="showMobileMenu = !showMobileMenu" class="mobile-menu-btn">
                        <template #icon>
                            <n-icon :component="MenuFilled" />
                        </template>
                        {{ t('menu') }}
                    </n-button>
                </div>
            </div>
        </div>
        <n-drawer v-model:show="showMobileMenu" placement="top" style="height: 100vh;">
            <n-drawer-content :title="t('menu')" closable>
                <n-menu :options="menuOptions" />
            </n-drawer-content>
        </n-drawer>
        <n-modal v-model:show="showAuth" :closable="false" :closeOnEsc="false" :maskClosable="false" preset="dialog"
            :title="t('accessHeader')">
            <p>{{ t('accessTip') }}</p>
            <n-input v-model:value="auth" type="password" show-password-on="click" />
            <template #action>
                <n-button :loading="loading" @click="authFunc" type="primary">
                    {{ t('ok') }}
                </n-button>
            </template>
        </n-modal>
    </div>
</template>

<style scoped>
/* 基础样式 */
.n-alert {
    margin-top: 10px;
    margin-bottom: 10px;
    text-align: center;
}

.n-card {
    margin-top: 10px;
}

.center {
    display: flex;
    text-align: left;
    place-items: center;
    justify-content: center;
    margin: 20px;
}

.n-form .n-button {
    margin-top: 10px;
}

/* 悬浮头部容器 */
.floating-header-wrapper {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 1000;
    padding: 16px 20px 0;
    pointer-events: none;
}

.floating-header {
    max-width: 1200px;
    margin: 0 auto;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(20px);
    border-radius: 20px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    box-shadow: 
        0 8px 32px rgba(0, 0, 0, 0.1),
        0 2px 8px rgba(0, 0, 0, 0.05),
        inset 0 1px 0 rgba(255, 255, 255, 0.3);
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    pointer-events: auto;
    overflow: hidden;
    position: relative;
}

.floating-header::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: linear-gradient(135deg, 
        rgba(255, 255, 255, 0.15) 0%,
        rgba(255, 255, 255, 0.05) 50%,
        rgba(255, 255, 255, 0.1) 100%);
    opacity: 0;
    transition: opacity 0.3s ease;
    pointer-events: none;
}

.floating-header:hover::before {
    opacity: 1;
}

.floating-header:hover {
    background: rgba(255, 255, 255, 0.2);
    border-color: rgba(255, 255, 255, 0.3);
    box-shadow: 
        0 12px 40px rgba(0, 0, 0, 0.15),
        0 4px 12px rgba(0, 0, 0, 0.08),
        inset 0 1px 0 rgba(255, 255, 255, 0.4);
    transform: translateY(-2px);
}

.header-content {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 12px 24px;
    position: relative;
    z-index: 1;
}

.header-left {
    display: flex;
    align-items: center;
    gap: 16px;
}

.header-right {
    display: flex;
    align-items: center;
}

/* Logo样式 */
.logo-container {
    cursor: pointer;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    padding: 4px;
    border-radius: 12px;
}

.logo-container:hover {
    transform: scale(1.1) rotate(5deg);
    background: rgba(24, 160, 88, 0.1);
}

.logo-avatar {
    border-radius: 10px !important;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
    transition: all 0.3s ease;
    border: 2px solid rgba(255, 255, 255, 0.3);
}

.logo-avatar:hover {
    box-shadow: 0 6px 20px rgba(24, 160, 88, 0.3);
    border-color: rgba(24, 160, 88, 0.5);
}

/* 标题样式 */
.header-title-container {
    cursor: pointer;
    padding: 8px 16px;
    border-radius: 12px;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    position: relative;
    overflow: hidden;
}

.header-title-container::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, 
        transparent, 
        rgba(24, 160, 88, 0.2), 
        transparent);
    transition: left 0.5s ease;
}

.header-title-container:hover::before {
    left: 100%;
}

.header-title-container:hover {
    background: rgba(24, 160, 88, 0.1);
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(24, 160, 88, 0.2);
}

.header-title {
    margin: 0;
    font-weight: 700;
    font-size: 1.2em;
    background: linear-gradient(135deg, #18a058, #36ad6a, #52c41a);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    transition: all 0.3s ease;
    position: relative;
    z-index: 1;
}

.header-title-container:hover .header-title {
    transform: translateY(-1px);
    filter: brightness(1.2);
}

/* 桌面菜单样式 */
.desktop-menu {
    border-radius: 12px;
    padding: 4px;
    transition: all 0.3s ease;
}

.desktop-menu:hover {
    background: rgba(255, 255, 255, 0.1);
}

/* 移动端菜单按钮 */
.mobile-menu-btn {
    border-radius: 12px;
    padding: 8px 16px;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    background: rgba(255, 255, 255, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.2);
}

.mobile-menu-btn:hover {
    background: rgba(24, 160, 88, 0.2);
    border-color: rgba(24, 160, 88, 0.3);
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(24, 160, 88, 0.2);
}

/* 暗色模式适配 */
html.dark .floating-header {
    background: rgba(16, 16, 20, 0.1);
    border-color: rgba(255, 255, 255, 0.1);
    box-shadow: 
        0 8px 32px rgba(0, 0, 0, 0.3),
        0 2px 8px rgba(0, 0, 0, 0.2),
        inset 0 1px 0 rgba(255, 255, 255, 0.1);
}

html.dark .floating-header::before {
    background: linear-gradient(135deg, 
        rgba(255, 255, 255, 0.05) 0%,
        rgba(255, 255, 255, 0.02) 50%,
        rgba(255, 255, 255, 0.05) 100%);
}

html.dark .floating-header:hover {
    background: rgba(16, 16, 20, 0.2);
    border-color: rgba(255, 255, 255, 0.2);
    box-shadow: 
        0 12px 40px rgba(0, 0, 0, 0.4),
        0 4px 12px rgba(0, 0, 0, 0.3),
        inset 0 1px 0 rgba(255, 255, 255, 0.2);
}

html.dark .logo-container:hover {
    background: rgba(24, 160, 88, 0.15);
}

html.dark .header-title-container:hover {
    background: rgba(24, 160, 88, 0.15);
}

html.dark .desktop-menu:hover {
    background: rgba(255, 255, 255, 0.05);
}

html.dark .mobile-menu-btn {
    background: rgba(255, 255, 255, 0.05);
    border-color: rgba(255, 255, 255, 0.1);
}

html.dark .mobile-menu-btn:hover {
    background: rgba(24, 160, 88, 0.2);
    border-color: rgba(24, 160, 88, 0.3);
}

/* 菜单项美化 */
:deep(.n-menu) {
    background: transparent !important;
}

:deep(.n-menu-item) {
    border-radius: 10px !important;
    margin: 0 4px !important;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1) !important;
    background: transparent !important;
}

:deep(.n-menu-item:hover) {
    background: rgba(24, 160, 88, 0.15) !important;
    transform: translateY(-2px) !important;
    box-shadow: 0 4px 12px rgba(24, 160, 88, 0.2) !important;
}

:deep(.n-menu-item.n-menu-item--selected) {
    background: rgba(24, 160, 88, 0.2) !important;
    box-shadow: 0 2px 8px rgba(24, 160, 88, 0.3) !important;
}

/* 抽屉样式美化 */
:deep(.n-drawer-content) {
    background: linear-gradient(135deg, 
        rgba(255, 255, 255, 0.95), 
        rgba(248, 250, 252, 0.95)) !important;
    backdrop-filter: blur(20px) !important;
    border-radius: 0 0 20px 20px !important;
}

html.dark :deep(.n-drawer-content) {
    background: linear-gradient(135deg, 
        rgba(16, 16, 20, 0.95), 
        rgba(24, 24, 28, 0.95)) !important;
}

/* 为页面内容添加顶部间距 */
:global(body) {
    padding-top: 80px;
}

/* 响应式设计 */
@media (max-width: 810px) {
    .floating-header-wrapper {
        padding: 12px 16px 0;
    }
    
    .floating-header {
        border-radius: 16px;
    }
    
    .header-content {
        padding: 10px 16px;
    }
    
    .header-left {
        gap: 12px;
    }
    
    .header-title {
        font-size: 1.1em;
    }
}

@media (max-width: 480px) {
    .floating-header-wrapper {
        padding: 8px 12px 0;
    }
    
    .floating-header {
        border-radius: 12px;
    }
    
    .header-content {
        padding: 8px 12px;
    }
    
    .header-left {
        gap: 8px;
    }
    
    .header-title {
        font-size: 1em;
    }
}
</style>
