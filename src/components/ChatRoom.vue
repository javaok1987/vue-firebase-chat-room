<template>
  <div class="container">
    <!-- 區塊：name area -->
    <div class="name">
      <h3>Name：{{ userName }}</h3>
      <!-- 註解：使用@click來偵測click，觸發時執行method中的setName() -->
      <div class="reset" @click="setName()">Reset Name</div>
    </div>
    <!-- 區塊：chat room -->
    <div class="chatRoom">
      <!-- 區塊：head -->
      <div class="roomHead">
        <div class="roomHead__topButtons">
          <div class="roomHead__button close"></div>
          <div class="roomHead__button minimize"></div>
          <div class="roomHead__button zoom"></div>
        </div>
        <img src="https://lorempixel.com/50/50/" class="roomHead__img" draggable="false">
        <div class="roomHead__title">聊天室</div>
        <div class="roomHead__notice">{{lastNotice.message}}</div>
      </div>
      <!-- 區塊：body -->
      <div id="js-roomBody" class="roomBody">
        <!-- 註解：使用template來當迴圈容器，或是判斷用的容器，當條件達成時產出template內容 -->
        <room-body v-for="(item, key) in messages"
          v-bind:key="key"
          v-bind:message="item"
          v-bind:userName="userName"
        ></room-body>
        <!-- 區塊：上傳進度條 -->
        <div v-show="upload" class="messageBox messageBox--self">
          <div class="messageBox__progress">
            <div id="js-progressBar" class="messageBox__progress--state"></div>
            <div class="messageBox__progress--number">{{progress}}</div>
          </div>
        </div>
      </div>
      <!-- 區塊：bottom -->
      <!-- 註解：使用:class來寫class是否顯示的判斷式{ class: 判斷式 } -->
      <div class="roomBottom" :class="{ disable: !userName }">
        <div class="roomBottom__tools">
          <div v-if="user.auth" class="roomBottom__tool_upload">
            <input type="file" accept="image/*" @change="sendImage($event)">
            <img src="../assets/icon-tools__file.png">
          </div>
          <div v-if="user.auth" class="roomBottom__tool_pin" @click="setNotice()">
            <img src="../assets/icon-tools__pin.png">
          </div>
        </div>
        <div class="roomBottom__input">
          <!-- 若要再帶入原生js的event(e)到function中，必須使用$event當參數傳入 -->
          <textarea id="js-message" class="roomBottom__input__textarea"
            :class="{ disable: !userName }" @keypress.enter="sendMessage($event)"></textarea>
        </div>
      </div>
    </div>
    <!-- 區塊：modal -->
    <div v-show="userNameSet || userName === ''" class="modal">
      <div class="modal__container">
        <header class="modal__header">
          <h2 class="view-title">輸入名稱</h2>
        </header>
        <div class="modal__body">
          <!-- 註解：使用@keypress.enter來偵測keypress enter，觸發時執行method中的saveName() -->
          <input type="text" id="js-userName" class="userName" maxlength="20"
            @keypress.enter="saveName()" :value="userName">
          <div class="button" @click="saveName()">設定</div>
        </div>
        <footer class="modal__footer"></footer>
      </div>
    </div>
    <div v-show="noticeSet && user.auth" class="modal">
      <div class="modal__container">
        <header class="modal__header">
          <h2 class="view-title">輸入公告</h2>
        </header>
        <div class="modal__body">
          <!-- 註解：使用@keypress.enter來偵測keypress enter，觸發時執行method中的saveName() -->
          <input type="text" id="js-notice" class="userName" maxlength="20"
            @keypress.enter="saveNotice()" :value="noticeContent">
          <div class="button" @click="saveNotice()">設定</div>
        </div>
        <footer class="modal__footer"></footer>
      </div>
    </div>
  </div>
</template>

<script>

/* global firebase */
/* eslint-env es6 */
/* eslint-disable no-console */

import '@/assets/css/chat-room.css';

import RoomBody from '@/components/RoomBody';

