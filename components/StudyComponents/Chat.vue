<template>
  <div id="classroom-chat">
    <div id="chat-messages">
      <h4>Chat</h4>
      <div class="chat-message" v-for="message in messages" :key="message.messageId">
        <div class="message-sender">
          <img :src="apiStaticUrl + '/uploads/images/' + message.sender.avatar" alt="User Avatar" />
        </div>
        <div class="chat-content">
          <h5>{{message.sender.name}}</h5>
          <p :class="{'message-by-me':message.sentByMe}" class="message-content">{{message.message}}</p>
        </div>
      </div>
    </div>

    <form @submit.prevent="sendMessage" class="chat-form">
      <input v-model="message" @focus="someoneIsTyping" type="text" :placeholder="placeholder" />
      <input type="submit" value="Send" />
    </form>
    <!-- <img :src="`/Images/${courseclass.teacher.avatar}`" alt="User Avatar" /> -->
  </div>
</template>

<script>
import { mapState } from 'vuex'

export default {
  data() {
    return {
      message: '',
      sendingMessage: false,
      messages: [],
      placeholder: 'type something',
      apiStaticUrl: ''
    }
  },
  computed: mapState({
    user: state => state.user
  }),
  methods: {
    sendMessage() {
      this.sendingMessage = true
      console.log(this.apiStaticUrl + '/uploads/images/' + this.user.avatar)
      const message = {
        message: this.message,
        timestamp: Date.now(),
        sentByMe: true,
        sender: {
          name: this.user.name,
          avatar: this.user.avatar
        }
      }

      if (this.sendingMessage) {
        this.messages.push(message)
        // this.message = ''
        // this.sendingMessage = false
      }

      this.$socket.emit('message_sent', {
        classroomId: this.$route.params.classroomId,
        message: {
          message: this.message,
          timestamp: Date.now(),
          sentBy: {
            name: this.user.name,
            avatar: this.user.avatar,
            userId: this.user._id
          }
        }
      })
      this.message = ''
      this.sendingMessage = false
    },
    someoneIsTyping() {
      this.$socket.emit('someoneIsTyping', {
        name: this.user.name,
        userId: this.user._id,
        classroomId: this.$route.params.classroomId
      })
    }
  },
  sockets: {
    message_received(message) {
      if (message.sentBy.userId !== this.user._id) {
        this.messages.push({
          message: message.message,
          timestamp: message.timestamp,
          sender: {
            name: message.sentBy.name,
            avatar: message.sentBy.avatar
          },
          messageId: message.timestamp,
          sentByMe: false
        })
      }
    },
    someone_is_typing(message) {
      if (message.userId !== this.user._id) {
        this.placeholder = `${message.name} is typing`
        setTimeout(() => {
          this.placeholder = 'type something'
        }, 5000)
      }
    }
  },
  mounted() {
    this.apiStaticUrl = process.env.baseUrl
  }
}
</script>
