<template>
    
    <ion-card>
        <ion-card-header>
            <div style="display: flex; justify-content: space-between; align-items:center">
                <ion-card-title>{{ incident.user.employee_info?.full_name }}</ion-card-title>
                <ion-button color="medium" @click="$emit('open')">
                    <ion-icon :icon="open" />
                </ion-button>
            </div>
        </ion-card-header>
        <ion-card-content>
            <div style="background-color: #111; padding: 8px; border-radius: 8px; margin-bottom: 8px;">
                <strong>{{ incident.title }}</strong>
                {{ incident.description }}
            </div>
            <div style="font-style: italic;">
                {{ incident.date_time }}
            </div>
        </ion-card-content>
    </ion-card>
    
</template>

<script setup>
import { IonCard, IonCardHeader, IonCardTitle, IonCardContent, IonButton, IonIcon } from '@ionic/vue';
import { ref } from 'vue';
import { open } from 'ionicons/icons';

const props = defineProps({
    incident: {
        type: Object,
        required: true
    }
})

const emit = defineEmits(['open']);

</script>
