<template>
  <ion-page>
    <ion-menu content-id="main-content">
      <ion-header>
        <ion-toolbar>
          <ion-title>Navigation</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-content>
        <div class="drawer-layout">
          <ion-list class="main-menu-list">
          <ion-menu-toggle auto-hide="true">
            <ion-item button href="/tabs/tab1">
              <ion-icon aria-hidden="true" :icon="home" slot="start" />
              <ion-label>Home</ion-label>
            </ion-item>
          </ion-menu-toggle>

          <template v-if="user">
            <ion-menu-toggle auto-hide="true">
              <ion-item button href="/tabs/tab2">
                <ion-icon aria-hidden="true" :icon="time" slot="start" />
                <ion-label>DTR</ion-label>
              </ion-item>
            </ion-menu-toggle>

            <ion-menu-toggle auto-hide="true">
              <ion-item button href="/tabs/leave-form">
                <ion-icon aria-hidden="true" :icon="documentText" slot="start" />
                <ion-label>Leave Application</ion-label>
              </ion-item>
            </ion-menu-toggle>

            <ion-menu-toggle auto-hide="true">
              <ion-item button href="/tabs/tab3">
                <ion-icon aria-hidden="true" :icon="clipboard" slot="start" />
                <ion-label>Pay Slip</ion-label>
              </ion-item>
            </ion-menu-toggle>

            <ion-menu-toggle auto-hide="true">
              <ion-item button href="/tabs/tab4">
                <ion-icon aria-hidden="true" :icon="people" slot="start" />
                <ion-label>Evaluation</ion-label>
              </ion-item>
            </ion-menu-toggle>

            <ion-menu-toggle auto-hide="true">
              <ion-item button href="/tabs/tab5">
                <ion-icon aria-hidden="true" :icon="fileTray" slot="start" />
                <ion-label>Incident</ion-label>
              </ion-item>
            </ion-menu-toggle>
          </template>
          </ion-list>

          <ion-list v-if="user" class="bottom-menu-list">
            <ion-menu-toggle auto-hide="true">
              <ion-item button @click="logout">
                <ion-icon aria-hidden="true" :icon="logOutOutline" slot="start" />
                <ion-label>Logout</ion-label>
              </ion-item>
            </ion-menu-toggle>
          </ion-list>
        </div>
      </ion-content>
    </ion-menu>

    <ion-router-outlet id="main-content"></ion-router-outlet>
  </ion-page>

  
</template>

<script setup>
import { IonMenu, IonHeader, IonToolbar, IonTitle, IonContent, IonList, IonMenuToggle, IonItem, IonLabel, IonIcon, IonPage, IonRouterOutlet } from '@ionic/vue';
import { home, clipboard, people, time, fileTray, logOutOutline, documentText } from 'ionicons/icons';
import axios from 'axios';
import { useToast } from 'vue-toastification';

import { inject } from 'vue';
import { useRouter } from 'vue-router';

const user = inject('user')
const toast = useToast()
const router = useRouter()

const logout = async () => {
  const host = localStorage.getItem('host') || 'http://localhost:8000/api'

  try {
    await axios.post(`${host}/logout`, {}, {
      headers: {
        Authorization: `Bearer ${localStorage.getItem('access_token')}`
      }
    })

    toast.success('Logout successful')
  } catch (error) {
    toast.error('Logout failed. ' + error.response?.data?.message)
  }

  localStorage.removeItem('access_token')
  localStorage.removeItem('user')
  user.value = null
  router.push('/tabs/tab1')
}
</script>

<style scoped>
.drawer-layout {
  min-height: 100%;
  display: flex;
  flex-direction: column;
}

.main-menu-list {
  margin-top: 0;
}

.bottom-menu-list {
  margin-top: auto;
  border-top: 1px solid var(--ion-color-step-150, #d7d8da);
  padding-top: 8px;
}
</style>
