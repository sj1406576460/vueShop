<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    </el-breadcrumb>

    <h2>欢迎来到电商后台管理系统</h2>

    <p>
      本项目源码已开源在<a href="https://gitee.com/sun_fang_jie/vue_shop">码云</a>
      本项目使用的技术：Vue+VueRouter+ElementUi+axios+MySql+express。<br>
      其中后端接口以及数据库的设计不是本人开发的。接口地址：http://sleepdevil.top:8888/api/private/v1/login <br>
      欢迎访问<a href="https://www.sunfangjie.com">我的博客</a>  <br> 里面有我个人的技术文章
    </p>
  </div>
</template>

<script>
export default {}
</script>

<style lang="scss" scoped>
h2 {
  font-size: 40px;
  margin-top: 0;
  margin-left: 510px;
}

p{
  margin-top: 50px;
  margin-left: 300px;
  font-size: 20px;
}

.el-breadcrumb {
  font-size: 20px;
}
</style>
