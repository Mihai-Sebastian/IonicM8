<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="main-toolbar">
        <ion-title>Iniciar Sessió</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <div class="auth-container">
        <div class="auth-logo">
          <ion-icon :icon="playCircle"></ion-icon>
          <h1>MediaVault</h1>
        </div>

        <ion-card class="auth-card">
          <ion-card-content>
            <h2 class="auth-title">Benvingut de nou</h2>
            <p class="auth-subtitle">Inicia sessió per accedir al teu contingut</p>

            <form @submit.prevent="login" class="auth-form">
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
                <div class="password-label">
                  <ion-label position="stacked">Contrasenya</ion-label>
                  <a class="forgot-password">Has oblidat la contrasenya?</a>
                </div>
                <ion-input
                    v-model="password"
                    :type="showPassword ? 'text' : 'password'"
                    placeholder="La teva contrasenya"
                    required
                    class="custom-input"
                    fill="solid"
                ></ion-input>
              </div>

              <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>

              <div class="auth-actions">
                <ion-button
                    expand="block"
                    type="submit"
                    class="login-button"
                    :disabled="!isFormValid || isLoading"
                >
                  <ion-icon :icon="logIn" slot="start"></ion-icon>
                  {{ isLoading ? 'Carregant...' : 'Iniciar Sessió' }}
                </ion-button>

                <div class="auth-divider">
                  <span>O</span>
                </div>

                <ion-button
                    expand="block"
                    fill="outline"
                    @click="goToRegister"
                    class="register-button"
                >
                  Crear un compte nou
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
import api from '@/api/axiosInstance';  // Assegura que aquest arxiu existeix i està ben configurat.
import {
  IonPage, IonContent, IonCard, IonCardContent,
  IonInput, IonButton, IonLabel, IonHeader,
  IonToolbar, IonTitle, IonIcon
} from '@ionic/vue';
import { logIn, playCircle } from 'ionicons/icons';

const email = ref('');
const password = ref('');
const isLoading = ref(false);
const errorMessage = ref('');
const router = useRouter();

const isFormValid = computed(() => {
  return email.value.trim() !== '' && password.value.trim() !== '';
});

const login = async () => {
  if (!isFormValid.value) return;
  isLoading.value = true;
  errorMessage.value = '';

  try {
    const response = await api.post('/login', {
      email: email.value,
      password: password.value
    });

    if (response.data.token) {
      localStorage.setItem('token', response.data.token);

      if (response.data.user?.id) {
        localStorage.setItem('userId', response.data.user.id);
      }

      router.push('/tabs/tab3');
    } else {
      throw new Error('Resposta invàlida del servidor');
    }
  } catch (error) {
    console.error('Error en iniciar sessió:', error);
    errorMessage.value =
        error.response?.data?.message || 'Error en iniciar sessió. Comprova les teves credencials.';
  } finally {
    isLoading.value = false;
  }
};

const goToRegister = () => {
  router.push('/register');
};
</script>

<style scoped>
/* Variables per al tema */
:root {
  --app-primary: #7c4dff;
  --app-background: #121212;
  --app-text: #ffffff;
}

/* Estils generals */
ion-content {
  --background: var(--app-background);
  --color: var(--app-text);
}

.main-toolbar {
  --background: var(--app-primary);
  --color: white;
}

.auth-container {
  display: flex;
  flex-direction: column;
  padding: 24px 16px;
  max-width: 500px;
  margin: 0 auto;
  height: 100%;
}

.auth-logo {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 32px 0;
}

.auth-logo ion-icon {
  font-size: 64px;
  color: var(--app-primary);
}

.auth-logo h1 {
  font-size: 28px;
  font-weight: 700;
  color: var(--app-primary);
}

.auth-card {
  border-radius: 16px;
  box-shadow: var(--app-shadow);
  margin: 0;
}

.auth-title {
  font-size: 24px;
  font-weight: 600;
  color: var(--app-text);
}

.auth-subtitle {
  font-size: 16px;
  color: var(--app-text);
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

.password-label {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.forgot-password {
  font-size: 14px;
  color: var(--app-primary);
}

.custom-input {
  --background: #333;
  --color: var(--app-text);
  --placeholder-color: #bbb;
  --border-radius: 8px;
  --padding-start: 16px;
  --padding-end: 16px;
}

.auth-actions {
  display: flex;
  flex-direction: column;
  gap: 24px;
  margin-top: 16px;
}

.error-message {
  color: #f44336;
  font-size: 14px;
  text-align: center;
  margin-top: -10px;
}

.login-button {
  --border-radius: 8px;
  height: 48px;
  font-weight: 600;
}

.register-button {
  --border-radius: 8px;
  height: 48px;
  --border-color: var(--app-primary);
  --color: var(--app-primary);
}

.auth-divider {
  display: flex;
  align-items: center;
  text-align: center;
  color: #bbb;
}

.auth-divider::before,
.auth-divider::after {
  content: '';
  flex: 1;
  border-bottom: 1px solid #444;
}

.auth-divider span {
  padding: 0 16px;
  font-size: 14px;
}
</style>
