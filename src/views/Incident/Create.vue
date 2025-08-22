<template>

    <ion-fab slot="fixed" vertical="top" horizontal="end" :edge="true">
      <ion-fab-button @click="$emit('changePage', 'index')">
        <ion-icon :icon="arrowBack"></ion-icon>
      </ion-fab-button>
    </ion-fab>

    <ion-title class="ion-padding">Make an Incident Report</ion-title>
    
    <hr />

    <div class="ion-padding">
        <ion-item>
            <ion-select label="Select Employee" type="select" v-model="incidentReport.user_id" labelPlacement="floating" required>
                <ion-select-option v-for="subordinate in subordinates" :key="subordinate.id" :value="subordinate.user_id">
                    {{ subordinate.last_name }}, {{ subordinate.first_name }}
                </ion-select-option>
            </ion-select>
        </ion-item>
        <ion-item>
            <ion-textarea label="Description" rows="3" label-placement="floating" v-model="incidentReport.description" required></ion-textarea>
        </ion-item>
        <ion-item>
            <ion-textarea label="Chronology of Events" rows="5" label-placement="floating" v-model="incidentReport.chronology" required></ion-textarea>
        </ion-item>
        <div style="display: flex; justify-content: stretch; align-items: center;">
            <ion-item style="flex: 1;">
                <ion-input type="date" v-model="incidentReport.date" label="Date of Incident" label-placement="floating" required></ion-input>
            </ion-item>
            <ion-item style="flex: 1;">
                <ion-input type="time" v-model="incidentReport.time" label="Time of Incident" label-placement="floating" required></ion-input>
            </ion-item>
        </div>
        <ion-item>
            <ion-toggle v-model="incidentReport.see_manager">
                See the Manager
            </ion-toggle>
        </ion-item>
        <ion-item>
            <ion-toggle v-model="incidentReport.explain">
                Explain in Writing
            </ion-toggle>
        </ion-item>
        <ion-item>
            <ion-textarea rows="3" type="text" v-model="incidentReport.other_actions" label="Other Actions Required" labelPlacement="floating" />
        </ion-item>

        <ion-item>
            <ion-select label="Select Employee Witnesses" type="select" v-model="incidentReport.witnesses" labelPlacement="floating" :multiple="true" required>
                <ion-select-option v-for="user in users" :key="user.id" :value="user.id">
                    {{ user.name }}
                </ion-select-option>
            </ion-select>
        </ion-item>

        <ion-button expand="full" @click="submitReport">Submit Report</ion-button>

    </div>


</template>

<script setup>
import { IonFab, IonFabButton, IonIcon, IonButton, IonTitle, IonToggle, IonItem, IonSelect, IonInput, IonTextarea, IonSelectOption, IonDatetimeButton, IonModal, IonDatetime, IonLabel } from '@ionic/vue';
import axios from 'axios';
import { arrowBack } from 'ionicons/icons';
import { inject, onMounted, ref } from 'vue';
import { useToast } from 'vue-toastification';

const emits = defineEmits(['changePage'])

const host = localStorage.getItem('host') || 'http://localhost:8000/api';
const user = inject('user');

const toast = useToast();

const incidentReport = ref({
    supervisor_id: user.value.id,
    user_id: '',
    see_manager: false,
    explain: false,
    other_actions: '',
    description: '',
    date: '',
    time: '',
    chronology: '',
    witnesses: []

});

const subordinates =  ref([]);
const users = ref([]);

const fetchSubordinates = () => {
    axios.get(host + '/employees/subordinates', {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    }).then(response => {
            subordinates.value = response.data
        })
        .catch(error => {
            console.log(error);
            toast.error("Failed to retrieve subordinates: " + error.message);
        });
}

const fetchUserList = () => {
    axios.get(host + '/employees/user-list', {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    }).then(response => {
            users.value = response.data
        })
        .catch(error => {
            console.log(error);
            toast.error("Failed to retrieve user list: " + error.message);
        });
}

const submitReport = () => {
    axios.post(host + '/incident-reports', incidentReport.value, {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    }).then(response => {
            console.log(response.data);
            emits('changePage', 'index');
            toast.success("Incident report submitted successfully.");
        })
        .catch(error => {
            console.log(error);
            toast.error("Failed to submit incident report: " + error.message);
        });
}

onMounted(()=>{
    fetchSubordinates()
    fetchUserList()
})

</script>
