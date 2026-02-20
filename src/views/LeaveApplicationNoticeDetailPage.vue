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
        <div class="actions-row" v-if="showFooterActions">
          <ion-button v-if="canForwardToHr" color="primary" @click="forwardToHr" :disabled="processing">
            Forward to HR
          </ion-button>
          <ion-button v-if="canDenyApplication" color="danger" @click="openDenyModal" :disabled="processing">
            Deny Application
          </ion-button>
          <ion-button v-if="canRecommendForApproval" color="primary" @click="recommendForApproval" :disabled="processing">
            Recommend for Approval
          </ion-button>
          <ion-button v-if="canApproveApplication" color="success" @click="approveApplication" :disabled="processing">
            Approve Application
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
const currentUser = ref(JSON.parse(localStorage.getItem('user') || 'null'));

const currentUserRoles = computed(() => {
  const roles = currentUser.value?.roles;

  if (Array.isArray(roles)) {
    return roles.map((role) => String(role).toLowerCase());
  }

  if (typeof currentUser.value?.role === 'string') {
    return [currentUser.value.role.toLowerCase()];
  }

  return [];
});

const hasRole = (role) => currentUserRoles.value.includes(role.toLowerCase());

const applicationStatus = computed(() => {
  if (!application.value) {
    return '';
  }

  return displayStatus(application.value);
});

const canForwardToHr = computed(() => applicationStatus.value === 'Pending');
const canRecommendForApproval = computed(() => hasRole('hr') && applicationStatus.value === 'Noted');
const canApproveApplication = computed(() => hasRole('admin') && applicationStatus.value === 'Recommended');
const canDenyApplication = computed(() => canRecommendForApproval.value || canApproveApplication.value);
const showFooterActions = computed(() => canForwardToHr.value || canDenyApplication.value || canRecommendForApproval.value || canApproveApplication.value);

const employeeName = computed(() => {
  const employeeInfo = application.value?.user?.employee_info;

  if (!employeeInfo) {
    return '';
  }

  return `${employeeInfo.last_name}, ${employeeInfo.first_name}`;
});

const actorName = (user) => {
  if (!user) {
    return 'Unknown User';
  }

  const employeeInfo = user.employee_info || user.employeeInfo;

  if (employeeInfo?.full_name) {
    return employeeInfo.full_name;
  }

  if (employeeInfo?.last_name || employeeInfo?.first_name) {
    return `${employeeInfo.last_name || ''}, ${employeeInfo.first_name || ''}`.replace(/^,\s*/, '').trim();
  }

  return user.name || 'Unknown User';
};

const processHistory = computed(() => {
  if (!application.value) {
    return [];
  }

  return [
    {
      key: 'noted',
      label: 'Noted',
      badgeClass: 'badge-noted',
      by: actorName(application.value.noted_by),
      at: application.value.noted_at
    },
    {
      key: 'recommended',
      label: 'Recommended for approval',
      badgeClass: 'badge-recommended',
      by: actorName(application.value.recommended_by),
      at: application.value.recommended_at
    },
    {
      key: 'approved',
      label: 'Approved',
      badgeClass: 'badge-approved',
      by: actorName(application.value.approved_by),
      at: application.value.approved_at
    },
    {
      key: 'denied',
      label: 'Denied',
      badgeClass: 'badge-denied',
      by: actorName(application.value.denied_by),
      at: application.value.denied_at,
      remarks: application.value.comments
    }
  ];
});

const visibleProcessHistory = computed(() => processHistory.value.filter((item) => !!item.at));

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

const formatLongDate = (value) => {
  if (!value) return '';

  return new Date(value).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
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
  denyReason.value = '';
  denyReasonError.value = '';
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

const recommendForApproval = async () => {
  if (processing.value) return;

  processing.value = true;

  try {
    await axios.post(`${host}/leave-applications/${route.params.id}/recommend`, {}, {
      headers: headers()
    });

    await showToast('Application recommended for approval.');
    router.replace('/tabs/leave-form');
  } finally {
    processing.value = false;
  }
};

const approveApplication = async () => {
  if (processing.value) return;

  processing.value = true;

  try {
    await axios.post(`${host}/leave-applications/${route.params.id}/approve`, {}, {
      headers: headers()
    });

    await showToast('Application approved successfully.');
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

.process-history-section {
  padding: 4px 0 12px;
}

.process-history-section h3 {
  margin: 8px 12px;
  font-size: 1rem;
}

.history-badge {
  display: inline-block;
  margin-right: 6px;
  padding: 2px 8px;
  border-radius: 999px;
  font-size: 0.72rem;
  font-weight: 600;
  line-height: 1.3;
}

.badge-noted {
  background: var(--ion-color-danger-tint);
  color: #fff;
}

.badge-recommended {
  background: var(--ion-color-warning);
  color: var(--ion-color-warning-contrast);
}

.badge-approved {
  background: var(--ion-color-success);
  color: var(--ion-color-success-contrast);
}

.badge-denied {
  background: var(--ion-color-danger);
  color: var(--ion-color-danger-contrast);
}

.history-remarks {
  margin-top: 6px;
  color: var(--ion-color-medium);
  font-size: 0.85rem;
}

.field-error {
  margin: 6px 0 12px;
  color: var(--ion-color-danger);
  font-size: 0.85rem;
}
</style>
