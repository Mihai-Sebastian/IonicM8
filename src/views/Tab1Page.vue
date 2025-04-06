<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="main-toolbar">
        <ion-title>Galeria de Continguts</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="goToAddVideo" class="add-button">
            <ion-icon :icon="add" slot="start"></ion-icon>
            Afegir
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <!-- Estat de càrrega -->
      <div v-if="loading" class="loading-container">
        <ion-spinner name="crescent" color="primary"></ion-spinner>
        <p>Carregant continguts...</p>
      </div>

      <!-- Missatge d'error -->
      <div v-if="error" class="error-container">
        <ion-icon :icon="alertCircle" color="danger" size="large"></ion-icon>
        <p>{{ error }}</p>
        <ion-button size="small" @click="getAllVideos">Tornar a intentar</ion-button>
      </div>

      <!-- Filtres i ordenació -->
      <div class="filters-container" v-if="!loading && videos.length > 0">
        <ion-segment v-model="currentFilter" class="filter-segment">
          <ion-segment-button value="all">Tots</ion-segment-button>
          <ion-segment-button value="videos">Vídeos</ion-segment-button>
          <ion-segment-button value="images">Imatges</ion-segment-button>
        </ion-segment>

        <ion-button fill="clear" size="small" @click="toggleSortMenu" class="sort-button">
          <ion-icon :icon="options" slot="start"></ion-icon>
          Ordenar
        </ion-button>
      </div>

      <!-- Galeria de continguts -->
      <div class="gallery-container" v-if="!loading && filteredVideos.length > 0">
        <div class="media-grid">
          <div v-for="video in filteredVideos" :key="video.id" class="media-card">
            <div class="media-thumbnail" @click="viewMedia(video)">
              <img
                  :src="getThumbnail(video)"
                  :alt="video.file_name"
                  loading="lazy"
              />
              <div class="media-type-badge" :class="{ 'video-badge': isVideo(video) }">
                <ion-icon :icon="isVideo(video) ? videocam : image"></ion-icon>
              </div>
              <div class="media-overlay">
                <ion-icon :icon="isVideo(video) ? play : eye" class="play-icon"></ion-icon>
              </div>
            </div>
            <div class="media-info">
              <h3 class="media-title">{{ getFileName(video) }}</h3>
              <p class="media-user">{{ video.user && video.user.name ? video.user.name : 'Usuari' }}</p>

              <div class="media-actions" v-if="isOwnMedia(video)">
                <ion-button fill="clear" size="small" @click="editVideo(video)">
                  <ion-icon :icon="pencil"></ion-icon>
                </ion-button>
                <ion-button fill="clear" size="small" color="danger" @click="confirmDelete(video.id)">
                  <ion-icon :icon="trash"></ion-icon>
                </ion-button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Estat buit -->
      <div v-if="!loading && filteredVideos.length === 0" class="empty-container">
        <ion-icon :icon="imagesOutline" size="large"></ion-icon>
        <h3>No hi ha continguts disponibles</h3>
        <p>Sigues el primer en pujar contingut multimèdia</p>
        <ion-button @click="goToAddVideo">Afegir contingut</ion-button>
      </div>

      <!-- Menú d'ordenació -->
      <ion-action-sheet
          :is-open="showSortMenu"
          header="Ordenar per"
          :buttons="sortOptions"
          @didDismiss="showSortMenu = false"
      ></ion-action-sheet>

      <!-- Alerta de confirmació d'eliminació -->
      <ion-alert
          :is-open="showDeleteAlert"
          header="Eliminar contingut"
          message="Estàs segur que vols eliminar aquest contingut?"
          :buttons="deleteAlertButtons"
          @didDismiss="showDeleteAlert = false"
      ></ion-alert>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonButtons, IonButton, IonIcon, IonSpinner, IonSegment,
  IonSegmentButton, IonActionSheet, IonAlert
} from '@ionic/vue';
import {
  add, alertCircle, videocam, image, eye, play,
  pencil, trash, options, imagesOutline
} from 'ionicons/icons';
import api from '@/api/axiosInstance';

const router = useRouter();
const videos = ref([]);
const loading = ref(true);
const error = ref('');
const currentFilter = ref('all');
const currentSort = ref('newest');
const showSortMenu = ref(false);
const showDeleteAlert = ref(false);
const videoToDelete = ref(null);

// Opcions d'ordenació
const sortOptions = [
  {
    text: 'Més recents primer',
    handler: () => {
      currentSort.value = 'newest';
    }
  },
  {
    text: 'Més antics primer',
    handler: () => {
      currentSort.value = 'oldest';
    }
  },
  {
    text: 'Alfabèticament (A-Z)',
    handler: () => {
      currentSort.value = 'nameAsc';
    }
  },
  {
    text: 'Alfabèticament (Z-A)',
    handler: () => {
      currentSort.value = 'nameDesc';
    }
  },
  {
    text: 'Cancel·lar',
    role: 'cancel'
  }
];

