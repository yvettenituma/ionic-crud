<template>
  <ion-page>
    <ion-header class="app-header">
      <ion-toolbar>
        <div class="topbar">
          <div class="brand">
            <div class="brand-mark">
              <ion-icon :icon="schoolOutline" />
            </div>
            <div>
              <div class="brand-title">StudentHub</div>
              <div class="brand-subtitle">Student record management</div>
            </div>
          </div>

          <div class="header-status">
            <span class="status-dot"></span>
            <span>Admin Workspace</span>
          </div>
        </div>
      </ion-toolbar>
    </ion-header>

    <ion-content class="page-content">
      <div class="dashboard-container">
        <!-- DASHBOARD INTRO -->
        <section class="intro-panel">
          <div class="intro-copy">
            <span class="eyebrow">ACADEMIC RECORDS</span>
            <h1>Student Management</h1>
            <p>Keep student information organized, updated, and easy to access.</p>
          </div>

          <div class="overview-card">
            <div class="overview-icon">
              <ion-icon :icon="peopleOutline" />
            </div>
            <div>
              <span>Total students</span>
              <strong>{{ students.length }}</strong>
            </div>
          </div>
        </section>

        <!-- MAIN WORKSPACE -->
        <div class="workspace-grid">
          <!-- FORM PANEL -->
          <ion-card class="dashboard-card form-card">
            <ion-card-content>
              <div class="section-top">
                <div class="section-title">
                  <span class="section-kicker">{{ editingId ? 'RECORD UPDATE' : 'NEW RECORD' }}</span>
                  <h2>{{ editingId ? 'Edit Student' : 'Add Student' }}</h2>
                  <p>{{ editingId ? 'Make changes to the selected student record.' : 'Enter the required student information.' }}</p>
                </div>
                <div class="section-symbol">
                  <ion-icon :icon="editingId ? createOutline : personAddOutline" />
                </div>
              </div>

              <form @submit.prevent="saveStudent">
                <div class="form-grid">
                  <div class="form-field">
                    <ion-input
                      v-model="form.name"
                      label="Full Name"
                      label-placement="stacked"
                      fill="outline"
                      placeholder="Enter student full name"
                      class="custom-input"
                    >
                      <ion-icon slot="start" :icon="personOutline" />
                    </ion-input>
                  </div>

                  <div class="form-field">
                    <ion-input
                      v-model="form.email"
                      type="email"
                      label="Email Address"
                      label-placement="stacked"
                      fill="outline"
                      placeholder="Enter email address"
                      class="custom-input"
                    >
                      <ion-icon slot="start" :icon="mailOutline" />
                    </ion-input>
                  </div>

                  <div class="form-field full-width">
                    <ion-input
                      v-model="form.course"
                      label="Course / Program"
                      label-placement="stacked"
                      fill="outline"
                      placeholder="Example: BS Information Technology"
                      class="custom-input"
                    >
                      <ion-icon slot="start" :icon="bookOutline" />
                    </ion-input>
                  </div>
                </div>

                <div class="form-actions">
                  <ion-button
                    type="submit"
                    class="primary-action"
                    :disabled="loading"
                  >
                    <ion-icon
                      slot="start"
                      :icon="editingId ? saveOutline : addCircleOutline"
                    />
                    {{ editingId ? 'Update Student' : 'Save Student' }}
                  </ion-button>

                  <ion-button
                    v-if="editingId"
                    type="button"
                    fill="outline"
                    class="cancel-action"
                    @click="resetForm"
                  >
                    <ion-icon slot="start" :icon="closeOutline" />
                    Cancel
                  </ion-button>
                </div>
              </form>

              <div
                v-if="message"
                class="message-box"
                :class="{
                  'message-error': message.toLowerCase().includes('unable'),
                  'message-success': !message.toLowerCase().includes('unable')
                }"
              >
                <ion-icon
                  :icon="
                    message.toLowerCase().includes('unable')
                      ? alertCircleOutline
                      : checkmarkCircleOutline
                  "
                />
                <span>{{ message }}</span>
              </div>
            </ion-card-content>
          </ion-card>

          <!-- CONNECTION PANEL -->
          <ion-card class="dashboard-card connection-card">
            <ion-card-content>
              <div class="connection-top">
                <div class="connection-icon">
                  <ion-icon :icon="cloudDoneOutline" />
                </div>
                <div>
                  <span class="section-kicker">SYSTEM STATUS</span>
                  <h2>Firebase</h2>
                </div>
              </div>

              <div class="connection-status">
                <ion-badge
                  v-if="firebaseStatus === 'checking'"
                  color="warning"
                  class="status-badge"
                >
                  <ion-icon :icon="syncOutline" />
                  Checking
                </ion-badge>

                <ion-badge
                  v-else-if="firebaseStatus === 'connected'"
                  color="success"
                  class="status-badge"
                >
                  <ion-icon :icon="checkmarkCircleOutline" />
                  Connected
                </ion-badge>

                <ion-badge
                  v-else
                  color="danger"
                  class="status-badge"
                >
                  <ion-icon :icon="closeCircleOutline" />
                  Disconnected
                </ion-badge>
              </div>

              <div class="connection-message">
                <ion-icon :icon="informationCircleOutline" />
                <p>{{ firebaseMessage }}</p>
              </div>

              <ion-button
                expand="block"
                fill="outline"
                class="connection-button"
                @click="checkFirebaseConnection"
              >
                <ion-icon slot="start" :icon="refreshOutline" />
                Refresh Connection
              </ion-button>
            </ion-card-content>
          </ion-card>
        </div>

        <!-- RECORDS -->
        <ion-card class="dashboard-card records-card">
          <ion-card-content>
            <div class="records-header">
              <div>
                <span class="section-kicker">DIRECTORY</span>
                <h2>Student Records</h2>
                <p>Registered students currently stored in the database.</p>
              </div>

              <div class="records-total">
                <strong>{{ students.length }}</strong>
                <span>records</span>
              </div>
            </div>

            <div v-if="loading" class="empty-state">
              <ion-spinner name="crescent" />
              <p>Loading student records...</p>
            </div>

            <div v-else-if="students.length === 0" class="empty-state">
              <div class="empty-icon">
                <ion-icon :icon="peopleOutline" />
              </div>
              <h4>No Student Records</h4>
              <p>Student records will appear here once you add a student.</p>
            </div>

            <ion-list v-else class="student-list">
              <ion-item
                v-for="(student, index) in students"
                :key="student.id"
                lines="none"
                class="student-item"
              >
                <div class="student-number">
                  {{ String(index + 1).padStart(2, '0') }}
                </div>

                <div class="student-avatar">
                  {{ student.name.charAt(0).toUpperCase() }}
                </div>

                <ion-label class="student-details">
                  <h2>{{ student.name }}</h2>
                  <div class="student-info">
                    <span>
                      <ion-icon :icon="mailOutline" />
                      {{ student.email }}
                    </span>
                    <span>
                      <ion-icon :icon="bookOutline" />
                      {{ student.course }}
                    </span>
                  </div>
                </ion-label>

                <div class="student-actions">
                  <ion-button
                    size="small"
                    fill="clear"
                    class="edit-button"
                    @click="startEdit(student)"
                  >
                    <ion-icon slot="start" :icon="createOutline" />
                    Edit
                  </ion-button>

                  <ion-button
                    size="small"
                    fill="clear"
                    color="danger"
                    class="delete-button"
                    @click="removeStudent(student.id)"
                  >
                    <ion-icon slot="start" :icon="trashOutline" />
                    Delete
                  </ion-button>
                </div>
              </ion-item>
            </ion-list>
          </ion-card-content>
        </ion-card>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonCard,
  IonCardContent,
  IonInput,
  IonButton,
  IonList,
  IonItem,
  IonLabel,
  IonBadge,
  IonIcon,
  IonSpinner
} from '@ionic/vue';

