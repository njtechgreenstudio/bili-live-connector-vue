<template>
  <div id="app">
    <ul>
      <div v-for="log in logs" :key="log.id">{{log.message}}</div>
    </ul>
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      logs: []
    };
  },
  methods: {
    addNewLog(sth) {
      this.logs.push({
        message: sth
      });
    }
  },
  mounted: function() {
    const _this = this;
    var TextEncoder = require("text-encoder-lite").TextEncoderLite;
    var TextDecoder = require("text-encoder-lite").TextDecoderLite;
    const textEncoder = new TextEncoder("utf-8");
    const textDecoder = new TextDecoder("utf-8");

    const readInt = function(buffer, start, len) {
      let result = 0;
      for (let i = len - 1; i >= 0; i--) {
        result += Math.pow(256, len - i - 1) * buffer[start + i];
      }
      return result;
    };
    const writeInt = function(buffer, start, len, value) {
      let i = 0;
      while (i < len) {
        buffer[start + i] = value / Math.pow(256, len - i - 1);
        i++;
      }
    };

    const encode = function(str, op) {
      let data = textEncoder.encode(str);
      let packetLen = 16 + data.byteLength;
      let header = [0, 0, 0, 0, 0, 16, 0, 1, 0, 0, 0, op, 0, 0, 0, 1];
      writeInt(header, 0, 4, packetLen);
      return new Uint8Array(header.concat(...data)).buffer;
    };
    const decode = function(blob) {
      return new Promise(function(resolve, reject) {
        let reader = new FileReader();
        reader.onload = function(e) {
          let buffer = new Uint8Array(e.target.result);
          let result = {};
          result.packetLen = readInt(buffer, 0, 4);
          result.headerLen = readInt(buffer, 4, 2);
          result.ver = readInt(buffer, 6, 2);
          result.op = readInt(buffer, 8, 4);
          result.seq = readInt(buffer, 12, 4);
          if (result.op === 5) {
            result.body = [];
            let offset = 0;
            while (offset < buffer.length) {
              let packetLen = readInt(buffer, offset + 0, 4);
              let headerLen = 16; // readInt(buffer,offset + 4,4)
              let data = buffer.slice(offset + headerLen, offset + packetLen);
              let body = textDecoder.decode(data);
              if (body) {
                result.body.push(JSON.parse(body));
              }
              offset += packetLen;
            }
          } else if (result.op === 3) {
            result.body = {
              count: readInt(buffer, 16, 4)
            };
          }
          resolve(result);
        };
        reader.readAsArrayBuffer(blob);
      });
    };

    //发送handshake，代号7
    const ws = new WebSocket("wss://broadcastlv.chat.bilibili.com:2245/sub");
    ws.onopen = function() {
      ws.send(
        encode(
          JSON.stringify({
            roomid: 360972
          }),
          7
        )
      );
    };

    //发送心跳包，代号2
    setInterval(function() {
      ws.send(encode("", 2));
    }, 30000);

    //接收心跳包实时数据
    ws.onmessage = async function(msgEvent) {
      const packet = await decode(msgEvent.data);
      switch (packet.op) {
        case 8:
          console.log("加入房间");
          break;
        case 3:
          const count = packet.body.count;
          console.log(`人气：${count}`);
          break;
        case 5:
          packet.body.forEach(body => {
            switch (body.cmd) {
              case "DANMU_MSG":
                console.log(`${body.info[2][1]}: ${body.info[1]}`);
                _this.addNewLog(`${body.info[2][1]}: ${body.info[1]}`);
                break;
              case "SEND_GIFT":
                console.log(
                  `${body.data.uname} ${body.data.action} ${body.data.num} 个 ${body.data.giftName}`
                );
                _this.addNewLog(
                  `${body.data.uname} ${body.data.action} ${body.data.num} 个 ${body.data.giftName}`
                );
                break;
              case "WELCOME":
                console.log(`欢迎 ${body.data.uname}`);
                _this.addNewLog(`欢迎 ${body.data.uname}`);
                break;
              // 此处省略很多其他通知类型
              default:
                console.log(`其他通知: `, body);
                break;
            }
          });
          break;
        default:
          console.log(packet);
      }
    };
  }
};
</script>

<style>
</style>
