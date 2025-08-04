<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Translator</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <div id="container">
        <div class="button-row">
          <ion-select
            aria-label="Source Language:"
            interface="popover"
            v-model="sourceLanguage">
            <ion-select-option
              v-for="lang in languages"
              :key="lang.value"
              :value="lang.value">
              {{ lang.label }}
            </ion-select-option>
          </ion-select>

          <ion-button @click="swapLanguages()" fill="clear">
            <ion-icon slot="icon-only" :icon="swapHorizontal"></ion-icon>
          </ion-button>

          <ion-select
            aria-label="Target Language:"
            interface="popover"
            v-model="targetLanguage">
            <ion-select-option
              v-for="lang in languages"
              :key="lang.value"
              :value="lang.value">
              {{ lang.label }}
            </ion-select-option>
          </ion-select>
        </div>

        <ion-item>
          <ion-textarea
            label="Source Text:"
            label-placement="stacked"
            placeholder="Enter text"
            auto-grow
            v-model="sourceText">
          </ion-textarea>
        </ion-item>

        <ion-item>
          <ion-textarea
            label="Translation:"
            label-placement="stacked"
            placeholder="Translation"
            auto-grow
            readonly
            v-model="translation">
          </ion-textarea>
        </ion-item>

        <div class="button-row">
          <ion-button @click="readAloud" fill="clear">
            <ion-icon slot="icon-only" :icon="volumeHigh"></ion-icon>
          </ion-button>
          <ion-button id="copyToClipboard" @click="copyToClipboard" fill="clear">
            <ion-icon slot="icon-only" :icon="copy"></ion-icon>
          </ion-button>
        </div>

        <ion-toast
          trigger="copyToClipboard"
          message="Text copied to clipboard."
          :duration="2000">
        </ion-toast>

        <ion-loading
          :is-open="loading"
          :message="loadingMessage"
          :backdrop-dismiss="false"
          duration="0">
        </ion-loading>
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
  IonToast,
  IonSelect,
  IonSelectOption,
  IonLoading,
  IonTextarea
} from '@ionic/vue';

import { defineComponent, ref, onMounted, watch } from 'vue';
import { swapHorizontal, volumeHigh, copy } from 'ionicons/icons';
import { Translation, Language } from '@capacitor-mlkit/translation';
import { Clipboard } from '@capacitor/clipboard';
import { SpeechSynthesis } from '@capawesome-team/capacitor-speech-synthesis';
import { Preferences } from '@capacitor/preferences';

export default defineComponent({
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonItem,
    IonToast,
    IonButton,
    IonSelect,
    IonSelectOption,
    IonLoading,
    IonTextarea
  },
  setup() {
    const loading = ref(false);
    const loadingMessage = ref('Loading language models...');
    const sourceLanguage = ref(Language.German);
    const targetLanguage = ref(Language.English);
    const sourceText = ref('');
    const translation = ref('');

    const languages = [
      { label: 'Deutsch', value: Language.German },
      { label: 'Englisch', value: Language.English },
      { label: 'Französisch', value: Language.French },
      { label: 'Spanisch', value: Language.Spanish },
      { label: 'Italienisch', value: Language.Italian },
    ];

    const translateText = async () => {
        const { text } = await Translation.translate({
          text: sourceText.value,
          sourceLanguage: sourceLanguage.value,
          targetLanguage: targetLanguage.value
        });
        translation.value = text;
    };

    const swapLanguages = async () => {
      const tempLang = sourceLanguage.value;
      sourceLanguage.value = targetLanguage.value;
      targetLanguage.value = tempLang;

      const tempText = sourceText.value;
      sourceText.value = translation.value;
      translation.value = tempText;

      await saveLanguagePreferences();
    };

    const saveLanguagePreferences = async () => {
      await Preferences.set({ key: 'sourceLanguage', value: sourceLanguage.value });
      await Preferences.set({ key: 'targetLanguage', value: targetLanguage.value });
    };


    const readAloud = async () => {
      await SpeechSynthesis.speak({
        text: translation.value,
        language: targetLanguage.value,
      });
    };

    const copyToClipboard = async () => {
      await Clipboard.write({
        string: translation.value
      });
    };

    watch([sourceLanguage, targetLanguage], async () => {
      await saveLanguagePreferences();
    });

    watch([sourceText, targetLanguage], () => {
      translateText();
    });

    onMounted(async () => {
      loading.value = true;
      await SpeechSynthesis.initialize();

      const { value: savedSourceLang } = await Preferences.get({ key: 'sourceLanguage' });
      const { value: savedTargetLang } = await Preferences.get({ key: 'targetLanguage' });

      if (savedSourceLang) sourceLanguage.value = savedSourceLang as Language;
      if (savedTargetLang) targetLanguage.value = savedTargetLang as Language;

      const requiredLanguages = [
        Language.German,
        Language.English,
        Language.French,
        Language.Spanish,
        Language.Italian,
      ];

      for (const lang of requiredLanguages) {
        const { languages } = await Translation.getDownloadedModels();
        if (!languages.includes(lang)) {
          try {
            await Translation.downloadModel({ language: lang });
          } catch (e) {
            console.error(`Failed to download model for ${lang}`, e);
          }
        }
      }
      loading.value = false;
    });

    return {
      // Icons
      swapHorizontal,
      volumeHigh,
      copy,

      // States
      loading,
      loadingMessage,
      sourceLanguage,
      targetLanguage,
      sourceText,
      translation,

      // Methods
      swapLanguages,
      translateText,
      saveLanguagePreferences,
      readAloud,
      copyToClipboard,

      // Language list
      languages
    };
  }
});
</script>

<style scoped>
#container {
  display: flex;
  flex-direction: column;
  gap: 16px;
  padding: 16px;
}

.button-row {
  display: flex;
  justify-content: center;
  gap: 12px;
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

ion-title {
  margin-top: 12px;
}
</style>