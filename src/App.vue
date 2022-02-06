<template>
  <div id="app">
    유저이름:
    <input
        v-model="userName"
        type="text"
    >
    내용: <input
      v-model="message"
      type="text"
      @keyup="sendMessage"
  >
    파일 업로드: <input
      type="file"
      @change="selectFile"
      ref="file"
      accept="image/*"
  >
    <button @click="sendPicture">파일 보내기</button>
    <button @click="state">state 보내기</button>
    <button @click="modify">수정하기</button>
    <button @click="deleteChat">삭제하기</button>
    <div
        v-for="(item, idx) in receiveList"
        :key="idx"
    >
      <h3>유저이름: {{ item.name }}</h3>
      <h3>내용: {{ item.message }}</h3>
      <h3>내용: {{ item.time }}</h3>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Stomp from 'webstomp-client'
import SockJS from 'sockjs-client'

export default {
  name: 'App',
  data() {
    return {
      userName: "",
      message: "",
      receiveList: [],
      img: ""
    }
  },
  created() {
    // App.vue가 생성되면 소켓 연결을 시도합니다.
    this.connect()
  },
  methods: {
    modify () {
      const msg = {
        id: "61f68247af040f66ae85707e",
        content: "카프카 연동",
        type: "modify"
      }
      this.stompClient.send("/kafka/send-direct-modify",JSON.stringify(msg),{});
    },
    deleteChat () {
      const msg = {
        id: "61f68247af040f66ae85707e",
        userId: "1",
        type: "delete"
      }
      this.stompClient.send("/kafka/send-direct-delete",JSON.stringify(msg),{});
    },
    selectFile () {
      this.img = this.$refs.file.files[0]
      console.log(this.img)
    },
    sendMessage (e) {
      if(e.keyCode === 13 && this.userName !== '' && this.message !== ''){
        this.send()
        this.message = ''
      }
    },
    sendPicture () {
      if (this.stompClient && this.stompClient.connected) {
        const formData = new FormData()
        formData.append("image",this.img)
        formData.append("thumbnail",this.img)
        formData.append("type","direct")
        formData.append("userId",1)
        formData.append("channelId",1)
        axios.post("http://localhost:8000/chat-server/file",formData)
      }
    },
    state() {
      const msg = {
        type:"state",
        channel_id: 1,
        user_id: 1
      }
      this.stompClient.send("/kafka/join-channel",JSON.stringify(msg),{});
    },
    send() {
      if (this.stompClient && this.stompClient.connected) {
        const msg = {
          content: this.message,
          channelId: 1,
          communityId: 1,
          accountId: 1,
          // userId: 1,
          name:"뵹찬",
          profileImage: "123123"
        };
        console.log(JSON.stringify(msg))
        this.stompClient.send("/kafka/send-channel-message", JSON.stringify(msg), {});
      }
    },
    connect() {
      const serverURL = "http://3.36.238.237:8080/my-chat"
      // const serverURL = "http://localhost:8080/my-chat"
      let socket = new SockJS(serverURL);
      this.stompClient = Stomp.over(socket);
      console.log(`소켓 연결을 시도합니다. 서버 주소: ${serverURL}`)
      this.stompClient.connect(
          {"access-token" : "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJwYWs3MzgwQG5hdmVyLmNvbSIsInJvbGUiOiJST0xFX0FETUlOIiwiaWQiOjIsImlhdCI6MTY0MTk5MDMyNSwiZXhwIjoxNjUwNjMwMzI1fQ.Gh53esjDbsLF838F_WjqpRwABFI4JaLYOtbZ7Phqu_g",
          "user-id" : "1"},
          frame => {
            // 소켓 연결 성공
            this.connected = true;
            console.log('소켓 연결 성공', frame);
            this.stompClient.subscribe("/topic/group/1", res => {
              console.log('구독으로 받은 메시지 입니다.', res.body);

              this.receiveList.push(JSON.parse(res.body))
              console.log(this.receiveList)
            });
          },
          error => {
            // 소켓 연결 실패
            console.log('소켓 연결 실패', error);
            this.connected = false;
          }
      );
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
