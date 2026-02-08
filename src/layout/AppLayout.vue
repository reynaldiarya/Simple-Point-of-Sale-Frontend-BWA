<script setup>
import { RouterView } from 'vue-router'
import AppSidebar from './AppSidebar.vue'
</script>

<template>
    <div class="flex min-h-screen bg-surface-50">
        <!-- Sidebar -->
        <AppSidebar />

        <div class="ml-64 flex-1 transition-all duration-300">
            <div class="w-full p-8">
                <RouterView />
            </div>
        </div>
    </div>
</template>
