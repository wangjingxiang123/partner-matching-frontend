<template>
  <van-form @submit="onSubmit">


    <div v-if="editUser.editKey=== 'avatarUrl'">
      <van-uploader
          v-model="editUser.currentValue"
          :name="editUser.editKey"
          :label="editUser.editName"
          :after-read="afterRead"
          multiple
          :max-count="1"
      >
        <!-- 图片回显 -->
        <template #preview="scope">
          <img :src="editUser.currentValue || scope.url" alt="预览" style="max-width: 100%; max-height: 100%;" />
        </template>
      </van-uploader>
    </div>

    <div v-if = "editUser.editKey === 'tags'">
      <van-divider content-position="left">已选标签</van-divider>
      <div v-if="activeIds.length === 0">请选择标签</div>
      <van-row gutter="16" style="padding: 0 16px">
        <van-col v-for="tag in activeIds">
          <van-tag closeable size="small" type="primary" @close="doClose(tag)">
            {{ tag }}
          </van-tag>
        </van-col>
      </van-row>
      <van-divider content-position="left">选择标签</van-divider>
      <van-tree-select
          v-model:active-id="activeIds"
          v-model:main-active-index="activeIndex"
          :items="tagList"
      />
    </div>

    <div v-if ="editUser.editKey === 'gender'">
      <van-radio-group v-model="editUser.currentValue"  >
        <van-radio name="0" value="0">男</van-radio>
        <van-radio name="1" value="1">女</van-radio>
      </van-radio-group>
    </div>


    <div v-if ="editUser.editKey !== 'gender' && editUser.editKey !== 'tags' && editUser.editKey !== 'avatarUrl'">
      <van-field
          v-model="editUser.currentValue"
          :name="editUser.editKey"
          :label="editUser.editName"
          :placeholder="`请输入${editUser.editName}`"
      />
    </div>


    <div style="margin: 16px;">
      <van-button round block type="primary" native-type="submit">
        提交
      </van-button>
    </div>
  </van-form>
</template>

<script setup lang="ts">
import { useRoute, useRouter } from "vue-router";
import { ref } from "vue";
import myAxios from "../plugins/myAxios";
import { Toast } from "vant";
import { getCurrentUser } from "../services/user";
// -------------------


const searchText = ref('');

const originTagList = [{
  text: '学习方向',
  children: [
    {text: 'Java', id: 'Java'},
    {text: 'python', id: 'python'},
    {text: 'C++', id: 'C++'},
    {text: 'PHP', id: 'PHP'},
    {text: 'Vue', id: 'Vue'},
  ],
},
  {
    text: '年级',
    children: [
      {
        "text": "freshman",
        "id": "freshman"
      },
      {
        "text": "sophomore",
        "id": "sophomore"
      },
      {
        "text": "junior",
        "id": "junior"
      },
      {
        "text": "senior",
        "id": "senior"
      }
    ],
  },
]

// 标签列表
let tagList = ref(originTagList);

/**
 * 搜索过滤
 * @param val
 */
const onSearch = (val) => {
  tagList.value = originTagList.map(parentTag => {
    const tempChildren = [...parentTag.children];
    const tempParentTag = {...parentTag};
    tempParentTag.children = tempChildren.filter(item => item.text.includes(searchText.value));
    return tempParentTag;
  });

}
const onCancel = () => {
  searchText.value = '';
  tagList.value = originTagList;
};

//-------------

const route = useRoute();
const router = useRouter();

const editUser = ref({
  editKey: route.query.editKey,
  currentValue: route.query.editKey !== "avatarUrl" ? route.query.currentValue : [],
  editName: route.query.editName,
});

console.log("log",editUser.value.currentValue[0])
// 用于存储上传的文件列表（包含图片信息）
const uploadedFiles = ref<File[]>([]);
const imageUrl = ref<string>("");

// 已选中的标签
const activeIds = ref([]);
const activeIndex = ref(0);


// 格式转化 string -> array
if (editUser.value.editKey === 'tags') {
  activeIds.value = JSON.parse(editUser.value.currentValue)
}

console.log("45678",activeIds.value)
// 移除标签
const doClose = (tag) => {
  activeIds.value = activeIds.value.filter(item => {
    return item !== tag;
  })
}

const afterRead = async (file: any) => {
  const currentUser = await getCurrentUser();
  if (!currentUser) {
    Toast.fail("用户未登录");
    return;
  }

  const formData = new FormData();
  formData.append("id", currentUser.id.toString());
  formData.append(editUser.value.editKey as string, editUser.value.currentValue);
  formData.append("file", file.file);

  try {
    const res = await myAxios.post("/user/uploadAvatar", formData, {
      headers: {
        "Content-Type": "multipart/form-data",
      },
    });
    if (res.code === 0) {
      imageUrl.value = res.data; // 假设接口返回图片地址在 data 字段
      // 更新上传文件列表，添加上传成功的图片信息
      uploadedFiles.value = [
        ...uploadedFiles.value,
        {
          file: file.file,
          url: imageUrl.value,
        },
      ];
      Toast.success("图片上传成功");
    } else {
      Toast.fail("图片上传失败");
    }
  } catch (error) {
    console.error("图片上传出错:", error);
    Toast.fail("图片上传出错");
  }
};

const onSubmit = async () => {


  const currentUser = await getCurrentUser();

  if (!currentUser) {
    Toast.fail("用户未登录");
    return;
  }

  if (editUser.value.editKey === "tags") {
    console.log("tags",activeIds.value)
    const tagParams = activeIds.value.map(tag => `tags=${encodeURIComponent(tag)}`).join('&');
    const url = `/user/updateTags?${tagParams}`;

    try {
      const res = await myAxios.get(url);
      console.log(res, "更新请求");
      if (res.code === 0) {
        Toast.success("修改成功");
        router.back();
      } else {
        Toast.fail("修改错误");
      }
    } catch (error) {
      console.error("更新请求出错:", error);
      Toast.fail("更新请求出错");
    }
  }else {
    console.log(currentUser, "当前用户");

    // 构建 JSON 数据
    const data = {
      id: currentUser.id.toString(),
      [editUser.value.editKey as string]: editUser.value.editKey === "avatarUrl" ? imageUrl.value : editUser.value.currentValue,
    };

    try {
      const res = await myAxios.post("/user/update", JSON.stringify(data), {
        headers: {
          "Content-Type": "application/json",
        },
      });
      console.log(res, "更新请求");
      if (res.code === 0 && res.data > 0) {
        Toast.success("修改成功");
        router.back();
      } else {
        Toast.fail("修改错误");
      }
    } catch (error) {
      console.error("更新请求出错:", error);
      Toast.fail("更新请求出错");

    }
  }

};
</script>

<style scoped>
</style>