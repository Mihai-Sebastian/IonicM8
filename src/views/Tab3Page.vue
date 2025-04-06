<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="main-toolbar">
        <ion-title>Perfil d'Usuari</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <!-- Estat de càrrega -->
      <div v-if="loading" class="loading-container">
        <ion-spinner name="crescent" color="primary"></ion-spinner>
        <p>Carregant perfil...</p>
      </div>

      <!-- Missatge d'error -->
      <div v-if="errorMessage && !loading" class="error-alert">
        <ion-icon :icon="alertCircle"></ion-icon>
        <span>{{ errorMessage }}</span>
      </div>

      <!-- Targeta de perfil -->
      <div class="profile-container" v-if="user && !loading">
        <ion-card class="profile-card">
          <div class="profile-header">
            <div class="avatar-container">
              <ion-avatar>
                <img :src="user.avatar || 'https://ionicframework.com/docs/img/demos/avatar.svg'" alt="Avatar" />
              </ion-avatar>
            </div>
            <ion-card-header>
              <ion-card-title class="profile-name">{{ user.name }}</ion-card-title>
              <ion-card-subtitle class="profile-email">{{ user.email }}</ion-card-subtitle>
            </ion-card-header>
          </div>
          <ion-card-content>
            <div class="profile-info">
              <div class="info-item">
                <span class="info-label">Usuari ID:</span>
                <span class="info-value">{{ user.id }}</span>
              </div>
              <div class="info-item">
                <span class="info-label">Continguts:</span>
                <span class="info-value">{{ videos.length }}</span>
              </div>
            </div>

          </ion-card-content>
        </ion-card>

        <!-- Secció de continguts -->
        <div class="content-section">
          <div class="section-header">
            <h2 class="section-title">Els Meus Continguts</h2>
            <ion-button size="small" fill="clear" @click="goToUpload" class="add-content-button">
              <ion-icon :icon="add"></ion-icon>
              Afegir
            </ion-button>
          </div>

          <!-- Estat buit -->
          <div v-if="videos.length === 0" class="empty-state">
            <ion-icon :icon="imagesOutline"></ion-icon>
            <h3>No tens cap contingut</h3>
            <p>Comença a pujar vídeos o imatges a la teva col·lecció</p>
            <ion-button @click="goToUpload">Afegir contingut</ion-button>
          </div>

          <!-- Graella de continguts -->
          <div v-else class="media-grid">
            <ion-card v-for="video in videos" :key="video.id" class="media-card">
              <div class="media-thumbnail" @click="viewMedia(video)">
                <img :src="getThumbnail(video)" :alt="video.file_name" />
                <div class="media-type-badge" :class="{ 'video-badge': isVideo(video) }">
                  <ion-icon :icon="isVideo(video) ? videocam : image"></ion-icon>
                </div>
                <div class="media-overlay">
                  <ion-icon :icon="isVideo(video) ? play : eye" class="play-icon"></ion-icon>
                </div>
              </div>
              <div class="media-info">
                <h3 class="media-title">{{ getFileName(video) }}</h3>
                <p class="media-date">{{ formatDate(video.created_at) }}</p>

                <div class="media-actions">
                  <ion-button fill="clear" size="small" @click="editVideo(video)">
                    <ion-icon :icon="pencil"></ion-icon>
                  </ion-button>
                  <ion-button fill="clear" size="small" color="danger" @click="confirmDelete(video.id)">
                    <ion-icon :icon="trash"></ion-icon>
                  </ion-button>
                </div>
              </div>
            </ion-card>
          </div>
        </div>
      </div>

      <!-- Alerta de confirmació d'eliminació -->
      <ion-alert
          :is-open="showDeleteAlert"
          header="Confirmació"
          message="Estàs segur que vols eliminar aquest contingut?"
          :buttons="[
          { text: 'Cancel·lar', role: 'cancel', handler: () => (showDeleteAlert = false) },
          { text: 'Eliminar', role: 'destructive', handler: deleteVideo }
        ]"
      ></ion-alert>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRouter } from 'vue-router';
import api from "@/api/axiosInstance";
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonCard, IonCardHeader, IonCardTitle, IonCardSubtitle, IonCardContent,
  IonText, IonSpinner, IonButton, IonAvatar, IonIcon, IonAlert
} from "@ionic/vue";
import {
  pencil, trash, videocam, image, alertCircle,
  add, imagesOutline, play, eye
} from 'ionicons/icons';

const router = useRouter();
const user = ref(null);
const loading = ref(true);
const errorMessage = ref("");
const videos = ref([]);
const showDeleteAlert = ref(false);
const videoToDelete = ref(null);

const fetchUserProfile = async () => {
  try {
    const response = await api.get("/user", {
      headers: { Authorization: `Bearer ${localStorage.getItem('token')}` }
    });
    user.value = response.data;
  } catch (error) {
    console.error("Error carregant el perfil:", error);
    errorMessage.value = "No s'ha pogut carregar el perfil.";
  } finally {
    loading.value = false;
  }
};

const getVideos = async () => {
  try {
    const response = await api.get('/multimedia', {
      headers: { Authorization: `Bearer ${localStorage.getItem('token')}` }
    });
    videos.value = response.data;
  } catch (error) {
    console.error('Error obtenint els continguts', error);
  }
};

const confirmDelete = (id) => {
  videoToDelete.value = id;
  showDeleteAlert.value = true;
};