import {
  schoolOutline,
  peopleOutline,
  cloudDoneOutline,
  syncOutline,
  checkmarkCircleOutline,
  closeCircleOutline,
  refreshOutline,
  informationCircleOutline,
  personAddOutline,
  createOutline,
  personOutline,
  mailOutline,
  bookOutline,
  saveOutline,
  addCircleOutline,
  closeOutline,
  alertCircleOutline,
  listOutline,
  trashOutline
} from 'ionicons/icons';

import { ref, onMounted } from 'vue';

import {
  ref as databaseRef,
  get,
  push,
  update,
  remove,
  serverTimestamp,
  query,
  limitToFirst
} from 'firebase/database';

import { db } from '@/firebase';

interface Student {
  id: string;
  name: string;
  email: string;
  course: string;
}

const students = ref<Student[]>([]);
const loading = ref(false);
const message = ref('');
const editingId = ref<string | null>(null);

const firebaseStatus = ref<
  'checking' | 'connected' | 'disconnected'
>('checking');

const firebaseMessage = ref(
  'Checking Firebase connection...'
);

const form = ref({
  name: '',
  email: '',
  course: ''
});

function describeFirebaseError(error: unknown): string {
  const detail =
    error instanceof Error
      ? error.message
      : String(error);

  if (/permission.?denied/i.test(detail)) {
    return 'Permission denied. Check Realtime Database rules for /students and Firebase Authentication.';
  }

  return detail;
}

