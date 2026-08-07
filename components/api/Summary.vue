<script setup lang="ts">
import { sleep } from '#shared/utils/helpers';
import { request } from '#shared/utils/request';
import CodeSegment from '~/components/api/CodeSegment.vue';
import toastFactory from '~/composables/toast';
import type { GetAuthKeyResult } from '~/types/types';

const toast = toastFactory();

// 会员/限速层配置（默认关闭；开启后才展示会员授权与限速说明）
const membership = useRuntimeConfig().public.membership;

const loading = ref(false);
const authKey = ref('');
async function getAuthKey() {
  loading.value = true;
  try {
    await sleep(1000);
    const resp = await request<GetAuthKeyResult>(`/api/public/v1/authkey`);
    if (resp.code === 0) {
      authKey.value = resp.data;
    } else {
      toast.error('获取密钥失败', resp.msg);
    }
  } finally {
    loading.value = false;
  }
}

const tiers = [
  { name: '查询类接口', desc: '搜索公众号 / 按链接搜索 / 文章列表', guest: '5 次/分钟', member: '100 次/分钟' },
  { name: '获取文章内容', desc: '文章下载 / 导出', guest: '1 次/分钟', member: '60 次/分钟' },
];

// 自助查询会员令牌详情
interface MemberInfo {
  status: 'valid' | 'expired' | 'notfound';
  expiresAt?: number;
  createdAt?: number | null;
  remainingDays?: number;
}

const memberToken = ref('');
const memberQuerying = ref(false);
const memberInfo = ref<MemberInfo | null>(null);

async function queryMemberInfo() {
  const tk = memberToken.value.trim();
  if (!tk) {
    toast.error('请输入令牌');
    return;
  }
  memberQuerying.value = true;
  memberInfo.value = null;
  try {
    memberInfo.value = await request<MemberInfo>('/api/public/v1/memberinfo', {
      headers: { 'X-Api-Token': tk },
    });
  } catch (e: any) {
    toast.error('查询失败', e?.data?.statusMessage || e?.data?.message || '请稍后重试');
  } finally {
    memberQuerying.value = false;
  }
}

function fmtDate(ms?: number | null) {
  // 固定东八区（北京时间），不随访问者本地时区变化
  return ms ? new Date(ms).toLocaleString('zh-CN', { timeZone: 'Asia/Shanghai', hour12: false }) : '-';
}
</script>

