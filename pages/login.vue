<template>
  <main class="min-h-screen flex items-center justify-center bg-gray-100">
    <section class="w-full max-w-md bg-white shadow-md rounded-lg p-6 space-y-4">
      <h1 class="text-2xl font-semibold text-center">Login</h1>

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

        <p v-if="error" class="text-sm text-red-600">{{ error }}</p>
        <p v-if="success" class="text-sm text-green-600">{{ success }}</p>

        <button
          type="submit"
          class="w-full bg-blue-600 hover:bg-blue-700 text-white text-sm font-medium py-2 rounded disabled:opacity-60"
          :disabled="loading"
        >
          <span v-if="loading">Logging in...</span>
          <span v-else>Login</span>
        </button>
      </form>

      <p class="text-xs text-center text-gray-600">
        Don't have an account?
        <NuxtLink to="/register" class="text-blue-600 hover:underline">Register</NuxtLink>
      </p>
    </section>
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

const form = ref({
  username: '',
  password: ''
})

const loading = ref(false)
const error = ref('')
const success = ref('')

const onSubmit = async () => {
  error.value = ''
  success.value = ''
  loading.value = true

  try {
    const data: any = await $fetch('http://localhost:3000/api/auth/login', {
      method: 'POST',
      body: form.value
    })

    // คาดหวังว่า backend ส่ง { token, role }
    if (data?.token) {
      localStorage.setItem('token', data.token)
    }
    if (data?.role) {
      localStorage.setItem('role', data.role)
    }

    success.value = 'Login success'

    // redirect ตาม role
    const role = data?.role
    if (role === 'admin') {
      router.push('/dashboard/admin')
    } else if (role === 'assessor') {
      router.push('/dashboard/assessor')
    } else {
      router.push('/dashboard/user')
    }
  } catch (err: any) {
    error.value = err?.data?.message || 'Login failed'
  } finally {
    loading.value = false
  }
}
</script>