const deleteVideo = async () => {
  try {
    await api.delete(`/multimedia/${videoToDelete.value}`, {
      headers: { Authorization: `Bearer ${localStorage.getItem('token')}` }
    });
    showDeleteAlert.value = false;
    getVideos();
  } catch (error) {
    console.error('Error eliminant el contingut', error);
  }
};

const editVideo = (video) => {
  router.push({ name: 'edit-video', params: { id: video.id } });
};

const viewMedia = (media) => {
  router.push({ name: 'view-media', params: { id: media.id } });
};

const goToUpload = () => {
  router.push('/tabs/tab2');
};

// Comprovar si és un vídeo
const isVideo = (media) => {
  return media.type && media.type.includes('video');
};

// Obtenir miniatura
const getThumbnail = (media) => {
  if (isVideo(media)) {
    // Retornar miniatura de vídeo o imatge per defecte
    const baseUrl = 'http://localhost:8000';
    const thumbnailUrl = media.thumbnail_path ? `${baseUrl}/storage/${media.thumbnail_path}` : 'https://www.techsmith.es/blog/wp-content/uploads/2023/03/how-to-make-a-youtube-video.png';

    return thumbnailUrl;  }
  else {
    const baseUrl = 'http://localhost:8000';
    // Per a imatges, utilitzar la URL de la imatge
    return media.file_path ? `${baseUrl}/storage/${media.file_path}` : 'https://www.techsmith.es/blog/wp-content/uploads/2023/03/how-to-make-a-youtube-video.png';
  }
};

// Obtenir nom de fitxer formatat
const getFileName = (media) => {
  // Mostrar títol si existeix, o nom de fitxer sense extensió
  if (media.title) return media.title;

  const fileName = media.file_name || '';
  return fileName.split('.').slice(0, -1).join('.') || fileName;
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

const formatFileType = (type) => {
  if (type.includes('video')) return 'Vídeo';
  if (type.includes('image')) return 'Imatge';
  return type;
};

onMounted(() => {
  fetchUserProfile();
  getVideos();
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

.error-alert {
  display: flex;
  align-items: center;
  gap: 8px;
  background-color: rgba(244, 67, 54, 0.1);
  color: var(--app-error);
  padding: 16px;
  border-radius: 8px;
  margin: 16px;
  font-size: 14px;
}

.error-alert ion-icon {
  font-size: 18px;
  flex-shrink: 0;
}

.profile-container {
  padding: 16px;
  max-width: 1200px;
  margin: 0 auto;
}

.profile-card {
  border-radius: 16px;
  margin-bottom: 24px;
  box-shadow: var(--app-shadow);
  overflow: hidden;
}

.profile-header {
  display: flex;
  align-items: center;
  padding: 16px 16px 0 16px;
}

.avatar-container {
  margin-right: 16px;
}

ion-avatar {
  width: 72px;
  height: 72px;
  border: 3px solid var(--app-primary);
}

.profile-name {
  font-size: 1.5rem;
  font-weight: 600;
  margin: 0;
  color: var(--app-text);
}

.profile-email {
  font-size: 0.9rem;
  color: var(--app-text-light);
}

.profile-info {
  margin-bottom: 16px;
}

.info-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  font-size: 0.9rem;
}

.info-label {
  color: var(--app-text-medium);
}

.info-value {
  font-weight: 500;
  color: var(--app-text);
}

.edit-profile-button {
  margin-top: 8px;
}

.content-section {
  margin-top: 32px;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.section-title {
  font-size: 1.25rem;
  font-weight: 600;
  margin: 0;
  color: var(--app-text);
}

.add-content-button {
  font-weight: 500;
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 48px 0;
  text-align: center;
}

.empty-state ion-icon {
  font-size: 64px;
  color: var(--app-text-light);
  opacity: 0.5;
  margin-bottom: 16px;
}

.empty-state h3 {
  font-size: 1.2rem;
  font-weight: 600;
  margin: 0 0 8px 0;
  color: var(--app-text);
}

.empty-state p {
  color: var(--app-text-light);
  margin: 0 0 16px 0;
  max-width: 300px;
}

.media-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 16px;
}

.media-card {
  border-radius: 12px;
  overflow: hidden;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  margin: 0;
  box-shadow: var(--app-shadow);
}

.media-card:hover {
  transform: translateY(-4px);
  box-shadow: var(--app-shadow-strong);
}

.media-thumbnail {
  position: relative;
  height: 140px;
  overflow: hidden;
  cursor: pointer;
}

.media-thumbnail img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.media-card:hover .media-thumbnail img {
  transform: scale(1.05);
}

.media-type-badge {
  position: absolute;
  top: 8px;
  right: 8px;
  background-color: rgba(0, 0, 0, 0.6);
  color: white;
  border-radius: 50%;
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2;
}

.video-badge {
  background-color: var(--app-primary);
}

.media-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.7) 0%, rgba(0, 0, 0, 0) 50%);
  opacity: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: opacity 0.3s ease;
}

.media-card:hover .media-overlay {
  opacity: 1;
}

.play-icon {
  font-size: 48px;
  color: white;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.3));
}

.media-info {
  padding: 12px;
}

.media-title {
  font-size: 0.95rem;
  font-weight: 600;
  margin: 0 0 4px 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  color: var(--app-text);
}

.media-date {
  font-size: 0.8rem;
  margin: 0 0 8px 0;
  color: var(--app-text-light);
}

.media-actions {
  display: flex;
  justify-content: flex-end;
  gap: 4px;
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
</style>