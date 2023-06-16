<template>
  <div class="main-full-middle">
    <div style="width: 642px">
      <div v-if="!loading">
        <el-result
          icon="success"
          v-if="success"
          title="绑定成功"
          subTitle="将在3秒后跳转到首页"
        >
          <template slot="extra">
            <el-button type="primary" size="medium" @click="handlePush">首页</el-button>
          </template>
        </el-result>

        <el-result
          icon="error"
          v-else
          title="绑定失败"
          subTitle="请重新绑定，将在3秒后跳转到首页"
        >
          <template slot="extra">
            <el-button type="primary" size="medium" @click="handlePush">返回</el-button>
          </template>
        </el-result>
      </div>
    </div>
  </div>
</template>

<script>
//api
import { emailValidate } from "@/api/login";

export default {
  data() {
    return {
      uuid: "",
      loading: true,
      success: "",
      //定时器
      timer:null,
      //3秒后执行
      count:3,
    };
  },

  //组件未挂载到DOM，对数据进行初始化
  created() {
    this.uuid = this.$route.params.uuid;
  },
  //DOM已挂载，发送验证异步请求
  mounted() {

    this.email(this.uuid);
  
  },
  //监听组件渲染变化，促发定时任务
  updated(){
    //启动定时器，每隔 1000 毫秒执行一次回调函数
    this.timer = setInterval(() => {
      this.count--;
      if (this.count <= 0) {
        //执行跳转操作并清除定时器
        this.$router.push("/");
        clearInterval(this.timer);
      }
    }, 1000); 
  },
  methods: {
    //获取所有内容
    email(uuid) {
      emailValidate(uuid)
        .then((response) => {
          this.loading = false;
          this.success = response.data;
        })
        .catch((response) => {});
      }, 

    //立即跳转并清除定时器
    handlePush() {
      this.$router.push("/");
      clearInterval(this.timer);
    }

},
    
};
</script>

<style scoped>
.main-full-middle {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}
.tag-group {
  text-align: center;
}
</style>