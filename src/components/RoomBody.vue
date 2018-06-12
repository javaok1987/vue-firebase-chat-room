<template>
    <div>
      <!-- other people -->
      <div v-if="message.userName != userName">
        <div class="messageBox">
          <img src="https://lorempixel.com/40/40/" class="messageBox__user" draggable="false">
          <div class="messageBox__content">
            <!-- 註解：Vue使用雙花括號{{}}來顯示script中data:的資料 -->
            <div :class="gmLabel">{{message.userName}}</div>
            <div v-if="message.type == 'text'" class="messageBox__message">
              <div class="messageBox__text">{{message.message}}</div>
              <div class="messageBox__readMore" @click="readMore($event)">顯示更多</div>
            </div>
            <div v-if="message.type == 'image'" class="messageBox__image"><img :src="message.message"></div>
          </div>
          <!-- <div class="messageBox__time">{{message.timeStamp}}</div> -->
          <div class="messageBox__time">{{ message.timeStamp | moment('YYYY-MM-DD HH:mm:ss') }}</div>
        </div>
      </div>
      <!-- 區塊：self -->
      <div v-if="message.userName == userName">
        <div class="messageBox messageBox--self">
          <div class="messageBox__time">{{message.timeStamp}}</div>
          <div class="messageBox__content">
            <div v-if="message.type == 'text'" class="messageBox__message">
              <div class="messageBox__text">{{message.message}}</div>
            </div>
            <div v-if="message.type == 'image'" class="messageBox__image"><img :src="message.message"></div>
          </div>
        </div>
      </div>
    </div>
</template>

<script>
export default {
  name: 'RoomBody',
  props: ['message', 'userName'],
  data() {
    return {
      manager: 'tBjoEkDl47VyGVPm82xeuFWAZQr2', // 管理者.
    };
  },
  computed: {
    gmLabel() {
      return (this.message.uid === this.manager) ? 'messageBox__name gmLabel' : 'messageBox__name';
    },
  },
};
</script>
