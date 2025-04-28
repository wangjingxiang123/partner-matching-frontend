<template>

  <!-- 先显示骨架屏，数据加载完成后隐藏 -->
  <van-skeleton title avatar :row="3" v-if="!userList || userList.length === 0" />
  <!-- 循环渲染用户卡片 -->
  <van-card
      v-for="item in userList"
      :key="item.id"
      :desc="item.profile"
      :title="`${item.username}（${item.planetCode}）`"
      :thumb="item.avatarUrl"
  >
    <template #tags>
      <van-tag plain type="danger" v-for="tag in item.tags" style="margin-right: 8px; margin-top: 8px">
        {{ tag }}
      </van-tag>
    </template>
    <template #footer>
      <van-button size="mini">联系我</van-button>
    </template>
  </van-card>
  <van-empty v-if="!userList || userList.length < 1" description="搜索结果为空" />
</template>

<script setup lang="ts">
import { onMounted, reactive, ref } from 'vue';
import { useRoute } from "vue-router";
import myAxios from "../plugins/myAxios";
import { Toast } from "vant";
import qs from 'qs';

const route = useRoute();
const { tags } = route.query;

const userList = ref([]);

onMounted(async () => {
  const userListData = await myAxios.get('/user/search/tags', {
    params: {
      tagNameList: tags
    },
    paramsSerializer: params => {
      return qs.stringify(params, { indices: false })
    }
  })
      .then(function (response) {
        console.log('/user/search/tags succeed', response);
        return response?.data;
      })
      .catch(function (error) {
        console.error('/user/search/tags error', error);
        Toast.fail('请求失败');
      })
  console.log(userListData)
  if (userListData) {
    userListData.forEach(user => {
      if (user.tags) {

        user.tags = JSON.parse(user.tags);
      }
    })
    userList.value = userListData;

  }
})
</script>

<style scoped>

</style>