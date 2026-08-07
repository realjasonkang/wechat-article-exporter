<template>
  <div class="flex flex-col h-screen">
    <!-- 全局提示（横跨侧边栏与顶部操作栏之上）-->
    <SiteNotice />

    <div class="flex flex-1 min-h-0">
      <!-- 左侧边栏 -->
      <SideBar />

      <div class="flex flex-col flex-1 overflow-hidden h-full">
        <!-- 顶部操作栏 -->
        <div
          class="flex h-[60px] flex-shrink-0 items-center justify-between border-b border-slate-6 dark:border-slate-600 px-6"
        >
          <div id="title"></div>
          <GlobalActions />
        </div>

        <!-- 页面容器 -->
        <div class="flex-1 overflow-hidden">
          <NuxtPage />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import GlobalActions from '~/components/dashboard/Actions.vue';
import SideBar from '~/components/dashboard/SideBar.vue';
import SiteNotice from '~/components/global/SiteNotice.vue';
</script>