const checkFirebaseConnection = async () => {
  firebaseStatus.value = 'checking';

  firebaseMessage.value =
    'Checking Firebase connection...';

  try {
    const testQuery = query(
      databaseRef(db, 'students'),
      limitToFirst(1)
    );

    await get(testQuery);

    firebaseStatus.value = 'connected';

    firebaseMessage.value =
      'Realtime Database connection is working properly.';
  } catch (error) {
    console.error('Firebase connection error:', error);

    firebaseStatus.value = 'disconnected';

    firebaseMessage.value =
      describeFirebaseError(error);
  }
};

const loadStudents = async () => {
  loading.value = true;

  try {
    const snapshot = await get(
      databaseRef(db, 'students')
    );

    const records: Student[] = [];

    snapshot.forEach((child) => {
      const data = child.val();

      records.push({
        id: child.key!,
        name: String(data?.name ?? ''),
        email: String(data?.email ?? ''),
        course: String(data?.course ?? '')
      });
    });

    students.value = records.sort((a, b) =>
      a.name.localeCompare(b.name)
    );
  } catch (error) {
    console.error(error);

    message.value =
      `Unable to load students: ${describeFirebaseError(error)}`;
  } finally {
    loading.value = false;
  }
};

const saveStudent = async () => {
  const name = form.value.name.trim();
  const email = form.value.email.trim();
  const course = form.value.course.trim();

  if (!name || !email || !course) {
    message.value = 'Please complete all fields.';
    return;
  }

  loading.value = true;

  try {
    if (editingId.value) {
      await update(
        databaseRef(
          db,
          `students/${editingId.value}`
        ),
        {
          name,
          email,
          course,
          updatedAt: serverTimestamp()
        }
      );

      message.value =
        'Student successfully updated.';
    } else {
      await push(
        databaseRef(db, 'students'),
        {
          name,
          email,
          course,
          createdAt: serverTimestamp()
        }
      );

      message.value =
        'Student successfully added.';
    }

    resetForm();

    await loadStudents();
    await checkFirebaseConnection();
  } catch (error) {
    console.error(error);

    message.value =
      `Unable to save student: ${describeFirebaseError(error)}`;
  } finally {
    loading.value = false;
  }
};

const startEdit = (student: Student) => {
  editingId.value = student.id;

  form.value = {
    name: student.name,
    email: student.email,
    course: student.course
  };

  window.scrollTo({
    top: 300,
    behavior: 'smooth'
  });
};

