<template>
  <main class="min-h-screen bg-gray-50 flex items-center justify-center px-4">
    <section class="w-full max-w-md bg-white rounded-lg shadow p-6 space-y-4">
      <h1 class="text-xl font-semibold">Profile</h1>
      <form class="space-y-3">
        <div>
          <label class="block text-xs font-medium mb-1" for="name">ชื่อ</label>
          <input id="name" v-model="name" class="w-full border rounded px-2 py-1.5 text-sm" />
        </div>
        <div>
          <label class="block text-xs font-medium mb-1" for="email">Email</label>
          <input id="email" type="email" v-model="email" class="w-full border rounded px-2 py-1.5 text-sm" />
        </div>
        <div>
          <label class="block text-xs font-medium mb-1" for="position">ตำแหน่ง</label>
          <input id="position" v-model="position" class="w-full border rounded px-2 py-1.5 text-sm" />
        </div>
        <div>
          <label class="block text-xs font-medium mb-1" for="avatar">รูปโปรไฟล์</label>
          <input id="avatar" type="file" accept="image/*" @change="onFile" class="w-full text-xs" />
          <div v-if="preview" class="mt-2 flex justify-center">
            <img :src="preview" alt="profile" class="h-16 w-16 rounded-full object-cover border" />
          </div>
        </div>
        <button type="button" class="w-full bg-blue-600 text-white text-sm font-medium py-2 rounded">
          บันทึก 
        </button>
      </form>
    </section>
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const name = ref('')
const email = ref('')
const position = ref('')
const preview = ref<string | null>(null)

const onFile = (e: Event) => {
  const file = (e.target as HTMLInputElement).files?.[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = () => {
    preview.value = reader.result as string
  }
  reader.readAsDataURL(file)
}
</script>
