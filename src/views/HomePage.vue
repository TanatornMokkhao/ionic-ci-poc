<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Push test</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="ion-padding">
      <p>Build: <strong>{{ appVersion }}</strong> · API: <strong>{{ apiEnv }}</strong></p>
      <p>Status: <strong>{{ status }}</strong></p>

      <ion-button expand="block" :disabled="!isNative" @click="register">Register for push</ion-button>
      <p v-if="!isNative">Push works only in the Android app, not in the browser.</p>

      <template v-if="token">
        <p>FCM token:</p>
        <ion-textarea :value="token" readonly auto-grow fill="outline" />
        <ion-button expand="block" fill="outline" @click="copyToken">Copy token</ion-button>
      </template>

      <template v-if="messages.length">
        <h3>Received</h3>
        <ion-list>
          <ion-item v-for="(m, i) in messages" :key="i">
            <ion-label>
              <h2>{{ m.title }}</h2>
              <p>{{ m.body }}</p>
            </ion-label>
          </ion-item>
        </ion-list>
      </template>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { Capacitor } from '@capacitor/core';
import { PushNotifications } from '@capacitor/push-notifications';
import {
  IonButton, IonContent, IonHeader, IonItem, IonLabel, IonList, IonPage, IonTextarea, IonTitle, IonToolbar,
} from '@ionic/vue';

const appVersion = import.meta.env.VITE_APP_VERSION || 'local';
const apiEnv = import.meta.env.VITE_API_ENV || 'dev';
const isNative = Capacitor.isNativePlatform();
const status = ref(isNative ? 'Not registered' : 'Browser');
const token = ref('');
const messages = ref<{ title: string; body: string }[]>([]);

onMounted(async () => {
  if (!isNative) return;
  await PushNotifications.addListener('registration', (t) => {
    token.value = t.value;
    status.value = 'Registered';
  });
  await PushNotifications.addListener('registrationError', (e) => {
    status.value = `Error: ${e.error}`;
  });
  await PushNotifications.addListener('pushNotificationReceived', (n) => {
    messages.value.unshift({ title: n.title ?? '', body: n.body ?? '' });
  });
});

async function register() {
  const perm = await PushNotifications.requestPermissions();
  if (perm.receive !== 'granted') {
    status.value = 'Permission denied';
    return;
  }
  status.value = 'Registering...';
  await PushNotifications.register();
}

async function copyToken() {
  await navigator.clipboard.writeText(token.value);
}
</script>
