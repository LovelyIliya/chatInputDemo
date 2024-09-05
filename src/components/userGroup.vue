<template>
  <!-- userGroupPosition父组件传递显示定位 -->
  <div class="user-group" :style="{ left: userGroupPosition.x, top: userGroupPosition.y }">
    <ul class="user-list">
      <li v-for="(user, index) in userGroup" :key="user.id" class="user-item" :class="{ active: activeIndex === index }" @click="selectAtUser(user)">
        <img :src="user.avatar" />
        <span>{{ user.name }}</span>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, defineProps, toRefs, onMounted, defineEmits } from "vue";
/**
 * @param userGroup 聊天群员列表
 * @param userGroupPosition 列表组件显示定位
 */
const props = defineProps(["userGroup", "userGroupPosition"]);
// 选择群员事件
const emit = defineEmits(["atUser"]);
const { userGroup, userGroupPosition } = toRefs(props);
// 此变量用来使用键盘上下按钮时，选择群员
let activeIndex = ref(-1);
// 给window挂载键盘事件，这样使用键盘上下键时可以选择群员
onMounted(() => {
  window.addEventListener("keydown", keyboardSelect);
});
// 键盘事件 通过修改activeIndex值，达到选择群员
function keyboardSelect(e) {
  if (e.key === "ArrowDown") {
    if (activeIndex.value !== userGroup.value.length - 1) {
      activeIndex.value++;
    } else {
      activeIndex.value = 0;
    }
  } else if (e.key === "ArrowUp") {
    if (activeIndex.value <= 0) {
      activeIndex.value = userGroup.value.length - 1;
    } else {
      activeIndex.value--;
    }
  } else if (e.key === "Enter") {
    selectAtUser(userGroup.value[activeIndex.value]);
  }
}
// 给父组件传递选择的群员
function selectAtUser(user) {
  console.log(user);
  emit("atUser", user);
}
</script>

<style scoped>
.user-group {
  position: fixed;
}
.user-list {
  list-style: none;
  padding: 0;
  border: 1px solid #e1e1e1;
  border-radius: 6px;
}

.user-item {
  height: 50px;
  line-height: 50px;
  padding: 0 20px;
  cursor: pointer;
  background-color: white;
  &:hover,
  &.active {
    background-color: #e1e1e1;
  }
  &:not(:last-of-type) {
    border-bottom: 1px solid #e1e1e1;
  }
  img {
    width: 30px;
    vertical-align: middle;
  }
}
</style>
