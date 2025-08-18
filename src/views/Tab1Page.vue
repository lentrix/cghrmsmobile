<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>
          <ion-icon :icon="home"></ion-icon>
          Home
        </ion-title>
        <ion-buttons slot="end">
          <ion-button @click="settingsIsOpen=!settingsIsOpen">
            Settings
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
      <SettingsModal :isOpen="settingsIsOpen" @close="close" />
      <LoginForm v-if="!user"></LoginForm>
      <HomePage v-else></HomePage>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonIcon, IonButton, IonButtons } from '@ionic/vue';
import LoginForm from '@/components/LoginForm.vue';
import HomePage from '@/components/HomePage.vue';
import SettingsModal from '../components/SettingsModal.vue';
import { onMounted, inject } from 'vue';
import { home } from 'ionicons/icons';
import {ref} from 'vue';

const user = inject('user')

const settingsIsOpen = ref(false);

const close = () =>{
  settingsIsOpen.value = false;
}

onMounted(() => {
  
  localStorage.getItem('user') ? user.value = JSON.parse(localStorage.getItem('user')) : user.value = null;

});

</script>
