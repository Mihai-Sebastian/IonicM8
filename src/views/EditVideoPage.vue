<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="main-toolbar">
        <ion-buttons slot="start">
          <ion-back-button default-href="/tabs/tab1" text=""></ion-back-button>
        </ion-buttons>
        <ion-title>Editar Contingut</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <!-- Estat de càrrega -->
      <div v-if="loading" class="loading-container">
        <ion-spinner name="crescent" color="primary"></ion-spinner>
        <p>Carregant contingut...</p>
      </div>

      <!-- Formulari d'edició -->
      <div v-else class="edit-container">
        <div class="edit-header">
          <h1 class="edit-title">Editar contingut multimèdia</h1>
          <p class="edit-subtitle">Modifica els detalls del teu contingut</p>
        </div>

        <div class="edit-content">
          <!-- Previsualització del contingut -->
          <div class="preview-container">
            <div class="media-preview">
              <img
                  :src="getThumbnail(video)"
                  :alt="video.file_name"
                  loading="lazy"
                  class="preview-image"
              />
              <div class="media-type-badge" :class="{ 'video-badge': isVideo(video) }">
                <ion-icon :icon="isVideo(video) ? videocam : image"></ion-icon>
              </div>
            </div>

            <!-- Informació del fitxer -->
            <div class="file-info">
              <div class="info-item">
                <span class="info-label">Tipus:</span>
                <span class="info-value">{{ formatFileType(video.type) }}</span>
              </div>
              <div class="info-item">
                <span class="info-label">Nom del fitxer:</span>
                <span class="info-value file-name">{{ video.file_name }}</span>
              </div>
              <div class="info-item">
                <span class="info-label">Data de pujada:</span>
                <span class="info-value">{{ formatDate(video.created_at) }}</span>
              </div>
            </div>
          </div>

          <!-- Formulari editable -->
          <form @submit.prevent="saveVideo" class="edit-form">
            <div class="form-group">
              <ion-label position="stacked">Títol</ion-label>
              <ion-input
                  v-model="editedTitle"
                  placeholder="Introdueix el títol del contingut"
                  class="custom-input"
                  fill="solid"
              ></ion-input>
            </div>

            <div class="form-group">
              <ion-label position="stacked">Descripció</ion-label>
              <ion-textarea
                  v-model="editedDescription"
                  placeholder="Afegeix una descripció"
                  class="custom-textarea"
                  rows="4"
                  fill="solid"
              ></ion-textarea>
            </div>

            <!-- Missatge d'error -->
            <div v-if="errorMessage" class="error-alert">
              <ion-icon :icon="alertCircle"></ion-icon>
              <span>{{ errorMessage }}</span>
            </div>

            <div class="form-actions">
              <ion-button type="submit" expand="block" class="save-button" :disabled="isSaving">
                <ion-spinner v-if="isSaving" name="crescent"></ion-spinner>
                <template v-else>
                  <ion-icon :icon="save" slot="start"></ion-icon>
                  Desar Canvis
                </template>
              </ion-button>
              <ion-button expand="block" fill="outline" class="cancel-button" @click="cancelEdit">
                Cancel·lar
              </ion-button>
            </div>
          </form>
        </div>
      </div>
    </ion-content>

    <!-- Toast de confirmació -->
    <ion-toast
        :is-open="showSuccessToast"
        color="success"
        message="Contingut desat correctament!"
        duration="3000"
        position="top"
        @didDismiss="showSuccessToast = false"
    ></ion-toast>
  </ion-page>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonButtons, IonBackButton, IonSpinner, IonInput, IonTextarea,
  IonLabel, IonButton, IonIcon, IonToast
} from '@ionic/vue';
import {
  videocam, image, alertCircle, save
} from 'ionicons/icons';
import api from '@/api/axiosInstance';

const route = useRoute();
const router = useRouter();
const videoId = route.params.id;

// Aquests són els valors que s'editaran
const video = ref({
  title: '',
  description: '',
  file_name: '',
  file_path: '',
  type: '',
  thumbnail_path: '',
  created_at: ''
});

const editedTitle = ref('');
const editedDescription = ref('');
const loading = ref(true);
const isSaving = ref(false);
const errorMessage = ref('');
const showSuccessToast = ref(false);

// Obtenir les detalls del vídeo per omplir el formulari
const getVideoDetails = async () => {
  try {
    const response = await api.get(`/multimedia/edit/${videoId}`, {
      headers: { Authorization: `Bearer ${localStorage.getItem('token')}` }
    });
    video.value = response.data;

    // Inicialitzar els valors editats amb els valors actuals
    editedTitle.value = video.value.title || '';
    editedDescription.value = video.value.description || '';
  } catch (error) {
    console.error('Error al carregar el contingut:', error);
    errorMessage.value = 'No s\'ha pogut carregar el contingut. Intenta-ho de nou.';
  } finally {
    loading.value = false;
  }
};

