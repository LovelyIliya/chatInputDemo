<template>
  <div class="chat-msg-record">
    <!-- <p v-html="msg"></p> -->
    <p>已发送的消息：</p>
    <template v-for="text in chatMsg.split(/(<[^>]+>)/g)">
      <template v-if="!text.startsWith('<emoji')">{{ text }}</template>
      <img v-else :src="imgSrc(text)[1]" class="emoji" />
    </template>
  </div>
</template>

<script setup>
import { ref,defineProps, toRefs, watch } from "vue";
const props = defineProps(["msg", "emojiList"]);
const { msg, emojiList } = toRefs(props);

let chatMsg = ref(msg.value)
watch(
  msg,
  () => {
    // 核心语句
    for (const key in emojiList.value) {
      const reg = new RegExp("(\\" + key + ")", "g");
      chatMsg.value = msg.value.replace(reg, `<emoji src="${emojiList.value[key]}">`);
    }
  },
  { immediate: true }
);
function imgSrc(txt) {
  return txt.match(/<emoji.*?src="(.*?)".*?>/);
}
</script>

<style scoped>
.chat-msg-record {
  width: 600px;
  height: 200px;
  border: 1px solid red;
  margin-top: 60px;
}
.emoji {
  width: 20px;
  height: 20px;
}
</style>
