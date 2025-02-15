<template>
  <div class="app">
    <UNotifications />
    <div class="row ">
      <!-- Форма входа -->
       <button @click="show">Show</button>
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
    </div>

    <Sidebar v-if="connect" />
    <div class="content" v-if="connect">
      <ChatList :chats="chats" @chat-selected="setSelectedChat" />
      <ChatWindow v-if="selectedChat" :chat="selectedChat" @send-message="sendMessage"/>
    </div>

  </div>
</template>

<script>

import { io } from "socket.io-client";
export default {
  name: "App",
  data() {
    return {
      chats: [
        { id: 1, name: "Ivan", messages: [] },
        { id: 2, name: "Anton", messages: [] },
      ],
      selectedChat: null, // { id: 1, name: "Ivan", messages: [] },
      username: '',
      connect: false
    };
  },
  methods: {
    setSelectedChat(chat) {
      this.selectedChat = chat;
      this.loadMessages();
    },
    async loadMessages() {
      let { data } = await useFetch(`/api/messages/${this.username}?sender=${this.selectedChat.name}`)
      this.selectedChat.messages = data
    },
    handleConnect() {
      if (this.username.length > 0) {
        const toast = useToast()
        // this.loadMessages()
        this.connect = true
        this.socket = io('ws://localhost:3000');
        this.socket.emit('logged_in', { username: this.username })
        this.socket.on('chat message', (data) => {
          this.chats.find(chat => chat.name === data.sender).messages.push({ text: data.text, sender: data.sender, receiver: data.receiver })
          toast.add({title: data.text});
        })
      }
    },
    show(){
      const toast = useToast()
      toast.add({title: 'test'});
    },

    sendMessage(data) {
      this.socket.emit('chat message', { sender: this.username, text: data.text, receiver: this.selectedChat.name })
    }
  },

};
</script>

<style>
.app {
  display: flex;
}

.content {
  flex: 1;
  display: flex;
}
</style>