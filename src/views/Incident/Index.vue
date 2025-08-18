<template>

    <ion-fab slot="fixed" vertical="bottom" horizontal="end">
      <ion-fab-button color="success" @click="fetchReports()">
        <ion-icon :icon="refresh"></ion-icon>
      </ion-fab-button>
    </ion-fab>

    <div style="display: flex; justify-content: space-between; align-items: center;">
        <ion-title class="ion-padding">Reported Incidents</ion-title>
        <ion-button @click="$emit('changePage', 'create')">
            <ion-icon :icon="add" />
        </ion-button>
    </div>

    <div>
        <IncidentCard
            v-for="incident in incidents"
            :key="incident.id"
            :incident="incident"
            @open="openIncident(incident)"
        />
    </div>

    <hr />

    <ion-title class="ion-padding">My Incidents</ion-title>
    
    <div>
        <IncidentCard
            v-for="incident in myIncidents"
            :key="incident.id"
            :incident="incident"
            @open="openIncident(incident)"
        />
    </div>


</template>

<script setup>
import { IonFab, IonFabButton, IonIcon, IonTitle, IonCard, IonCardHeader, IonCardTitle, IonCardContent, IonButton } from '@ionic/vue';
import { add, open, refresh } from 'ionicons/icons';
import { inject, onMounted, ref } from 'vue';
import axios from 'axios';
import { useToast } from 'vue-toastification';
import IncidentCard from '@/components/IncidentCard.vue';

const toast = useToast();

const user = inject('user');
const host = localStorage.getItem('host') || 'http://localhost:8000/api';

const incidents = ref([]);
const myIncidents = ref([]);

const fetchReports = async () => {
    axios.get(host + '/incident-reports/index', {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    }).then(response => {
            incidents.value = response.data.incidents;
            myIncidents.value = response.data.my_incidents;
        })
        .catch(error => {
            console.log(error);
            toast.error("Failed to retrieve incident reports: " + error.message);
        });
}

const openIncident = (incident) => {
    emits('changePage', 'open', incident);
};

const emits = defineEmits(['changePage']);

onMounted(() => {
    fetchReports();
});


</script>