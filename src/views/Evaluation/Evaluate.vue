<script setup>
import { inject, onMounted, ref } from 'vue';
import { IonTitle, IonCardContent, IonCardTitle, IonCard, IonCardHeader, IonTextarea, IonButton } from '@ionic/vue';
import { IonIcon } from '@ionic/vue';
import { arrowBack } from 'ionicons/icons';
import { defineEmits } from 'vue';
import axios from 'axios';
import EvalItem from '../../components/EvalItem.vue';
import { useToast } from 'vue-toastification';

const props = defineProps({
    evaluation: {
        type: Object,
        required: true
    }
});

const toast = useToast();

const user = inject('user');
const host = localStorage.getItem('host') || 'http://localhost:8000/api';

const evalItems = ref([]);
const comments = ref('');

const fetchEvalItems = async () => {
    try {
        const response = await axios.get(`${host}/evaluation/items/${props.evaluation.id}`, {
            headers: {
                Authorization: `Bearer ${localStorage.getItem('access_token')}`
            }
        });
        // Handle the response data as needed

        evalItems.value = response.data;
    } catch (error) {
        console.error("Failed to fetch evaluation items:", error);
    }
};

onMounted(() => {
    fetchEvalItems();
});

const emits = defineEmits(['back']);

const onRate = (item, rating) => {
    // Emit the rating event with itemId and rating
    item.rating = rating; // Update the item's rating locally
};

const onSubmit = () => {
    // Handle the submission of the evaluation
    axios.post(`${host}/evaluation/submit`, {
        evaluation_id: props.evaluation.id,
        items: evalItems.value.items.map(item => ({
            id: item.id,
            rating: item.rating
        })),
        comments: comments.value
    }, {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    }).then(response => {
        console.log("Evaluation submitted successfully:", response.data);
        toast.success("Evaluation submitted successfully!");
        emits('back'); // Emit back event to return to the index page
    }).catch(error => {
        console.error("Failed to submit evaluation:", error);
        toast.error("Failed to submit evaluation: " + error.message);
    });
};

</script>

<template>
    <ion-card>
        <ion-card-header>
            <div style="display: flex; justify-content: space-between; align-items: center;">
                <ion-card-title>{{ props.evaluation.evaluatee }}</ion-card-title>
                <ion-button fill="outline" size="small" @click="$emit('back')">
                    <ion-icon :icon="arrowBack"></ion-icon>
                </ion-button>
            </div>
        </ion-card-header>
        <ion-card-content>
            <div>Position: {{ props.evaluation.position }}</div>
            <div>Deadline: {{ props.evaluation.deadline }}</div>
        </ion-card-content>
    </ion-card>

    <hr>

    <ion-title class="ion-padding">Evaluation Items</ion-title>

    <EvalItem v-for="(item) in evalItems.items" :key="item.id" :item="item" :labels="evalItems.labels" @rate="onRate(item, $event)" />

    <hr>

    <ion-card>
        <ion-card-content>
            <div>
                <ion-textarea v-model="comments" placeholder="Add comments (optional)" rows="3"></ion-textarea>
            </div>
        </ion-card-content>
    </ion-card>

    <hr>

    <ion-button expand="full" @click="onSubmit()">Submit Evaluation</ion-button>
</template>