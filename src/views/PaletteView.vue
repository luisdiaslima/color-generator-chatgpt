<template>
  <main>
    <div class="palette-container">
      <form @submit.prevent="handleChatGPT" class="palette-container__form">
        <input
          type="text"
          class="palette-container__form__search"
          placeholder="Type some vibe"
          @input="handleSearch"
        />
        <button type="submit" class="palette-container__form__button">?</button>
      </form>

      <div class="palette-container__colors">
        <span v-for="(color, index) in colors" :key="index"> {{ color }} </span>
      </div>
      <div
        :class="{ 'fade-in': fadeIn }"
        class="palette-container__generated-color"
      >
        <span
          v-for="(color, index) in colors"
          :key="color"
          :id="'color' + index"
        />
      </div>
      <loader-infinite v-if="isLoading" />
    </div>
  </main>
</template>

<script>
import axios from "axios";
import LoaderInfinite from "../components/InfiniteLoader/Component.vue";
export default {
  components: { LoaderInfinite },
  data() {
    return {
      colors: [],
      fadeIn: false,
      search: "",
      isLoading: false,
    };
  },
  mounted() {
    this.generatePalette();
  },
  methods: {
    generatePalette() {
      for (let i = 0; i < 4; i++) {
        const randomColor = Math.floor(Math.random() * 16777215).toString(16);
        this.colors[i] = `#${randomColor}`.toUpperCase();

        this.fadeIn = true;
        setTimeout(() => (this.fadeIn = false), 400);
      }
    },
    handleSearch(e) {
      this.search = e.target.value;
    },
    async handleChatGPT() {
      this.isLoading = true;
      const { data } = await axios.post(
        "https://api.openai.com/v1/completions",
        {
          model: "text-davinci-003",
          prompt: `${import.meta.env.VITE_CHAT_GPT_QUESTION} ${this.search}`,
          temperature: 0,
          max_tokens: 1000,
        },
        {
          headers: {
            Authorization: `Bearer ${
              import.meta.env.VITE_CHAT_GPT_PRIVATE_KEY
            }`,
          },
        }
      );
      const newColors = eval(data.choices[0].text);
      this.colors = newColors;
      this.isLoading = false;
    },
  },
};
</script>

<style lang="scss" scoped>
@import "./styles.scss";

#color0 {
  background: v-bind("colors[0]");
}

#color1 {
  background: v-bind("colors[1]");
}

#color2 {
  background: v-bind("colors[2]");
}

#color3 {
  background: v-bind("colors[3]");
}
</style>
