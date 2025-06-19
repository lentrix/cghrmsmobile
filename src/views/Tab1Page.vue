<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>
          <ion-icon :icon="home"></ion-icon>
          Home
        </ion-title>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
      <LoginForm v-if="!user"></LoginForm>
      <HomePage v-else></HomePage>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonIcon } from '@ionic/vue';
import LoginForm from '@/components/LoginForm.vue';
import HomePage from '@/components/HomePage.vue';
import { onMounted, inject } from 'vue';
import { home } from 'ionicons/icons';

const user = inject('user')

onMounted(() => {
  
  localStorage.getItem('user') ? user.value = JSON.parse(localStorage.getItem('user')) : user.value = null;

});

</script>
