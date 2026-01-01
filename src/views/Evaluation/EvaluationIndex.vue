<script setup>
import { IonTitle, IonCardContent, IonCardTitle, IonCard, IonCardHeader, IonGrid, IonRow, IonCol, IonButton, IonSelect, IonSelectOption, IonItem, IonLabel } from '@ionic/vue';
import { IonIcon, IonFab, IonFabButton } from '@ionic/vue';
import { refresh } from 'ionicons/icons';
import axios from 'axios';
import { onMounted, ref } from 'vue';
import { useToast } from 'vue-toastification';

const toast = useToast();

// const user = inject('user');
let host = localStorage.getItem('host') || 'http://localhost:8000/api';

const evaluationsRequirements = ref([]);
const performanceAppraisal = ref([]);
const selectedScheduleId = ref(null);


const fetchEvalRequirements = async () => {

    axios.get(host + '/evaluation/requirements/', {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    }).then(response => {
            evaluationsRequirements.value = response.data;
        })
        .catch(error => {
            console.log(error)
            toast.error("Failed to retrieve recent logs: " + error.message);
        });
}
const fetchPerformanceAppraisal = async () => {
    console.log(selectedScheduleId.value);
    axios.get(host + '/perf-appr/' + (selectedScheduleId.value ? selectedScheduleId.value : ''), {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    }).then(response => {
            performanceAppraisal.value = response.data;
        })
        .catch(error => {
            console.log(error)
            toast.error("Failed to retrieve recent logs: " + error.message);
        });
}

const emits = defineEmits(['evaluate']);

const onEvaluate = (requirement) => {
    emits('evaluate', requirement);
};

const fetchWithSched = (e) => {
    const scheduleId = e.detail.value;

    axios.get(host + '/evaluation/requirements/' + scheduleId, {
        headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
        }
    }).then(response => {
        evaluationsRequirements.value = response.data;
    })
    .catch(error => {
        console.log(error)
        toast.error("Failed to retrieve recent logs: " + error.message);
    });

    fetchPerformanceAppraisal(scheduleId);
}

onMounted(() => {
    fetchEvalRequirements();
    fetchPerformanceAppraisal();
});

</script>

