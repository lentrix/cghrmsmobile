<template>
  <ion-page>
    <ion-tabs>
      <ion-router-outlet></ion-router-outlet>
      <ion-tab-bar slot="bottom">
        <ion-tab-button tab="tab1" href="/tabs/tab1">
          <ion-icon aria-hidden="true" :icon="home" />
          <ion-label>Home</ion-label>
        </ion-tab-button>

        <template v-if="user">
          
          <ion-tab-button tab="tab2" href="/tabs/tab2">
            <ion-icon aria-hidden="true" :icon="time" />
            <ion-label>DTR</ion-label>
          </ion-tab-button>

          <ion-tab-button tab="tab3" href="/tabs/tab3">
            <ion-icon aria-hidden="true" :icon="clipboard" />
            <ion-label>Pay Slip</ion-label>
          </ion-tab-button>

          <ion-tab-button tab="tab4" href="/tabs/tab4">
            <ion-icon aria-hidden="true" :icon="people" />
            <ion-label>Evaluation</ion-label>
          </ion-tab-button>

        </template>

      </ion-tab-bar>
    </ion-tabs>
  </ion-page>

  
</template>

<script setup>
import { IonTabBar, IonTabButton, IonTabs, IonLabel, IonIcon, IonPage, IonRouterOutlet } from '@ionic/vue';
import { home, clipboard, people, time } from 'ionicons/icons';

import { inject } from 'vue';

const user = inject('user')
</script>
