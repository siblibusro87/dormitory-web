<template>
  <v-app :dark="dark">
    <!-- 左侧导航条 -->
    <v-navigation-drawer
      :dark="dark"
      persistent
      :mini-variant="miniVariant"
      v-model="drawer"
      enable-resize-watcher
      fixed
      app
    >
      <userinfo :userinfo="userInfo"></userinfo>
      <v-divider/>
      <!-- 左侧菜单 -->
      <v-list class="pt-0" dense>
        <v-list-group
          v-model="item.active"
          v-for="item in items"
          :key="item.title"
          :prepend-icon="item.action"
          no-action
        >
          <!--一级菜单 -->
          <v-list-tile slot="activator">
            <v-list-tile-content>
              <v-list-tile-title>{{ item.title }}</v-list-tile-title>
            </v-list-tile-content>
          </v-list-tile>
          <!-- 二级菜单 -->
          <v-list-tile v-for="subItem in item.items" :key="subItem.title" :to="item.path + subItem.path">
            <v-list-tile-action>
              <v-icon>{{ subItem.action }}</v-icon>
            </v-list-tile-action>
            <v-list-tile-content>
              <v-list-tile-title>{{ subItem.title }}</v-list-tile-title>
            </v-list-tile-content>
          </v-list-tile>
        </v-list-group>
      </v-list>
    </v-navigation-drawer>
    <v-toolbar
      app
      dark
      :color="dark ? 'secondary' : 'primary'"
    >
      <!-- 隐藏左侧菜单的按钮-->
      <v-toolbar-side-icon @click.stop="drawer = !drawer"/>
      <!-- 顶部导航标题 -->
      <v-flex xs4></v-flex>
      <v-toolbar-title v-text="title"/>
      <v-spacer/>

      <!-- 调色板 -->
      <v-btn icon @click.stop="dark = !dark">
        <v-icon>invert_colors</v-icon>
      </v-btn>
      <!-- &lt;!&ndash; 顶部导航用户菜单 &ndash;&gt;
       <v-btn icon @click.stop="dark = !dark">
         <v-icon>account_box</v-icon>
       </v-btn>-->
    </v-toolbar>
    <v-content>
      <v-breadcrumbs>
        <v-icon slot="divider">chevron_right</v-icon>
        <v-breadcrumbs-item>{{item1}}</v-breadcrumbs-item>
        <v-breadcrumbs-item>{{item2}}</v-breadcrumbs-item>
      </v-breadcrumbs>
      <div>
        <router-view/>
      </div>
    </v-content>
  </v-app>
</template>

<script>
  import menus from "../menu";
  import Userinfo from "./user/Teacherinfo"

  export default {
    data() {
      return {
        dark: false,// 是否暗黑主题
        drawer: true,// 左侧导航是否隐藏
        miniVariant: false,// 左侧导航是否收起
        title: '学生宿舍管理系统',// 顶部导航条名称,
        menuMap: {},
        userInfo: {
          id: "",
          username: ""
        }
      }
    },
    components: {
      Userinfo
    },
    computed: {
      items() {
        return menus;
      },
      item1() {
        const arr = this.$route.path.split("/");
        return this.menuMap[arr[1]].name;
      },
      item2() {
        const arr = this.$route.path.split("/");
        return this.menuMap[arr[1]][arr[2]];
      }
    },
    name: 'App',
    watch: {},
    created() {
      menus.forEach(m => {
        const p1 = m.path.slice(1);
        this.menuMap[p1] = {name: m.title};
        m.items.forEach(i => {
          this.menuMap[p1][i.path.slice(1)] = i.title;
        })
      });
    },
    mounted() {
      this.$http.get("/auth/verify")
        .then(resp => { // 获取响应结果对象
          if (resp.data.teacher) {
            this.userInfo = Object.deepCopy(resp.data);
          } else {
            this.$router.push("/layoutStudent");
          }
        }).catch(() => {
        this.$router.push("/login");
      });

    }
  }
</script>

<style scoped>
  .box {
    width: 90%;
  }
</style>
