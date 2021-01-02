<template>
  <el-container>
    <el-header>
      <div>
        <img src="@/assets/logo.png" alt="" />
        <span>电商后台管理系统</span>
      </div>
      <div class="outDiv">
        <el-button type="info" @click="logout">退出</el-button>
      </div>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleAside">|||</div>
        <el-menu 
        :unique-opened="true"
        background-color="#373d41"   
        text-color="#fff"
        active-text-color="blue"
        :collapse="isCollapse"
        :collapse-transition="false"
        router
        :default-active="actPath"
        >
          <el-submenu :index="'/' + item.path" v-for="item in menulist" :key="item.id" :router="true">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item :index="'/' + citem.path" v-for="citem in item.children" :key="citem.id" @click="saveActPath('/' + citem.path)">
                 <template slot="title">
              <i class="el-icon-menu"></i>
              <span>{{citem.authName}}</span>
            </template>
            </el-menu-item>
          </el-submenu>
          </el-menu-item>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      menulist: [],
      isCollapse: false,
      actPath: window.sessionStorage.getItem("actPath"),
    };
  },
  methods: {
    logout() {
      window.sessionStorage.clear();
      this.$router.push("/login");
      this.$message.success("退出成功!");
    },
    async getMenulist() {
      const { data: res } = await this.$http.get("menus");
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.$message.success(res.meta.msg);
      this.menulist = res.data;
      // console.log(this.menulist);
    },
    toggleAside() {
      this.isCollapse = !this.isCollapse;
    },
    saveActPath(actPath) {
      window.sessionStorage.setItem("actPath", actPath);
    },
  },
  components: {},
  created() {
    this.getMenulist();
  },
};
</script>

<style lang="less" scoped>
.el-container {
  height: 100%;
}
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding: 0;

  img {
    vertical-align: middle;
    height: 100%;
  }
  span {
    font-size: 24px;
    font-weight: 800;
    color: #eaedf1;
  }
  .outDiv {
    height: 100%;
    padding: 10px 0;
  }
}
.el-aside {
  background-color: #333744;
}
.el-main {
  background-color: #eaedf1;
}
.toggle-button {
  text-align: center;
  background-color: #4a5064;
  font-size: 10px;
  height: 24px;
  line-height: 24px;
  color: #fff;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>