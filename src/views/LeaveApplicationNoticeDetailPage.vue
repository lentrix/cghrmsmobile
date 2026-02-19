<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button default-href="/tabs/leave-form"></ion-back-button>
        </ion-buttons>
        <ion-title>Leave Application Details</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div class="details-content" v-if="application">
        <ion-list>
          <ion-item>
            <ion-label>
              <h3>Employee</h3>
              <p>{{ employeeName }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-label>
              <h3>Date Applied</h3>
              <p>{{ formatDate(application.created_at) }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-label>
              <h3>Type</h3>
              <p>{{ displayType(application) }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-label>
              <h3>Start Date</h3>
              <p>{{ formatDate(application.start_date) }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-label>
              <h3>End Date</h3>
              <p>{{ formatDate(application.end_date) }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-label>
              <h3>With Pay</h3>
              <p>{{ application.with_pay ? 'Yes' : 'No' }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-label>
              <h3>Reason</h3>
              <p>{{ application.reason }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-label>
              <h3>Status</h3>
              <p>{{ displayStatus(application) }}</p>
            </ion-label>
          </ion-item>
        </ion-list>
      </div>
    </ion-content>

    <ion-footer>
      <ion-toolbar>
        <div class="actions-row">
          <ion-button color="primary" @click="forwardToHr" :disabled="processing">
            Forward to HR
          </ion-button>
          <ion-button color="danger" @click="openDenyModal" :disabled="processing">
            Deny Application
          </ion-button>
        </div>
      </ion-toolbar>
    </ion-footer>

    <ion-modal :is-open="isDenyModalOpen" @didDismiss="closeDenyModal">
      <ion-header>
        <ion-toolbar>
          <ion-title>Deny Application</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="closeDenyModal">Close</ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
      <ion-content>
        <div class="modal-content">
          <ion-item>
            <ion-label position="stacked">Reason of Denial</ion-label>
            <ion-textarea v-model="denyReason" :rows="5"></ion-textarea>
          </ion-item>
          <p v-if="denyReasonError" class="field-error">{{ denyReasonError }}</p>

          <ion-button expand="block" @click="submitDenyApplication" :disabled="processing">
            Submit
          </ion-button>
        </div>
      </ion-content>
    </ion-modal>
  </ion-page>
</template>

<script setup>
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonButtons,
  IonBackButton,
  IonList,
  IonItem,
  IonLabel,
  IonFooter,
  IonButton,
  IonModal,
  IonTextarea,
  toastController
} from '@ionic/vue';
import axios from 'axios';
import { computed, onMounted, ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const host = localStorage.getItem('host') || 'http://localhost:8000/api';

const route = useRoute();
const router = useRouter();

const application = ref(null);
const processing = ref(false);
const isDenyModalOpen = ref(false);
const denyReason = ref('');
const denyReasonError = ref('');

const employeeName = computed(() => {
  const employeeInfo = application.value?.user?.employee_info;

  if (!employeeInfo) {
    return '';
  }

  return `${employeeInfo.last_name}, ${employeeInfo.first_name}`;
});

const headers = () => ({
  Authorization: `Bearer ${localStorage.getItem('access_token')}`
});

const formatDate = (value) => {
  if (!value) return '';

  return new Date(value).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'short',
    day: '2-digit'
  });
};

const displayType = (item) => {
  if (item.type === 'Others' && item.other_type) {
    return item.other_type;
  }

  return item.type || '';
};

const displayStatus = (item) => {
  if (item.status) {
    return item.status;
  }

  if (item.denied_at) {
    return 'Denied';
  }

  if (item.approved_at) {
    return 'Approved';
  }

  if (item.recommended_at) {
    return 'Recommended';
  }

  if (item.noted_at) {
    return 'Noted';
  }

  return 'Pending';
};

const fetchApplication = async () => {
  const response = await axios.get(`${host}/leave-applications/${route.params.id}`, {
    headers: headers()
  });

  application.value = response.data;
};

const showToast = async (message, color = 'success') => {
  const toast = await toastController.create({
    message,
    duration: 1500,
    color,
    position: 'bottom'
  });

  await toast.present();
  await toast.onDidDismiss();
};

const forwardToHr = async () => {
  if (processing.value) return;

  processing.value = true;

  try {
    await axios.post(`${host}/leave-applications/${route.params.id}/forward-to-hr`, {}, {
      headers: headers()
    });

    await showToast('Application forwarded to HR.');
    router.replace('/tabs/leave-form');
  } finally {
    processing.value = false;
  }
};

const openDenyModal = () => {
  denyReasonError.value = '';
  isDenyModalOpen.value = true;
};

const closeDenyModal = () => {
  isDenyModalOpen.value = false;
};

const submitDenyApplication = async () => {
  if (processing.value) return;

  if (!denyReason.value.trim()) {
    denyReasonError.value = 'Reason of denial is required.';
    return;
  }

  processing.value = true;

  try {
    await axios.post(`${host}/leave-applications/${route.params.id}/deny`, {
      reason: denyReason.value.trim()
    }, {
      headers: headers()
    });

    closeDenyModal();
    await showToast('Application denied successfully.');
    router.replace('/tabs/leave-form');
  } finally {
    processing.value = false;
  }
};

onMounted(() => {
  fetchApplication();
});
</script>

<style scoped>
.details-content {
  padding: 12px;
}

.actions-row {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
  padding: 8px 12px;
}

.modal-content {
  padding: 16px;
}

.field-error {
  margin: 6px 0 12px;
  color: var(--ion-color-danger);
  font-size: 0.85rem;
}
</style>
