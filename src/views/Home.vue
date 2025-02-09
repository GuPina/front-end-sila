<template>
    <div class="flex">
      <Sidebar />
      <div class="flex-1 p-6">
        <Header />
        <DashboardCard />
      </div>
    </div>
  </template>
  
  <script>
  import Sidebar from '@/components/Dashboard/Sidebar.vue'
  import Header from '@/components/Dashboard/Header.vue'
  import DashboardCard from '@/components/Dashboard/DashboardCard.vue';
  
  export default {
    name: 'Home',
    components: {
      Sidebar,
      Header,
      DashboardCard
    }
  }
  </script>
  
  <style scoped>
  /* Estilos personalizados para a página inicial */
  </style>
  