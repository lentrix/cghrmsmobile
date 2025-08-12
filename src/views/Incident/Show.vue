<template>
    <ion-fab slot="fixed" vertical="top" horizontal="end" :edge="true">
      <ion-fab-button @click="$emit('changePage', 'index')">
        <ion-icon :icon="arrowBack"></ion-icon>
      </ion-fab-button>
    </ion-fab>

    <ion-title class="ion-padding">View Incident Report</ion-title>

    <ion-card>
        <ion-card-header>
            <ion-card-title>
                {{ incident.user.employee_info.full_name }}
            </ion-card-title>
        </ion-card-header>
        <ion-card-content>
            <div>{{ incident.user.employee_info.position }}</div>
        </ion-card-content>
    </ion-card>

    <ion-card>
        <ion-card-header>
            <ion-card-title>
                {{ incident.date_time }}
            </ion-card-title>
        </ion-card-header>
    </ion-card>

    <ion-card>
        <ion-card-header>
            <ion-card-title>
                Description
            </ion-card-title>
        </ion-card-header>
        <ion-card-content>
            <div>{{ incident.description }}</div>
        </ion-card-content>
    </ion-card>

    <ion-card>
        <ion-card-header>
            <ion-card-title>
                Chronology of Events
            </ion-card-title>
        </ion-card-header>
        <ion-card-content>
            <div>{{ incident.chronology }}</div>
        </ion-card-content>
    </ion-card>

    <ion-card>
        <ion-card-header>
            <ion-card-title>
                Requirements
            </ion-card-title>
        </ion-card-header>
        <ion-card-content>
            <ion-list>
                <ion-item>
                    <ion-label>See the manager</ion-label>
                    <ion-icon :icon="incident.see_manager ? checkmark : closeCircle" />
                </ion-item>
                <ion-item>
                    <ion-label>Explain in writing</ion-label>
                    <ion-icon :icon="incident.explain ? checkmark : closeCircle" />
                </ion-item>
                <ion-item>
                    <div>{{ incident.other_actions }}</div>
                </ion-item>
            </ion-list>
        </ion-card-content>
    </ion-card>

    <ion-card>
        <ion-card-header>
            <ion-card-title>
                Management Comments
            </ion-card-title>
        </ion-card-header>
        <ion-card-content>
            <div>{{ incident.mgt_comments ? incident.mgt_comments : 'none' }}</div>
        </ion-card-content>
    </ion-card>

    <ion-card>
        <ion-card-header>
            <ion-card-title>
                <div>Employee Comments</div>
            </ion-card-title>
        </ion-card-header>
        <ion-card-content>
            <div>{{ incident.employee_comments ? incident.employee_comments : 'none' }}</div>
            <template v-if="user.id==incident.user_id && !incident.employee_comments">
                <ion-button expand="full" id="open-modal" style="margin-top: 8px">
                    <ion-icon :icon="add" />
                    Add Comments
                </ion-button>
                <ion-modal ref="modal" trigger="open-modal">
                    <ion-header>
                        <ion-toolbar>
                            <ion-buttons slot="end">
                                <ion-button @click="cancel()">
                                    <ion-icon :icon="closeCircle"></ion-icon>
                                    &nbsp;Cancel
                                </ion-button>
                            </ion-buttons>
                            <ion-title>You Comments</ion-title>
                            
                        </ion-toolbar>
                    </ion-header>
                    <ion-content class="ion-padding">
                        <ion-item>
                            <ion-textarea 
                                label="Enter your comments" 
                                label-placement="floating" 
                                v-model="comments.employee_comments"
                                rows="8"
                            ></ion-textarea>
                        </ion-item>
                        <ion-button expand="full" @click="submitComments">
                            Submit Comments
                        </ion-button>
                    </ion-content>
                </ion-modal>
            </template>
        </ion-card-content>
    </ion-card>

</template>

<script setup>
import { 
    IonTitle, IonFab, IonLabel, IonFabButton, IonIcon, IonCard, IonCardHeader, IonCardTitle, 
    IonCardContent, IonList, IonItem, IonButton, IonContent, IonTextarea,
    IonHeader, IonToolbar, IonButtons, IonInput, IonModal} from '@ionic/vue';
import { ref, inject } from 'vue';
import { add, arrowBack, checkmark, closeCircle } from 'ionicons/icons';
import axios from 'axios';

const emit = defineEmits(['changePage','updateComments']);

const user = inject('user');
const host = inject('host');

const modal = ref();

const comments = ref({
    employee_comments: ''
});

const props = defineProps({
    incident: {
        type: Object,
        required: true
    }
});

const cancel = () => {
    modal.value.$el.dismiss(null, 'cancel')
}

const submitComments = () => {
    axios.post(host + '/incident-reports/employee-comments', {
        incident_report_id: props.incident.id,
        employee_comments: comments.value.employee_comments
    }, {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    }).then(response => {
        if (response.data.success) {
            modal.value.$el.dismiss(null, 'cancel')
            emit('updateComments', response.data.employee_comments)
        } else {
            console.error("Submit Comments Err: ",response.data.message);
        }
    }).catch(error => {
        console.error(error);
    });
};


</script>
