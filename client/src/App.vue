<script setup>
import {onBeforeMount, ref} from "vue";
import {io} from 'socket.io-client'

const socket = io('http://localhost:3001')

const messages = ref([])
const messageText = ref('')
const joined = ref(false)
const name = ref('')
const typingDisplay = ref('')

onBeforeMount(() => {
  socket.emit('findAllMessages', {}, (response) => {
    messages.value = response
  })

  socket.on('message', (message) => {
    messages.value.push(message)
  })

  socket.on('typing', ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} is typing...`
    } else {
      typingDisplay.value = ''
    }
  })
})

const join = () => {
  socket.emit('join', {name: name.value}, () => {
    joined.value = true
  })
}

const sendMessage = () => {
  socket.emit(
      'createMessage',
      {
        text: messageText.value
      }, () => {
        messageText.value = ''
      })
}

let timeout
const emitTyping = () => {
  socket.emit('typing', {isTyping: true})
  timeout = setTimeout(() => {
    socket.emit('typing', {isTyping: false})
  }, 2000)
}
</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form class="chat-ask" @submit.prevent="join">
        <label class="chat-ask__label">What's your name?</label>
        <input class="chat-ask__input" v-model="name" placeholder="Enter your name"/>
        <button class="chat-ask__button" type="submit">Send</button>
      </form>
    </div>
    <div v-if="joined">
      <div class="chat-container">
        <div v-for="message in messages">
          [{{ message.name }}]: {{ message.text }}
        </div>
      </div>
      <div v-if="typingDisplay">
        {{ typingDisplay }}
      </div>
      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messageText" @input="emitTyping"/>

          <button>Send message</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style>
@import 'assets/base.css';

.chat {
  background: #212020;
  min-height: 100vh;
  color: #f0f0f0;
  padding: 20px;
}

.chat-ask {
  display: flex;
  flex-flow: column;
  margin-bottom: 20px;
  width: 400px;
}

.chat-ask__label {
  display: block;
  font-size: 1.2em;
  font-weight: bold;
  margin-bottom: 8px;
}

.chat-ask__input {
  background: none;
  border: 1px solid #f0f0f0;
  padding: 8px;
  border-radius: 5px;
  color: #f0f0f0;
  margin-bottom: 8px;
}

.chat-ask__button {
  appearance: none;
  outline: none;
  background: none;
  border: none;
  color: #f0f0f0;
  background: #4d951c;
  padding: 8px;
  border-radius: 5px;
}
</style>