const removeStudent = async (id: string) => {
  const confirmed = window.confirm(
    'Are you sure you want to delete this student?'
  );

  if (!confirmed) {
    return;
  }

  loading.value = true;

  try {
    await remove(
      databaseRef(
        db,
        `students/${id}`
      )
    );

    if (editingId.value === id) {
      resetForm();
    }

    message.value =
      'Student successfully deleted.';

    await loadStudents();
    await checkFirebaseConnection();
  } catch (error) {
    console.error(error);

    message.value =
      `Unable to delete student: ${describeFirebaseError(error)}`;
  } finally {
    loading.value = false;
  }
};

const resetForm = () => {
  editingId.value = null;

  form.value = {
    name: '',
    email: '',
    course: ''
  };
};

onMounted(async () => {
  await checkFirebaseConnection();
  await loadStudents();
});
</script>


<style scoped>
ion-content {
  --background: #eef2f7;
}

.app-header ion-toolbar {
  --background: #071b36;
  --color: #ffffff;
  --min-height: 72px;
  padding: 0 20px;
}

.topbar {
  min-height: 72px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 18px;
}

.brand {
  display: flex;
  align-items: center;
  gap: 12px;
}

.brand-mark {
  width: 42px;
  height: 42px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 11px;
  background: #d8b25c;
  color: #071b36;
}

.brand-mark ion-icon {
  font-size: 23px;
}

.brand-title {
  color: #ffffff;
  font-size: 18px;
  font-weight: 800;
  letter-spacing: 0.2px;
}

.brand-subtitle {
  margin-top: 2px;
  color: #aebed2;
  font-size: 11px;
}

.header-status {
  display: flex;
  align-items: center;
  gap: 7px;
  padding: 7px 11px;
  border: 1px solid rgba(255,255,255,0.13);
  border-radius: 20px;
  color: #d8e2ee;
  font-size: 11px;
}

.status-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #63c174;
  box-shadow: 0 0 0 3px rgba(99,193,116,0.12);
}

.page-content {
  --padding-top: 0;
  --padding-bottom: 35px;
}

.dashboard-container {
  width: 100%;
  max-width: 1120px;
  margin: 0 auto;
  padding: 28px 20px 45px;
}

.intro-panel {
  display: grid;
  grid-template-columns: 1fr 250px;
  align-items: stretch;
  gap: 20px;
  margin-bottom: 20px;
}

.intro-copy {
  padding: 8px 2px;
}

.eyebrow,
.section-kicker {
  color: #45688f;
  font-size: 10px;
  font-weight: 850;
  letter-spacing: 1.4px;
}

.intro-copy h1 {
  margin: 7px 0 6px;
  color: #102b4d;
  font-size: 30px;
  line-height: 1.1;
  font-weight: 850;
}

.intro-copy p {
  margin: 0;
  max-width: 560px;
  color: #66778d;
  font-size: 13px;
}

.overview-card {
  display: flex;
  align-items: center;
  gap: 13px;
  padding: 18px;
  border: 1px solid #dbe3ec;
  border-radius: 15px;
  background: #ffffff;
  box-shadow: 0 7px 22px rgba(7,27,54,0.055);
}

.overview-icon {
  width: 46px;
  height: 46px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  background: #e5ebf3;
  color: #0c3159;
}

.overview-icon ion-icon {
  font-size: 24px;
}

.overview-card span {
  display: block;
  color: #718198;
  font-size: 11px;
}

.overview-card strong {
  display: block;
  margin-top: 2px;
  color: #0b2748;
  font-size: 24px;
  line-height: 1;
}

.workspace-grid {
  display: grid;
  grid-template-columns: minmax(0, 1.65fr) minmax(270px, 0.8fr);
  gap: 18px;
  align-items: start;
  margin-bottom: 18px;
}

.dashboard-card {
  margin: 0;
  border: 1px solid #dbe3ec;
  border-radius: 16px;
  background: #ffffff;
  box-shadow: 0 7px 25px rgba(7,27,54,0.055);
  overflow: hidden;
}

.dashboard-card ion-card-content {
  padding: 22px;
}

.section-top {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 14px;
  margin-bottom: 21px;
}