// msgRef = firebase中的資料表/messages/，若沒有的會自動建立
const msgRef = firebase.database().ref('/messages/');
const noticeRef = firebase.database().ref('/notice/');
const storageRef = firebase.storage().ref('/images/');
export default {
  // 指定此頁使用的name
  name: 'ChatRoom',
  // 資料位置，於html中可用{{}}渲染出來
  data() {
    return {
      userNameSet: false, // 姓名輸入框
      noticeSet: false, // 公告輸入框
      userName: '', // 名稱
      messages: {}, // 訊息內容
      lastNotice: '', // 最新公告
      noticeContent: '', // 公告內容
      upload: false, // 上傳進度框
      progress: '', // 上傳進度%數
      user: { // 使用者
        uid: '',
        email: '',
        auth: false,
      },
    };
  },
  components: {
    'room-body': RoomBody,
  },

  // 這個頁面的functions
  methods: {
    /** 彈出設定視窗 */
    setName() {
      this.userNameSet = true;
    },

    setNotice() {
      this.noticeSet = true;
    },

    /** 儲存設定名稱 */
    saveName() {
      // vue的mtthod中this是指export中這整塊的資料
      const vm = this;
      const userName = document.querySelector('#js-userName').value;
      if (userName.trim() === '') {
        return;
      }
      // 這裡的vm.userName(this.userName)就是data()裡面的userName
      vm.userName = userName;
      vm.userNameSet = false;
    },

    /** 儲存公告 */
    saveNotice() {
      const vm = this;
      const notice = document.querySelector('#js-notice').value;
      if (notice.trim() === '') {
        return;
      }
      noticeRef.push({
        type: 'text',
        message: notice.trim(),
        timeStamp: vm.getTime(),
      });
      vm.noticeContent = notice;
      vm.noticeSet = false;
    },

    /** 取得時間 */
    getTime() {
      // 由 firebase 提供系統時間.
      // const now = new Date();
      // const hours = now.getHours();
      // const minutes = now.getMinutes();
      // return `${hours >= 12 ? '下午' : '上午'} ${hours}:${minutes < 10 ? `0${minutes}` : minutes}`;
      return firebase.database.ServerValue.TIMESTAMP;
    },

    /** 傳送訊息 */
    sendMessage(e) {
      const vm = this;
      const userName = document.querySelector('#js-userName');
      const message = document.querySelector('#js-message');
      // 如果是按住shift則不傳送訊息(多行輸入)
      if (e.shiftKey) {
        return false;
      }
      // 如果輸入是空則不傳送訊息
      if (message.value.length <= 1 && message.value.trim() === '') {
        // 避免enter產生的空白換行
        e.preventDefault();
        return false;
      }
      // 對firebase的db做push，db只能接受json物件格式，若要用陣列要先轉字串來存
      msgRef.push({
        userName: userName.value,
        type: 'text',
        message: message.value,
        // 取得時間，這裡的vm.getTime()就是method中的getTime
        timeStamp: vm.getTime(),
        uid: vm.user.uid,
        auth: vm.user.auth,
      });
      // 清空輸入欄位並避免enter產生的空白換行
      message.value = '';
      e.preventDefault();
      return false;
    },

    /** 傳送圖片 */
    sendImage(e) {
      const vm = this;
      const userName = document.querySelector('#js-userName');
      // 取得上傳檔案的資料
      const file = e.target.files[0];
      const fileName = `${Math.floor(Date.now() / 1000)}_${file.name}`;
      const metadata = {
        contentType: 'image/*',
      };
      // 取得HTML進度條元素
      const progressBar = document.querySelector('#js-progressBar');
      // 上傳資訊設定
      const uploadTask = storageRef.child(fileName).put(file, metadata);
      // 上傳狀態處理
      uploadTask.on(
        firebase.storage.TaskEvent.STATE_CHANGED,
        /* 上傳進度 */
        (snapshot) => {
          const progress = Math.floor((snapshot.bytesTransferred / snapshot.totalBytes) * 100);
          if (progress < 100) {
            // 開啟進度條
            vm.upload = true;
            vm.progress = `${progress}%`;
            progressBar.setAttribute('style', `width:${progress}%`);
          }
        },
        /* 錯誤處理 */
        (error) => {
          msgRef.child('bug/').push({
            userName: userName.value,
            type: 'image',
            message: error.code,
            timeStamp: vm.getTime(),
          });
        },
        /* 上傳結束處理 */
        () => {
          const downloadURL = uploadTask.snapshot.downloadURL;
          msgRef.push({
            userName: userName.value,
            type: 'image',
            message: downloadURL,
            timeStamp: vm.getTime(),
          });
          // 關閉進度條
          vm.upload = false;
        },
      );
    },

    /** 顯示更多 */
    readMore(e) {
      // 把內容高度限制取消
      e.target.previousElementSibling.setAttribute(
        'style',
        'max-height: 100%;',
      );
      // 隱藏"顯示更多"按紐
      e.target.setAttribute('style', 'display: none;');
    },
  },


  // mounted是vue的生命週期之一，代表模板已編譯完成，已經取值準備渲染元件了
  mounted() {
    const vm = this;
    msgRef.on('value', (snapshot) => {
      vm.messages = (snapshot.val() === null) ? {} : snapshot.val();
    });

    noticeRef.on('value', (snapshot) => {
      const val = snapshot.val();
      const temp = { ...val };
      const { [Object.keys(temp).pop()]: lastItem } = temp;
      vm.lastNotice = lastItem;
    });

    // login
    firebase.auth().setPersistence(firebase.auth.Auth.Persistence.SESSION)
      .then(() => firebase.auth().signInWithEmailAndPassword('write@housefun.com.tw', 'writewrite'))
      .catch((error) => {
        // Handle Errors here.
        const errorCode = error.code;
        const errorMessage = error.message;
        console.log(errorCode, errorMessage);
      });

    // 設置身份驗證狀態觀察者並獲取用戶數據
    firebase.auth().onAuthStateChanged((user) => {
      if (user) {
        // User is signed in.
        vm.user = {
          uid: user.uid,
          email: user.email,
          auth: true,
        };
      } else {
        // User is signed out.
        // ...
      }
      console.log('登入狀態', vm.user.auth);
    });
  },


  // update是vue的生命週期之一，在元件渲染完成後執行
  updated() {
    // 判斷內容高度超過300就隱藏起來，把"顯示更多"按紐打開
    const messages = document.querySelectorAll('.messageBox__message');
    messages.forEach((message) => {
      if (message.offsetHeight > 300) {
        message
          .querySelector('.messageBox__readMore')
          .setAttribute('style', 'display: block');
      }
    });
    // 當畫面渲染完成，把聊天視窗滾到最底部(讀取最新消息)
    const roomBody = document.querySelector('#js-roomBody');
    roomBody.scrollTop = roomBody.scrollHeight;
  },
};
</script>