// Comprovar si és un vídeo
const isVideo = (media) => {
  return media.type && media.type.includes('video');
};

// Funció per obtenir la miniatura del vídeo o la imatge
const getThumbnail = (media) => {
  const baseUrl = import.meta.env.VITE_API_URL || 'http://localhost:8000';
  if (isVideo(media)) {
    // Si és un vídeo, utilitzar la miniatura o una imatge per defecte
    return media.thumbnail_path
        ? `${baseUrl}/storage/${media.thumbnail_path}`
        : 'https://ionicframework.com/docs/img/demos/thumbnail.svg';
  } else {
    // Si no és un vídeo, utilitzar la URL de la imatge
    return media.file_path
        ? `${baseUrl}/storage/${media.file_path}`
        : 'https://ionicframework.com/docs/img/demos/thumbnail.svg';
  }
};

// Formatar tipus de fitxer
const formatFileType = (type) => {
  if (!type) return 'Desconegut';

  if (type.includes('video')) return 'Vídeo';
  if (type.includes('image')) return 'Imatge';

  return type.split('/').pop().toUpperCase();
};

// Formatar data
const formatDate = (dateString) => {
  if (!dateString) return '';

  const date = new Date(dateString);
  return date.toLocaleDateString('ca-ES', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric'
  });
};

// Desar els canvis realitzats en el formulari
const saveVideo = async () => {
  if (!editedTitle.value.trim()) {
    errorMessage.value = 'El títol és obligatori';
    return;
  }

  isSaving.value = true;
  errorMessage.value = '';

  try {
    // Fer la petició PUT amb les dades actualitzades
    const response = await api.put('/multimedia/edit', {
      id: videoId,
      title: editedTitle.value,
      description: editedDescription.value,
    }, {
      headers: { Authorization: `Bearer ${localStorage.getItem('token')}` }
    });

    console.log('Resposta de l\'API:', response.data.message);

    // Mostrar missatge d'èxit
    showSuccessToast.value = true;

    // Redirigir després d'uns segons
    setTimeout(() => {
      router.push('/tabs/tab1');
    }, 2000);
  } catch (error) {
    console.error('Error al desar els canvis:', error);
    errorMessage.value = error.response?.data?.message || 'Error al desar els canvis. Intenta-ho de nou.';
  } finally {
    isSaving.value = false;
  }
};

// Cancel·lar l'edició
const cancelEdit = () => {
  router.push('/tabs/tab1');
};

// Obtenir els detalls del vídeo quan es carrega el component
onMounted(() => {
  getVideoDetails();
});
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

.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 200px;
  gap: 16px;
  color: var(--app-text-medium);
}

.edit-container {
  padding: 16px;
  max-width: 800px;
  margin: 0 auto;
}

.edit-header {
  margin-bottom: 24px;
  text-align: center;
}

.edit-title {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 8px;
  color: var(--app-primary);
}

.edit-subtitle {
  color: var(--app-text-light);
  font-size: 0.9rem;
}

.edit-content {
  background-color: var(--app-surface);
  border-radius: 12px;
  box-shadow: var(--app-shadow);
  overflow: hidden;
}

.preview-container {
  padding: 16px;
  border-bottom: 1px solid #eee;
}

.media-preview {
  position: relative;
  width: 100%;
  height: 200px;
  border-radius: 8px;
  overflow: hidden;
  margin-bottom: 16px;
  background-color: #f0f0f0;
}

.preview-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.media-type-badge {
  position: absolute;
  top: 8px;
  right: 8px;
  background-color: rgba(0, 0, 0, 0.6);
  color: white;
  border-radius: 50%;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.video-badge {
  background-color: var(--app-primary);
}

.file-info {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.info-item {
  display: flex;
  align-items: center;
  font-size: 0.9rem;
}

.info-label {
  font-weight: 500;
  color: var(--app-text-medium);
  width: 120px;
  flex-shrink: 0;
}

.info-value {
  color: var(--app-text);
}

.file-name {
  font-family: monospace;
  word-break: break-all;
}

.edit-form {
  padding: 16px;
}

.form-group {
  margin-bottom: 16px;
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

.error-alert {
  display: flex;
  align-items: center;
  gap: 8px;
  background-color: rgba(244, 67, 54, 0.1);
  color: var(--app-error);
  padding: 12px;
  border-radius: 8px;
  font-size: 14px;
  margin-bottom: 16px;
}

.error-alert ion-icon {
  font-size: 18px;
  flex-shrink: 0;
}

.form-actions {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.save-button {
  --border-radius: 8px;
  height: 48px;
  font-weight: 600;
  display: flex;
  justify-content: center;
  align-items: center;
}

.save-button ion-spinner {
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

.dark-theme .media-preview {
  background-color: #2a2a2a;
}

.dark-theme .preview-container {
  border-color: #333;
}
</style>