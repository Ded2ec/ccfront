<template>
  <section class="space-y-4">
    <h1 class="text-2xl font-semibold">Admin Dashboard</h1>
    <p class="text-sm text-gray-600">สำหรับผู้ใช้ที่มี role: <span class="font-mono">admin</span></p>

    <div class="grid gap-4 md:grid-cols-3">
      <div class="rounded-lg border bg-white p-4 shadow-sm">
        <p class="text-xs uppercase tracking-wide text-gray-500">Overview</p>
        <p class="mt-2 text-lg font-semibold">ระบบจัดการทั้งหมด</p>
      </div>
      <div class="rounded-lg border bg-white p-4 shadow-sm">
        <p class="text-xs uppercase tracking-wide text-gray-500">Users</p>
        <p class="mt-2 text-sm text-gray-700">คุณสามารถต่อยอดมาแสดงจำนวนผู้ใช้จาก backend</p>
      </div>
      <div class="rounded-lg border bg-white p-4 shadow-sm">
        <p class="text-xs uppercase tracking-wide text-gray-500">Settings</p>
        <p class="mt-2 text-sm text-gray-700">ส่วนสำหรับจัดการการตั้งค่าระบบ</p>
      </div>
    </div>
  </section>
</template>
