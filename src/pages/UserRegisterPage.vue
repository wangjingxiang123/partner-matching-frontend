<template>
  <van-form @submit="onSubmit">
    <van-cell-group inset>
      <van-field
          v-model="userName"
          name="userName"
          label="用户名"
          placeholder="请输入用户名"
          :rules="[{ required: true, message: '请填写用户名' }]"
      />
      <van-field
          v-model="userAccount"
          name="userAccount"
          label="账号"
          placeholder="请输入账号"
          :rules="[{ required: true, message: '请填写账号' }]"
      />
      <van-field
          v-model="userPassword"
          type="password"
          name="userPassword"
          label="密码"
          placeholder="请输入密码"
          :rules="[{ required: true, message: '请填写密码' }]"
      />
      <van-field
          v-model="confirmPassword"
          type="password"
          name="confirmPassword"
          label="确认密码"
          placeholder="请再次输入密码"
          :rules="[{ required: true, message: '请再次输入密码' }, { validator: validatePasswordMatch, message: '两次输入的密码不一致' }]"
      />
      <van-field
          v-model="userPhone"
          name="userPhone"
          label="电话"
          placeholder="请输入电话"
          :rules="[{ required: true, message: '请填写电话' }]"
      />
      <van-field
          v-model="userEmail"
          name="userEmail"
          label="邮箱"
          placeholder="请输入邮箱"
          :rules="[{ required: true, message: '请填写邮箱' }, { validator: validateEmail, message: '请输入有效的邮箱地址' }]"
      />
      <van-field
          v-model="planetCode"
          name="planetCode"
          label="星球编号"
          placeholder="请输入星球编号"
          :rules="[{ required: true, message: '请填写星球编号' }]"
      />
      <van-radio-group v-model="gender">
        <van-radio name="0" value="0">男</van-radio>
        <van-radio name="1" value="1">女</van-radio>
      </van-radio-group>
<!--      <van-uploader-->
<!--          v-model="avatarList"-->
<!--          :max-count="1"-->
<!--          @after-read="afterReadAvatar"-->
<!--          :before-delete="beforeDeleteAvatar"-->
<!--      >-->
<!--        <template #upload>-->
<!--          <van-button icon="plus" type="primary">上传头像</van-button>-->
<!--        </template>-->
<!--      </van-uploader>-->
    </van-cell-group>
    <div style="margin: 16px;">
      <van-button round block type="primary" native-type="submit">
        注册
      </van-button>
    </div>
  </van-form>
</template>

<script setup lang="ts">
import { ref } from "vue";
import myAxios from "../plugins/myAxios";
import { Toast } from "vant";

const userName = ref('');
const userAccount = ref('');
const userPassword = ref('');
const confirmPassword = ref('');
const userPhone = ref('');
const userEmail = ref('');
const planetCode = ref('');
const tagInput = ref('');
const gender = ref('');
const avatarList = ref<File[]>([]);

const validatePasswordMatch = (value: string) => {
  return value === userPassword.value;
};

const validateEmail = (value: string) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(value);
};

const afterReadAvatar = (file: File) => {
  avatarList.value = [file];
};

const beforeDeleteAvatar = () => {
  avatarList.value = [];
  return true;
};

const onSubmit = async () => {
  const tagsArray = tagInput.value.split(',').filter(tag => tag.trim()!== '');
  const tagsJson = JSON.stringify(tagsArray);

  // 构建 JSON 格式的数据对象
  const formData = {
    userName: userName.value,
    userAccount: userAccount.value,
    userPassword: userPassword.value,
    userPhone: userPhone.value,
    userEmail: userEmail.value,
    planetCode: planetCode.value,
    tags: tagsJson,
    gender: gender.value
  };
  // if (avatarList.value.length > 0) {
  //   // 处理头像上传，这里假设后端支持 base64 编码的头像数据
  //   const reader = new FileReader();
  //   reader.readAsDataURL(avatarList.value[0]);
  //   reader.onloadend = () => {
  //     formData.avatar = reader.result as string;
  //   };
  // }

  const res = await myAxios.post('/user/register', JSON.stringify(formData), {
    headers: {
      'Content-Type': 'application/json'
    },
    skipLoginCheck: true // 添加跳过登录检查的配置
  });
  console.log(res, '用户注册');
  if (res.code === 0 && res.data) {
    Toast.success('注册成功');
    // 注册成功后跳转到登录页面
    window.location.href = '/user/login';
  } else {
    Toast.fail('注册失败');
  }
};
</script>

<style scoped>

</style>