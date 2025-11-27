<template>
  <main class="min-h-screen flex items-center justify-center bg-gray-100">
    <section class="w-full max-w-md bg-white shadow-md rounded-lg p-6 space-y-4">
      <h1 class="text-2xl font-semibold text-center">Register</h1>

      <form @submit.prevent="onSubmit" class="space-y-4">
        <div>
          <label class="block text-sm font-medium mb-1" for="username">Username</label>
          <input
            id="username"
            v-model="form.username"
            type="text"
            class="w-full border rounded px-3 py-2 text-sm focus:outline-none focus:ring focus:border-blue-400"
            required
          />
        </div>

        <div>
          <label class="block text-sm font-medium mb-1" for="password">Password</label>
          <input
            id="password"
            v-model="form.password"
            type="password"
            class="w-full border rounded px-3 py-2 text-sm focus:outline-none focus:ring focus:border-blue-400"
            required
          />
        </div>

        <div>
          <label class="block text-sm font-medium mb-1" for="role">Role</label>
          <select
            id="role"
            v-model="form.role"
            class="w-full border rounded px-3 py-2 text-sm focus:outline-none focus:ring focus:border-blue-400"
          >
            <option value="user">user</option>
            <option value="admin">admin</option>
            <option value="assessor">assessor</option>
          </select>
        </div>

        <p v-if="error" class="text-sm text-red-600">{{ error }}</p>
        <p v-if="success" class="text-sm text-green-600">{{ success }}</p>

        <button
          type="submit"
          class="w-full bg-emerald-600 hover:bg-emerald-700 text-white text-sm font-medium py-2 rounded disabled:opacity-60"
          :disabled="loading"
        >
          <span v-if="loading">Registering...</span>
          <span v-else>Register</span>
        </button>
      </form>

      <p class="text-xs text-center text-gray-600">
        Already have an account?
        <NuxtLink to="/login" class="text-blue-600 hover:underline">Login</NuxtLink>
      </p>
    </section>
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const form = ref({
  username: '',
  password: '',
  role: 'user'
})

const loading = ref(false)
const error = ref('')
const success = ref('')

const onSubmit = async () => {
  error.value = ''
  success.value = ''
  loading.value = true

  try {
    await $fetch('http://localhost:3000/api/auth/register', {
      method: 'POST',
      body: form.value
    })

    success.value = 'Register success'
  } catch (err: any) {
    error.value = err?.data?.message || 'Register failed'
  } finally {
    loading.value = false
  }
}
</script>
