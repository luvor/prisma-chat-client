<template>
  <div class="page">
    <div class="messages">
      <div
        v-for="msg in messages"
        :key="msg.id"
        class="message"
        :class="{
          'message-me': msg.user === user,
        }"
      >
        <div class="message-user">
          {{ msg.user }}
        </div>
        <div class="message-content">
          {{ msg.content }}
        </div>
        <div v-if="msg.fileUrl">
          <a :href="msg.fileUrl" target="_blank" download>Скачать</a>
        </div>
      </div>
    </div>
    <div class="user-actions">
      <input class="user-name" v-model="user" placeholder="имя" />
      <input class="user-input" v-model="content" @keyup.enter="sendMessage" placeholder="сообщение" />
      <button @click="sendMessage">Отправить</button>
    </div>
    <div class="upload-area" @drop.prevent="handleDrop" @dragover.prevent @click="triggerFileInput">
      <input type="file" multiple @change="handleFileChange" ref="fileInput" class="file-input" />
      <p>Перетащите в область или кликните для загрузки</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      ws: null,
      user: '',
      content: '',
      fileUrl: null,
      messages: [],
    }
  },
  created() {
    this.ws = new WebSocket('ws://localhost:3000')
    this.ws.onmessage = (event) => {
      const message = JSON.parse(event.data)
      if (message.type === 'oldMessages') {
        this.messages = message.data
      } else {
        this.messages.push(message)
      }
    }
  },
  methods: {
    sendMessage() {
      if (this.user && (this.content || this.fileUrl)) {
        const message = { user: this.user, content: this.content, fileUrl: this.fileUrl }
        this.ws.send(JSON.stringify(message))
        this.content = ''
        this.fileUrl = null
      }
    },
    async uploadFiles(files) {
      for (let i = 0; i < files.length; i++) {
        const file = files[i]
        const formData = new FormData()
        formData.append('file', file)

        const response = await axios.post('http://localhost:3000/upload', formData, {
          headers: {
            'Content-Type': 'multipart/form-data',
          },
        })

        console.log('lol', response)
        this.content = file.name
        this.fileUrl = response.data.fileUrl
        this.sendMessage()
      }
    },
    triggerFileInput() {
      this.$refs.fileInput.click()
    },
    handleDrop(event) {
      if (!event.dataTransfer?.files) return
      this.uploadFiles(event.dataTransfer.files)
    },
    handleFileChange(event) {
      this.uploadFiles(event.target.files)
    },
  },
}
</script>

<style scoped>
.page {
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100%;
  padding: 2rem;
}
.messages {
  flex: 1;
  overflow-y: auto;
  padding-right: 20px;
  padding-bottom: 20px;
  scrollbar-color: black #333;
  scrollbar-width: thin;
}
.message {
  margin-bottom: 10px;
  padding: 10px 20px;
  background: #1f1f1f;
  width: fit-content;
  border-radius: 8px;
}
.message-me {
  background: #333;
  margin-left: auto;
}
.message-user {
  font-weight: bold;
}
.user-actions {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  margin-top: 10px;
  gap: 10px;
}
.user-name {
  border-radius: 8px;
  border: none;
  padding: 5px 10px;
}
.user-input {
  flex: 1;
  border-radius: 8px;
  border: none;
  padding: 5px 10px;
}
.upload-area {
  border: 2px dashed #ccc;
  padding: 20px;
  text-align: center;
  cursor: pointer;
  border-radius: 8px;
  margin-top: 10px;
}
.file-input {
  display: none;
}
</style>
