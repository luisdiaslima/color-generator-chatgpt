

<template>
  <main class="main-container">
    <div class="palette-container">
      <input
        type="text"
        class="palette-container__search"
        v-model="search"
        placeholder="Type a color or vibe..."
        @keyup.enter="handleSearch"
      />

      <div class="palette-container__colors">
        <span
          v-for="(color, index) in colors"
          :key="index"
          :style="{ color: color }"
          @click="copyToClipboard(color)"
          class="color-label"
        >
          {{ color }}
        </span>
      </div>

      <div class="palette-container__generated-color">
        <span
          v-for="(color, index) in colors"
          :key="index"
          :style="{ backgroundColor: isLoading ? 'transparent' : color }"
          :class="{ 'shimmer-effect': isLoading }"
          class="color-box"
        />
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import Groq from "groq-sdk";

const groq = new Groq({
  apiKey: import.meta.env.VITE_API_KEY,
  dangerouslyAllowBrowser: true,
});

const colors = ref([]);
const search = ref("");
const isLoading = ref(false);

onMounted(() => {
  generateRandomPalette();
});

const generateRandomPalette = () => {
  colors.value = Array.from({ length: 4 }, () => {
    return `#${Math.floor(Math.random() * 16777215).toString(16).padStart(6, '0').toUpperCase()}`;
  });
};

const updateColors = (newColors) => {
  if (newColors.length > 4) {
    colors.value = newColors.slice(0, 4);
  } else {
    colors.value = newColors;
  }
};

const copyToClipboard = (color) => {
  navigator.clipboard.writeText(color).then(() => {
    alert(`Copied: ${color}`);
  });
};

const handleSearch = async () => {
  if (!search.value.trim()) {
    generateRandomPalette();
    return;
  }

  isLoading.value = true;
  try {
    const response = await groq.chat.completions.create({
      messages: [
        {
          role: "user",
          content: `Give me 4 hexadecimal colors based on: ${search.value}. Return only the hex codes as an array.`,
        },
      ],
      model: "llama-3.3-70b-versatile",
    });

    const hexColors = extractHexColors(response.choices[0].message.content);
    updateColors(hexColors);
  } catch (error) {
    console.error("Error fetching colors:", error);
    alert("Failed to generate palette. Please try again.");
  } finally {
    isLoading.value = false;
  }
};

// Extrai códigos hexadecimais de uma string
const extractHexColors = (str) => {
  const hexPattern = /#([0-9A-Fa-f]{3,6})\b/gi;
  return (str.match(hexPattern) || []).map((color) => color.toUpperCase());
};
</script>

<style lang="scss" scoped>
@import './styles.scss'
</style>