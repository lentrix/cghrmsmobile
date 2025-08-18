<script setup>
import { IonTitle, IonCardContent, IonCardTitle, IonCard, IonCardHeader, IonGrid, IonRow, IonCol, IonButton } from '@ionic/vue';
import { IonIcon, IonFab, IonFabButton } from '@ionic/vue';
import { refresh } from 'ionicons/icons';
import axios from 'axios';
import { inject, onMounted, ref } from 'vue';

const user = inject('user');
let host = localStorage.getItem('host') || 'http://localhost:8000/api';

const evaluationsRequirements = ref([]);

const fetchEvalRequirements = async () => {

    axios.get(host + '/evaluation/requirements/', {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    }).then(response => {
            evaluationsRequirements.value = response.data;
        })
        .catch(error => {
            console.log(error)
            toast.error("Failed to retrieve recent logs: " + error.message);
        });
}

const emits = defineEmits(['evaluate']);

const onEvaluate = (requirement) => {
    emits('evaluate', requirement);
};

onMounted(() => {
    fetchEvalRequirements();
});

</script>

<template>
    <ion-fab slot="fixed" vertical="top" horizontal="end" :edge="true">
      <ion-fab-button @click="fetchEvalRequirements">
        <ion-icon :icon="refresh"></ion-icon>
      </ion-fab-button>
    </ion-fab>
    <ion-title class="ion-padding">Evaluation Requirements</ion-title>
    <ion-grid>
        <ion-row v-for="(requirement, index) in evaluationsRequirements" :key="index">
            <ion-col>
                <ion-card>
                    <ion-card-header>
                        <div style="display: flex; justify-content: space-between; align-items: center;">
                            <ion-card-title>{{ requirement.evaluatee }}</ion-card-title>
                            <ion-button fill="outline" size="small" @click="onEvaluate(requirement)">Evaluate</ion-button>
                        </div>
                    </ion-card-header>
                    <ion-card-content>
                        <div>Position: {{ requirement.position }}</div>
                        <div>Deadline: {{ requirement.deadline }}</div>
                    </ion-card-content>
                </ion-card>
            </ion-col>
        </ion-row>
    </ion-grid>
</template>