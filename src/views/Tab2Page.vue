<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="main-toolbar">
        <ion-title>Pujar contingut</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <div class="upload-container">
        <div class="upload-header">
          <h2>Afegir nou contingut multimèdia</h2>
          <p class="upload-subtitle">Puja vídeos o imatges a la teva col·lecció</p>
        </div>

        <form @submit.prevent="handleSubmit" class="upload-form">
          <div class="form-group">
            <ion-label position="stacked">Títol</ion-label>
            <ion-input
                v-model="videoTitle"
                placeholder="Escriviu el títol"
                class="custom-input"
                fill="solid"
            ></ion-input>
          </div>

          <div class="form-group">
            <ion-label position="stacked">Descripció</ion-label>
            <ion-textarea
                v-model="videoDescription"
                placeholder="Descripció del contingut"
                class="custom-textarea"
                rows="4"
                fill="solid"
            ></ion-textarea>
          </div>

          <div class="file-upload-container">
            <!-- FilePond component -->
            <file-pond
                ref="pond"
                name="file"
                label-idle="Arrossega i deixa anar els teus fitxers o <span class='filepond--label-action'>Tria'n un</span>"
                :accepted-file-types="acceptedFileTypes"
                allow-multiple="false"
                v-on:addfile="handleFilePondAdd"
                class="custom-filepond"
            />

            <div class="file-type-info" v-if="fileTypeMessage">
              <ion-icon :icon="fileIcon" class="file-icon"></ion-icon>
              <span>{{ fileTypeMessage }}</span>
            </div>
          </div>

          <div v-if="errorMessage" class="error-alert">
            <ion-icon :icon="alertCircle"></ion-icon>
            <span>{{ errorMessage }}</span>
          </div>

          <div class="upload-actions">
            <ion-button expand="block" type="submit" class="upload-button" :disabled="isUploading">
              <ion-spinner v-if="isUploading" name="crescent"></ion-spinner>
              <template v-else>
                <ion-icon :icon="cloudUpload" slot="start"></ion-icon>
                Pujar contingut
              </template>
            </ion-button>
            <ion-button expand="block" fill="outline" type="button" class="cancel-button" @click="cancelUpload">
              Borrar
            </ion-button>
          </div>
        </form>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { useRouter } from 'vue-router';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonInput, IonTextarea, IonButton, IonLabel, IonIcon,
  IonSpinner
} from '@ionic/vue';
import {
  cloudUpload, image as imageIcon, videocam,
  documentText, alertCircle
} from 'ionicons/icons';
import api from '@/api/axiosInstance';

// Importa FilePond i el plugin per validar el tipus de fitxer
import vueFilePond from 'vue-filepond';
import FilePondPluginFileValidateType from 'filepond-plugin-file-validate-type';
import 'filepond/dist/filepond.min.css';

// Registra el component FilePond amb el plugin
const FilePond = vueFilePond(FilePondPluginFileValidateType);

const router = useRouter();

// Definició de variables reactives
const videoTitle = ref('');
const videoDescription = ref('');
const videoFile = ref(null);
const fileType = ref('');
const isUploading = ref(false);
const errorMessage = ref('');

// Definir els tipus de fitxers acceptats
const acceptedFileTypes = ref([
  'image/*',
  'video/mp4',
  'video/x-matroska'
]);

// Missatge basat en el tipus de fitxer
const fileTypeMessage = computed(() => {
  if (!fileType.value) return '';

  if (fileType.value.includes('image')) {
    return 'Imatge seleccionada';
  } else if (fileType.value.includes('video')) {
    return 'Vídeo seleccionat';
  } else {
    return 'Fitxer seleccionat';
  }
});

// Icona basada en el tipus de fitxer
const fileIcon = computed(() => {
  if (fileType.value.includes('image')) {
    return imageIcon;
  } else if (fileType.value.includes('video')) {
    return videocam;
  } else {
    return documentText;
  }
});

// Quan s'afegeix un fitxer a FilePond
const handleFilePondAdd = (error, file) => {
  if (error) {
    console.error('Error afegint el fitxer:', error);
    errorMessage.value = 'Error afegint el fitxer. Comprova el format.';
    return;
  }
  videoFile.value = file.file;
  fileType.value = file.file.type;
  errorMessage.value = '';
};

// Quan s'envia el formulari
const handleSubmit = async () => {
  errorMessage.value = '';

  if (!videoFile.value) {
    errorMessage.value = 'Si us plau, selecciona un fitxer!';
    return;
  }

  if (!videoTitle.value.trim()) {
    errorMessage.value = 'Si us plau, afegeix un títol!';
    return;
  }

  isUploading.value = true;
  const formData = new FormData();
  formData.append('title', videoTitle.value);
  formData.append('description', videoDescription.value);
  formData.append('file', videoFile.value);

  try {
    const response = await api.post('/multimedia', formData, {
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      },
    });
    console.log('Fitxer pujat:', response.data);

    // Redirigir a la galeria després de pujar correctament
    router.push('/tabs/tab1');
  } catch (error) {
    console.error('Error pujant fitxer:', error);
    errorMessage.value = error.response?.data?.message || 'Ha fallat la càrrega del fitxer. Intenta-ho de nou.';
  } finally {
    isUploading.value = false;
  }
};