// Botons d'alerta d'eliminació
const deleteAlertButtons = [
  {
    text: 'Cancel·lar',
    role: 'cancel'
  },
  {
    text: 'Eliminar',
    role: 'destructive',
    handler: () => {
      if (videoToDelete.value) {
        deleteVideo(videoToDelete.value);
      }
    }
  }
];

// Obtenir tots els vídeos (sense filtre d'usuari)
const getAllVideos = async () => {
  loading.value = true;
  error.value = '';

  try {
    const token = localStorage.getItem('token');
    const headers = token ? { Authorization: `Bearer ${token}` } : {};

    const response = await api.get('/multimedia/all', { headers });
    videos.value = response.data;
  } catch (err) {
    console.error('Error obtenint els continguts', err);
    error.value = 'No s\'han pogut carregar els continguts. Torna a intentar-ho.';
  } finally {
    loading.value = false;
  }
};

// Filtrar vídeos segons el filtre actual
const filteredVideos = computed(() => {
  let filtered = [...videos.value];

  // Aplicar filtre per tipus
  if (currentFilter.value === 'videos') {
    filtered = filtered.filter(v => isVideo(v));
  } else if (currentFilter.value === 'images') {
    filtered = filtered.filter(v => !isVideo(v));
  }

  // Aplicar ordenació
  if (currentSort.value === 'newest') {
    filtered.sort((a, b) => new Date(b.created_at) - new Date(a.created_at));
  } else if (currentSort.value === 'oldest') {
    filtered.sort((a, b) => new Date(a.created_at) - new Date(b.created_at));
  } else if (currentSort.value === 'nameAsc') {
    filtered.sort((a, b) => a.file_name.localeCompare(b.file_name));
  } else if (currentSort.value === 'nameDesc') {
    filtered.sort((a, b) => b.file_name.localeCompare(a.file_name));
  }

  return filtered;
});

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

// Comprovar si el contingut pertany a l'usuari actual
const isOwnMedia = (media) => {
  const userId = localStorage.getItem('userId');
  return userId && media.user_id === parseInt(userId);
};

// Confirmar eliminació
const confirmDelete = (id) => {
  videoToDelete.value = id;
  showDeleteAlert.value = true;
};

// Eliminar vídeo
const deleteVideo = async (id) => {
  try {
    const token = localStorage.getItem('token');
    await api.delete(`/multimedia/${id}`, {
      headers: { Authorization: `Bearer ${token}` }
    });
    getAllVideos();
  } catch (error) {
    console.error('Error eliminant el contingut', error);
  }
};

// Editar vídeo
const editVideo = (video) => {
  router.push({ name: 'edit-video', params: { id: video.id } });
};

// Veure contingut
const viewMedia = (media) => {
  router.push({ name: 'view-media', params: { id: media.id } });
};

// Mostrar menú d'ordenació
const toggleSortMenu = () => {
  showSortMenu.value = true;
};

// Anar a afegir vídeo
const goToAddVideo = () => {
  router.push('/tabs/tab2');
};

// Carregar vídeos en muntar el component
onMounted(() => {
  getAllVideos();
});
</script>


<style>
.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 200px;
  gap: 16px;
  color: var(--app-text-medium);
}

.error-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 200px;
  gap: 16px;
  color: var(--app-error);
  text-align: center;
  padding: 0 32px;
}

.filters-container {
  padding: 12px 16px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: var(--app-surface);
  position: sticky;
  top: 0;
  z-index: 10;
}

.filter-segment {
  max-width: 70%;
}

.sort-button {
  font-size: 0.9rem;
}

.gallery-container {
  padding: 12px;
}

.media-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 16px;
}

.media-card {
  border-radius: 12px;
  overflow: hidden;
  background-color: var(--app-card);
  box-shadow: var(--app-shadow);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.media-card:hover {
  transform: translateY(-4px);
  box-shadow: var(--app-shadow-strong);
}

.media-thumbnail {
  position: relative;
  height: 160px;
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

.media-user {
  font-size: 0.8rem;
  margin: 0;
  color: var(--app-text-light);
}

.media-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 8px;
}

.empty-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 400px;
  gap: 16px;
  text-align: center;
  padding: 0 32px;
}

.empty-container ion-icon {
  font-size: 64px;
  color: var(--app-text-light);
  opacity: 0.5;
}

.empty-container h3 {
  font-size: 1.2rem;
  font-weight: 600;
  margin: 0;
  color: var(--app-text);
}

.empty-container p {
  color: var(--app-text-light);
  margin: 0;
}

.add-button {
  --color: white;
}

/* Estil per a la barra d'eines principal */
.main-toolbar {
  --background: var(--app-primary);
  --color: white;
}
</style>