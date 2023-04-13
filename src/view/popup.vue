<template>
  <div class="main_app">
    <input v-model="OpenAIAPIKey" placeholder="Your OpenAI APIKey: sk-..." style="width:100px">
    <select v-model="ReplyLanguage">
      <option value="English">English</option>
      <option value="中文">中文</option>
      <option value="Spanish">Spanish</option>
      <option value="Japanese">Japanese</option>
    </select>
    <select v-model="isReadAloud">
      <option value="ON">🔊</option>
      <option value="OFF">🔇</option>
    </select>
    <button @click="saveAndReloadResponse()">✅</button>
    <input v-model="AdditionalInstruction" placeholder="Additional Instructions" style="width:250px;"><br>
    <h1>{{ phrase }}</h1>
    <h2>{{ explanation }}</h2>
  </div>
</template>

<script>
const franc = require('franc');
export default {
  name: 'popupView',
  data() {
    return {
      phrase: 'Nothing is selected',
      explanation: '',
      OpenAIAPIKey: '',
      ReplyLanguage: 'English',
      isReadAloud: 'ON',
      AdditionalInstruction: '',
    }
  },
  methods: {
    async saveAndReloadResponse() {
      await chrome.storage.sync.set({ 'OpenAIAPIKey': this.OpenAIAPIKey });
      await chrome.storage.sync.set({ 'ReplyLanguage': this.ReplyLanguage });
      await chrome.storage.sync.set({ 'isReadAloud': this.isReadAloud });
      await chrome.storage.sync.set({ 'AdditionalInstruction': this.AdditionalInstruction });
      await this.reloadResponse();
    },
    async getSelectedText() {
      const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
      let result;
      try {
        [{ result }] = await chrome.scripting.executeScript({
          target: { tabId: tab.id },
          function: () => {
            const selection = getSelection().toString().trim();
            let context = getSelection().baseNode.parentNode.innerText;
            if (context.length > 200) {
              const index = context.indexOf(selection);
              const start = Math.max(0, index - 100);
              const end = Math.min(context.length, index + selection.length + 100);
              context = "..." + context.substring(start, end) + "...";
              console.log(`context is too long, shorten to ${context}`);
            }
            return [selection, context];
          },
        });
      } catch (e) {
        console.log(e);
        console.log("ignoring an unsupported page like chrome://extensions");
        return null; // ignoring an unsupported page like chrome://extensions
      }
      if (this.isReadAloud=="ON") {
        const [phrase, context] = result;
        const languageCode = franc.franc(phrase.repeat(20)); //hack: franc doesn't work if the text is too short.
        chrome.tts.speak(phrase, {
          voiceName: languageCode=='cmn'?'Google 普通话（中国大陆）':'Google UK English Female',
        });
      }
      return result;
    },
    async getOpenAIResponse(phrase, context) {
      const { Configuration, OpenAIApi } = require("openai");
      const apiKey = this.OpenAIAPIKey;
      const languageCode = franc.franc(phrase.repeat(20)); //hack: franc doesn't work if the text is too short.
      console.log(languageCode);
      let ReplyInLanguage = `Please reply in ${this.ReplyLanguage}.`;
      
      //hack: to make sure AI uses Chinese when I choose Chinese
      if (this.ReplyLanguage=='中文') {
        ReplyInLanguage += `请用中文回答，开始：`;
      }
      
      const promptExplainWord = `--Context--\n${context}\n--End--\nBased on the context above, please explain the phrase "${phrase}" using no more than 30 words.\n${languageCode == 'cmn' ? ' Please add PinYin to the phrase.' : ''} ${ReplyInLanguage}\n${this.AdditionalInstruction}\n`;
      console.log(promptExplainWord);

      const openai = new OpenAIApi(new Configuration({
        apiKey: apiKey,
      }));
      const response = await openai.createChatCompletion({
        model: "gpt-3.5-turbo",
        messages: [
          { role: "system", content: "You are a software application named Simple Explainer, designed to assist users in acquiring new phrases." },
          { role: "user", content: promptExplainWord }
        ],
      });
      return response;
    },
    async reloadResponse() {
      let result = await chrome.storage.sync.get("OpenAIAPIKey");
      this.OpenAIAPIKey = result.OpenAIAPIKey || "";

      result = await chrome.storage.sync.get("ReplyLanguage");
      console.log(result);
      this.ReplyLanguage = result.ReplyLanguage || "English";

      result = await chrome.storage.sync.get("isReadAloud");
      console.log(result);
      this.isReadAloud = result.isReadAloud || "ON";

      result = await chrome.storage.sync.get("AdditionalInstruction");
      console.log(result);
      this.AdditionalInstruction = result.AdditionalInstruction || "";

      console.log(this.OpenAIAPIKey);
      const [phrase, context] = await this.getSelectedText();
      console.log(phrase);
      if (phrase) {
        this.phrase = phrase;
        this.explanation = "Loading...";
        const response = await this.getOpenAIResponse(phrase, context);
        this.explanation = response.data.choices[0].message.content;
      }

    }
  },
  async mounted() {
    await this.reloadResponse();
  },

}

</script>

<style>
.main_app {
  width: 320px;
  padding: 0 10px;
}

h2 {
  padding: 10px;
  font-weight: normal;
  background-color: aliceblue;
}
</style>
