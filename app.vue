<template>
  <div class="min-h-screen bg-gray-50 text-gray-900">
    <header class="border-b bg-white/80 backdrop-blur sticky top-0 z-10">
      <nav class="max-w-4xl mx-auto flex items-center justify-between px-4 py-3">
        <h1 class="text-lg font-semibold text-blue-600">Contest App</h1>
        <div class="space-x-3 text-sm">
          <NuxtLink to="/login" class="hover:underline">Login</NuxtLink>
          <NuxtLink to="/register" class="hover:underline">Register</NuxtLink>
        </div>
      </nav>
    </header>

    <main class="max-w-4xl mx-auto px-4 py-8">
      <NuxtPage />
    </main>
  </div>
</template>