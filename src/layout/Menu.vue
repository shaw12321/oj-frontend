<template>
  <template v-for="item in menuList" :key="item.path">
    <!--没有子路由-->
    <template v-if="!item.children">
      <el-menu-item
        :index="item.path"
        v-if="!item.meta.hidden"
        @click="goRoute(item.path)"
      >
        <template #title>
          <svg-icon :name="item.meta.icon" size="19px" />
          <span>{{ item.meta.title }}</span>
        </template>
      </el-menu-item>
    </template>
    <!-- 有子路由但是只有一个子路由 -->
    <template v-if="item.children && item.children.length == 1">
      <el-menu-item
        :index="item.children[0].path"
        v-if="!item.children[0].meta.hidden"
        @click="goRoute"
      >
        <el-icon>
          <component :is="item.children[0].meta.icon"></component>
        </el-icon>
        <template #title>
          <span>{{ item.children[0].meta.title }}</span>
        </template>
      </el-menu-item>
    </template>
    <!-- 有子路由且个数大于一个1 -->
    <el-sub-menu
      :index="item.path"
      v-if="item.children && item.children.length > 1"
    >
      <template #title>
        <el-icon>
          <component :is="item.meta.icon"></component>
        </el-icon>
        <div>{{ item.meta.title }}</div>
      </template>
      <Menu :menuList="item.children"></Menu>
    </el-sub-menu>
  </template>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router'
//获取父组件传递过来的全部路由数组
defineProps(['menuList'])
//获取路由器对象
let $router = useRouter()
//点击菜单的回调
const goRoute = (path: any) => {
  //路由跳转
  $router.push(path)
}
</script>
<script lang="ts"></script>

<style scoped>
.is-active {
  background-color: #eff0ff;
}
.el-menu-item {
  border-radius: 8px;
  height: 50px;
  margin: 10px 0;
  font-size: 15px;
  display: flex;
  align-items: center;
  gap: 10px;
  letter-spacing: 1px;
}
</style>
