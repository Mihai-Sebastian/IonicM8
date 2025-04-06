<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="main-toolbar">
        <ion-buttons slot="start">
          <ion-back-button default-href="/tabs/tab1" text=""></ion-back-button>
        </ion-buttons>
        <ion-title>{{ media?.title || 'Carregant contingut...' }}</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <!-- Estat de càrrega -->
      <div v-if="loading" class="loading-container">
        <ion-spinner name="crescent" color="primary"></ion-spinner>
        <p>Carregant contingut...</p>
      </div>

      <!-- Missatge d'error -->
      <div v-if="errorMessage && !loading" class="error-alert">
        <ion-icon :icon="alertCircle"></ion-icon>
        <span>{{ errorMessage }}</span>
      </div>

      <!-- Contingut principal -->
      <div v-if="media && !loading" class="media-view-container">
        <div class="media-main-content">
          <!-- Reproductor de vídeo o visualitzador d'imatge -->
          <div class="media-player-container">
            <!-- Reproductor de vídeo -->
            <div v-if="isVideo(media)" class="video-player">
              <video
                  controls
                  autoplay
                  playsinline
                  :src="getVideoUrl(media)"
                  :poster="getThumbnail(media)"
                  class="video-element"
                  @loadedmetadata="onLoadedMetadata"
                  @timeupdate="onTimeUpdate"
              >
              </video>
            </div>

            <!-- Visualitzador d'imatge -->
            <div v-else class="image-viewer">
              <img :src="getThumbnail(media)" :alt="media.title" class="image-element" />
            </div>
          </div>

          <!-- Informació del contingut -->
          <div class="media-info-container">
            <h1 class="media-title">{{ media.title }}</h1>



            <!-- Informació de l'usuari -->
            <div class="user-info">
              <ion-avatar class="user-avatar">
                <img :src="media.user?.avatar || 'https://ionicframework.com/docs/img/demos/avatar.svg'" alt="Avatar" />
              </ion-avatar>
              <div class="user-details">
                <h3 class="user-name">{{ media.user?.name || 'Usuari' }}</h3>
                <p class="user-meta">{{ formatDate(media.user?.created_at) }}</p>
              </div>
              <ion-button fill="outline" size="small" class="follow-button">
                Seguir
              </ion-button>
            </div>

            <!-- Descripció -->
            <div class="media-description">
              <p v-if="media.description">{{ media.description }}</p>
              <p v-else class="no-description">Sense descripció</p>
            </div>

            <!-- Accions -->
            <div class="media-actions">
              <ion-button fill="clear" size="small">
                <ion-icon :icon="thumbsUp" slot="start"></ion-icon>
                Like
              </ion-button>
              <ion-button fill="clear" size="small">
                <ion-icon :icon="share" slot="start"></ion-icon>
                Compartir
              </ion-button>
              <ion-button fill="clear" size="small" @click="handleDownload">
                <ion-icon :icon="download" slot="start"></ion-icon>
                Descarregar
              </ion-button>
            </div>
          </div>
        </div>

        <!-- Continguts relacionats -->
        <div class="related-content">
          <h2 class="related-title">Continguts relacionats</h2>

          <!-- Estat buit -->
          <div v-if="relatedVideos.length === 0" class="empty-related">
            <ion-icon :icon="videocamOutline"></ion-icon>
            <p>No hi ha continguts relacionats</p>
          </div>

          <!-- Llista de continguts relacionats -->
          <div v-else class="related-list">
            <div
                v-for="video in relatedVideos"
                :key="video.id"
                class="related-item"
                @click="goToVideo(video.id)"
            >
              <div class="related-thumbnail">
                <img :src="getThumbnail(video)" :alt="video.title" />
                <div class="related-type-badge" :class="{ 'video-badge': isVideo(video) }">
                  <ion-icon :icon="isVideo(video) ? videocam : image"></ion-icon>
                </div>
              </div>
              <div class="related-info">
                <h3 class="related-title">{{ video.title || 'Sense títol' }}</h3>
                <p class="related-user">{{ video.user?.name || 'Usuari' }}</p>
                <p class="related-date">{{ formatDate(video.created_at) }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonButtons, IonBackButton, IonSpinner, IonIcon,
  IonAvatar, IonButton
} from '@ionic/vue';
import {
  videocam, image, alertCircle, thumbsUp, share,
  download, videocamOutline
} from 'ionicons/icons';
import api from '@/api/axiosInstance';

const route = useRoute();
const router = useRouter();
const media = ref(null);
const relatedVideos = ref([]);
const loading = ref(true);
const errorMessage = ref('');

// Funció per identificar si el fitxer és un vídeo
const isVideo = (media) => {
  return media.type && media.type.includes('video');
};

// Funció per obtenir la URL del vídeo
const getVideoUrl = (media) => {
  const baseUrl = import.meta.env.VITE_API_URL || 'http://localhost:8000';
  return `${baseUrl}/storage/${media.file_path}`;
};

// Funció per obtenir la miniatura o la imatge segons el tipus de fitxer
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

// Funció per formatar dates
const formatDate = (dateString) => {
  if (!dateString) return '';

  const date = new Date(dateString);
  return date.toLocaleDateString('ca-ES', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric'
  });
};

