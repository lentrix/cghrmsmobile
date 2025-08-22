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
      <div class="ion-padding" v-if="online">
        Online...
      </div>
      <div class="ion-padding" v-else>
        Offline...
      </div>
      <LoginForm v-if="!user"></LoginForm>
      <HomePage v-else></HomePage>
    </ion-content>

    <ion-fab slot="fixed" vertical="bottom" horizontal="end">
      <ion-fab-button @click="refreshPage">
        <ion-icon :icon="refresh"></ion-icon>
      </ion-fab-button>
    </ion-fab>

  </ion-page>
</template>

<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonIcon, IonButton, IonButtons, IonFab, IonFabButton } from '@ionic/vue';
import LoginForm from '@/components/LoginForm.vue';
import HomePage from '@/components/HomePage.vue';
import SettingsModal from '../components/SettingsModal.vue';
import { onMounted, inject } from 'vue';
import { home, refresh } from 'ionicons/icons';
import {ref} from 'vue';
import axios from 'axios';

const user = inject('user')

const online = ref(false)

const settingsIsOpen = ref(false);

const refreshPage = () =>{
  location.reload();
}

const close = () =>{
  settingsIsOpen.value = false;
}

onMounted(() => {
  
  localStorage.getItem('user') ? user.value = JSON.parse(localStorage.getItem('user')) : user.value = null;

  let host = localStorage.getItem('host');

  axios.get(`${host}/test`)
    .then(response => {
      online.value = true;
    })
    .catch(error => {
      console.error('Error fetching online status:', error);
    });

});

</script>