<template>
    <IonFab slot="fixed" vertical="top" horizontal="end" :edge="true">
      <IonFab-button @click="fetchEvalRequirements">
        <IonIcon :icon="refresh"></IonIcon>
      </IonFab-button>
    </IonFab>
    <div>
        <IonCard class="ion-margin">
            <IonCardHeader>
                <IonCardTitle>Select Schedule</IonCardTitle>
            </IonCardHeader>
            <IonCardContent>
                <IonGrid>
                    <IonRow>
                        <IonCol>
                            <IonItem>
                                <IonLabel>Select Schedule</IonLabel>
                                <IonSelect v-model="selectedScheduleId" placeholder="Choose a schedule" @ion-change="(e)=>fetchWithSched(e)">
                                    <IonSelectOption v-for="schedule in evaluationsRequirements.schedules" :key="schedule.id" :value="schedule.id">
                                        {{ schedule.title }}
                                    </IonSelectOption>
                                </IonSelect>
                            </IonItem>
                        </IonCol>
                    </IonRow>
                </IonGrid>
            </IonCardContent>
        </IonCard>
    </div>
    <div>
        <IonTitle class="ion-padding">Evaluation Requirements</IonTitle>
        <IonCard v-if="evaluationsRequirements?.evaluation?.length==0" class="ion-margin">
            <IonCardHeader>
                <IonCardTitle>No Evaluation Requirements</IonCardTitle>
            </IonCardHeader>
            <IonCardContent>
                There are currently no evaluation requirements remaining assigned to you.
            </IonCardContent>
        </IonCard>
        <IonGrid>
            <IonRow v-for="(requirement, index) in evaluationsRequirements.evaluation" :key="index">
                <IonCol>
                    <IonCard>
                        <IonCardHeader>
                            <div style="display: flex; justify-content: space-between; align-items: center;">
                                <IonCardTitle>{{ requirement.evaluatee }}</IonCardTitle>
                                <IonButton fill="outline" size="small" @click="onEvaluate(requirement)">Evaluate</IonButton>
                            </div>
                        </IonCardHeader>
                        <IonCardContent>
                            <div>Position: {{ requirement.position }}</div>
                            <div>Deadline: {{ requirement.deadline }}</div>
                        </IonCardContent>
                    </IonCard>
                </IonCol>
            </IonRow>
        </IonGrid>
    </div>
    <div v-if="evaluationsRequirements?.evaluation?.length==0">
        <IonCard class="ion-margin">
            <IonCardHeader>
                <IonCardTitle>My Personality Evaluation</IonCardTitle>
            </IonCardHeader>
            <IonCardContent v-if="evaluationsRequirements.message">
                {{ evaluationsRequirements.message }}
            </IonCardContent>
            <IonCardContent v-else>
                <IonGrid>
                    <IonRow class="table-header">
                        <IonCol size="9">Item</IonCol>
                        <IonCol size="3">Average Rating</IonCol>
                    </IonRow>
                    <IonRow v-for="(item, index) in evaluationsRequirements.my_personality_eval?.per_item_average" :key="item.item_id" :class="index % 2 === 0 ? 'stripe-even' : 'stripe-odd'">
                        <IonCol size="9">
                            {{ item.description }}
                        </IonCol>
                        <IonCol size="3">
                            {{ item.average_rating }}
                        </IonCol>
                    </IonRow>
                    <IonRow style="font-size: 1.4em">
                        <IonCol size="9" class="table-header">
                            Overall Average
                        </IonCol>
                        <IonCol size="3" class="table-header">
                            {{ evaluationsRequirements.my_personality_eval?.average_rating }}
                        </IonCol>
                    </IonRow>
                </IonGrid>
            </IonCardContent>
        </IonCard>
        
    </div>
    <div v-else>
        <IonCard class="ion-margin">
            <IonCardHeader>
                <IonCardTitle>My Personality Evaluation</IonCardTitle>
            </IonCardHeader>
            <IonCardContent>
                Your personality evaluation will be displayed here once you have completed all your assigned evaluations.
            </IonCardContent>
        </IonCard>
    </div>

    <div>
        <IonTitle>Performance Appraisal</IonTitle>
        <IonCard v-if="performanceAppraisal.perf_appr==null" class="ion-margin">
            <IonCardHeader>
                <IonCardTitle>No Performance Appraisal</IonCardTitle>
            </IonCardHeader>
            <IonCardContent>
                There is currently no performance appraisal available for you.
            </IonCardContent>
        </IonCard>
        <div v-else>
            <IonCard v-for="kra in performanceAppraisal.perf_appr.kras" :key="kra.id">
                <IonGrid>
                    <IonRow>
                        <IonCol size="8" class="table-header">
                            <strong>{{ kra.kra_template.description }}</strong>
                        </IonCol>
                        <IonCol class="table-header" size="2">Acc</IonCol>
                        <IonCol size="2" class="table-header">
                            {{ kra.weight }}
                        </IonCol>
                    </IonRow>
                </IonGrid>
                <IonGrid>
                    <IonRow v-for="kpri in kra.kpris" :key="kpri.id">
                        <IonCol size="8">
                            <strong>{{ kpri.substituted_description }}</strong>
                        </IonCol>
                        <IonCol size="2">{{ kpri.accomplishment }}</IonCol>
                        <IonCol size="2">
                            {{ kpri.computed_rating.toFixed(2) }}
                        </IonCol>
                    </IonRow>
                </IonGrid>
                <IonGrid>
                    <IonRow>
                        <IonCol size="8" class="table-header">
                            <strong>Computed Weight Value ({{ kra.weight }} x {{ kra.kpris[0].computed_rating.toFixed(2)*100 }}%)</strong>
                        </IonCol>
                        <IonCol size="2" class="table-header"></IonCol>
                        <IonCol size="2" class="table-header">
                            {{ kra.computed_weight.toFixed(2) }}
                        </IonCol>
                    </IonRow>
                </IonGrid>
            </IonCard>

            <IonCard>
                <IonGrid>
                    <IonRow style="font-size: 1.3em">
                        <IonCol size="8" class="table-header">
                            <strong>Overall Performance Rating</strong>
                        </IonCol>
                        <IonCol size="2" class="table-header"></IonCol>
                        <IonCol size="2" class="table-header">
                            {{ performanceAppraisal.perf_appr.overall_rating.toFixed(2)*100 }}%
                        </IonCol>
                    </IonRow>
                    <IonRow style="font-size: 1.3em">
                        <IonCol class="table-header">
                            <strong>Interpretation</strong>
                        </IonCol>
                        <IonCol size="2" class="table-header"></IonCol>
                        <IonCol class="table-header" :style="{ color: performanceAppraisal.perf_appr.interpretation_color }">
                            {{ performanceAppraisal.perf_appr.interpretation }}
                        </IonCol>
                    </IonRow>
                </IonGrid>
            </IonCard>
        </div>
    </div>
</template>

<style>
.table-header {
    font-weight: bold;
    background-color: #2d2d2d;
    color: #ffffff;
}
.stripe-even {
    background-color: #1e1e1e;
    color: #ffffff;
}
.stripe-odd {
    background-color: #2a2a2a;
    color: #ffffff;
}
</style>