// Cancel·lar la pujada i netejar el formulari
const cancelUpload = () => {
  videoTitle.value = '';
  videoDescription.value = '';
  videoFile.value = null;
  fileType.value = '';
  errorMessage.value = '';

  // Reseteja FilePond
  if (pond.value) {
    pond.value.removeFiles();
  }
};

const pond = ref(null);
</script>


<style>
/* Variables CSS globals - Assegurar que coincideixen amb les altres pàgines */
:root {
  /* Colors principals */
  --app-primary: #7c4dff;
  --app-primary-shade: #6a3df7;
  --app-primary-tint: #9670ff;
  --app-secondary: #ff4d94;
  --app-accent: #00e5ff;

  /* Colors de fons */
  --app-background: #f8f9fa;
  --app-surface: #ffffff;
  --app-card: #ffffff;

  /* Colors de text */
  --app-text: #333333;
  --app-text-medium: #666666;
  --app-text-light: #999999;

  /* Colors d'estat */
  --app-success: #4caf50;
  --app-warning: #ff9800;
  --app-error: #f44336;

  /* Ombres */
  --app-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  --app-shadow-strong: 0 8px 24px rgba(0, 0, 0, 0.12);
}

/* Aplicar variables a Ionic */
ion-app {
  --ion-color-primary: var(--app-primary);
  --ion-color-primary-rgb: 124, 77, 255;
  --ion-color-primary-contrast: #ffffff;
  --ion-color-primary-contrast-rgb: 255, 255, 255;
  --ion-color-primary-shade: var(--app-primary-shade);
  --ion-color-primary-tint: var(--app-primary-tint);

  --ion-background-color: var(--app-background);
  --ion-text-color: var(--app-text);

  --ion-toolbar-background: var(--app-surface);
  --ion-toolbar-color: var(--app-text);

  --ion-item-background: var(--app-surface);
}

/* Estils específics del component */
.main-toolbar {
  --background: var(--app-primary);
  --color: white;
  --border-color: transparent;
}

.upload-container {
  padding: 24px 16px;
  display: flex;
  flex-direction: column;
  max-width: 800px;
  margin: 0 auto;
}

.upload-header {
  margin-bottom: 24px;
  text-align: center;
}

.upload-header h2 {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 8px;
  color: var(--app-primary);
}

.upload-subtitle {
  color: var(--app-text-light);
  font-size: 0.9rem;
}

.upload-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

ion-label {
  font-weight: 500;
  margin-bottom: 4px;
  font-size: 0.9rem;
}

.custom-input,
.custom-textarea {
  --background: #f5f5f5;
  --color: var(--app-text);
  --placeholder-color: var(--app-text-light);
  --border-radius: 8px;
  --padding-start: 16px;
  --padding-end: 16px;
}

.file-upload-container {
  margin: 16px 0;
  border-radius: 12px;
  overflow: hidden;
}

.custom-filepond {
  --filepond-primary-color: var(--app-primary);
  background-color: #f5f5f5;
  border-radius: 8px;
}

:deep(.filepond--panel-root) {
  background-color: #f5f5f5;
}

:deep(.filepond--drop-label) {
  color: var(--app-text-medium);
}

:deep(.filepond--label-action) {
  color: var(--app-primary);
  text-decoration-color: var(--app-primary);
}

.file-type-info {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 12px;
  padding: 8px 12px;
  background-color: rgba(var(--ion-color-primary-rgb), 0.1);
  border-radius: 8px;
  font-size: 0.9rem;
}

.file-icon {
  font-size: 1.2rem;
  color: var(--app-primary);
}

.error-alert {
  display: flex;
  align-items: center;
  gap: 8px;
  background-color: rgba(244, 67, 54, 0.1);
  color: var(--app-error);
  padding: 12px;
  border-radius: 8px;
  font-size: 14px;
}

.error-alert ion-icon {
  font-size: 18px;
  flex-shrink: 0;
}

.upload-actions {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 16px;
}

.upload-button {
  --border-radius: 8px;
  margin-bottom: 8px;
  height: 48px;
  font-weight: 600;
  display: flex;
  justify-content: center;
  align-items: center;
}

.upload-button ion-spinner {
  width: 24px;
  height: 24px;
}

.cancel-button {
  --border-radius: 8px;
  height: 48px;
}

/* Estils per a tema fosc (si s'aplica) */
:root.dark-theme {
  --app-background: #121212;
  --app-surface: #1e1e1e;
  --app-card: #1e1e1e;
  --app-text: #ffffff;
  --app-text-medium: #cccccc;
  --app-text-light: #999999;
}

.dark-theme .custom-input,
.dark-theme .custom-textarea {
  --background: #2a2a2a;
  --color: #ffffff;
  --placeholder-color: #999;
}

.dark-theme .custom-filepond {
  background-color: #2a2a2a;
}

.dark-theme :deep(.filepond--panel-root) {
  background-color: #2a2a2a;
}

.dark-theme :deep(.filepond--drop-label) {
  color: #ccc;
}
</style>