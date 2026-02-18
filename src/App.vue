<template>
  <ion-app>
    <ion-router-outlet />
  </ion-app>
</template>

<script setup>
import { IonApp, IonRouterOutlet } from '@ionic/vue';
import {ref, provide} from 'vue';

const getStoredUser = () => {
  const storedUser = localStorage.getItem('user')

  if (!storedUser) {
    return null
  }

  try {
    return JSON.parse(storedUser)
  } catch {
    return null
  }
}

const user = ref(getStoredUser())
// const host = ref(localStorage.getItem('host') || 'http://localhost:8000/api')

provide('user', user)
</script>
