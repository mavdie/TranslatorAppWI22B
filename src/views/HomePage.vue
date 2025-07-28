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
          <ion-button @click="translateText" :disabled="disableButton">Translate</ion-button>
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
import { IonContent,
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
  IonTextarea} from '@ionic/vue';
import { defineComponent } from "vue";
import { swapHorizontal, volumeHigh, copy } from 'ionicons/icons';
import { Translation, Language } from '@capacitor-mlkit/translation';
import { Clipboard } from '@capacitor/clipboard';
import { SpeechSynthesis } from '@capawesome-team/capacitor-speech-synthesis';
import { Preferences } from '@capacitor/preferences';
import { onMounted, ref } from "vue";

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

    // Use refs for sourceLanguage and targetLanguage
    const sourceLanguage = ref(Language.German);
    const targetLanguage = ref(Language.English);

    onMounted(async () => {
      loading.value = true;
      await SpeechSynthesis.initialize();
      
      // Load saved languages from preferences
      const { value: savedSourceLang } = await Preferences.get({ key: 'sourceLanguage' });
      const { value: savedTargetLang } = await Preferences.get({ key: 'targetLanguage' });
      
      // Apply saved languages if they exist
      if (savedSourceLang) {
        sourceLanguage.value = savedSourceLang as Language;
      }
      if (savedTargetLang) {
        targetLanguage.value = savedTargetLang as Language;
      }

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

    return { swapHorizontal, volumeHigh, copy, loading, loadingMessage, sourceLanguage, targetLanguage };
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
      sourceText: "",
      translation: ""
    };
  },
  computed: {
  },
  methods: {
    async translateText() {
      this.disableButton = true
      const { text } = await Translation.translate({
        text: this.sourceText,
        sourceLanguage: this.sourceLanguage,
        targetLanguage: this.targetLanguage
      });
      this.translation = text
      this.disableButton = false
    },
    async swapLanguages() {
      // Swap the selected languages
      const temp = this.sourceLanguage;
      this.sourceLanguage = this.targetLanguage;
      this.targetLanguage = temp;
      
      // Save to preferences
      await this.saveLanguagePreferences();
      
      // Swap the text in the textareas as well
      const tempText = this.sourceText;
      this.sourceText = this.translation;
      this.translation = tempText;
    },
    async saveLanguagePreferences() {
      await Preferences.set({
        key: 'sourceLanguage',
        value: this.sourceLanguage
      });
      await Preferences.set({
        key: 'targetLanguage',
        value: this.targetLanguage
      });
    },
    async readAloud() {
      await SpeechSynthesis.speak({
        text: this.translation,
        language: this.targetLanguage,
      })
    },
    async copyToClipboard() {
      await Clipboard.write({
        string: this.translation
      });
    }
  },
  watch: {
    // Save preferences when languages change
    sourceLanguage() {
      this.saveLanguagePreferences();
    },
    targetLanguage() {
      this.saveLanguagePreferences();
    }
  }
})
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
