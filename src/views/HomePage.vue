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
            interface="popover"
            label="Source Language:"
            v-model="sourceLanguage">
            <ion-select-option
              v-for="lang in languages"
              :key="lang.value"
              :value="lang.value">
              {{ lang.label }}
            </ion-select-option>
          </ion-select>
        </ion-item>

        <div class="center-button">
          <ion-button size="small" @click="swapLanguages">
            <ion-icon slot="icon-only" :icon="swapHorizontal" />
          </ion-button>
        </div>

        <ion-item>
          <ion-select
            interface="popover"
            label="Target Language:"
            v-model="targetLanguage">
            <ion-select-option
              v-for="lang in languages"
              :key="lang.value"
              :value="lang.value">
              {{ lang.label }}
            </ion-select-option>
          </ion-select>
        </ion-item>

        <ion-item>
          <ion-textarea
            label="Source Text"
            v-model="sourceText"
            auto-grow
          ></ion-textarea>
        </ion-item>

        <ion-item>
          <ion-textarea
            label="Translation"
            v-model="translation"
            auto-grow
            readonly
          ></ion-textarea>
        </ion-item>

        <ion-button expand="block" @click="translateText" :disabled="disableButton">
          Translate
        </ion-button>

        <ion-button expand="block" color="medium" @click="readAloud">
          <ion-icon slot="start" :icon="volumeHigh" />
          Read Aloud
        </ion-button>
        <ion-button>
          <ion-icon
            id="copyToClipboard"
            @click="copyToClipboard()"
            slot="icon-only"
            :icon="copy">
          </ion-icon>
        </ion-button>
        <ion-toast trigger="copyToClipboard" message="Text copied to clipboard." :duration="5000"></ion-toast>
      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonItem,
  IonButton,
  IonSelect,
  IonSelectOption,
  IonTextarea,
  IonIcon
} from '@ionic/vue';
import { defineComponent } from 'vue';
import { swapHorizontal, volumeHigh } from 'ionicons/icons';
  IonInput,
  IonButton,
  IonToast, } from '@ionic/vue';
import { defineComponent } from "vue";
import { swapHorizontal, volumeHigh, copy } from 'ionicons/icons';
import { Translation, Language } from '@capacitor-mlkit/translation';
import { SpeechSynthesis } from '@capawesome-team/capacitor-speech-synthesis';

export default defineComponent({
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonItem,
    IonButton,
    IonSelect,
    IonSelectOption,
    IonTextarea,
    IonIcon
    IonInput,
    IonToast,
    IonButton
  },
  setup() {
    return { swapHorizontal, volumeHigh };
    return {swapHorizontal, volumeHigh, copy};
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
      sourceText: '',
      translation: ''
    };
  },
  methods: {
    async translateText() {
      const { text } = await Translation.translate({
        text: this.sourceText,
        sourceLanguage: this.sourceLanguage,
        targetLanguage: this.targetLanguage
      });
      this.translation = text;
  this.disableButton = true
  const { text } = await Translation.translate({
    text: this.sourceText,
    sourceLanguage: this.sourceLanguage,
    targetLanguage: this.targetLanguage
  })
  this.translation = text
  this.disableButton = false
    },
    swapLanguages() {
      const temp = this.sourceLanguage;
      this.sourceLanguage = this.targetLanguage;
      this.targetLanguage = temp;
    async swapLanguages() {
      // Swap the selected languages
      const temp = this.sourceLanguage;
      this.sourceLanguage = this.targetLanguage;
      this.targetLanguage = temp;
      // Swap the text in the textareas as well
      const tempText = this.sourceText;
      this.sourceText = this.translation;
      this.translation = tempText;
    },
    async readAloud() {
      await SpeechSynthesis.speak({
        text: this.translation,
        language: this.targetLanguage,
        rate: 1.0
      });
      await SpeechSynthesis.speak({
        text: this.translation
      });
    },
    async copyToClipboard() {
      await Clipboard.write({
        string: this.translation
      });
    }
  }
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
  display: flex;
  flex-direction: column;
  gap: 16px;
  padding: 16px;
}

#container p {
  font-size: 16px;
  line-height: 22px;

  color: #8c8c8c;

  margin: 0;
}

#container a {
  text-decoration: none;
.center-button {
  display: flex;
  justify-content: center;
}
</style>
