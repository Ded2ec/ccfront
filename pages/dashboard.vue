<template>
  <section class="space-y-6">
    <header class="flex items-center justify-between">
      <div>
        <h1 class="text-2xl font-semibold">Dashboard</h1>
        <p class="text-sm text-gray-600">สรุปข้อมูลผู้ใช้หลังจาก Login</p>
      </div>
    </header>

    <div class="grid gap-4 md:grid-cols-3">
      <div class="rounded-lg border bg-white p-4 shadow-sm">
        <p class="text-xs uppercase tracking-wide text-gray-500">Status</p>
        <p class="mt-2 text-lg font-semibold text-emerald-600">Online</p>
      </div>
      <div class="rounded-lg border bg-white p-4 shadow-sm">
        <p class="text-xs uppercase tracking-wide text-gray-500">Role</p>
        <p class="mt-2 text-lg font-semibold">user</p>
      </div>
      <div class="rounded-lg border bg-white p-4 shadow-sm">
        <p class="text-xs uppercase tracking-wide text-gray-500">Actions</p>
        <p class="mt-2 text-sm text-gray-700">คุณสามารถต่อยอดแสดงข้อมูลจริงจาก backend ได้ที่หน้านี้</p>
      </div>
    </div>
  </section>
</template>
