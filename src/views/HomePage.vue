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
            label="Source Language:"
            v-model="sourceLanguage">
            <ion-select-option v-for="lang in languages" :key="lang.value" :value="lang.value">
              {{ lang.label }}
            </ion-select-option>
          </ion-select>
          <ion-button>
          <ion-icon
            @click="swapLanguages()"
            slot="icon-only"
            :icon="swapHorizontal">
          </ion-icon>
        </ion-button>
          <ion-select
            label="Target Language:"
            v-model="targetLanguage">
            <ion-select-option v-for="lang in languages" :key="lang.value" :value="lang.value">
              {{ lang.label }}
            </ion-select-option>
        </ion-select>
        </ion-item>
          <ion-textarea
            label="Source Text:"
            v-model="sourceText"
          ></ion-textarea>
          <ion-textarea
            label="Translation:"
            readonly
            v-model="translation"
          ></ion-textarea>
        <ion-button
          @click="translateText()"
          :disabled="disableButton"
          >Translate</ion-button>
        <ion-button>
          <ion-icon
            @click="readAloud()"
            slot="icon-only"
            :icon="volumeHigh">
          </ion-icon>
        </ion-button>
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
import { swapHorizontal, volumeHigh } from 'ionicons/icons';
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
    IonButton
  },
  setup() {
    return {swapHorizontal, volumeHigh};
  },
  data() {
    return {
      disableButton: false,
      sourceLanguage: Language.German,
      targetLanguage: Language.English,
    languages: [
      { label: 'Deutsch', value: Language.German },
      { label: 'Englisch', value: Language.English },
      { label: 'Französisch', value: Language.French },
      { label: 'Spanisch', value: Language.Spanish },
      { label: 'Italienisch', value: Language.Italian },
    ],
      sourceText: "Enter Text",
      translation: "Translation"
    };
  },
  computed: {
  },
  methods: {
    async translateText() {
    const { text } = await Translation.translate({
    text: this.sourceText,
    sourceLanguage: this.sourceLanguage,
    targetLanguage: this.targetLanguage,
  });
  this.translation = text;
    },
    async swapLanguages() {
    },
    async readAloud() {
    }
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