.section-title h2,
.records-header h2,
.connection-top h2 {
  margin: 4px 0 3px;
  color: #142f50;
  font-size: 18px;
  font-weight: 800;
}

.section-title p,
.records-header p {
  margin: 0;
  color: #77869a;
  font-size: 11px;
  line-height: 1.45;
}

.section-symbol,
.connection-icon {
  width: 43px;
  height: 43px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex: 0 0 43px;
  border-radius: 12px;
  background: #e9eef5;
  color: #0c3159;
}

.section-symbol ion-icon,
.connection-icon ion-icon {
  font-size: 22px;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 14px;
}

.form-field.full-width {
  grid-column: 1 / -1;
}

.custom-input {
  --background: #fbfcfe;
  --border-color: #d3dce7;
  --border-radius: 10px;
  --highlight-color-focused: #173e68;
  --padding-start: 12px;
  --padding-end: 12px;
  min-height: 57px;
  color: #18324f;
  font-size: 13px;
}

.custom-input ion-icon {
  color: #718198;
  margin-right: 7px;
}

.form-actions {
  display: flex;
  gap: 10px;
  margin-top: 18px;
}

.form-actions ion-button {
  height: 44px;
  margin: 0;
  border-radius: 9px;
  font-size: 12px;
  font-weight: 750;
  text-transform: none;
}

.primary-action {
  flex: 1;
  --background: #0b2d52;
  --background-hover: #173e68;
  --border-radius: 9px;
  --box-shadow: none;
}

.cancel-action {
  flex: 0 0 135px;
  --color: #38516e;
  --border-color: #bac7d6;
  --border-radius: 9px;
}

.message-box {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 15px;
  padding: 11px 13px;
  border-radius: 9px;
  font-size: 12px;
  line-height: 1.4;
}

.message-box ion-icon {
  font-size: 17px;
  flex: 0 0 auto;
}

.message-success {
  color: #166534;
  background: #f0f8f2;
  border: 1px solid #c6e6ce;
}

.message-error {
  color: #b42318;
  background: #fff4f3;
  border: 1px solid #f2cbc7;
}

.connection-card {
  min-height: 100%;
}

.connection-top {
  display: flex;
  align-items: center;
  gap: 12px;
}

.connection-top .section-kicker {
  color: #718198;
}

.connection-top h2 {
  margin-top: 2px;
}

.connection-status {
  display: flex;
  justify-content: flex-start;
  margin: 19px 0 14px;
}

.status-badge {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 7px 10px;
  border-radius: 20px;
  font-size: 10px;
  font-weight: 750;
}

.status-badge ion-icon {
  font-size: 13px;
}

.connection-message {
  min-height: 82px;
  display: flex;
  align-items: flex-start;
  gap: 9px;
  padding: 13px;
  border-radius: 10px;
  background: #f4f7fa;
  color: #68798f;
}

.connection-message ion-icon {
  flex: 0 0 auto;
  margin-top: 1px;
  color: #244d77;
  font-size: 17px;
}

.connection-message p {
  margin: 0;
  font-size: 11px;
  line-height: 1.55;
}

.connection-button {
  margin-top: 14px;
  --border-color: #274d75;
  --color: #173e68;
  --border-radius: 9px;
  height: 42px;
  font-size: 11px;
  font-weight: 750;
  text-transform: none;
}

.records-header {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 15px;
  margin-bottom: 17px;
}

.records-total {
  display: flex;
  align-items: baseline;
  gap: 5px;
  padding: 8px 12px;
  border-radius: 10px;
  background: #0b2d52;
  color: #ffffff;
  white-space: nowrap;
}

.records-total strong {
  font-size: 16px;
}

.records-total span {
  color: #c4d2e1;
  font-size: 10px;
}

.student-list {
  padding: 0;
  background: transparent;
}