<template>
  <div class="space-y-8">
    <!-- 简介 -->
    <p class="text-base leading-relaxed text-gray-600 dark:text-gray-300">
      为了方便第三方开发人员进行个性化定制，本网站将其主要功能（包括但不限于公众号查询、历史文章列表查询、文章下载等）提供
      API 以供接入。
    </p>

    <!-- 下线提示（仅公开托管站显示；fork 私有部署 API 正常可用）-->
    <div
      v-if="membership.enabled"
      class="flex gap-3 rounded-xl border border-rose-200 bg-rose-50 p-4 dark:border-rose-900/50 dark:bg-rose-950/30"
    >
      <UIcon name="i-lucide:alert-triangle" class="mt-0.5 size-5 shrink-0 text-rose-500" />
      <p class="text-sm text-rose-800 dark:text-rose-200">
        <span class="font-semibold">本站 API 已下线。</span>
        公开 API 不再对外开放，已停止受理新的会员开通与技术支持。已持有密钥 /
        令牌的用户仍可继续调用现有接口，但不再保证可用性；有稳定调用需求请自行私有部署。以下文档与调试工具仅供存量用户参考。
      </p>
    </div>

    <!-- 密钥 -->
    <section class="rounded-xl border border-gray-200 p-5 dark:border-gray-800">
      <h3 class="mb-4 flex items-center gap-2 text-xl font-semibold">
        <UIcon name="i-lucide:key-square" class="text-blue-500" />
        <span>密钥</span>
        <UBadge color="gray" variant="soft" size="xs" class="ml-1">登录集成 · 免费</UBadge>
      </h3>
      <ol class="list-decimal space-y-2 pl-5 text-sm leading-relaxed marker:text-gray-400">
        <li>
          查询类接口需携带密钥调用（下载接口无需）。密钥可通过两种方式传输：
          <div class="mt-1 space-y-0.5 text-gray-600 dark:text-gray-400">
            <p>a. 请求头 <code class="rounded bg-gray-100 px-1 font-mono text-rose-500 dark:bg-gray-800">X-Auth-Key</code></p>
            <p>
              b. name 为 <code class="rounded bg-gray-100 px-1 font-mono text-rose-500 dark:bg-gray-800">auth-key</code> 的
              Cookie
            </p>
          </div>
        </li>
        <li>密钥与本网站登录集成，扫码登录后会自动刷新。</li>
        <li>网站登录信息失效时，对应密钥同时失效。</li>
      </ol>
      <UButton class="mt-4" color="blue" :loading="loading" @click="getAuthKey">
        查询 API 密钥 (确保当前登录信息有效)
      </UButton>
      <div v-if="authKey" class="mt-4">
        <p class="mb-2 text-sm text-gray-500">当前密钥：</p>
        <CodeSegment :code="authKey" lang="text" class="max-w-xl" />
      </div>
    </section>

    <!-- 会员授权（已停止开通，仅存量令牌，仅开启会员/限速时显示）-->
    <section v-if="membership.enabled" class="rounded-xl border border-gray-200 p-5 dark:border-gray-800">
      <h3 class="mb-4 flex items-center gap-2 text-xl font-semibold">
        <UIcon name="i-lucide:crown" class="text-amber-500" />
        <span>会员授权</span>
        <UBadge color="gray" variant="soft" size="xs" class="ml-1">已停止开通</UBadge>
      </h3>

      <div class="space-y-4">
        <!-- 频率对比表 -->
        <div class="overflow-hidden rounded-xl border border-gray-200 dark:border-gray-800">
          <table class="w-full text-sm">
            <thead class="bg-gray-50 text-xs text-gray-500 dark:bg-gray-900">
              <tr>
                <th class="px-4 py-2.5 text-left font-medium">接口</th>
                <th class="px-4 py-2.5 text-center font-medium">游客 · 免费</th>
                <th class="px-4 py-2.5 text-center font-medium text-amber-600 dark:text-amber-400">会员</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-100 dark:divide-gray-800">
              <tr v-for="t in tiers" :key="t.name">
                <td class="px-4 py-3">
                  <p class="font-medium">{{ t.name }}</p>
                  <p class="mt-0.5 text-xs text-gray-400">{{ t.desc }}</p>
                </td>
                <td class="px-4 py-3 text-center text-gray-600 dark:text-gray-300">{{ t.guest }}</td>
                <td class="px-4 py-3 text-center font-semibold text-amber-700 dark:text-amber-300">{{ t.member }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- 用法 -->
        <div class="space-y-1.5 text-sm">
          <p>
            会员调用时，在请求头携带专属令牌：<code class="rounded bg-gray-100 px-1 font-mono text-rose-500 dark:bg-gray-800"
              >X-Api-Token: 你的令牌</code
            >
          </p>
          <p class="text-gray-500">令牌到期后自动降级为游客速率。查询类接口仍需登录密钥（X-Auth-Key）。</p>
        </div>
      </div>

      <!-- 查询我的令牌 -->
      <div class="mt-5 rounded-xl border border-gray-200 p-4 dark:border-gray-800">
        <p class="mb-2 flex items-center gap-2 text-sm font-medium">
          <UIcon name="i-lucide:search" class="size-4 text-gray-400" />
          查询我的令牌
        </p>
        <div class="flex flex-col gap-2 sm:flex-row">
          <UInput
            v-model="memberToken"
            placeholder="粘贴你的会员令牌 (X-Api-Token)"
            class="flex-1"
            @keyup.enter="queryMemberInfo"
          />
          <UButton color="amber" :loading="memberQuerying" @click="queryMemberInfo">查询</UButton>
        </div>

        <div v-if="memberInfo" class="mt-3 text-sm">
          <template v-if="memberInfo.status === 'valid'">
            <p class="flex items-center gap-1.5 font-medium text-green-600 dark:text-green-400">
              <UIcon name="i-lucide:badge-check" class="size-4" />会员有效
            </p>
            <div class="mt-1.5 space-y-0.5 text-gray-600 dark:text-gray-300">
              <p>剩余天数：<span class="font-medium">{{ memberInfo.remainingDays }}</span> 天</p>
              <p>到期时间：{{ fmtDate(memberInfo.expiresAt) }} <span class="text-xs text-gray-400">北京时间</span></p>
              <p v-if="memberInfo.createdAt">开通时间：{{ fmtDate(memberInfo.createdAt) }}</p>
            </div>
          </template>
          <template v-else-if="memberInfo.status === 'expired'">
            <p class="flex items-center gap-1.5 font-medium text-rose-500">
              <UIcon name="i-lucide:alert-triangle" class="size-4" />会员已过期
            </p>
            <div class="mt-1.5 space-y-0.5 text-gray-600 dark:text-gray-300">
              <p>到期时间：{{ fmtDate(memberInfo.expiresAt) }} <span class="text-xs text-gray-400">北京时间</span></p>
              <p class="text-rose-500">请续费后恢复会员额度。</p>
            </div>
          </template>
          <template v-else>
            <p class="flex items-center gap-1.5 font-medium text-gray-500">
              <UIcon name="i-lucide:x-circle" class="size-4" />未找到该令牌（无效或已过期清理）
            </p>
          </template>
        </div>
      </div>
    </section>
  </div>
</template>
