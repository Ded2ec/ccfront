<template>
  <main class="min-h-screen flex items-center justify-center bg-slate-950 text-slate-50 px-4">
    <section class="max-w-md w-full text-center space-y-4">
      <p class="text-xs uppercase tracking-[0.3em] text-sky-400/80">Error</p>
      <h1 class="text-3xl font-semibold">
        {{ statusCode === 404 ? 'ไม่พบหน้า' : 'เกิดข้อผิดพลาดภายในระบบ' }}
      </h1>
      <p class="text-sm text-slate-300" v-if="message">
        {{ message }}
      </p>
      <div class="flex items-center justify-center gap-3 pt-2">
        <NuxtLink
          to="/"
          class="px-4 py-2 rounded-full bg-sky-500 text-sm font-medium text-slate-950 hover:bg-sky-400"
        >
          กลับหน้าแรก
        </NuxtLink>
        <button
          type="button"
          class="px-4 py-2 rounded-full border border-slate-600 text-sm"
          @click="refresh"
        >
          ลองใหม่อีกครั้ง
        </button>
      </div>
    </section>
  </main>
</template>

<script setup lang="ts">
const props = defineProps<{ error: { statusCode?: number; message?: string } }>()
const statusCode = computed(() => props.error.statusCode || 500)
const message = computed(() => props.error.message || '')

const refresh = () => {
  clearError({ redirect: '/' })
}
</script>
