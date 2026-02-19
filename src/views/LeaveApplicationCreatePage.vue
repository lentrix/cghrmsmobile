<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button default-href="/tabs/leave-form"></ion-back-button>
        </ion-buttons>
        <ion-title>Create Leave Application</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div class="form-content">
        <ion-item>
          <ion-label position="stacked">Type</ion-label>
          <ion-select v-model="leaveType" placeholder="Select leave type">
            <ion-select-option value="Service Incentive Leave">Service Incentive Leave</ion-select-option>
            <ion-select-option value="Sick Leave">Sick Leave</ion-select-option>
            <ion-select-option value="Solo Parent Leave">Solo Parent Leave</ion-select-option>
            <ion-select-option value="Others">Others</ion-select-option>
          </ion-select>
        </ion-item>
        <p v-if="errors.leaveType" class="field-error">{{ errors.leaveType }}</p>

        <ion-item v-if="leaveType === 'Others'">
          <ion-label position="stacked">Specific Type of Leave</ion-label>
          <ion-input v-model="specificLeaveType" type="text"></ion-input>
        </ion-item>

        <section class="dates-section">
          <h2>Applicable Dates</h2>
          <div class="dates-row">
            <ion-item class="date-item">
              <ion-label position="stacked">Start Date</ion-label>
              <ion-input v-model="startDate" type="date"></ion-input>
              <p v-if="errors.startDate" class="field-error">{{ errors.startDate }}</p>
            </ion-item>

            <ion-item class="date-item">
              <ion-label position="stacked">End Date</ion-label>
              <ion-input v-model="endDate" type="date"></ion-input>
              <p v-if="errors.endDate" class="field-error">{{ errors.endDate }}</p>
            </ion-item>
          </div>
        </section>

        <div style="margin-block: 12pt;">
            <ion-item>
            <ion-checkbox v-model="withPay">With Pay</ion-checkbox>
            </ion-item>
            <p v-if="errors.withPay" class="field-error">{{ errors.withPay }}</p>
        </div>

        <ion-item>
          <ion-label position="stacked">Reason</ion-label>
          <ion-textarea v-model="reason" :rows="4"></ion-textarea>
        </ion-item>
        <p v-if="errors.reason" class="field-error">{{ errors.reason }}</p>

        <ion-button expand="block" type="button" class="submit-btn" @click="submitLeaveApplication" :disabled="submitting">Submit</ion-button>
      </div>
    </ion-content>
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
    IonItem,
    IonLabel,
    IonSelect,
    IonSelectOption,
    IonInput,
    IonCheckbox,
    IonTextarea,
    IonButton,
    toastController
    } from '@ionic/vue';
import axios from 'axios';
    import { inject, ref, watch } from 'vue';
    import { useRouter } from 'vue-router';

    const host = localStorage.getItem('host') || 'http://localhost:8000/api';
    const user = inject('user');
    const router = useRouter();

    const leaveType = ref('');
    const specificLeaveType = ref('');
    const startDate = ref('');
    const endDate = ref('');
    const withPay = ref(false);
    const reason = ref('');
    const submitting = ref(false);
    const errors = ref({
      leaveType: '',
      startDate: '',
      endDate: '',
      withPay: '',
      reason: ''
    });

    const validateForm = () => {
      errors.value = {
        leaveType: '',
        startDate: '',
        endDate: '',
        withPay: '',
        reason: ''
      };

      if (!leaveType.value) {
        errors.value.leaveType = 'Type is required.';
      }

      if (!startDate.value) {
        errors.value.startDate = 'Start date is required.';
      }

      if (!endDate.value) {
        errors.value.endDate = 'End date is required.';
      }

      if (!withPay.value) {
        errors.value.withPay = 'With pay is required.';
      }

      if (!reason.value.trim()) {
        errors.value.reason = 'Reason is required.';
      }

      return Object.values(errors.value).every((message) => !message);
    };

    watch(leaveType, (currentValue) => {
      if (currentValue !== 'Others') {
        specificLeaveType.value = '';
      }
    });

    const submitLeaveApplication = async () => {
        if (submitting.value) return;

        if (!validateForm()) {
          return;
        }

        submitting.value = true;

        try {
          await axios.post(`${host}/leave-applications`, {
            user_id: user.value.id,
            type: leaveType.value,
            other_type: leaveType.value === 'Others' ? specificLeaveType.value.trim() || null : null,
            start_date: startDate.value,
            end_date: endDate.value,
            with_pay: withPay.value,
            reason: reason.value
          }, {
            headers: {
              Authorization: `Bearer ${localStorage.getItem('access_token')}`
            }
          });

          const successToast = await toastController.create({
            message: 'Leave application submitted successfully.',
            duration: 1500,
            color: 'success',
            position: 'bottom'
          });

          await successToast.present();
          await successToast.onDidDismiss();
          router.replace('/tabs/leave-form');
        } finally {
          submitting.value = false;
        }
    }
</script>

<style scoped>
.form-content {
  padding: 16px;
}

.dates-section {
  margin-top: 16px;
}

.dates-section h2 {
  margin: 0 0 8px;
  font-size: 1rem;
}

.dates-row {
  display: flex;
  gap: 12px;
}

.date-item {
  flex: 1;
}

.submit-btn {
  margin-top: 16px;
}

.field-error {
  margin: 6px 0 0;
  color: var(--ion-color-danger);
  font-size: 0.85rem;
}
</style>
