<template>
  <div class="book">
    <n-space vertical>
      <h1>This is an books page</h1>
      <n-tag type="success">{{ book }}</n-tag>
      <n-button type="success" @click="getBooks">Получить книгу</n-button>
    </n-space>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, onMounted, onBeforeUnmount } from "vue";
import { useMessage, NInput, NSpace, NTag } from "naive-ui";

const book = ref("");

const getBooks = async () => {
  const { data } = await axios.get(`http://localhost:3000/book`);

  book.value = data;
};
</script>

<style>
@media (min-width: 1024px) {
  .book {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
