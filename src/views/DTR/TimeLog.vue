<script setup>
import { IonContent, IonHeader, IonToolbar, IonTitle, IonButton, IonCard } from '@ionic/vue';
import axios from 'axios';
import { inject, onMounted, onUnmounted, ref } from 'vue';
import { useToast } from 'vue-toastification';
import { QrcodeStream } from 'vue3-qrcode-reader';


const emit = defineEmits(['back']);
const toast = useToast()
const isOnline = ref(false);
const host = inject('host');
const user = inject('user');
var interval = null;
const coords = ref(null);
const timestamp = ref(null);

const props = defineProps({
    logType: {
        type: String,
        required: true
    },
    logPeriod: {
        type: String,
        required: true
    }
});

const onDecode = (decodedString) => {
    
    const qrObject = validateQRCode(decodedString)

    if(!qrObject) {
        toast.error("Invalid QR Code")
        emit('back')
    }
    
    const proximity = computeProximity(qrObject.lat, qrObject.lon, coords.value.lat, coords.value.lon)

    if (proximity > 100) {
        toast.error('You are too far from the location: ' + proximity + " meters away.")
        emit('back')
        return
    }

    const d = new Date(timestamp.value)
    const checkTime = d.getFullYear() + "-" + ("00" + (d.getMonth()+1)).slice(-2) + "-" + ("00" + d.getDate()).slice(-2) 
        + " " + d.getHours() + ":" + ("00" + d.getMinutes()).slice(-2) + ":" + ("00" + d.getSeconds()).slice(-2)
    
    console.log("User: ", user.value)
    const data = {
        user_id: user.value.id,
        location_id: qrObject.id,
        check_time: checkTime,
        check_type: props.logPeriod + "_" + props.logType,
        location: qrObject.name,
        serial: Date.now(),
    }
    console.log("Data: ", data)

    if(isOnline.value) {
        postTimeLog(data)
    }else {
        // let logs = JSON.parse(localStorage.getItem('timeLogs')) || []
        // logs.push(data)
        // localStorage.setItem('timeLogs', JSON.stringify(logs))
        // toast.success('Successfully logged TIME ' + props.logType + ' ' + checkTime +  ' @ ' + qrObject.name + ' (Offline)')
        toast.error('Sorry! It seems you are offline.')
    }

    emit('back')
}

const validateQRCode = (decodedString) => {
    try{
        const qrObject = JSON.parse(decodedString) ? JSON.parse(decodedString) : false

        return (qrObject && qrObject.id && qrObject.name && qrObject.lon && qrObject.lat) ? qrObject : false;

    }catch(e) {
        console.log(e)
        return false;
    }

}

const computeProximity = (locLat, locLon, devLat, devLon) => {
  const toRad = (value) => (value * Math.PI) / 180;
  const R = 6371e3; // Earth's radius in meters

  const lat1 = toRad(locLat);
  const lon1 = toRad(locLon);
  const lat2 = toRad(devLat);
  const lon2 = toRad(devLon);

  const dLat = lat2 - lat1;
  const dLon = lon2 - lon1;

  const a =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(lat1) * Math.cos(lat2) *
    Math.sin(dLon / 2) * Math.sin(dLon / 2);

  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  return R * c; // in meters
}

onMounted(async () => {
    navigator.geolocation.getCurrentPosition((position) => {
        coords.value = {
            lon: position.coords.longitude,
            lat: position.coords.latitude
        }
        timestamp.value = position.timestamp
    }, (error) => {
        toast.error('Error getting location:', error);
    });

    interval = setInterval(() => {
        checkConnection();
    }, 3000);
});

const checkConnection = async () => {
    try {
        const response = await axios.get(host + '/test');
        if (response.status === 200) {
            isOnline.value = true;
        } else {
            isOnline.value = false;
        }
    } catch (error) {
        isOnline.value = false;
    }
};

const postTimeLog = async (data) => {
    try {
        const response = await axios.post(host + '/dtr/timelog', data, {
            headers: {
                Authorization: `Bearer ${localStorage.getItem('access_token')}`
            }
        });
        toast.success('Successfully logged TIME ' + props.logType + ' ' + data.check_time + ' @ ' + data.location + ' (Online)')
    } catch (error) {
        console.log(error.response?.data)
        toast.error('Error logging time: ' + error.response?.data?.error);
    }
}

onUnmounted(() => {
    if (interval) {
        clearInterval(interval);
        interval = null;
    }
})

</script>

<template>
    <ion-content class="ion-padding">
        <ion-header>
            <ion-toolbar>
                <div style="display: flex; justify-content: space-between; align-items: center;">
                    <ion-title>
                        TIME {{ logType }} {{ logPeriod }}
                        <span style="color: #00ff00" v-if="isOnline">(Online)</span>
                        <span style="color: #ff1100" v-else>(Offline)</span>
                    </ion-title>
                    <ion-button fill="solid" color="primary" @click="$emit('back')">Back</ion-button>
                </div>
            </ion-toolbar>
        </ion-header>


        <ion-card class="ion-padding">
            <template v-if="coords">
                <ion-content style="height: 400px">
                    <qrcode-stream @decode="onDecode"></qrcode-stream>
                </ion-content>
                <div>Device Coordinates: {{ coords.lat }}, {{ coords.lon }} Timestamp: {{ new Date(timestamp) }}</div>
            </template>
            <div v-else class="blinking-text">Detecting Device position...</div>
            
        </ion-card>

    </ion-content>
</template>

<style scoped>
    .blinking-text {
        animation: blink 0.6s steps(2, start) infinite;
    }
    @keyframes blink {
        to {
            visibility: hidden;
        }
    }
</style>