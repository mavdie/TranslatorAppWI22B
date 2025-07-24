<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Translator</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <div id="container">
        <ion-item>
          <ion-select
            label="Source Language"
            v-model="sourceLanguage"
            :options="languages"
          ></ion-select>
          <ion-textarea
            label="Source Text:"
            v-model="sourceText"
          ></ion-textarea>
        </ion-item>
        <ion-button
          @click="translateText()"
          :disabled="disableButton"
          >Translate</ion-button>
        <ion-button>
          <ion-icon
            @click="swapLanguages()"
            slot="icon-only"
            :icon="swapVertical">
          </ion-icon>
        </ion-button>
        <ion-select
          label="Target Language"
          v-model="targetLanguage"
          :options="languages"
        ></ion-select>
        <ion-textarea
            label="Translation:"
            readonly
            v-model="translation"
          ></ion-textarea>
      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonButtons,
  IonBackButton,
  IonItem,
  IonInput,
  IonButton, } from '@ionic/vue';
import { defineComponent } from "vue";
import { swapVertical } from 'ionicons/icons';
import { Translation, Language } from '@capacitor-mlkit/translation';
import { Clipboard } from '@capacitor/clipboard';
import { SpeechSynthesis, AudioSessionCategory, QueueStrategy } from '@capawesome-team/capacitor-speech-synthesis';

export default defineComponent({
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonButtons,
    IonBackButton,
    IonItem,
    IonInput,
    IonButton,
  },
  data() {
    return {
      batteryLevel: undefined as number | undefined,
      disableButton: false,
    };
  },
  computed: {
  },
  methods: {
    async translate() {
    },
    async swapLanguages() {
    },
  },

});
</script>

<style scoped>
#container {
  text-align: center;
  
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  
  color: #8c8c8c;
  
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
