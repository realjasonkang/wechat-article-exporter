<script setup lang="ts">
// 域名到期时间（东八区），到期后网站将无法访问
const SITE_EXPIRE_DATE = '2026-10-30';

const daysLeft = computed(() => {
  const diff = new Date(`${SITE_EXPIRE_DATE}T00:00:00+08:00`).getTime() - Date.now();
  return Math.max(0, Math.ceil(diff / 86_400_000));
});

/**
 * 随剩余天数升级提示强度，避免一上来就拉满导致用户脱敏：
 * - 30 天以上：琥珀色中性提醒，可关闭
 * - 8 ~ 30 天：玫红警告，可关闭
 * - 7 天以内：加深底色 + 实心倒计时，不可关闭
 *
 * class 全部写成字面量，便于 Tailwind 静态扫描（勿改为拼接字符串）。
 */
const tone = computed(() => {
  if (daysLeft.value <= 7) {
    return {
      dismissible: false,
      bar: 'border-rose-300 bg-rose-100 dark:border-rose-800 dark:bg-rose-950/60',
      icon: 'i-lucide:alert-triangle',
      iconClass: 'text-rose-600 dark:text-rose-400',
      title: 'text-rose-900 dark:text-rose-100',
      body: 'text-rose-800 dark:text-rose-200',
      pill: 'bg-rose-600 text-white text-sm font-bold dark:bg-rose-500',
      cta: 'bg-rose-600 text-white hover:bg-rose-700 dark:bg-rose-500 dark:hover:bg-rose-400',
    };
  }
  if (daysLeft.value <= 30) {
    return {
      dismissible: true,
      bar: 'border-rose-200 bg-rose-50 dark:border-rose-900/50 dark:bg-rose-950/30',
      icon: 'i-lucide:alert-triangle',
      iconClass: 'text-rose-500',
      title: 'text-rose-900 dark:text-rose-100',
      body: 'text-rose-800 dark:text-rose-200',
      pill: 'bg-rose-100 text-rose-900 text-xs font-semibold dark:bg-rose-900/50 dark:text-rose-100',
      cta: 'bg-rose-600 text-white hover:bg-rose-700 dark:bg-rose-500 dark:hover:bg-rose-400',
    };
  }
  return {
    dismissible: true,
    bar: 'border-amber-200 bg-amber-50 dark:border-amber-900/50 dark:bg-amber-950/30',
    icon: 'i-lucide:info',
    iconClass: 'text-amber-500',
    title: 'text-amber-900 dark:text-amber-100',
    body: 'text-amber-800 dark:text-amber-200',
    pill: 'bg-amber-100 text-amber-900 text-xs font-semibold dark:bg-amber-900/50 dark:text-amber-100',
    cta: 'bg-amber-600 text-white hover:bg-amber-700 dark:bg-amber-500 dark:hover:bg-amber-400',
  };
});

// 仅本次会话生效：关掉后当前标签页内不再弹出，重开标签页会再次提醒。
// 域名到期是硬期限，不做永久免打扰。换新公告时更换该 key 即可。
const dismissed = useSessionStorage('notice:site-shutdown-2026-10-30', false);
const visible = computed(() => !(tone.value.dismissible && dismissed.value));
</script>

<template>
  <div v-if="visible" class="flex flex-shrink-0 items-center gap-4 border-b px-6 py-3" :class="tone.bar">
    <UIcon :name="tone.icon" class="size-5 shrink-0" :class="tone.iconClass" />

    <!-- 正文限宽 max-w-3xl：避免宽屏下单行拉到 1600px，行长过长影响扫读 -->
    <div class="min-w-0 flex-1">
      <p class="max-w-3xl text-sm font-semibold leading-snug" :class="tone.title">
        本站域名将于 {{ SITE_EXPIRE_DATE }} 到期
      </p>
      <p class="max-w-3xl text-sm leading-snug" :class="tone.body">
        届时网站将无法访问，请在此之前导出你需要保留的文章数据。
      </p>
    </div>

    <span
      v-if="daysLeft > 0"
      class="shrink-0 whitespace-nowrap rounded-full px-2.5 py-1 leading-none tabular-nums"
      :class="tone.pill"
    >
      剩余 {{ daysLeft }} 天
    </span>

    <NuxtLink
      to="/dashboard/article"
      class="inline-flex shrink-0 items-center gap-1.5 rounded-md px-3 py-1.5 text-sm font-medium shadow-sm transition-colors"
      :class="tone.cta"
    >
      <UIcon name="i-lucide:arrow-up-from-line" class="size-4" />
      去导出文章
    </NuxtLink>

    <!-- 明确关闭的作用域，避免用户误以为是永久免打扰 -->
    <UTooltip v-if="tone.dismissible" text="本次浏览不再提示（重开标签页会再次提醒）" class="shrink-0">
      <UButton
        square
        variant="link"
        color="gray"
        class="-mr-2"
        aria-label="关闭提示，本次浏览不再显示"
        @click="dismissed = true"
      >
        <UIcon name="i-lucide:x" class="size-5" />
      </UButton>
    </UTooltip>
  </div>
</template>
