<template>
  <v-app>
    <AppHeader />
    <router-view />
    <AppFooter />
  </v-app>
</template>

<script lang="ts" setup>
import AppHeader from '@/components/AppHeader.vue'
import AppFooter from '@/components/AppFooter.vue'
</script>
