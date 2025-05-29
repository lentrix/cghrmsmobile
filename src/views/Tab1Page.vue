<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Home</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Home</ion-title>
        </ion-toolbar>
      </ion-header>

      <LoginForm v-if="!user"></LoginForm>
      <HomePage v-else></HomePage>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/vue';
import LoginForm from '@/components/LoginForm.vue';
import HomePage from '@/components/HomePage.vue';
import { onMounted, inject } from 'vue';

const user = inject('user')

onMounted(() => {
  
  localStorage.getItem('user') ? user.value = JSON.parse(localStorage.getItem('user')) : user.value = null;

});

</script>
