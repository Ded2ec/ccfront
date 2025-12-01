<template>
  <section class="space-y-4">
    <nav class="flex items-center justify-between border-b pb-2">
      <div class="flex items-center gap-4 text-sm">
        <NuxtLink to="/dashboard" class="font-semibold">Dashboard</NuxtLink>
        <button type="button" class="text-gray-600 hover:text-gray-900" @click="showProfile = true">Profile</button>
      </div>
      <button
        type="button"
        class="inline-flex items-center rounded-md bg-red-500 px-3 py-1.5 text-xs font-medium text-white shadow-sm hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-400 focus:ring-offset-1"
        @click="logout"
      >
        ลงชื่อออก
      </button>
    </nav>

    <div class="rounded-lg border bg-white p-4 shadow-sm flex items-center gap-4">
      <div class="h-12 w-12 rounded-full bg-sky-500 text-white flex items-center justify-center text-lg font-semibold overflow-hidden">
        <img
          v-if="preview"
          :src="preview as string"
          alt="avatar"
          class="h-full w-full object-cover"
        />
        <span v-else>
          {{ name.charAt(0).toUpperCase() }}
        </span>
      </div>
      <div class="text-sm">
        <p class="font-semibold">{{ name }}</p>
        <p class="text-xs text-gray-500">Role: {{ role || 'unknown' }}</p>
      </div>
    </div>

    <div class="grid gap-4 md:grid-cols-3">
      <div class="rounded-lg border bg-white p-4 shadow-sm">
        <p class="text-xs uppercase tracking-wide text-gray-500">Status</p>
        <p class="mt-2 text-lg font-semibold text-emerald-600">Online</p>
      </div>
      <div class="rounded-lg border bg-white p-4 shadow-sm md:col-span-2">
        <p class="text-xs uppercase tracking-wide text-gray-500">Actions</p>
        <p class="mt-2 text-sm text-gray-700">คุณสามารถต่อยอดแสดงข้อมูลจริงจาก backend ได้ที่หน้านี้</p>
      </div>
    </div>

    <div v-if="showProfile" class="fixed inset-0 z-20 flex items-center justify-center bg-black/40">
      <section class="w-full max-w-md bg-white rounded-lg shadow p-6 space-y-4">
        <header class="flex items-center justify-between">
          <h2 class="text-lg font-semibold">Profile</h2>
          <button type="button" class="text-sm text-gray-500 hover:text-gray-800" @click="showProfile = false">✕</button>
        </header>
        <form class="space-y-3">
          <div>
            <label class="block text-xs font-medium mb-1" for="pf-name">ชื่อ</label>
            <input id="pf-name" v-model="name" class="w-full border rounded px-2 py-1.5 text-sm" />
          </div>
          <div>
            <label class="block text-xs font-medium mb-1" for="pf-email">Email</label>
            <input id="pf-email" type="email" v-model="email" class="w-full border rounded px-2 py-1.5 text-sm" />
          </div>
          <div>
            <label class="block text-xs font-medium mb-1" for="pf-pos">ตำแหน่ง</label>
            <input id="pf-pos" v-model="position" class="w-full border rounded px-2 py-1.5 text-sm" />
          </div>
          <div>
            <label class="block text-xs font-medium mb-1" for="pf-avatar">รูปโปรไฟล์</label>
            <input id="pf-avatar" type="file" accept="image/*" @change="onFile" class="w-full text-xs" />
            <div v-if="preview" class="mt-2 flex justify-center">
              <img :src="preview" alt="profile" class="h-16 w-16 rounded-full object-cover border" />
            </div>
          </div>
          <button
            type="button"
            class="w-full bg-blue-600 text-white text-sm font-medium py-2 rounded disabled:opacity-60"
            :disabled="saving"
            @click="saveProfile"
          >
            {{ saving ? 'กำลังบันทึก...' : 'บันทึก' }}
          </button>
          <p v-if="profileMessage" class="text-xs text-center text-emerald-600">
            {{ profileMessage }}
          </p>
        </form>
      </section>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'

definePageMeta({
  middleware: 'auth'
})

const router = useRouter()
const name = ref('ผู้ใช้งาน')
const role = ref<string | null>(null)
const showProfile = ref(false)
const email = ref('')
const position = ref('')
const preview = ref<string | null>(null)
const saving = ref(false)
const profileMessage = ref('')

const apiBase = 'http://localhost:3333'

const fetchProfile = async () => {
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return
  try {
    const data: any = await $fetch(apiBase + '/api/profile', {
      headers: { Authorization: `Bearer ${token}` }
    })
    name.value = data.full_name || name.value
    email.value = data.email || ''
    position.value = data.position || ''
    preview.value = data.avatar_url || null
  } catch (e) {
    console.error(e)
  }
}

onMounted(() => {
  if (typeof window !== 'undefined') {
    name.value = localStorage.getItem('username') || 'ผู้ใช้งาน'
    role.value = localStorage.getItem('role')
  }
  fetchProfile()
})

const logout = () => {
  if (typeof window !== 'undefined') {
    localStorage.clear()
    sessionStorage.clear()
  }
  router.push('/login')
}

const onFile = (e: Event) => {
  const file = (e.target as HTMLInputElement).files?.[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = () => {
    preview.value = reader.result as string
  }
  reader.readAsDataURL(file)
}

const saveProfile = async () => {
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return
  saving.value = true
  profileMessage.value = ''
  try {
    await $fetch(apiBase + '/api/profile', {
      method: 'PUT',
      headers: { Authorization: `Bearer ${token}` },
      body: {
        full_name: name.value,
        email: email.value,
        position: position.value,
        avatar_url: preview.value
      }
    })
    profileMessage.value = 'บันทึกข้อมูลสำเร็จ'
  } catch (e: any) {
    profileMessage.value = e?.data?.message || 'บันทึกไม่สำเร็จ'
  } finally {
    saving.value = false
  }
}
</script>
