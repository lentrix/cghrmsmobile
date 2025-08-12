<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>
          <ion-icon aria-hidden="true" :icon="fileTray" />
          Incident Report
        </ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">

        <Index v-if="activePage === 'index'" @changePage="changePage" />
        <Create v-else-if="activePage === 'create'" @changePage="changePage" />
        <Show v-else-if="activePage === 'open'" :incident="incidentReport" @changePage="changePage" @updateComments="updateComments" />

    </ion-content>
  </ion-page>
</template>


<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonIcon } from '@ionic/vue';
import { fileTray } from 'ionicons/icons';
import { inject, ref } from 'vue';
import Index from '@/views/Incident/Index.vue';
import Create from './Incident/Create.vue';
import Show from './Incident/Show.vue';

const user = inject('user');
const activePage = ref('index');

const incidentReport = ref(null);

const changePage = (page, param) => {
  activePage.value = page;
  if (param) {
    incidentReport.value = param;
  } else {
    incidentReport.value = null;
  }
};

const onBack = () => {
  activePage.value = 'index';
};

const updateComments = (comments) => {
  console.log("Update Comments: ", comments)
  incidentReport.value.employee_comments = comments
}

</script>