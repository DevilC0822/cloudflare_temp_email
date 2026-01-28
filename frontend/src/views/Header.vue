<script setup>
import { ref, h, computed, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useHead } from '@unhead/vue'
import { useRoute, useRouter } from 'vue-router'
import { useIsMobile } from '../utils/composables'
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
    toggleDark, isDark, isTelegram,
    showAuth, auth, loading, openSettings, userSettings
} = useGlobalState()
const route = useRoute()
const router = useRouter()
const isMobile = useIsMobile()

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

const goTo = async (path) => {
    await router.push(getRouterPathWithLang(path, locale.value));
    showMobileMenu.value = false;
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
            title: 'Cloudflare Temp Email',
            subtitle: 'Private, fast, and temporary inbox',
            dark: 'Dark',
            light: 'Light',
            accessHeader: 'Access Password',
            accessTip: 'Please enter the correct access password',
            home: 'Home',
            menu: 'Menu',
            user: 'User',
            admin: 'Admin',
            ok: 'OK',
            toAdminHint: 'Click {count} more times to enter Admin',
            goAdmin: 'Go to Admin',
        },
        zh: {
            title: 'Cloudflare 临时邮箱',
            subtitle: '简洁、安全的临时邮箱',
            dark: '暗色',
            light: '亮色',
            accessHeader: '访问密码',
            accessTip: '请输入站点访问密码',
            home: '主页',
            menu: '菜单',
            user: '用户',
            admin: '管理',
            ok: '确定',
            toAdminHint: '再点击 {count} 次进入管理',
            goAdmin: '进入管理',
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
                onClick: async () => await goTo('/')
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
                onClick: async () => await goTo('/user')
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
                    await goTo('/admin');
                    loading.value = false;
                    showMobileMenu.value = false;
                }
            },
            {
                default: () => t('admin'),
                icon: () => h(NIcon, { component: AdminPanelSettingsFilled }),
            }
        ),
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
                default: () => locale.value == 'zh' ? "English" : "中文",
                icon: () => h(
                    NIcon, { component: Language }
                )
            }
        ),
        key: "lang"
    },
    {
        label: () => h(
            NButton,
            {
                text: true,
                size: "small",
                style: "width: 100%",
                tag: "a",
                target: "_blank",
                href: "https://github.com/dreamhunter2333/cloudflare_temp_email",
            },
            {
                default: () => version || "Github",
                icon: () => h(NIcon, { component: GithubAlt })
            }
        ),
        show: openSettings.value?.showGithub,
        key: "github"
    }
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
        message.info(t('goAdmin'));
        loading.value = true;
        await goTo('/admin');
        loading.value = false;
    } else {
        logoClickCount.value++;
    }
    if (logoClickCount.value > 0) {
        message.info(t('toAdminHint', { count: 5 - logoClickCount.value + 1 }));
    }
}

onMounted(async () => {
    await api.getOpenSettings(message, notification);
    // make sure user_id is fetched
    if (!userSettings.value.user_id) await api.getUserSettings(message);
});
</script>

<template>
    <header class="app-header">
        <div class="app-container">
            <div class="header-card app-glass">
                <div class="header-left" @click="logoClick" @keydown.enter.prevent="logoClick"
                    @keydown.space.prevent="logoClick" role="button" tabindex="0" aria-label="logo">
                    <n-avatar class="header-logo" src="/logo.png" />
                    <div class="header-titles">
                        <div class="header-title">
                            {{ openSettings.title || t('title') }}
                        </div>
                        <div class="header-subtitle">
                            {{ openSettings.description || t('subtitle') }}
                        </div>
                    </div>
                </div>

                <nav v-if="!isMobile" class="header-nav">
                    <n-button quaternary :type="menuValue === 'home' ? 'primary' : 'default'" @click="goTo('/')">
                        <template #icon><n-icon :component="Home" /></template>
                        {{ t('home') }}
                    </n-button>
                    <n-button v-if="!isTelegram" quaternary :type="menuValue === 'user' ? 'primary' : 'default'"
                        @click="goTo('/user')">
                        <template #icon><n-icon :component="User" /></template>
                        {{ t('user') }}
                    </n-button>
                    <n-button quaternary :type="menuValue === 'admin' ? 'primary' : 'default'" @click="goTo('/admin')">
                        <template #icon><n-icon :component="AdminPanelSettingsFilled" /></template>
                        {{ t('admin') }}
                    </n-button>
                </nav>

                <div class="header-actions">
                    <n-button quaternary circle @click="toggleDark()" :aria-label="isDark ? t('light') : t('dark')">
                        <n-icon :component="isDark ? LightModeFilled : DarkModeFilled" />
                    </n-button>
                    <n-button quaternary circle @click="locale == 'zh' ? changeLocale('en') : changeLocale('zh')"
                        aria-label="language">
                        <n-icon :component="Language" />
                    </n-button>
                    <n-button v-if="openSettings?.showGithub" quaternary tag="a" target="_blank"
                        href="https://github.com/dreamhunter2333/cloudflare_temp_email">
                        <template #icon><n-icon :component="GithubAlt" /></template>
                        {{ version || 'GitHub' }}
                    </n-button>
                    <n-button v-if="isMobile" quaternary @click="showMobileMenu = true" aria-label="menu">
                        <template #icon><n-icon :component="MenuFilled" /></template>
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
    </header>
</template>

<style scoped>
.app-header {
    position: sticky;
    top: 0;
    z-index: 50;
    padding: 12px 0;
}

.header-card {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    padding: 10px 12px;
}

.header-left {
    display: flex;
    align-items: center;
    gap: 10px;
    min-width: 0;
    cursor: pointer;
    user-select: none;
}

.header-logo {
    flex: 0 0 auto;
}

.header-titles {
    min-width: 0;
    text-align: left;
}

.header-title {
    font-size: 14px;
    font-weight: 700;
    letter-spacing: 0.2px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.header-subtitle {
    margin-top: 2px;
    font-size: 12px;
    color: var(--app-text-muted);
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.header-nav {
    display: flex;
    gap: 4px;
    align-items: center;
    justify-content: center;
    flex: 1 1 auto;
    min-width: 0;
}

.header-actions {
    display: flex;
    gap: 6px;
    align-items: center;
    justify-content: flex-end;
    flex: 0 0 auto;
    min-width: 0;
}

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
</style>
