<template>
  <div class="dashboard-container">
    <h4 style="text-align: center">websocket 测试</h4>
    <el-container>
      <el-aside width="400px">
        <div class="aside-main">
          <div class="connect-box" style="margin-top: 15px;">
            <el-input placeholder="请输入内容" v-model="input" style="width: auto;">
              <el-select class="type-select" v-model="select" slot="prepend" placeholder="请选择">
                <el-option label="wss://" value="wss://"></el-option>
                <el-option label="ws://" value="ws://"></el-option>
              </el-select>
            </el-input>
            <div class="button-group">
              <el-button size="mini" plain id="connect" type="primary" :disabled="connectState" @click="init">连接</el-button>
              <el-button size="mini" plain type="danger" @click="close">断开</el-button>
            </div>
          </div>

          <div class="send-box">
            <el-input
              v-model="textArea"
              type="textarea"
              :rows="4"
              placeholder="请输入内容"
            >
            </el-input>
            <el-button style="margin-top: 20px; width: 100%;" size="medium" type="primary" @click="send">发送</el-button>

          </div>

        </div>
      </el-aside>
      <el-container>
        <el-main>
          <el-card class="message-block">
            <el-timeline>
              <el-timeline-item class="message-item" :type=" val.type === 'user' ? 'success' : 'primary' " :class="val.type === 'user' ? 'indent' : '' " v-for="( val, key ) in messages" :key="key" :timestamp="val.time" placement="top">
                <div class="content" v-html="val.data">
                </div>
              </el-timeline-item>
            </el-timeline>
          </el-card>
          <el-button style="margin-top: 12px" size="mini" plain type="danger" @click="clearAll">清空消息</el-button>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script>

export default {
  name: 'Dashboard',
  data() {
    return {
      select: 'ws://',
      input: '121.40.165.18:8800/',
      textArea: '',
      messages: [],
      ws: {},
      readyState: null
    }
  },
  computed: {
    connectState() {
      // console.log(this.ws)
      return this.readyState === 1
    }
  },
  methods: {
    init() {
      const url = this.select + this.input
      if (!url) {
        alert('连接地址不能为空')
        return false
      }
      if ('WebSocket' in window) {
        this.ws = new WebSocket(url)
        this.readyState = this.ws.readyState
        const that = this
        const msgData = this.messages
        this.ws.onopen = function() {
          // console.log(83, this)
          that.readyState = this.readyState
          // console.log('websocket 已连接上，此时可以使用send发送数据')
        }
        this.ws.onclose = function() {
          // console.log(88, this)
          that.readyState = this.readyState
          // console.log('websocket 已断开')
        }
        this.ws.onmessage = function(evt) {
          that.readyState = this.readyState
          msgData.push({
            data: evt.data,
            time: that._getTime(),
            type: 'server'
          })
        }
        this.ws.onerror = function(err) {
          that.readyState = this.readyState
          // eslint-disable-next-line no-console
          console.log('错误信息', err)
        }
      } else {
        alert('浏览器不支持socket')
      }
    },
    close() {
      if (!this.ws) {
        alert('暂无连接')
        return false
      }
      // console.log('断开连接')
      this.ws.close()
      this.readyState = this.ws.readyState
    },
    send() {
      // console.log(this.ws)
      if (!this._isConnnect()) {
        alert('暂无连接或连接已断开')
        return false
      }
      this.ws.send(this.textArea)
      this.messages.push({
        data: this.textArea,
        time: this._getTime(),
        type: 'user'
      })
    },
    clearAll() {
      this.messages = []
    },
    _getTime() {
      const date = new Date()
      const hour = date.getHours()
      const minutes = date.getMinutes()
      const second = date.getSeconds()
      return `${hour}:${minutes}:${second}`
    },
    _isConnnect() {
      return !!(this.ws && this.ws.readyState === 1)
    }
  }
}
</script>

<style lang="css" scoped>
  .message-block{
    height: 75vh;
    overflow: auto;
  }

  .connect-box{
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
  }
  .button-group{
      width: 128px;
  }
  .send-box{
    margin-top: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between;
  }
  .type-select{
    width: 80px;
  }
</style>
