<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Daily Time Record</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      

      <DTRIndex v-if="activePage=='index'" @logTime="logTime"></DTRIndex>
      <TimeLog v-if="activePage=='timelog'" :logType="logType" :logPeriod="logPeriod" @back="activePage='index'" @decoded="handleDecode"></TimeLog>
      
    </ion-content>
  </ion-page>
</template>

<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent  } from '@ionic/vue';
import DTRIndex from '@/views/DTR/DTRIndex.vue';
import TimeLog from '@/views/DTR/TimeLog.vue';
import { ref, inject } from 'vue';
import { useRouter } from 'vue-router';
import { useToast } from 'vue-toastification';

const toast = useToast()
const activePage = ref('index')
const logType = ref(null)
const logPeriod = ref(null)

const router = useRouter()
const user = inject('user')

const handleDecode = (payLoad) => {

  activePage.value = 'index'

  try {
    const loc = JSON.parse(payLoad.location)

    if (loc && loc.id && loc.name && loc.lon && loc.lat) {

      const proximity = computeProximity(loc, payLoad.coords)

      if (proximity > 600) {
        toast.error('You are too far from the location')
        return
      }

      const d = new Date()
      const logTime = d.getFullYear() + "-" + ("00" + (d.getMonth()+1)).slice(-2) + "-" + ("00" + d.getDate()).slice(-2) 
        + " " + d.getHours() + ":" + ("00" + d.getMinutes()).slice(-2) + ":" + ("00" + d.getSeconds()).slice(-2)

      const data = {
        user_id: user.value.id,
        location_id: loc.id,
        check_time: logTime,
        check_type: logType.value,
        location: loc.name,
      }

      let logs = JSON.parse(localStorage.getItem('timeLogs')) || []
      logs.push(data)
      localStorage.setItem('timeLogs', JSON.stringify(logs))
      toast.success('Successfully logged TIME ' + logType.value + ' @ ' + loc.name)
      logType.value = null
      activePage.value = 'index'

    } else {
      // Handle missing properties
      toast.error('Invalid QR Code')
    }

  } catch (e) {
    // Handle JSON parsing error
    toast.error('Invalid QR Code')
    console.error('Error parsing QR code:', e)
    return
  }
} 

const logTime = (period, type) => {
  logType.value = type
  logPeriod.value = period
  activePage.value = 'timelog'
  console.log("Log Type: ", logType.value)
  console.log("Log Period: ", logPeriod.value)  
}

</script>
