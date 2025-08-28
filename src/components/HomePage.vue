<script setup>
import { IonIcon, IonCard, IonCol, IonCardHeader, IonCardTitle, IonCardContent, IonButton, IonRow, IonGrid } from '@ionic/vue';
import { calendar, mic } from 'ionicons/icons';
import { inject, onMounted, ref } from 'vue'
import axios from 'axios'
import { useToast } from 'vue-toastification'

const toast = useToast()

const host = localStorage.getItem('host') || 'http://localhost:8000/api'
const user = inject('user')

const events = ref([])
const announcements = ref([])

const logout = async () => {

    try {
        await axios.post(`${host}/logout`, {}, {
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

onMounted(() => {
    // get upcoming calendar events
    axios.get(`${host}/home-data`, {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    })
    .then(response => {
        events.value = response.data.events || []
        announcements.value = response.data.announcements || []
    })
    .catch(error => {
        toast.error('Failed to fetch user information. ' + error.response?.data?.message)
    })
})

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

    <div>

        <h3 class="ion-padding">
            Upcoming Events
        </h3>

        <div v-if="events.size==0">No upcoming events.</div>
        <template v-else>

            <ion-card v-for="event in events" :key="event.id" class="ion-margin-bottom" color="tertiary">
                <ion-card-header>
                    <ion-card-title>
                        <ion-icon :icon="calendar" />
                        {{ event.title }}
                    </ion-card-title>
                </ion-card-header>
                <ion-card-content>
                    <p>
                        {{
                            new Date(event.start).toLocaleString('en-US', {
                                month: 'short',
                                day: 'numeric',
                                year: 'numeric',
                                hour: 'numeric',
                                minute: '2-digit',
                                hour12: true
                            })
                        }}
                        -
                        {{
                            new Date(event.end).toLocaleString('en-US', {
                                month: 'short',
                                day: 'numeric',
                                year: 'numeric',
                                hour: 'numeric',
                                minute: '2-digit',
                                hour12: true
                            })
                        }}
                    </p>
                </ion-card-content>
            </ion-card>

        </template>

    </div>

    <div>

        <h3 class="ion-padding">Announcements</h3>

        <div v-if="events.size==0">No announcements.</div>
        <template v-else>

            <ion-card v-for="announcement in announcements" :key="announcement.id" class="ion-margin-bottom" color="success">
                <ion-card-header>
                    <ion-card-title>
                        <ion-icon :icon="mic"></ion-icon>
                        {{ announcement.title }}
                    </ion-card-title>
                </ion-card-header>
                <ion-card-content>
                    <p>{{ announcement.description }}</p>
                </ion-card-content>
            </ion-card>

        </template>

    </div>

    <ion-button expand="full" class="ion-padding" @click="logout">Logout</ion-button>
</template>