<script setup>
import { IonContent, IonButton, IonGrid, IonRow, IonCol, IonIcon, IonFab, IonFabButton } from '@ionic/vue';
import { refresh } from 'ionicons/icons';
import axios from 'axios';
import { onMounted, ref, inject } from 'vue';
import { useToast } from 'vue-toastification';

const records = ref()

const user = inject('user')
const host = inject('host')

const toast = useToast()

const isOffline = ref(true)
const recentLogs = ref([])

const emits = defineEmits(['logTime'])

const onUpload = () => {
    // retrieveLocalLogs()

    records.value.forEach(record => {
        const data = {
            user_id: user.value.id,
            location_id: record.location_id,
            check_time: record.check_time,
            check_type: record.check_type,
            serial: record.serial,
        }

        axios.post(host + '/dtr/timelog', data, {
            headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
            }
        })
            .then(response => {
                console.log("Upload successful:", response.data);
                removeLocalLog(record.serial);
                retrieveRecentLogs()

            })
            .catch(error => {
                toast.error("Upload failed: " + error.message);
                return;
            });
    });
}

const onDownload = () => {
    retrieveRecentLogs()
}

// const retrieveLocalLogs = () => {
//     const logs = localStorage.getItem('timeLogs') ? JSON.parse(localStorage.getItem('timeLogs')) : []
//     records.value = logs.filter(log => log.user_id === user.value.id)
// }

// const removeLocalLog = (serial) => {
//     let logs = JSON.parse(localStorage.getItem('timeLogs')) || []
//     logs = logs.filter(log => log.serial !== serial)
//     localStorage.setItem('timeLogs', JSON.stringify(logs))
//     retrieveLocalLogs()
// }

const retrieveRecentLogs = () => {
    axios.get(host + '/dtr/timelog', {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    })
        .then(response => {
            recentLogs.value = response.data.slice(0, 20);
            isOffline.value = false;
        })
        .catch(error => {
            isOffline.value = true;
            console.log(error)
            toast.error("Failed to retrieve recent logs: " + error.message);
        });
}

onMounted(() => {
    // retrieveLocalLogs()
    retrieveRecentLogs()
})

</script>

<template>
    <ion-content class="ion-padding">
        <ion-grid>
            <ion-row style="align-items: center;">
                <ion-col size="2" style="font-size: 12pt; font-weight: bold;">
                    AM
                </ion-col>
                <ion-col>
                    <ion-button expand="block" color="success" @click="$emit('logTime','AM','IN')">Time In</ion-button>
                </ion-col>
                <ion-col>
                    <ion-button expand="block" color="danger" @click="$emit('logTime','AM','OUT')">Time Out</ion-button>
                </ion-col>
            </ion-row>
            <ion-row style="align-items: center;">
                <ion-col size="2" style="font-size: 12pt; font-weight: bold;">
                    PM
                </ion-col>
                <ion-col>
                    <ion-button expand="block" color="success" @click="$emit('logTime','PM','IN')">Time In</ion-button>
                </ion-col>
                <ion-col>
                    <ion-button expand="block" color="danger" @click="$emit('logTime','PM','OUT')">Time Out</ion-button>
                </ion-col>
            </ion-row>
            <ion-row style="align-items: center;">
                <ion-col size="2" style="font-size: 12pt; font-weight: bold;">
                    OT
                </ion-col>
                <ion-col>
                    <ion-button expand="block" color="success" @click="$emit('logTime','OT','IN')">Time In</ion-button>
                </ion-col>
                <ion-col>
                    <ion-button expand="block" color="danger" @click="$emit('logTime','OT','OUT')">Time Out</ion-button>
                </ion-col>
            </ion-row>
        </ion-grid>

        <!-- <h3>Pending Time Logs</h3>
        <ion-grid>
            <ion-row style="border-bottom: 1px solid #ccc;">
                <ion-col><strong>Log Time</strong></ion-col>
                <ion-col><strong>Log Type</strong></ion-col>
                <ion-col><strong>Location</strong></ion-col>
            </ion-row>
            <ion-row
                v-for="(record, index) in records"
                :key="index"
                style="border-bottom: 1px solid #ccc;"
            >
                <ion-col>{{ record.check_time }}</ion-col>
                <ion-col>{{ record.check_type }}</ion-col>
                <ion-col>{{ record.location }}</ion-col>
            </ion-row>
        </ion-grid>

        <ion-grid>
            <ion-row>
                <ion-col>
                    <ion-button expand="block" color="primary" @click="onUpload()">Upload</ion-button>
                </ion-col>
                <ion-col>
                    <ion-button expand="block" color="success" @click="onDownload()">Download</ion-button>
                </ion-col>
            </ion-row>
        </ion-grid> -->

        <h3>Recent Time Logs (Last 20)</h3>
        <div v-if="isOffline" style="color:red">Unable to fetch data. (You are offline)</div>
        <ion-grid v-else>
            <ion-row style="border-bottom: 1px solid #ccc;">
                <ion-col size="5"><strong>Log Time</strong></ion-col>
                <ion-col size="3"><strong>Log Type</strong></ion-col>
                <ion-col size="4"><strong>Location</strong></ion-col>
            </ion-row>
            <ion-row
                v-for="(recentLog) in recentLogs"
                :key="recentLog.id"
                style="border-bottom: 1px solid #ccc;"
            >
                <ion-col size="5">{{ recentLog.check_time }}</ion-col>
                <ion-col size="3">{{ recentLog.check_type }}</ion-col>
                <ion-col size="4">{{ recentLog.location }}</ion-col>
            </ion-row>
        </ion-grid>

    </ion-content>

    <ion-fab vertical="bottom" horizontal="end" slot="fixed">
        <ion-fab-button color="primary" @click="onDownload">
            <ion-icon :icon="refresh"></ion-icon>
        </ion-fab-button>
    </ion-fab>
</template>

<style scoped>
ion-col {
    font-size: 10pt;
}
</style>