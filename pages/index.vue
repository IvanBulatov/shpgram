<script>
import { io } from "socket.io-client";

export default {
  data() {
    return {
      messages: [],
      username: '',
      connect: false,
      socket: null,
      text: '',
      receiver: ''
    };
  },
  methods: {
    async loadMessages() {
      let { data } = await useFetch(`/api/messages/${this.username}`)
      this.messages = data
      console.log(this.messages)
    },
    handleConnect() {
      if (this.username.length > 0) {
        this.loadMessages()
        this.connect = true
        this.socket = io('ws://localhost:3000');
        this.socket.emit('logged_in', { username: this.username })
        this.socket.on('chat message', (data) => {
          this.messages.push({ text: data.text, sender: data.username, receiver: data.receiver })
        })
      }
    },

    sendMessage() {
      this.socket.emit('chat message', { sender: this.username, text: this.text, receiver: this.receiver })
    }
  },
};
</script>

<template>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet"
    integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
  <h2>Всем разрабам привет. Я стажёр, но уже положил всю базу данных )) Извините ...</h2>
  <div class="container py-3">
    <h1>Чат</h1>
  </div>
  <div class="container">
    <div class="row ">
      <!-- Форма входа -->
      <div class="col-4 form-container" v-if="!connect">
        <form @submit.prevent="handleConnect">
          <div class="card">
            <div class="card-body">
              <!-- Название  -->
              <div class="mb-3">
                <input type="text" class="form-control" placeholder="Имя пользователя" v-model="username">
              </div>

              <button class="btn btn-primary">Войти в чат</button>
            </div>
          </div>
        </form>
      </div>
      <!-- Форма добавления заметки -->
      <div class="col-4 form-container" v-if="connect">
        <form @submit.prevent="sendMessage">
          <div class="card">
            <div class="card-body">
              <!-- Название  -->
              <div class="mb-3">
                <textarea type="text" class="form-control" rows="10" v-model="text"></textarea>
                <input type="text" class="form-control" placeholder="Имя пользователя" v-model="receiver">
              </div>

              <button class="btn btn-primary">Отправить сообщение</button>
            </div>
          </div>
        </form>
      </div>

      <!-- Список заметок  -->
      <div class="col-8 notes-container">

        <div class="card mb-3" v-if="connect" v-for="message in messages">
          <div class="card-body">
            <h3 class="card-title">
              <span>{{ message.sender }}: {{ message.text }}</span>
            </h3>

          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<style>
.card-link {
  cursor: pointer;
}
</style>
