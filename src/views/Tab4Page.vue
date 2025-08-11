<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>
          <ion-icon aria-hidden="true" :icon="people" />
          Evaluation
        </ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <Index v-if="activePage=='index'" @evaluate="onEvaluate" />
      <Evaluate v-else-if="activePage=='evaluate'" :evaluation="evaluation" @back="onBack" />
    </ion-content>
  </ion-page>
</template>

<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonIcon } from '@ionic/vue';
import {people} from 'ionicons/icons';
import Index from './Evaluation/Index.vue';
import Evaluate from './Evaluation/Evaluate.vue';
import { ref } from 'vue';

const activePage = ref('index');

const evaluation = ref(null);

const onEvaluate = (requirement) => {
    activePage.value = 'evaluate';
    evaluation.value = requirement;
};

const onBack = () => {
    activePage.value = 'index';
    evaluation.value = null;
};


</script>