// Funció per obtenir el mitjà i els vídeos relacionats
const fetchMediaWithRelated = async () => {
  const id = route.params.id;
  loading.value = true;
  errorMessage.value = '';

  try {
    // Realitzar la crida a l'API per obtenir tant el vídeo actual com els vídeos relacionats
    const response = await api.get(`/multimedia/${id}/related`, {
      headers: { Authorization: `Bearer ${localStorage.getItem('token')}` }
    });
    media.value = response.data.media;
    relatedVideos.value = response.data.relatedVideos || [];
  } catch (error) {
    console.error('Error fetching media and related videos:', error);
    errorMessage.value = 'No s\'ha pogut carregar el contingut. Intenta-ho de nou.';
  } finally {
    loading.value = false;
  }
};

// Funció per redirigir a la vista d'un vídeo relacionat
const goToVideo = (id) => {
  router.push({ name: 'view-media', params: { id } });
};

const handleDownload = () => {
  const fileUrl = getVideoUrl(media.value);

  const link = document.createElement('a');
  link.href = fileUrl;
  link.download = media.value.file_path.split('/').pop();
  link.click();
};

onMounted(() => {
  fetchMediaWithRelated();
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

.media-view-container {
  display: flex;
  flex-direction: column;
  padding: 16px;
  max-width: 1400px;
  margin: 0 auto;
}

@media (min-width: 992px) {
  .media-view-container {
    flex-direction: row;
    gap: 24px;
  }

  .media-main-content {
    flex: 1;
    max-width: 70%;
  }

  .related-content {
    width: 30%;
    min-width: 300px;
  }
}

.media-player-container {
  margin-bottom: 16px;
  border-radius: 12px;
  overflow: hidden;
  background-color: #000;
  box-shadow: var(--app-shadow);
}

.video-player, .image-viewer {
  width: 100%;
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  align-items: center;
  justify-content: center;
}

.video-element {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.image-element {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.media-info-container {
  background-color: var(--app-surface);
  border-radius: 12px;
  padding: 16px;
  margin-bottom: 24px;
  box-shadow: var(--app-shadow);
}

.media-title {
  font-size: 1.5rem;
  font-weight: 600;
  margin: 0 0 12px 0;
  color: var(--app-text);
}

.media-stats {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
}

.media-type-badge {
  display: flex;
  align-items: center;
  gap: 4px;
  background-color: #f0f0f0;
  color: var(--app-text-medium);
  padding: 4px 8px;
  border-radius: 16px;
  font-size: 0.8rem;
}

.video-badge {
  background-color: rgba(var(--ion-color-primary-rgb), 0.1);
  color: var(--app-primary);
}

.media-date {
  font-size: 0.85rem;
  color: var(--app-text-light);
}

.user-info {
  display: flex;
  align-items: center;
  padding: 16px 0;
  border-top: 1px solid #eee;
  border-bottom: 1px solid #eee;
  margin-bottom: 16px;
}

.user-avatar {
  width: 48px;
  height: 48px;
  margin-right: 12px;
}

.user-details {
  flex: 1;
}

.user-name {
  font-size: 1rem;
  font-weight: 600;
  margin: 0 0 4px 0;
  color: var(--app-text);
}

.user-meta {
  font-size: 0.8rem;
  color: var(--app-text-light);
  margin: 0;
}

.follow-button {
  --border-radius: 20px;
  font-weight: 500;
}

.media-description {
  margin-bottom: 16px;
  font-size: 0.95rem;
  line-height: 1.5;
  color: var(--app-text);
}

.no-description {
  color: var(--app-text-light);
  font-style: italic;
}

.media-actions {
  display: flex;
  gap: 8px;
  border-top: 1px solid #eee;
  padding-top: 16px;
}

.related-content {
  margin-top: 16px;
}

.related-title {
  font-size: 1.2rem;
  font-weight: 600;
  margin: 0 0 16px 0;
  color: var(--app-text);
}

.empty-related {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 32px 0;
  text-align: center;
  background-color: var(--app-surface);
  border-radius: 12px;
  box-shadow: var(--app-shadow);
}

.empty-related ion-icon {
  font-size: 48px;
  color: var(--app-text-light);
  opacity: 0.5;
  margin-bottom: 12px;
}

.empty-related p {
  color: var(--app-text-light);
  margin: 0;
}

.related-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.related-item {
  display: flex;
  gap: 12px;
  padding: 12px;
  background-color: var(--app-surface);
  border-radius: 12px;
  cursor: pointer;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  box-shadow: var(--app-shadow);
}

.related-item:hover {
  transform: translateY(-2px);
  box-shadow: var(--app-shadow-strong);
}

.related-thumbnail {
  position: relative;
  width: 120px;
  height: 68px;
  border-radius: 8px;
  overflow: hidden;
  flex-shrink: 0;
}

.related-thumbnail img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.related-type-badge {
  position: absolute;
  top: 4px;
  right: 4px;
  background-color: rgba(0, 0, 0, 0.6);
  color: white;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
}

.related-type-badge.video-badge {
  background-color: var(--app-primary);
}

.related-info {
  flex: 1;
  min-width: 0;
}

.related-info h3 {
  font-size: 0.9rem;
  font-weight: 600;
  margin: 0 0 4px 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  color: var(--app-text);
}

.related-user {
  font-size: 0.8rem;
  margin: 0 0 2px 0;
  color: var(--app-text-medium);
}

.related-date {
  font-size: 0.75rem;
  margin: 0;
  color: var(--app-text-light);
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

.dark-theme .media-type-badge {
  background-color: #333;
}

.dark-theme .user-info,
.dark-theme .media-actions {
  border-color: #333;
}
</style>