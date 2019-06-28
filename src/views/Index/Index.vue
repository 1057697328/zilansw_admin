<!--suppress ALL -->
<template>
  <div>
    <el-container>
      <el-header style="background-color: rgb(238, 241, 246)" :height="'80px'">
        <h1 style="padding:1rem 0 0 0;color:gray">湖南紫澜生物科技有限公司</h1>
        <h4>微信公众号管理系统</h4>
      </el-header>
      <el-container>
        <el-aside style="background-color: rgb(238, 241, 246)" >
          <el-menu :default-openeds="['1']">
            <el-submenu index="1">
              <template slot="title"><i class="el-icon-setting"></i>后台管理</template>
              <el-menu-item-group title="轮播">
                <el-menu-item index="1-1" @click="chooseComponent('bannermanager')"><i class="el-icon-picture-outline"></i>首页轮播管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group title="商品">
                <el-menu-item index="1-2"><i class="el-icon-collection"></i>商品类型管理</el-menu-item>
                <el-menu-item index="1-3"><i class="el-icon-goods"></i>商品管理</el-menu-item>
                <el-menu-item index="1-4"><i class=el-icon-notebook-1></i>库存管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group title="订单">
                <el-menu-item index="3-1"><i class=el-icon-tickets></i>订单管理</el-menu-item>
              </el-menu-item-group>
              <el-menu-item-group title="其它">
                <el-menu-item index="4-1"><i class=el-icon-folder-opened></i>商城公告管理</el-menu-item>
                <el-menu-item index="4-2"><i class="el-icon-user"></i>用户管理</el-menu-item>
                <el-menu-item index="4-3"><i class="el-icon-star-off"></i>商品评价管理</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
          </el-menu>
        </el-aside>
        <el-main>
          <bannermanager v-if="componentName==='bannermanager'"></bannermanager>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script>
  /**
   * 轮播管理
   */
  import BannerManager from '@/components/banner/bannermanager'
  import Bannermanager from "../../components/banner/bannermanager";
  export default {
    name: "Index",
    components: {Bannermanager},
    data(){
      return {
        componentName:''
      }
    },
    methods:{
      /**
       * 展示组件
       * @param name 组件名称
       */
      chooseComponent(name){
        this.componentName=name;
      }
    }
  }
</script>

<style scoped>

</style>
