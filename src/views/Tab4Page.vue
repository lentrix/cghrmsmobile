<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Evaluation</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Evaluation</ion-title>
        </ion-toolbar>
      </ion-header>

      <ExploreContainer name="Evaluation page" />
    </ion-content>
  </ion-page>
</template>

<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/vue';
import ExploreContainer from '@/components/ExploreContainer.vue';
</script>
