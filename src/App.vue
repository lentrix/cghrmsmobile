<template>
  <ion-app>
    <ion-router-outlet />
  </ion-app>
</template>

<script setup>
import { IonApp, IonRouterOutlet } from '@ionic/vue';
import {ref, provide} from 'vue';

const user = ref(null)

provide('user', user)
provide('host', 'http://localhost:8000/api')
</script>
