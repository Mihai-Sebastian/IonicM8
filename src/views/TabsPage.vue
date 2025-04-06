<template>
  <ion-page>
    <!-- Navbar superior -->
    <ion-header>
      <ion-toolbar class="main-toolbar">
        <div class="app-header">
          <ion-title>
            <div class="app-title">
              <ion-icon :icon="playCircle" class="app-logo"></ion-icon>
              <span>MediaVault</span>
            </div>
          </ion-title>
        </div>
        <ion-buttons slot="end">
          <ion-button v-if="!isAuthenticated" @click="goToLogin" class="auth-button">
            <ion-icon :icon="logIn" slot="start"></ion-icon>
            Iniciar sessió
          </ion-button>
          <ion-button v-if="isAuthenticated" @click="showLogoutConfirm" class="auth-button">
            <ion-icon :icon="logOut" slot="start"></ion-icon>
            Sortir
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <ion-tabs>
        <!-- Clau dinàmica per forçar el re-render -->
        <ion-router-outlet :key="$route.fullPath"></ion-router-outlet>

        <ion-tab-bar slot="bottom" class="custom-tab-bar">
          <ion-tab-button tab="tab1" href="/tabs/tab1" class="tab-button">
            <ion-icon :icon="grid" />
            <ion-label>Galeria</ion-label>
          </ion-tab-button>

          <ion-tab-button tab="tab2" href="/tabs/tab2" class="tab-button upload-tab">
            <div class="upload-icon-container">
              <ion-icon :icon="add" />
            </div>
            <ion-label>Pujar</ion-label>
          </ion-tab-button>

          <ion-tab-button tab="tab3" href="/tabs/tab3" class="tab-button">
            <ion-icon :icon="person" />
            <ion-label>Perfil</ion-label>
          </ion-tab-button>
        </ion-tab-bar>
      </ion-tabs>
    </ion-content>

    <!-- Alerta de confirmació de tancament de sessió -->
    <ion-alert
        :is-open="showAlert"
        header="Tancar sessió"
        message="Estàs segur que vols tancar la sessió?"
        :buttons="alertButtons"
        @didDismiss="showAlert = false"
    ></ion-alert>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { useRouter } from 'vue-router';
import {
  IonTabBar, IonTabButton, IonTabs, IonLabel, IonIcon,
  IonPage, IonRouterOutlet, IonHeader, IonToolbar,
  IonTitle, IonButtons, IonButton, IonContent, IonAlert
} from '@ionic/vue';
import {
  grid, add, person, logIn, logOut,
  playCircle
} from 'ionicons/icons';

const router = useRouter();
const isAuthenticated = computed(() => !!localStorage.getItem('token'));
const showAlert = ref(false);

const alertButtons = [
  {
    text: 'Cancel·lar',
    role: 'cancel',
    handler: () => {
      console.log('Logout cancelled');
    }
  },
  {
    text: 'Sortir',
    role: 'confirm',
    handler: () => {
      logout();
    }
  }
];

const showLogoutConfirm = () => {
  showAlert.value = true;
};

const logout = () => {
  localStorage.removeItem('token');
  router.push('/login');
};

const goToLogin = () => {
  router.push('/login');
};
</script>

<style>
/* Variables CSS globals per a tota l'aplicació */
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

.app-header {
  display: flex;
  align-items: center;
}

.app-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
}

.app-logo {
  font-size: 24px;
  color: white;
}

.auth-button {
  font-weight: 500;
  --padding-start: 12px;
  --padding-end: 12px;
  --color: white;
}

.custom-tab-bar {
  --background: var(--app-surface);
  --border: none;
  box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.05);
  padding: 6px 0;
  height: 60px;
}

.tab-button {
  --color: var(--app-text-light);
  --color-selected: var(--app-primary);
  position: relative;
  transition: all 0.2s ease;
}

.tab-button::before {
  content: '';
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%) scaleX(0);
  width: 24px;
  height: 3px;
  background-color: var(--app-primary);
  border-radius: 0 0 4px 4px;
  transition: transform 0.2s ease;
}

.tab-button.tab-selected::before {
  transform: translateX(-50%) scaleX(1);
}

.upload-tab {
  --color: white;
}

.upload-icon-container {
  background: linear-gradient(135deg, var(--app-primary), var(--app-secondary));
  border-radius: 50%;
  width: 44px;
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 2px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.upload-icon-container ion-icon {
  font-size: 24px;
}
</style>