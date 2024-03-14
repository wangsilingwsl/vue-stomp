<template>
  <div id="app">
    <!-- 发送消息表单 -->
    <van-form @submit="onSubmit">
      <van-field
        v-model="content"
        name="内容"
        label="内容"
        rows="3"
        autosize
        type="textarea"
        placeholder="请输入内容"
      />
      <div style="margin: 16px">
        <van-button round block type="info" native-type="submit"
          >提交</van-button
        >
      </div>
    </van-form>

    <!-- 消息回复体 -->
    <van-cell-group>
      <van-cell
        v-for="(msgItem, msgIndex) in msgList"
        :key="msgIndex"
        :title="'回复消息' + (msgIndex + 1)"
        value=""
        :label="msgItem"
      />
    </van-cell-group>
  </div>
</template>

<script>
import Stomp from "stompjs";
let socketTimer = null;

export default {
  name: "App",
  created() {
    this.username = "admin";
    this.initWebsocket();
  },
  data() {
    return {
      content: "",
      stompClient: null,
      msgList: [],
      username: "admin",
    };
  },
  methods: {
    initWebsocket() {
      this.stompClient = Stomp.client(
        "ws://192.168.1.109:7010/websocket/bpm/runFlow"
      );
      this.stompClient.debug = null;
      const headers = {
        username: this.username,
      };
      this.stompClient.connect(
        headers, // 自定义请求头
        () => {
          this.stompClient.subscribe(
            "/user/websocket/topic/websocket/bpm/runFlow",
            (res) => {
              this.msgList.push(JSON.stringify(res));
            }
          );
        },
        (err) => {
          console.log("err", err);
          this.$toast("连接失败：" + JSON.stringify(err));
          // 监听报错信息，手动发起重连
          if (socketTimer) {
            clearInterval(socketTimer);
          }
          // 10s后重新连接一次
          socketTimer = setTimeout(() => {
            this.initWebsocket();
          }, 10000);
        }
      );
      // this.stompClient.heartbeat.outgoing = 10000;
      // 若使用STOMP 1.1 版本，默认开启了心跳检测机制（默认值都是10000ms）
      // this.stompClient.heartbeat.incoming = 0;
      // 客户端不从服务端接收心跳包
    },
    closeWebsocket() {
      if (this.stompClient !== null) {
        this.stompClient.disconnect(() => {
          console.log("关闭连接");
        });
      }
    },
    onSubmit() {
      // 发送信息
      // 转成json对象
      this.stompClient.send(
        "/websocket/app/websocket/bpm/runFlow",
        {},
        // JSON.stringify({ content: this.content })
        this.content
      );
    },
  },
  destroyed() {
    // 页面销毁后记得关闭定时器
    clearInterval(socketTimer);
    this.closeWebsocket();
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
