<template>
    <ion-modal :is-open="isOpen">
        <ion-header>
            <ion-toolbar>
                <ion-title>Settings</ion-title>
                <ion-buttons slot="end">
                    <ion-button @click="$emit('close')">Close</ion-button>
                </ion-buttons>
            </ion-toolbar>
        </ion-header>
        <ion-content class="ion-padding">
            <ion-list>
                <ion-item>
                    <ion-input label="API Host" v-model="settings.apiHost" label-placement="floating" placeholder="Enter API Host URL"></ion-input>
                </ion-item>
            </ion-list>

            <ion-button expand="full" @click="saveSettings" class="ion-margin-top">
                Save Settings
            </ion-button>
        </ion-content>
    </ion-modal>
</template>

<script setup>
import {IonToolbar, IonHeader, IonTitle, IonButtons, IonButton, IonContent, IonModal, IonList, IonItem, IonInput } from '@ionic/vue';
import { ref,inject } from 'vue';

const props = defineProps({
    isOpen: {
        type: Boolean,
        required: true
    }
})

const emit = defineEmits(['close'])
const host = localStorage.getItem('host') || 'http://localhost:8000/api'

const settings = ref({
    apiHost: localStorage.getItem('host') || 'http://localhost:8000/api'
})

const saveSettings = () => {
    localStorage.setItem('host', settings.value.apiHost)
    emit('close')
}

</script>

<style scoped>

</style>