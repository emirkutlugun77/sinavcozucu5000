<template>
  <div class="chat-container">
    <div class="chat-box left-chat-box">
      <div
        class="chat-message"
        v-for="(message, index) in messages"
        :key="index"
      >
        <p class="chat-text">{{ message.text }}</p>
      </div>
      <form class="chat-form" @submit.prevent="sendMessage">
        <textarea
          type="text"
          v-model="newMessage"
          placeholder="Soru"
          style="width: 80%"
        ></textarea>
        <div class="lesson-name">
          <input type="text" v-model="lessonName" placeholder="Dersin Adı" />
        </div>
        <button type="submit">Send</button>
      </form>
    </div>
    <div class="chat-box right-chat-box">
      <div v-if="!isLoading">
        <div class="chat-message">
          <p class="chat-text" v-html="formattedResponse"></p>
        </div>
      </div>
      <div v-if="isLoading" class="loader-container">
        <div class="loader"></div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      isLoading: false,
      messages: [],
      newMessage: "",
      lessonName: "",
      response: "",
    };
  },
  computed: {
    formattedResponse() {
      if (this.response) {
        return this.response
          .replace(/(Step\s+\d+:)/g, '<span class="step">$1</span>')
          .replace(/\n/g, "<br>");
      } else {
        return "";
      }
    },
  },
  methods: {
    async sendMessage() {
      if (this.newMessage) {
        const payload = {
          question: this.newMessage,
          lessonName: this.lessonName,
        };
        this.messages.push({ text: this.newMessage });
        this.newMessage = "";
        this.isLoading = true;
        try {
          const response = await axios.post(
            "https://sinavcozucu.herokuapp.com/gpt-service/askQuestion",
            payload
          ); //answer.choices[0].message.content
          console.log(JSON.stringify(response));

          this.response = response.data;
        } catch (error) {
          console.error(error);
        } finally {
          this.isLoading = false;
        }
      }
    },
  },
};
</script>

<style scoped>
body {
  height: 100%;
}
.chat-container {
  display: flex;
  flex-wrap: wrap;
  height: 90vh;
}

.chat-box {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 10px;
  margin: 10px;
  display: flex;
  flex-direction: column;
  flex-grow: 1;
  overflow-y: auto;
}

.chat-message {
  margin: 10px;
  max-width: 70%;
  padding: 10px;
  background-color: #eee;
  border-radius: 10px;
  align-self: flex-start;
  word-wrap: break-word;
}

.chat-message.right-chat-message {
  align-self: flex-end;
}

.chat-text {
  text-align: left;
  font-size: 14px;
  color: #333;
}

.chat-form {
  margin-top: auto;
  display: flex;
}

.chat-form input {
  flex: 1;
  padding: 5px 10px;
  border-radius: 5px;
  border: none;
  margin-right: 10px;
  font-size: 14px;
  min-height: 30px;
}

.chat-form button {
  background-color: #4caf50;
  color: #fff;
  border: none;
  border-radius: 5px;
  font-size: 18px;
  padding: 5px 10px;
  cursor: pointer;
}

.chat-box.left-chat-box {
  flex-basis: 30%;
}

.chat-box.right-chat-box {
  flex-basis: 65%;
  position: relative;
}
.chat-box.right-chat-box .loader {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.loader {
  border: 10px solid #f3f3f3;
  border-top: 10px solid #4caf50;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.chat-text .step {
  color: red;
  font-weight: bold;
}
</style>
