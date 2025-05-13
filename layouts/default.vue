<template>
    <div class="default-layout">
        <AppHeader v-if="!isAdmin" />
        <AdminHeader v-if="isAdmin" />
        <main>
            <NuxtPage />
        </main>
        <AppFooter />
    </div>
</template>

<script>
import AppHeader from '@/components/AppHeader.vue'
import AppFooter from '@/components/AppFooter.vue'
import AdminHeader from '~/components/AdminHeader.vue';

export default {
    name: 'DefaultLayout',
    components: {
        AppHeader
    },
    computed: {
        isAdmin() {
            return this.$route.path.startsWith('/admin');
        }
    }
}
</script>

<style scoped>
.default-layout {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}
</style>