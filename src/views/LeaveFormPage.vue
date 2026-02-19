<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button></ion-menu-button>
        </ion-buttons>
        <ion-title>Leave Application</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div class="page-content">
        <section class="table-section">
          <h2>Application Notice</h2>
            <p class="section-note" style="color: var(--ion-color-medium); background: var(--ion-color-light); padding: 10px 12px; border-radius: 8px;">
              List of leave applications filed by your subordinates requiring your notice to be forwarded to the HR
            </p>
          <table class="leave-table">
            <thead>
              <tr>
                <th>Employee</th>
                <th>Date Applied</th>
                <th>Type</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="application in applicationsForNotice"
                :key="application.id"
                class="clickable-row"
                @click="openNoticeDetails(application.id)"
              >
                <td>{{ application.user.employee_info.last_name }}, {{ application.user.employee_info.first_name }}</td>
                <td>{{ formatDate(application.created_at) }}</td>
                <td>{{ displayType(application) }}</td>
              </tr>
            </tbody>
          </table>
        </section>

        <section class="table-section">
          <h2>My Leave Applications</h2>
          <p class="section-note" style="color: var(--ion-color-medium); background: var(--ion-color-light); padding: 10px 12px; border-radius: 8px;">
              List of your leave applications with their current status. You can also view the details of each application by clicking on them.
            </p>
          <table class="leave-table">
            <thead>
              <tr>
                <th>Date Applied</th>
                <th>Type</th>
                <th>Status</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="application in leaveApplications"
                :key="application.id"
                class="clickable-row"
                @click="openMyApplicationModal(application)"
              >
                <td>{{ formatDate(application.created_at) }}</td>
                <td>{{ displayType(application) }}</td>
                <td :class="statusClass(application)">{{ displayStatus(application) }}</td>
              </tr>
            </tbody>
          </table>
        </section>
      </div>
    </ion-content>

    <ion-fab slot="fixed" vertical="bottom" horizontal="end">
      <ion-fab-button router-link="/tabs/leave-form/create">
        <ion-icon :icon="add"></ion-icon>
      </ion-fab-button>
    </ion-fab>

    <ion-modal :is-open="isMyApplicationModalOpen" @didDismiss="closeMyApplicationModal">
      <ion-header>
        <ion-toolbar>
          <ion-title>Application Details</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="closeMyApplicationModal">Close</ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
      <ion-content>
        <div class="modal-content" v-if="selectedMyApplication">
          <ion-list>
            <ion-item>
              <ion-label>
                <h3>Date Applied</h3>
                <p>{{ formatDate(selectedMyApplication.created_at) }}</p>
              </ion-label>
            </ion-item>
            <ion-item>
              <ion-label>
                <h3>Type</h3>
                <p>{{ displayType(selectedMyApplication) }}</p>
              </ion-label>
            </ion-item>
            <ion-item>
              <ion-label>
                <h3>Status</h3>
                <p>{{ displayStatus(selectedMyApplication) }}</p>
              </ion-label>
            </ion-item>
            <ion-item>
              <ion-label>
                <h3>Start Date</h3>
                <p>{{ formatDate(selectedMyApplication.start_date) }}</p>
              </ion-label>
            </ion-item>
            <ion-item>
              <ion-label>
                <h3>End Date</h3>
                <p>{{ formatDate(selectedMyApplication.end_date) }}</p>
              </ion-label>
            </ion-item>
            <ion-item>
              <ion-label>
                <h3>With Pay</h3>
                <p>{{ selectedMyApplication.with_pay ? 'Yes' : 'No' }}</p>
              </ion-label>
            </ion-item>
            <ion-item>
              <ion-label>
                <h3>Reason</h3>
                <p>{{ selectedMyApplication.reason }}</p>
              </ion-label>
            </ion-item>
            <ion-item v-if="selectedMyApplication.comments">
              <ion-label>
                <h3>Comments</h3>
                <p>{{ selectedMyApplication.comments }}</p>
              </ion-label>
            </ion-item>
          </ion-list>
        </div>
      </ion-content>
    </ion-modal>
  </ion-page>
