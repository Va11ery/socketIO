<template>
  <div>
    <h1>Vue.js Socket.IO Chat</h1>

    <n-space vertical v-if="!authenticated">
      <p>Введите пароль для соединения:</p>
      <n-input v-model:value="password" type="password" />
      <n-button type="success" @click="authenticate">Подключиться</n-button>
      <p>{{ authErrorMessage }}</p>
    </n-space>

    <div v-else>
      <n-space style="margin-bottom: 25px">
        <n-tag type="success" v-for="message in messages" :key="message.id">{{
          message.text
        }}</n-tag>
      </n-space>

      <p>Введите собщение:</p>
      <n-input
        style="margin-bottom: 25px"
        v-model:value="newMessage"
        @keyup.enter="sendMessage"
        placeholder=""
      />

      <p>Введите название папки:</p>
      <n-input
        style="margin-bottom: 25px"
        v-model:value="nameFolder"
        placeholder="Type a message..."
      />

      <n-button type="success" @click="createFolder">Создать папки</n-button>

      <n-space style="margin-bottom: 25px">
        <p>Загрузка файла:</p>
        <input type="file" ref="fileInput" @change="handleFileUpload" />
        <n-button @click="uploadFile">Upload File</n-button>
      </n-space>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import { useMessage, NInput, NSpace, NTag } from "naive-ui";
import io from "socket.io-client";
import axios from "axios";

const message = useMessage();
const socket = ref(null);
const messages = ref([]);
const newMessage = ref("");
const password = ref("");
const authErrorMessage = ref("");
const nameFolder = ref("");
const authenticated = ref(false);
const selectedFile = ref(null);
const fileInput = ref(null);

onMounted(() => {
  // Подключение к серверу сокетов
  socket.value = io("http://localhost:3000");

  // Обработка события при установке соединения
  socket.value.on("connect", () => {
    message.success("Соединение установлено");
  });

  // Обработка события при разрыве соединения
  socket.value.on("disconnect", () => {
    message.error("Соединение разорвано");
  });

  // Обработка события при неудачной попытке соединения
  socket.value.on("connect_error", (error) => {
    message.error("Ошибка соединения");
  });

  // Обработка события нового сообщения от сервера
  socket.value.on("sendUserData", (msg) => {
    messages.value.push({ id: new Date().getTime(), text: msg });
  });
});

const authenticate = () => {
  if (password.value) {
    socket.value.emit("authenticate", password.value, (auth, errorMessage) => {
      console.log(auth);
      if (auth) {
        message.success("Соединение установлено");
        authenticated.value = true;
      } else {
        message.error(errorMessage);
      }
    });
  }
};

const createFolder = async () => {
  const { data } = await axios.post(`http://localhost:3000/createFolder`, {
    nameFolder: nameFolder.value,
  });

  message.success(data);
};

const handleFileUpload = () => {
  // Обработка выбора файла
  selectedFile.value = fileInput.value.files[0];
};

const uploadFile = async () => {
  try {
    // Создание объекта FormData для отправки файла
    const formData = new FormData();
    formData.append("file", selectedFile.value);

    // Выполнение HTTP-запроса на ваш сервер (Express)
    await axios.post("http://localhost:3000/uploadFile", formData, {
      headers: {
        "Content-Type": "multipart/form-data",
      },
      params: {
        nameFolder: nameFolder.value,
      },
    });

    message.success("File uploaded successfully!");
  } catch (error) {
    message.error("Error uploading file:", error);
  }
};

const sendMessage = () => {
  if (newMessage.value) {
    // Отправка сообщения на сервер
    socket.value.emit("sendUserData", newMessage.value);
    newMessage.value = "";
  }
};

onBeforeUnmount(() => {
  // Отключение сокета при уничтожении компонента
  if (socket.value) {
    socket.value.disconnect();
  }
});
</script>

<style scoped>
/* Ваши стили компонента здесь */
</style>
