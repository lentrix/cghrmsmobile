<script setup>
import { IonContent, IonCard, IonCol, IonCardHeader, IonCardTitle, IonCardContent, IonItem, IonLabel, IonInput, IonButton, IonRow, IonGrid } from '@ionic/vue';
import { inject } from 'vue'
import axios from 'axios'
import { useToast } from 'vue-toastification'

const toast = useToast()

const host = inject('host')
const user = inject('user')

const logout = async () => {

    try {
        const response = await axios.post(`${host}/logout`, {}, {
            headers: {
                Authorization: `Bearer ${localStorage.getItem('access_token')}`
            }
        })
        
        toast.success('Logout successful')

    } catch (error) {
        toast.error('Logout failed. ' + error.response?.data?.message)
    }

    localStorage.removeItem('access_token')
    localStorage.removeItem('user')
    user.value = null
}

</script>

<template>        
    <ion-card>
        <ion-card-header>
            <ion-card-title>User Information</ion-card-title>
        </ion-card-header>
        <ion-card-content>
            <ion-grid>
                <ion-row style="border-bottom: 1px solid #ccc;">
                    <ion-col size="4"><strong>ID</strong></ion-col>
                    <ion-col>{{ user.id }}</ion-col>
                </ion-row>
                <ion-row style="border-bottom: 1px solid #ccc;">
                    <ion-col size="4"><strong>Name</strong></ion-col>
                    <ion-col>{{ user.name }}</ion-col>
                </ion-row>
                <ion-row style="border-bottom: 1px solid #ccc;">
                    <ion-col size="4"><strong>Email</strong></ion-col>
                    <ion-col>{{ user.email }}</ion-col>
                </ion-row>
            </ion-grid>
        </ion-card-content>
    </ion-card>

    <ion-card>
        <ion-card-header>
            <ion-card-title>Employee Information</ion-card-title>
        </ion-card-header>
        <ion-card-content>
            <ion-grid>
                <ion-row style="border-bottom: 1px solid #ccc;">
                    <ion-col size="4"><strong>Employee ID</strong></ion-col>
                    <ion-col>{{ user.employee_info?.id }}</ion-col>
                </ion-row>
                <ion-row style="border-bottom: 1px solid #ccc;">
                    <ion-col size="4"><strong>Full Name</strong></ion-col>
                    <ion-col>{{ user.employee_info?.last_name }}, {{ user.employee_info?.first_name }}</ion-col>
                </ion-row>
                <ion-row style="border-bottom: 1px solid #ccc;">
                    <ion-col size="4"><strong>Position</strong></ion-col>
                    <ion-col>{{ user.employee_info?.position }}</ion-col>
                </ion-row>
            </ion-grid>
        </ion-card-content>
    </ion-card>

    <ion-button expand="full" class="ion-padding" @click="logout">Logout</ion-button>
</template>