</template>

<script setup>
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonButtons, IonButton, IonMenuButton, IonFab, IonFabButton, IonIcon, IonModal, IonList, IonItem, IonLabel, onIonViewWillEnter } from '@ionic/vue';
import axios from 'axios';
import { ref } from 'vue';
import { add } from 'ionicons/icons';
import { useRouter } from 'vue-router';

const host = localStorage.getItem('host') || 'http://localhost:8000/api';
const router = useRouter();

const leaveApplications = ref([]);
const applicationsForNotice = ref([]);
const selectedMyApplication = ref(null);
const isMyApplicationModalOpen = ref(false);

const formatDate = (value) => {
  if (!value) return '';

  return new Date(value).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'short',
    day: '2-digit'
  });
};

const displayType = (application) => {
  if (application.type === 'Others' && application.other_type) {
    return application.other_type;
  }

  return application.type || '';
};

const displayStatus = (application) => {
  if (application.status) {
    return application.status;
  }

  if (application.denied_at) {
    return 'Denied';
  }

  if (application.approved_at) {
    return 'Approved';
  }

  if (application.recommended_at) {
    return 'Recommended';
  }

  if (application.noted_at) {
    return 'Noted';
  }

  return 'Pending';
};

const statusClass = (application) => {
  const status = displayStatus(application);

  if (status === 'Noted') {
    return 'status-noted';
  }

  if (status === 'Recommended') {
    return 'status-recommended';
  }

  if (status === 'Approved') {
    return 'status-approved';
  }

  if (status === 'Denied') {
    return 'status-denied';
  }

  return 'status-pending';
};

const fetchLeaveApplications = async () => {
  try {
    const response = await axios.get(`${host}/leave-application`, {
      headers: {
        Authorization: `Bearer ${localStorage.getItem('access_token')}`
      }
    });

    leaveApplications.value = response.data.leaveApplications || [];
    applicationsForNotice.value = response.data.applicationsForNotice || [];
  } catch {
    leaveApplications.value = [];
    applicationsForNotice.value = [];
  }
};

const openNoticeDetails = (id) => {
  router.push(`/tabs/leave-form/notice/${id}`);
};

const openMyApplicationModal = (application) => {
  selectedMyApplication.value = application;
  isMyApplicationModalOpen.value = true;
};

const closeMyApplicationModal = () => {
  isMyApplicationModalOpen.value = false;
};

onIonViewWillEnter(() => {
  fetchLeaveApplications();
});
</script>

<style scoped>
.page-content {
  padding: 16px;
}

.table-section {
  margin-bottom: 24px;
}

.table-section h2 {
  margin: 0 0 12px;
  font-size: 1rem;
}

.section-note {
  margin: 0 0 12px;
  font-size: 0.9rem;
}

.leave-table {
  width: 100%;
  border-collapse: collapse;
  background: var(--ion-color-light);
}

.leave-table th {
  padding: 10px 8px;
  text-align: left;
  border-bottom: 1px solid var(--ion-color-step-150);
  font-weight: 600;
  background: var(--ion-color-step-100);
}

.leave-table td {
  padding: 10px 8px;
  border-bottom: 1px solid var(--ion-color-step-100);
}

.leave-table tbody tr:nth-child(odd) {
  background: #333;
}

.leave-table tbody tr:nth-child(even) {
  background: var(--ion-color-light);
}

.clickable-row {
  cursor: pointer;
}

.status-pending {
  color: var(--ion-color-medium);
  font-weight: 600;
}

.status-noted {
  color: var(--ion-color-danger-tint);
  font-weight: 600;
}

.status-recommended {
  color: var(--ion-color-warning);
  font-weight: 600;
}

.status-approved {
  color: var(--ion-color-success);
  font-weight: 600;
}

.status-denied {
  color: var(--ion-color-danger);
  font-weight: 600;
}

.modal-content {
  padding: 12px;
}

</style>
