<script setup>
import { defineProps } from 'vue';
import { IonCard, IonText, IonCardContent, IonButton } from '@ionic/vue';

const props = defineProps({
    item: {
        type: Object,
        required: true
    },
    labels: {
        type: Array,
        required: true
    }
});

const emits = defineEmits(['rate']);

</script>


<template>

<ion-card>
    <ion-card-content>
        <div>
            <ion-text color="dark">{{ props.item.description }}</ion-text>
            <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin-top: 10px;">
                <ion-button size="small" @click="$emit('rate', 4)" :color="item.rating == 4 ? 'primary' : 'medium'">
                    {{ props.labels[0] }}
                </ion-button>
                <ion-button size="small" @click="$emit('rate', 3)" :color="item.rating == 3 ? 'primary' : 'medium'">
                    {{ props.labels[1] }}
                </ion-button>
                <ion-button size="small" @click="$emit('rate', 2)" :color="item.rating == 2 ? 'primary' : 'medium'">
                    {{ props.labels[2] }}
                </ion-button>
                <ion-button size="small" @click="$emit('rate', 1)" :color="item.rating == 1 ? 'primary' : 'medium'">
                    {{ props.labels[3] }}
                </ion-button>
            </div>
        </div>
    </ion-card-content>
</ion-card>

</template>