<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="main-toolbar">
        <ion-buttons slot="start">
          <ion-button @click="goToLogin">
            <ion-icon :icon="arrowBack"></ion-icon>
          </ion-button>
        </ion-buttons>
        <ion-title>Crear Compte</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <div class="auth-container">
        <ion-card class="auth-card">
          <ion-card-content>
            <h2 class="auth-title">Uneix-te a MediaVault</h2>
            <p class="auth-subtitle">Crea un compte per començar a pujar contingut</p>

            <form @submit.prevent="register" class="auth-form">
              <div class="form-group">
                <ion-label position="stacked">Nom</ion-label>
                <ion-input
                    v-model="name"
                    placeholder="El teu nom"
                    required
                    class="custom-input"
                    fill="solid"
                ></ion-input>
              </div>

              <div class="form-group">
                <ion-label position="stacked">Correu electrònic</ion-label>
                <ion-input
                    v-model="email"
                    type="email"
                    placeholder="El teu correu electrònic"
                    required
                    class="custom-input"
                    fill="solid"
                ></ion-input>
              </div>

              <div class="form-group">
                <ion-label position="stacked">Contrasenya</ion-label>
                <ion-input
                    v-model="password"
                    type="password"
                    placeholder="Crea una contrasenya"
                    required
                    class="custom-input"
                    fill="solid"
                ></ion-input>
              </div>

              <div class="form-group">
                <ion-label position="stacked">Confirmar Contrasenya</ion-label>
                <ion-input
                    v-model="password_confirmation"
                    type="password"
                    placeholder="Confirma la contrasenya"
                    required
                    class="custom-input"
                    fill="solid"
                ></ion-input>
                <div v-if="passwordMismatch" class="error-message">
                  Les contrasenyes no coincideixen
                </div>
              </div>

              <div class="terms-agreement">
                <ion-checkbox v-model="agreeToTerms"></ion-checkbox>
                <span>Accepto els <a>Termes i Condicions</a> i la <a >Política de Privacitat</a></span>
              </div>

              <div v-if="errorMessage" class="error-alert">
                <ion-icon :icon="alertCircle"></ion-icon>
                {{ errorMessage }}
              </div>

              <div class="auth-actions">
                <ion-button
                    expand="block"
                    type="submit"
                    class="register-button"
                    :disabled="!isFormValid || isLoading"
                >
                  <ion-spinner v-if="isLoading" name="crescent"></ion-spinner>
                  <span v-else>Crear Compte</span>
                </ion-button>

                <div class="auth-divider">
                  <span>Ja tens un compte?</span>
                </div>

                <ion-button
                    expand="block"
                    fill="outline"
                    @click="goToLogin"
                    class="login-button"
                >
                  Iniciar Sessió
                </ion-button>
              </div>
            </form>
          </ion-card-content>
        </ion-card>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref, computed } from 'vue';
import { useRouter } from 'vue-router';
import {
  IonCheckbox, IonPage, IonContent, IonCard, IonCardContent,
  IonInput, IonButton, IonLabel, IonHeader, IonToolbar,
  IonTitle, IonButtons, IonIcon, IonSpinner
} from '@ionic/vue';
import { arrowBack, alertCircle } from 'ionicons/icons';
import api from '@/api/axiosInstance';

const name = ref('');
const email = ref('');
const password = ref('');
const password_confirmation = ref('');
const agreeToTerms = ref(false);
const isLoading = ref(false);
const errorMessage = ref('');
const router = useRouter();

const passwordMismatch = computed(() =>
    password.value &&
    password_confirmation.value &&
    password.value !== password_confirmation.value
);

const isFormValid = computed(() =>
    name.value.trim() !== '' &&
    email.value.trim() !== '' &&
    password.value.trim() !== '' &&
    password.value === password_confirmation.value &&
    agreeToTerms.value
);

const register = async () => {
  if (!isFormValid.value) return;
  isLoading.value = true;
  errorMessage.value = '';

  try {
    const response = await api.post('/register', {
      name: name.value,
      email: email.value,
      password: password.value,
      password_confirmation: password_confirmation.value
    });

    localStorage.setItem('token', response.data.token);
    if (response.data.user && response.data.user.id) {
      localStorage.setItem('userId', response.data.user.id);
    }

    router.push('/tabs/tab3');
  } catch (error) {
    console.error('Error en registrar-se:', error);
    errorMessage.value = error.response?.data?.message || 'Error en crear el compte. Intenta-ho de nou.';
  } finally {
    isLoading.value = false;
  }
};

const goToLogin = () => {
  router.push('/login');
};
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

.auth-container {
  display: flex;
  flex-direction: column;
  padding: 24px 16px;
  max-width: 500px;
  margin: 0 auto;
  height: 100%;
}

.auth-card {
  border-radius: 16px;
  box-shadow: var(--app-shadow);
  margin: 0;
}

.auth-title {
  font-size: 24px;
  font-weight: 600;
  margin: 0 0 8px 0;
  color: var(--app-text);
}

.auth-subtitle {
  font-size: 16px;
  color: var(--app-text-light);
  margin: 0 0 32px 0;
}

.auth-form {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.custom-input {
  --background: #f5f5f5;
  --color: var(--app-text);
  --placeholder-color: var(--app-text-light);
  --border-radius: 8px;
  --padding-start: 16px;
  --padding-end: 16px;
}

.error-message {
  color: var(--app-error);
  font-size: 14px;
  margin-top: 4px;
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
  margin-top: 8px;
}

.error-alert ion-icon {
  font-size: 18px;
  flex-shrink: 0;
}

.terms-agreement {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  font-size: 14px;
  color: var(--app-text-medium);
}

.terms-agreement a {
  color: var(--app-primary);
  text-decoration: none;
}

.auth-actions {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-top: 16px;
}

.register-button {
  --border-radius: 8px;
  height: 48px;
  font-weight: 600;
  display: flex;
  justify-content: center;
  align-items: center;
}

.register-button ion-spinner {
  width: 24px;
  height: 24px;
}

.login-button {
  --border-radius: 8px;
  height: 48px;
  --border-color: var(--app-primary);
  --color: var(--app-primary);
}

.auth-divider {
  display: flex;
  align-items: center;
  text-align: center;
  color: var(--app-text-light);
  margin: 8px 0;
}

.auth-divider::before,
.auth-divider::after {
  content: '';
  flex: 1;
  border-bottom: 1px solid #e0e0e0;
}

.auth-divider span {
  padding: 0 16px;
  font-size: 14px;
}
</style>