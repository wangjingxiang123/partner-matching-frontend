<template>
  <template v-if="user">
    <van-cell title="当前用户" :value="user?.username" />
    <van-cell title="修改信息" is-link to="/user/update" />
    <van-cell title="我创建的队伍" is-link to="/user/team/create" />
    <van-cell title="我加入的队伍" is-link to="/user/team/join" />
    <van-cell title="退出登录" @click="handleLogout" />
  </template>
</template>

<script setup lang="ts">
import { useRouter } from "vue-router";
import { onMounted, ref } from "vue";
import { getCurrentUser } from "../services/user";
// 假设 myAxios 是已经配置好的 HTTP 请求工具
import myAxios from "../plugins/myAxios";

const user = ref();
const router = useRouter();

onMounted(async () => {
  user.value = await getCurrentUser();
});

const toEdit = (editKey: string, editName: string, currentValue: string) => {
  router.push({
    path: '/user/edit',
    query: {
      editKey,
      editName,
      currentValue,
    }
  });
};

const handleLogout = async () => {
  try {
    // 调用 user/logout 接口
    const response = await myAxios.post('/user/logout');
    if (response && response.data) {
      // 退出登录成功，跳转到登录页面
      const redirectUrl = window.location.href;
      window.location.href = `/user/login?redirect=${redirectUrl}`;
      // 清空当前用户信息
      user.value = null;
    } else {
      console.error('退出登录失败');
    }
  } catch (error) {
    console.error('退出登录出错:', error);
  }
};
</script>

<style scoped>
</style>