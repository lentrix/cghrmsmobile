<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title slot="start">
          <ion-icon aria-hidden="true" :icon="clipboard" />
          Pay Slip
        </ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">

      <ion-item>
        <ion-label>Year</ion-label>
        <ion-select v-model="selectedYear" placeholder="Select Year" @ionChange="getPaySlips()">
          <ion-select-option v-for="year in getYearRange(5)" :key="year" :value="year">
            {{ year }}
          </ion-select-option>
        </ion-select>
      </ion-item>

      <ion-grid>
        <ion-row style="border-bottom: 1px solid #aaa;">
          <ion-col><strong>Pay Date</strong></ion-col>
          <ion-col><strong>Gross Pay</strong></ion-col>
          <ion-col><strong>Net Pay</strong></ion-col>
          <ion-col>
            <ion-icon aria-hidden="true" :icon="cog" />
          </ion-col>
        </ion-row>
        <ion-row
          v-for="slip in paySlips.payrolls"
          :key="slip.id"
          style="border-bottom: 1px solid #aaa;"
        >
            <ion-col>{{ slip.month }}</ion-col>
          <ion-col>{{ slip.gross_pay }}</ion-col>
          <ion-col>{{ slip.net_pay }}</ion-col>
          <ion-col>
            <ion-button
              size="small"
              @click="$router.push({ path: '/tabs/payslip', query: { id: slip.id } })"
            >
              View
            </ion-button>
          </ion-col>
        </ion-row>
      </ion-grid>
      
    </ion-content>
  </ion-page>
</template>

<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonIcon, IonLabel, IonSelectOption, IonSelect, IonItem, IonCol, IonRow, IonGrid, IonButton } from '@ionic/vue';
import { onMounted, ref, inject } from 'vue';
import { clipboard, cog } from 'ionicons/icons';
import axios from 'axios';

const user = inject('user')
const host = localStorage.getItem('host') || 'http://localhost:8000/api'

const selectedYear = ref(new Date().getFullYear())

const paySlips = ref([])

function getYearRange(yearsBack) {
  const currentYear = new Date().getFullYear();
  return Array.from({ length: yearsBack + 1 }, (_, i) => currentYear - yearsBack + i);
}


const getPaySlips = () => {
  axios.get(host + '/pay-slips/' + selectedYear.value, {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    })
        .then(response => {
            paySlips.value = response.data
        })
        .catch(error => {
            isOffline.value = true;
            console.log(error)
            toast.error("Failed to pay slips: " + error.message);
        });
}

onMounted(()=>{

  getPaySlips()
  
})

</script>
