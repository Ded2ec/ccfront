<template>
  <div class="min-h-screen bg-gradient-to-b from-slate-900 via-slate-950 to-slate-900 text-slate-50 flex items-center justify-center px-4">
    <div class="max-w-2xl w-full text-center">
      <p class="text-xs font-medium tracking-[0.3em] uppercase text-sky-400/80 mb-4">
        Contest Platform
      </p>
      <h1 class="text-3xl sm:text-4xl md:text-5xl font-semibold tracking-tight mb-4">
        ระบบประเมินออนไลน์
      </h1>
      <p class="text-sm sm:text-base text-slate-300/90 mb-8 max-w-xl mx-auto">
       "เข้าสู่ระบบเพื่อจัดการข้อมูล หรือสมัครสมาชิกเพื่อเริ่มต้น"
  (หรือใช้เป็นปุ่มคู่กันไปเลย) <br></br> [ เข้าสู่ระบบ ] [ ลงทะเบียน ]
      </p>

      <div class="flex flex-col sm:flex-row items-center justify-center gap-3 sm:gap-4 mb-6">
        <NuxtLink
          to="/login"
          class="inline-flex items-center justify-center rounded-full bg-sky-500 px-6 py-2.5 text-sm font-medium text-slate-950 shadow-lg shadow-sky-500/30 hover:bg-sky-400 transition-colors w-full sm:w-auto"
        >
          เข้าสู่ระบบ
        </NuxtLink>
        <NuxtLink
          to="/register"
          class="inline-flex items-center justify-center rounded-full border border-slate-600/80 px-6 py-2.5 text-sm font-medium text-slate-100 hover:bg-slate-800/70 transition-colors w-full sm:w-auto"
        >
          สมัครสมาชิก
        </NuxtLink>
      </div>

      <p class="text-[11px] sm:text-xs text-slate-400/80">
        รองรับการใช้งานทั้งบนคอมพิวเตอร์และมือถือ
      </p>
    </div>
  </div>
</template>