.student-item {
  --background: #f9fafc;
  --padding-start: 10px;
  --padding-end: 10px;
  --inner-padding-end: 0;
  --min-height: 76px;
  margin-bottom: 9px;
  border: 1px solid #e1e7ee;
  border-radius: 11px;
  transition: transform 0.15s ease, border-color 0.15s ease;
}

.student-item:hover {
  border-color: #b9c7d7;
  transform: translateY(-1px);
}

.student-item:last-child {
  margin-bottom: 0;
}

.student-number {
  width: 28px;
  margin-right: 8px;
  color: #9aa8b9;
  font-size: 10px;
  font-weight: 800;
  text-align: center;
}

.student-avatar {
  width: 40px;
  height: 40px;
  flex: 0 0 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 12px;
  border-radius: 11px;
  background: #dce6f1;
  color: #102f52;
  font-size: 16px;
  font-weight: 850;
}

.student-details {
  min-width: 0;
}

.student-details h2 {
  margin: 0 0 5px;
  color: #203650;
  font-size: 13px;
  font-weight: 800;
}

.student-info {
  display: flex;
  flex-wrap: wrap;
  gap: 6px 16px;
}

.student-info span {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  color: #728197;
  font-size: 10px;
}

.student-info ion-icon {
  color: #315b84;
  font-size: 12px;
}

.student-actions {
  display: flex;
  align-items: center;
  gap: 2px;
  margin-left: 10px;
}

.student-actions ion-button {
  --padding-start: 7px;
  --padding-end: 7px;
  height: 32px;
  margin: 0;
  font-size: 10px;
  font-weight: 750;
  text-transform: none;
}

.edit-button {
  --color: #173e68;
}

.delete-button {
  --color: #c62828;
}

.empty-state {
  display: flex;
  align-items: center;
  flex-direction: column;
  justify-content: center;
  padding: 36px 15px 28px;
  text-align: center;
  color: #728197;
}

.empty-state ion-spinner {
  margin-bottom: 11px;
  color: #173e68;
}

.empty-icon {
  width: 58px;
  height: 58px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 11px;
  border-radius: 14px;
  background: #e5ebf3;
  color: #173e68;
}

.empty-icon ion-icon {
  font-size: 28px;
}

.empty-state h4 {
  margin: 0 0 5px;
  color: #334b67;
  font-size: 14px;
  font-weight: 800;
}

.empty-state p {
  max-width: 300px;
  margin: 0;
  color: #93a0b1;
  font-size: 11px;
  line-height: 1.5;
}

@media (max-width: 800px) {
  .intro-panel,
  .workspace-grid {
    grid-template-columns: 1fr;
  }

  .connection-card {
    min-height: auto;
  }
}

@media (max-width: 650px) {
  .dashboard-container {
    padding: 20px 12px 32px;
  }

  .app-header ion-toolbar {
    padding: 0 12px;
  }

  .header-status {
    display: none;
  }

  .intro-copy h1 {
    font-size: 25px;
  }

  .overview-card {
    width: 100%;
  }

  .dashboard-card ion-card-content {
    padding: 17px;
  }

  .form-grid {
    grid-template-columns: 1fr;
    gap: 12px;
  }

  .form-field.full-width {
    grid-column: auto;
  }

  .form-actions {
    flex-direction: column;
  }

  .cancel-action {
    flex: 1;
  }

  .records-header {
    align-items: flex-start;
  }

  .student-item {
    --min-height: auto;
    padding-top: 11px;
    padding-bottom: 11px;
  }

  .student-number {
    display: none;
  }

  .student-actions {
    flex-direction: column;
    align-items: flex-end;
  }

  .student-info {
    flex-direction: column;
    gap: 3px;
  }
}

@media (max-width: 420px) {
  .brand-title {
    font-size: 16px;
  }

  .brand-subtitle {
    font-size: 10px;
  }

  .section-title h2,
  .records-header h2,
  .connection-top h2 {
    font-size: 16px;
  }

  .student-avatar {
    width: 36px;
    height: 36px;
    flex-basis: 36px;
    font-size: 14px;
  }

  .student-actions ion-button {
    font-size: 9px;
  }
}
</style>
