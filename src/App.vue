<template>
  <div id="app">
    <div class="msg-box">
      <!-- 消息输入框 -->
      <div class="msg-input" ref="msgInput" contenteditable @blur="getAfterBlurIndex" @keydown="msgInputKeyDown" @input="msgInputFun"></div>
      <!-- 表情列表 -->
      <div class="emoji-list">
        <p>表情列表</p>
        <img v-for="(emoji, key) in emojiList" :key="key" :src="emoji" @click="selectEmoji(key)" />
      </div>
      <button @click="sendMsg">发送</button>
    </div>
    <p>数据本体：{{ chatMsgRecord }}</p>
    <!-- 消息回显框 -->
    <chatMsgEl :msg="chatMsgRecord" :emojiList="emojiList"></chatMsgEl>
    <!-- 可艾特的聊天群员 -->
    <userGroup :userGroup="userGroupData" :userGroupPosition="userGroupPosition" v-if="showAtSelect" @at-user="selectAtUser"></userGroup>
  </div>
</template>

<script setup>
import chatMsgEl from "./components/chatMsgEl.vue";
import userGroup from "./components/userGroup.vue";
import { reactive, ref } from "vue";

const emojiList = {
  "[vueLogo]": require("./assets/logo.png"),
};

let msgInput = ref(); // 输入框ref绑定

let chatMsgRecord = ref(""); // 发送的消息体

let focusNode = reactive({}); // 存储光标聚焦节点
let focusOffset = ref(0); // 存储光标聚焦偏移量
let chatInputOffset = reactive({}); // 存储光标聚焦的元素

// 聊天输入框失焦时获取失焦前的光标位置
function getAfterBlurIndex() {
  setTimeout(() => {
    showAtSelect.value = false;
  }, 100);
  if (window.getSelection) {
    let sel = window.getSelection();
    if (sel.getRangeAt && sel.rangeCount) {
      focusNode = sel.focusNode;
      focusOffset.value = sel.focusOffset;
      chatInputOffset = sel.getRangeAt(0);
      console.log("focusNode:", focusNode);
      console.log("focusOffset:", focusOffset.value);
      console.log("chatInputOffset:", chatInputOffset);
    }
  }
}
// 获取输入框选取
function getInputSelection() {
  let sel = window.getSelection();
  // 插入内容之前输入框是否已经聚焦获得选区
  if (
    !chatInputOffset.endContainer ||
    (chatInputOffset.endContainer.className != "msg-input" &&
      chatInputOffset.endContainer.parentNode.className != "msg-input" &&
      chatInputOffset.endContainer.parentNode.parentNode.className != "msg-input")
  ) {
    chatInputOffset = document.createRange();
    //用于设置 Range，使其包含一个 Node的内容。
    chatInputOffset.selectNodeContents(document.querySelector(".msg-input"));
    //将包含着的这段内容的光标设置到最后去，true 折叠到 Range 的 start 节点，false 折叠到 end 节点。如果省略，则默认为 false .
    chatInputOffset.collapse(false);
    sel = window.getSelection();
    sel.removeAllRanges();
    sel.addRange(chatInputOffset);
  }
}
// 点击选择表情
function selectEmoji(emoji) {
  getInputSelection(); // 先获得一下输入框焦点
  // 创建图片元素，回显到输入框内
  const img = `<img src="${emojiList[emoji]}" alt='${emoji}' class="emoji" style="width: 36px;height: 36px;object-fit: contain" >`;
  chatInputOffset.collapse(false); //光标移至最后
  // 创建节点并插入
  const node = chatInputOffset.createContextualFragment(img);
  let c = node.lastChild;
  chatInputOffset.insertNode(node);
  if (c) {
    chatInputOffset.setEndAfter(c);
    chatInputOffset.setStartAfter(c);
  }
  let j = window.getSelection();
  j.removeAllRanges();
  j.addRange(chatInputOffset);
}

let msgType = ref("text"); // 发送的消息类型

function sendMsg() {
  console.log("msgInput:", msgInput);
  chatMsgRecord.value = ""; // 先清空一下旧消息
  msgType.value = "text"; // 初始化消息类型
  msgInput.value.childNodes.forEach((element) => {
    // 如果是emoji表情图片的话，则转义
    if (element.nodeName === "IMG" && element.className === "emoji") {
      chatMsgRecord.value += element.alt;
    } else if (element.className === "at-msg") {
      chatMsgRecord.value += element.innerText + " ";
      msgType.value = "groupChatAt"; // 根据实际约束类型修改
    } else {
      chatMsgRecord.value += element.data;
    }
  });
  // 清空输入框中的内容
  msgInput.value.innerHTML = "";
  msgInput.value.innerText = "";
  // 在这里使用websocket把数据发送给后端
  // socketSend({text:chatMsgRecord,type:msgType})
}
// 群员列表
const userGroupData = reactive([
  { name: "用户A", id: 1, avatar: require("@/assets/logo.png") },
  { name: "用户B", id: 2, avatar: require("@/assets/logo.png") },
  { name: "用户C", id: 3, avatar: require("@/assets/logo.png") },
]);
// 用户列表定位
const userGroupPosition = reactive({
  x: "0px",
  y: "0px",
});
let showAtSelect = ref(false);
// 获取输入框中的光标相对页面的坐标
function getCursorPosition() {
  let sel = window.getSelection();
  let range = document.createRange();
  range.selectNode(sel.focusNode);
  range.setStart(sel.focusNode, sel.focusOffset);
  const { x, y } = range.getBoundingClientRect();
  return { x, y };
}
function msgInputFun(e) {
  if (e.data === "@") {
    setTimeout(() => {
      const { x, y } = getCursorPosition();
      showAtSelect.value = true;
      userGroupPosition.x = x + "px";
      userGroupPosition.y = y + "px";
    }, 100);
  } else {
    showAtSelect.value = false;
  }
}
// 显示艾特选择组件时，阻止输入框默认上下按键及回车行为
function msgInputKeyDown(e) {
  if (showAtSelect.value && ["ArrowUp", "ArrowDown", "Enter"].includes(e.key)) {
    return e.preventDefault();
  }
  if (e.key === "Enter") {
    e.preventDefault();
    sendMsg();
  }
}
function selectAtUser(user) {
  showAtSelect.value = false;
  msgInput.value.blur();
  getInputSelection();
  chatInputOffset.setStart(focusNode, focusOffset.value - 1);
  chatInputOffset.setEnd(focusNode, focusOffset.value);
  chatInputOffset.deleteContents();
  const atElement = `<span userId='${user.id}' userName='${user.name}' contentEditable="false" class="at-msg" style="color:#2E77E5">@${user.name}&nbsp;</span>`;
  chatInputOffset.collapse(false);
  const node = chatInputOffset.createContextualFragment(atElement);
  let c = node.lastChild;
  chatInputOffset.insertNode(node);
  if (c) {
    chatInputOffset.setEndAfter(c);
    chatInputOffset.setStartAfter(c);
  }
  let j = window.getSelection();
  j.removeAllRanges();
  j.addRange(chatInputOffset);
}
</script>

<style scoped>
.msg-box {
  display: flex;
}
.emoji-list {
  width: 200px;
  border: 1px solid greenyellow;
  padding: 5px;
  img {
    width: 30px;
    cursor: pointer;
  }
}
.msg-input {
  width: 300px;
  height: 150px;
  border-radius: 6px;
  border: 1px solid #a1a1a1;
  outline: none;
  &:focus {
    border-color: aqua;
  }
}
</style>
