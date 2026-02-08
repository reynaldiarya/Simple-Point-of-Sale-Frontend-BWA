<script setup>
import router from '@/router'
import { useAuthStore } from '@/stores/auth.store'
import { Button, Dialog } from 'primevue'
import { ref } from 'vue'
import { useRoute } from 'vue-router'

const authStore = useAuthStore()
const { user } = authStore

const logoutDialog = ref(false)

const handleLogout = async () => {
    await authStore.logout()
    logoutDialog.value = false
    router.push({ name: 'login' })
}

const route = useRoute()

const menuItems = ref([
    {
        label: 'General',
        items: [
            { icon: 'pi pi-th-large', to: '/', label: 'Dashboard' },
            { icon: 'pi pi-shopping-cart', to: '/pos', label: 'POS' },
        ],
    },
    {
        label: 'Management',
        items: [
            {
                icon: 'pi pi-tag',
                to: '/product-categories',
                label: 'Product Categories',
            },
            { icon: 'pi pi-box', to: '/products', label: 'Products' },
            { icon: 'pi pi-users', to: '/customers', label: 'Customers' },
            {
                icon: 'pi pi-receipt',
                to: '/transactions',
                label: 'Transactions',
            },
        ],
    },
])
</script>

<template>
    <div
        class="fixed top-0 left-0 z-50 flex h-full w-64 flex-col border-r border-surface-200 bg-white transition-all duration-300"
    >
        <!-- logo -->
        <div class="flex h-20 items-center border-b border-surface-100 px-8">
            <div
                class="mr-3 flex h-8 w-8 items-center justify-center rounded-lg bg-primary-600 text-white"
            >
                <i class="pi pi-bolt text-lg"></i>
            </div>
            <span class="text-xl font-bold text-surface-900">BwaPOS</span>
        </div>

        <!-- menu -->
        <div class="flex flex-1 flex-col gap-6 overflow-y-auto px-4 py-6">
            <div v-for="(section, i) in menuItems" :key="i">
                <div
                    class="mb-3 px-3 text-xs font-semibold tracking-wider text-surface-400 uppercase"
                >
                    {{ section.label }}
                </div>

                <div class="flex flex-col gap-1">
                    <router-link
                        v-for="(item, j) in section.items"
                        :to="item.to"
                        :key="index"
                        class="flex items-center gap-3 rounded-lg px-3 py-2.5 transition-colors duration-200"
                        :class="[
                            route.path == item.to
                                ? 'bg-surface-100 text-primary-600'
                                : 'text-surface-900 hover:bg-surface-100',
                        ]"
                    >
                        <i :class="[item.icon, 'text-lg']"></i>
                        <span class="text-sm font-medium">{{
                            item.label
                        }}</span>
                    </router-link>
                </div>
            </div>
        </div>

        <!-- User Profile -->
        <div class="border-t border-surface-200 p-4">
            <button
                @click="logoutDialog = true"
                class="flex w-full items-center gap-3 rounded-xl p-3 hover:bg-surface-50"
            >
                <div
                    class="flex h-9 w-9 items-center justify-center overflow-hidden rounded-full bg-surface-200"
                >
                    <i class="pi pi-user text-lg text-surface-600"></i>
                </div>
                <div class="text-left">
                    <div class="text-sm font-semibold text-surface-900">
                        {{ user?.name }}
                    </div>
                    <div class="text-xs text-surface-500">
                        {{ user?.email }}
                    </div>
                </div>
                <div
                    class="ml-auto flex h-8 w-8 items-center justify-center rounded-lg text-surface-400 transition-colors group-hover:bg-red-100 group-hover:text-red-600"
                >
                    <i class="pi pi-sign-out text-lg"></i>
                </div>
            </button>
        </div>
    </div>

    <Dialog
        v-model:visible="logoutDialog"
        header="Confirm Logout"
        :modal="true"
        class="w-100"
    >
        <span class="mb-8 block text-surface-500">Are u sure to logout?</span>
        <div class="flex justify-end gap-2">
            <Button
                type="button"
                label="Cancel"
                severity="secondary"
                @click="logoutDialog = false"
            />
            <Button
                type="button"
                label="Logout"
                severity="danger"
                @click="handleLogout"
            />
        </div>
    </Dialog>
</template>
