<script setup>
import io from "socket.io-client";
import { onBeforeMount, ref } from "vue";
const socket = io("https://nest-socket-test.herokuapp.com/");
const messages = ref([]);
const messageText = ref("");
const joined = ref(false);
const name = ref("");
const typingDisplay = ref("");

onBeforeMount(() => {
  socket.emit("findAllMessages", {}, (response) => {
    messages.value = response;
  });
  socket.on("message", (message) => {
    messages.value.push(message);
  });

  socket.on("typing", ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} 正在輸入...`;
    } else {
      typingDisplay.value = "";
    }
  });
});

const join = () => {
  socket.emit("join", { name: name.value }, () => {
    joined.value = true;
  });
};

const sendMessage = () => {
  socket.emit("createMessage", { text: messageText.value }, () => {
    messageText.value = "";
  });
};


const emitTyping = () => {
  socket.emit("typing", { isTyping: true });
  setTimeout(() => {
    socket.emit("typing", { isTyping: false });
  }, 2000);
};
</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label> 你的名字? </label>
        <input v-model="name" />
        <button type="submit">送出</button>
      </form>
    </div>
    <div class="chat-container">

      <div class="messages-container">
        <div v-for="(message,index) in messages" :key="message.text+index">
          [{{message.name}}]: {{message.text}}
        </div>
      </div>

      <div v-if="typingDisplay">{{typingDisplay}}</div>

      <hr />

      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label>訊息:</label>
          <input
            v-model="messageText"
            @input="emitTyping"
          />
          <button type="submit">送出</button>
        </form>
      </div>

    </div>
  </div>
</template>

<style scoped>
@import "./assets/base.css";

.chat {
  padding: 20px;
  height: 100vh;
}

.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.messages-container {
  flex: 1;
}
</style>
