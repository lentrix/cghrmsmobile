<script setup>
import { ref, inject } from 'vue'
import axios from 'axios'
import { IonCardTitle, IonCard, IonCardHeader, IonCardContent, IonLabel, IonInput, IonItem, IonButton, IonContent } from '@ionic/vue'
import { useToast } from 'vue-toastification'


const toast = useToast()

const credentials = ref({
  email: '',
  password: ''
})

const host = inject('host')
const user = inject('user')

const login = async () => {
    try {
        const response = await axios.post(`${host}/login`, {
            email: credentials.value.email,
            password: credentials.value.password
        })
        // handle successful login, e.g., save token or redirect

        user.value = response.data.user // assuming the API returns user data
        localStorage.setItem('access_token', response.data.access_token) // save token to local storage
        localStorage.setItem('user', JSON.stringify(response.data.user)) // save user data to local storage

        console.log('Login successful:', response.data)
    } catch (error) {
        // handle error, e.g., show error message
        toast.error('Login failed. ' + error.response?.data?.message)
    }
}

</script>

<template>
  <ion-content class="ion-padding">
    <ion-card>
      <ion-card-header>
        <ion-card-title>Login</ion-card-title>
      </ion-card-header>
      <ion-card-content>
        <ion-item>
          <ion-label position="stacked">Email</ion-label>
          <ion-input v-model="credentials.email" type="text"></ion-input>
        </ion-item>
        <ion-item>
          <ion-label position="stacked">Password</ion-label>
          <ion-input v-model="credentials.password" type="password"></ion-input>
        </ion-item>
        <div class="login-button">
          <ion-button class="my-4" expand="full" @click="login">Login</ion-button>
        </div>
      </ion-card-content>
    </ion-card>
  </ion-content>
</template>