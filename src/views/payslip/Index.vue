<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Pay Slip View</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
        <ion-card>
            <ion-card-header>
                <ion-card-title>
                    {{ payroll ? 'Basic' : 'Loading...' }}
                </ion-card-title>
            </ion-card-header>
            <ion-card-content>
                <div v-if="payroll">
                    <ion-grid>
                        <ion-row>
                            <ion-col size="5"><strong>Period:</strong></ion-col>
                            <ion-col>
                                {{ formatDate(payroll.pay_period.start_date) }} to {{ formatDate(payroll.pay_period.end_date) }}
                            </ion-col>
                        </ion-row>
                        <ion-row>
                            <ion-col size="5"><strong>Pay Date:</strong></ion-col>
                            <ion-col>
                                {{ formatDate(payroll?.pay_period?.pay_date) }}
                            </ion-col>
                        </ion-row>
                        <ion-row>
                            <ion-col size="5"><strong>Hours Worked:</strong></ion-col>
                            <ion-col>
                                {{ formatHoursMinutes(payroll.minutes_worked) }}
                            </ion-col>
                        </ion-row>
                        <ion-row>
                            <ion-col size="5"><strong>Gross Pay:</strong></ion-col>
                            <ion-col class="ion-text-right">
                                <strong>{{ (payroll.gross_pay*1).toLocaleString('en-US', { style: 'currency', currency: 'PHP' }) }}</strong>
                            </ion-col>
                        </ion-row>
                    </ion-grid>
                </div>
            </ion-card-content>
        </ion-card>
        <ion-card v-if="payroll && payroll.deductions && payroll.deductions.length">
            <ion-card-header>
                <ion-card-title>Deductions</ion-card-title>
            </ion-card-header>
            <ion-card-content>
                <ion-grid>
                    <ion-row v-for="(deduction, idx) in payroll.deductions" :key="idx">
                        <ion-col size="7">{{ deduction.name }}</ion-col>
                        <ion-col size="5" class="ion-text-right">{{ deduction.amount }}</ion-col>
                    </ion-row>
                    <ion-row>
                        <ion-col size="7"><strong>Total Deductions</strong></ion-col>
                        <ion-col size="5" class="ion-text-right">
                            <strong>
                                {{
                                    payroll.deductions.reduce((sum, d) => sum + Number(d.amount), 0).toLocaleString('en-US', { style: 'currency', currency: 'PHP' })
                                }}
                            </strong>
                        </ion-col>
                    </ion-row>
                </ion-grid>
            </ion-card-content>
        </ion-card>

        <ion-card v-if="payroll">
            <ion-card-header>
                <ion-card-title>
                    <ion-grid>
                        <ion-row>
                            <ion-col size="7"><strong>Net Pay</strong></ion-col>
                            <ion-col size="5" class="ion-text-right">
                                <strong>{{ payroll.net_pay.toLocaleString('en-US', { style: 'currency', currency: 'PHP' }) }}</strong>
                            </ion-col>
                        </ion-row>
                    </ion-grid>
                </ion-card-title>
            </ion-card-header>
        </ion-card>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonCard, IonCardHeader, IonCardContent, IonCardTitle, IonCol, IonGrid, IonRow } from '@ionic/vue';
import axios from 'axios';
import { ref, watch, inject, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import { useToast } from 'vue-toastification';

const toast = useToast()
const host = localStorage.getItem('host') || 'http://localhost:8000/api'

const route = useRoute();
let payroll = ref(null);

const fetchPayroll = async () => {

    axios.get(host + '/payroll/' + route.query.id, {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    }).then(response => {
            payroll.value = response.data;
        })
        .catch(error => {
            console.log(error)
            toast.error("Failed to retrieve recent logs: " + error.message);
        });
}

const formatDate = (dateString) => {
    if (!dateString) return '';
    const date = new Date(dateString);
    return date.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' });
};

const formatHoursMinutes = (minutes) => {
    if (typeof minutes !== 'number' || isNaN(minutes)) return '';
    const hrs = Math.floor(minutes / 60);
    const mins = minutes % 60;
    return `${hrs}:${mins.toString().padStart(2, '0')}`;
};

onMounted(()=>{
    fetchPayroll(route.query.id)
})

watch(() => route.query.id, (newId) => {
    if(newId) {
        payroll.value = null
        fetchPayroll(newId)
    }
}, { immediate: true });
</script>
