<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-50 p-6">
    <div class="max-w-7xl mx-auto">
      <div class="bg-white rounded-2xl shadow-xl overflow-hidden">
        <div class="bg-gradient-to-r from-blue-600 to-indigo-600 p-6">
          <h1 class="text-3xl font-bold text-white flex items-center gap-3">
            <span class="inline-flex h-8 w-8 items-center justify-center rounded-lg bg-white/10">
              📊
            </span>
            ระบบประเมินตำแหน่งงาน
          </h1>
          <p class="text-blue-100 mt-2">
            จัดการและดำเนินการประเมินผลการทำงาน
            <span v-if="role" class="ml-2 text-xs px-2 py-0.5 rounded-full bg-white/10">Role: {{ role }}</span>
          </p>
        </div>

        <div class="border-b border-gray-200">
          <div class="flex gap-1 p-4 overflow-x-auto">
            <button
              v-for="tab in tabs"
              :key="tab.id"
              type="button"
              @click="activeTab = tab.id"
              class="flex items-center gap-2 px-4 py-2 rounded-lg font-medium transition-all whitespace-nowrap"
              :class="
                activeTab === tab.id
                  ? 'bg-blue-600 text-white shadow-md'
                  : 'text-gray-600 hover:bg-gray-100'
              "
            >
              <span class="flex h-4 w-4 items-center justify-center text-base">
                {{ tab.icon }}
              </span>
              {{ tab.label }}
            </button>
          </div>
        </div>

        <div class="p-6">
          <section v-if="activeTab === 'periods'" class="space-y-4">
            <div class="flex justify-between items-center mb-6">
              <h2 class="text-2xl font-bold text-gray-800">ระยะเวลาการประเมิน</h2>
              <button
                type="button"
                @click="addPeriod"
                class="flex items-center gap-2 bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors"
              >
                <span class="text-lg">＋</span>
                เพิ่มระยะเวลา
              </button>
            </div>
            <div v-for="period in periods" :key="period.id" class="bg-gray-50 p-4 rounded-lg border border-gray-200">
              <div v-if="editingPeriod === period.id" class="space-y-3">
                <input
                  type="text"
                  v-model="period.name"
                  class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                  placeholder="ชื่อรอบการประเมิน"
                />
                <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                  <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">วันที่เริ่มต้น</label>
                    <input
                      type="date"
                      v-model="period.startDate"
                      class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                    />
                  </div>
                  <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">วันที่สิ้นสุด</label>
                    <input
                      type="date"
                      v-model="period.endDate"
                      class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                    />
                  </div>
                </div>
                <div class="flex flex-wrap gap-2">
                  <button
                    type="button"
                    @click="editingPeriod = null"
                    class="flex items-center gap-2 bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700"
                  >
                    <span class="text-sm">💾</span>
                    บันทึก
                  </button>
                  <button
                    type="button"
                    @click="deletePeriod(period.id)"
                    class="flex items-center gap-2 bg-red-600 text-white px-4 py-2 rounded-lg hover:bg-red-700"
                  >
                    <span class="text-sm">🗑</span>
                    ลบ
                  </button>
                </div>
              </div>
              <div v-else class="flex justify-between items-center">
                <div>
                  <h3 class="font-semibold text-lg text-gray-800">{{ period.name }}</h3>
                  <p class="text-gray-600">
                    {{ period.startDate }} ถึง {{ period.endDate }}
                  </p>
                  <span
                    class="inline-block mt-2 px-3 py-1 rounded-full text-xs font-medium"
                    :class="
                      period.status === 'active'
                        ? 'bg-green-100 text-green-800'
                        : 'bg-gray-100 text-gray-800'
                    "
                  >
                    {{ period.status === 'active' ? 'ใช้งานอยู่' : 'แบบร่าง' }}
                  </span>
                </div>
                <button
                  type="button"
                  @click="editingPeriod = period.id"
                  class="text-blue-600 hover:text-blue-700 text-xl"
                >
                  ✏
                </button>
              </div>
            </div>

            <!-- แสดงหัวข้อการประเมินที่กำหนดช่วงวันที่ และสถานะว่าช่วงประเมินเปิด/ปิดแล้ว -->
            <div v-if="criteriaWithPeriod.length" class="mt-8 space-y-3">
              <h3 class="text-lg font-semibold text-gray-800">หัวข้อที่กำหนดระยะเวลาประเมิน</h3>
              <div
                v-for="c in criteriaWithPeriod"
                :key="c.id"
                class="flex items-center justify-between bg-gray-50 border border-gray-200 rounded-lg px-4 py-3 text-sm"
              >
                <div>
                  <p class="font-medium text-gray-800">{{ c.name }}</p>
                  <p class="text-gray-600 text-xs">
                    {{ c.periodStart }} - {{ c.periodEnd }}
                  </p>
                </div>
                <span
                  class="px-3 py-1 rounded-full text-xs font-medium"
                  :class="
                    c.status === 'upcoming'
                      ? 'bg-gray-100 text-gray-700'
                      : c.status === 'active'
                        ? 'bg-green-100 text-green-800'
                        : 'bg-red-100 text-red-800'
                  "
                >
                  {{
                    c.status === 'upcoming'
                      ? 'ยังไม่ถึงช่วงประเมิน'
                      : c.status === 'active'
                        ? 'สามารถประเมินได้'
                        : 'ผ่านช่วงประเมินแล้ว'
                  }}
                </span>
              </div>
            </div>

            <!-- แสดงหัวข้อการประเมินที่ไม่กำหนดระยะเวลาการประเมิน -->
            <div v-if="criteriaWithoutPeriod.length" class="mt-8 space-y-2">
              <h3 class="text-lg font-semibold text-gray-800">ไม่กำหนดระยะเวลาการประเมิน</h3>
              <div
                v-for="c in criteriaWithoutPeriod"
                :key="c.id"
                class="flex items-center justify-between bg-gray-50 border border-gray-200 rounded-lg px-4 py-2 text-sm"
              >
                <p class="font-medium text-gray-800 truncate" :title="c.name">{{ c.name }}</p>
                <span class="px-3 py-1 rounded-full text-xs font-medium bg-green-100 text-green-800">
                  ไม่กำหนดระยะเวลา
                </span>
              </div>
            </div>
          </section>

          <section v-else-if="activeTab === 'criteria'" class="space-y-4">
            <div class="flex flex-col gap-3 mb-4">
              <div class="flex justify-between items-center">
                <h2 class="text-2xl font-bold text-gray-800">หัวข้อการประเมิน</h2>
                <button
                  type="button"
                  @click="addCriteria"
                  class="flex items-center gap-2 bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700"
                >
                  <span class="text-lg">＋</span>
                  เพิ่มหัวข้อ
                </button>
              </div>
              <label v-if="periods.length" class="inline-flex items-center gap-2 text-sm text-gray-700">
                <input
                  type="checkbox"
                  v-model="showCriteriaPeriods"
                  class="text-blue-600 border-gray-300 rounded"
                />
                กำหนดระยะเวลาการประเมินสำหรับหัวข้อนี้
              </label>
              <div
                v-if="showCriteriaPeriods && periods.length"
                class="bg-gray-50 border border-gray-200 rounded-lg p-3 text-xs text-gray-700"
              >
                <p class="font-medium mb-2">เลือกระยะเวลาการประเมิน</p>
                <div class="flex flex-wrap gap-2">
                  <label
                    v-for="p in periods"
                    :key="p.id"
                    class="inline-flex items-center gap-1 cursor-pointer"
                  >
                    <input
                      type="radio"
                      class="text-blue-600 border-gray-300"
                      :value="p.id"
                      v-model="selectedCriteriaPeriodId"
                    />
                    <span>
                      {{ p.name }}
                      <span class="text-[10px] text-gray-500">({{ p.startDate }} - {{ p.endDate }})</span>
                    </span>
                  </label>
                </div>
              </div>
            </div>
            <div v-for="criterion in criteria" :key="criterion.id" class="bg-gray-50 p-4 rounded-lg border border-gray-200">
              <div v-if="editingCriteria === criterion.id" class="space-y-3">
                <input
                  type="text"
                  v-model="criterion.name"
                  class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                  placeholder="ชื่อหัวข้อ"
                />
                <textarea
                  v-model="criterion.description"
                  class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                  placeholder="รายละเอียด"
                  rows="2"
                />
                <label class="flex items-center gap-2 text-gray-700">
                  <input
                    type="checkbox"
                    v-model="criterion.requiresEvidence"
                    class="w-4 h-4 text-blue-600 rounded focus:ring-2 focus:ring-blue-500"
                  />
                  กำหนดระยะเวลาประเมิน (เลือกวันที่เริ่มต้นและสิ้นสุด)
                </label>
                <div
                  v-if="criterion.requiresEvidence"
                  class="grid grid-cols-1 md:grid-cols-2 gap-3 text-sm text-gray-700"
                >
                  <div>
                    <label class="block mb-1">วันที่เริ่มต้น</label>
                    <input
                      type="date"
                      v-model="criterion.periodStart"
                      class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                    />
                  </div>
                  <div>
                    <label class="block mb-1">วันที่สิ้นสุด</label>
                    <input
                      type="date"
                      v-model="criterion.periodEnd"
                      class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                    />
                  </div>
                </div>

                <div class="mt-4 pl-4 border-l-2 border-blue-300">
                  <div class="flex justify-between items-center mb-2">
                    <h4 class="font-medium text-gray-700">หัวข้อย่อย</h4>
                    <button
                      type="button"
                      @click="addSubItem(criterion.id)"
                      class="text-sm text-blue-600 hover:text-blue-700"
                    >
                      + เพิ่มหัวข้อย่อย
                    </button>
                  </div>
                  <div
                    v-for="subItem in criterion.subItems"
                    :key="subItem.id"
                    class="bg-white p-3 rounded-lg mb-2 flex gap-2"
                  >
                    <input
                      type="text"
                      v-model="subItem.name"
                      class="flex-1 px-2 py-1 border border-gray-300 rounded text-sm"
                      placeholder="ชื่อหัวข้อย่อย"
                    />
                    <select
                      v-model="subItem.type"
                      class="px-2 py-1 border border-gray-300 rounded text-sm"
                    >
                      <option value="score">ให้คะแนน</option>
                      <option value="document">อัพโหลดเอกสาร</option>
                    </select>
                    <button
                      type="button"
                      @click="deleteSubItem(criterion.id, subItem.id)"
                      class="text-red-600 hover:text-red-700 text-sm"
                    >
                      🗑
                    </button>
                  </div>
                </div>

                <div class="flex gap-2 flex-wrap mt-2">
                  <button
                    type="button"
                    @click="saveCriteria(criterion)"
                    class="flex items-center gap-2 bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700"
                  >
                    <span class="text-sm">💾</span>
                    บันทึก
                  </button>
                  <button
                    type="button"
                    @click="deleteCriteria(criterion.id)"
                    class="flex items-center gap-2 bg-red-600 text-white px-4 py-2 rounded-lg hover:bg-red-700"
                  >
                    <span class="text-sm">🗑</span>
                    ลบหัวข้อ
                  </button>
                </div>
              </div>
              <div v-else class="flex justify-between items-start">
                <div class="flex-1">
                  <h3 class="font-semibold text-lg text-gray-800">{{ criterion.name }}</h3>
                  <p class="text-gray-600 mt-1">{{ criterion.description }}</p>
                  <div v-if="criterion.subItems.length" class="mt-3 pl-4 border-l-2 border-gray-300">
                    <p class="text-sm font-medium text-gray-700 mb-1">หัวข้อย่อย:</p>
                    <div
                      v-for="sub in criterion.subItems"
                      :key="sub.id"
                      class="text-sm text-gray-600 flex items-center gap-2"
                    >
                      <span>• {{ sub.name }}</span>
                      <span class="text-xs bg-gray-200 px-2 py-0.5 rounded">
                        {{ sub.type === 'score' ? 'ให้คะแนน' : 'เอกสาร' }}
                      </span>
                    </div>
                  </div>
                </div>
                <button
                  type="button"
                  @click="editingCriteria = criterion.id"
                  class="text-blue-600 hover:text-blue-700 text-xl"
                >
                  ✏
                </button>
              </div>
            </div>
          </section>

          <section v-else-if="activeTab === 'positions'" class="space-y-4">
            <h2 class="text-2xl font-bold text-gray-800 mb-6">ตำแหน่งและบุคลากร</h2>
            <div
              v-for="position in positions"
              :key="position.id"
              class="bg-gray-50 p-4 rounded-lg border border-gray-200"
            >
              <div class="flex items-center justify-between mb-3">
                <div>
                  <h3 class="font-semibold text-lg text-gray-800">{{ position.name }}</h3>
                  <p class="text-sm text-gray-600">กลุ่ม: {{ position.group }}</p>
                </div>
              </div>

              <!-- เลือกหัวข้อการประเมินสำหรับตำแหน่งนี้ -->
              <div class="mb-3">
                <p class="text-sm font-medium text-gray-700">หัวข้อการประเมินสำหรับตำแหน่งนี้:</p>
                <div class="flex flex-wrap gap-2 mt-1 text-xs">
                  <label
                    v-for="c in criteria"
                    :key="c.id"
                    class="inline-flex items-center gap-1 cursor-pointer bg-white px-2 py-1 rounded border border-gray-200"
                  >
                    <input
                      type="checkbox"
                      class="text-blue-600 border-gray-300 rounded"
                      :value="c.id"
                      :checked="positionCriteriaMap[position.name]?.includes(c.id)"
                      @change="togglePositionCriteria(position.name, c.id, ($event.target as HTMLInputElement).checked)"
                    />
                    <span class="truncate max-w-[160px]" :title="c.name">{{ c.name }}</span>
                  </label>
                </div>
              </div>

              <div class="space-y-2">
                <p class="text-sm font-medium text-gray-700">บุคลากร:</p>

                <!-- เลือกทั้งหมดภายในตำแหน่งนี้ -->
                <div
                  v-if="filteredEmployees(position.employees).length"
                  class="flex items-center justify-between mb-1"
                >
                  <label class="inline-flex items-center gap-2 text-xs text-gray-700">
                    <input
                      type="checkbox"
                      class="text-blue-600 border-gray-300 rounded"
                      :checked="areAllEmployeesSelected(position.name, filteredEmployees(position.employees))"
                      @change="
                        toggleSelectAllEmployees(
                          position.name,
                          filteredEmployees(position.employees),
                          ($event.target as HTMLInputElement).checked
                        )
                      "
                    />
                    เลือกบุคลากรทั้งหมด
                  </label>
                </div>

                <div
                  v-for="emp in filteredEmployees(position.employees)"
                  :key="emp.userId"
                  class="flex items-center justify-between bg-white p-3 rounded-lg"
                >
                  <div class="flex items-center gap-2">
                    <input
                      type="checkbox"
                      class="text-blue-600 border-gray-300 rounded"
                      :checked="isEmployeeSelected(position.name, emp.userId)"
                      @change="
                        togglePositionEmployee(
                          position.name,
                          emp.userId,
                          ($event.target as HTMLInputElement).checked
                        )
                      "
                    />
                    <span class="text-gray-800">{{ emp.name }}</span>
                  </div>
                </div>

                <div class="mt-3 flex justify-end">
                  <button
                    type="button"
                    class="inline-flex items-center gap-2 px-4 py-2 rounded-lg text-sm font-medium"
                    :class="
                      canStartForPosition(position)
                        ? 'bg-blue-600 text-white hover:bg-blue-700'
                        : 'bg-gray-200 text-gray-500 cursor-not-allowed'
                    "
                    :disabled="!canStartForPosition(position)"
                    @click="startEvaluationsForPosition(position)"
                  >
                    <span>เริ่มประเมินสำหรับที่เลือก</span>
                  </button>
                </div>
              </div>
            </div>
          </section>

          <section v-else-if="activeTab === 'evaluate'" class="space-y-4">
            <h2 class="text-2xl font-bold text-gray-800 mb-2">ดำเนินการประเมิน</h2>

            <div v-if="visibleEmployees.length === 0" class="text-center py-12 text-gray-500">
              <div class="mx-auto mb-4 h-12 w-12 rounded-full bg-gray-100 flex items-center justify-center text-2xl">
                📄
              </div>
              <p>ยังไม่พบบุคลากรที่สามารถประเมินได้</p>
            </div>

            <div v-else class="space-y-6">
              <!-- กลุ่มยังไม่ประเมิน -->
              <div v-if="pendingEmployees.length" class="space-y-2">
                <h3 class="text-sm font-semibold text-gray-800">ยังไม่ประเมิน</h3>
                <div
                  v-for="emp in pendingEmployees"
                  :key="emp"
                  class="bg-gray-50 rounded-lg border border-gray-200 overflow-hidden"
                >
                  <button
                    type="button"
                    class="w-full flex items-center justify-between px-4 py-3 text-left hover:bg-gray-100"
                    @click="toggleEmployee(emp)"
                  >
                    <div>
                      <p class="font-semibold text-gray-800">{{ emp }}</p>
                      <p class="text-xs text-gray-500">
                        {{ getPendingEvaluationsForEmployee(emp).length }} รายการยังไม่ประเมิน
                        /
                        {{ getCompletedEvaluationsForEmployee(emp).length }} รายการประเมินแล้ว
                      </p>
                    </div>
                    <span class="text-sm text-gray-500">
                      {{ expandedEmployee === emp ? 'ซ่อน' : 'ดูรายละเอียด' }}
                    </span>
                  </button>

                  <div v-if="expandedEmployee === emp" class="border-t border-gray-200 p-4 space-y-4 bg-white">
                    <!-- ส่วนด้านใน: คงโครงเดิม -->
                    <div class="space-y-2">
                      <h3 class="font-medium text-gray-800 text-sm">ยังไม่ประเมิน</h3>
                      <div v-if="getPendingEvaluationsForEmployee(emp).length === 0" class="text-xs text-gray-500">
                        <p>ยังไม่มีรายการที่ยังไม่เสร็จสิ้น</p>
                      </div>
                      <div v-else class="space-y-1 text-xs text-gray-700">
                        <div
                          v-for="ev in getPendingEvaluationsForEmployee(emp)"
                          :key="ev.id"
                          class="flex items-center justify-between bg-gray-50 px-3 py-2 rounded"
                        >
                          <div>
                            <p class="text-sm text-gray-800">รอบ: {{ ev.period || 'ไม่ระบุรอบ' }}</p>
                            <p class="text-xs text-gray-500">สถานะ: {{ ev.status }}</p>
                          </div>
                          <button
                            type="button"
                            class="text-xs bg-blue-600 text-white px-3 py-1 rounded hover:bg-blue-700"
                            @click.stop="selectEvaluation(ev)"
                          >
                            ประเมิน
                          </button>
                        </div>
                      </div>
                    </div>

                    <div class="space-y-2">
                      <h3 class="font-medium text-gray-800 text-sm">ประเมินแล้ว</h3>
                      <div
                        v-if="getCompletedEvaluationsForEmployee(emp).length === 0"
                        class="text-xs text-gray-500"
                      >
                        ยังไม่มีรายการประเมินที่เสร็จสิ้น
                      </div>
                      <div
                        v-else
                        v-for="ev in getCompletedEvaluationsForEmployee(emp)"
                        :key="ev.id"
                        class="flex items-center justify-between bg-gray-50 px-3 py-2 rounded"
                      >
                        <div>
                          <p class="text-sm text-gray-800">รอบ: {{ ev.period || 'ไม่ระบุรอบ' }}</p>
                          <p class="text-xs text-gray-500">สถานะ: {{ ev.status }}</p>
                        </div>
                        <button
                          type="button"
                          class="text-xs bg-blue-600 text-white px-3 py-1 rounded hover:bg-blue-700"
                          @click.stop="selectEvaluation(ev)"
                        >
                          ประเมิน
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              </div>

              <!-- กลุ่มประเมินแล้ว -->
              <div v-if="completedEmployees.length" class="space-y-2">
                <h3 class="text-sm font-semibold text-gray-800">ประเมินแล้ว</h3>
                <div
                  v-for="emp in completedEmployees"
                  :key="emp"
                  class="bg-gray-50 rounded-lg border border-gray-200 overflow-hidden"
                >
                  <button
                    type="button"
                    class="w-full flex items-center justify-between px-4 py-3 text-left hover:bg-gray-100"
                    @click="toggleEmployee(emp)"
                  >
                    <div>
                      <p class="font-semibold text-gray-800">{{ emp }}</p>
                      <p class="text-xs text-gray-500">
                        {{ getPendingEvaluationsForEmployee(emp).length }} รายการยังไม่ประเมิน
                        /
                        {{ getCompletedEvaluationsForEmployee(emp).length }} รายการประเมินแล้ว
                      </p>
                    </div>
                    <span class="text-sm text-gray-500">
                      {{ expandedEmployee === emp ? 'ซ่อน' : 'ดูรายละเอียด' }}
                    </span>
                  </button>

                  <div v-if="expandedEmployee === emp" class="border-t border-gray-200 p-4 space-y-4 bg-white">
                    <!-- ส่วนด้านใน: คงโครงเดิม -->
                    <div class="space-y-2">
                      <h3 class="font-medium text-gray-800 text-sm">ยังไม่ประเมิน</h3>
                      <div v-if="getPendingEvaluationsForEmployee(emp).length === 0" class="text-xs text-gray-500">
                        <p>ยังไม่มีรายการที่ยังไม่เสร็จสิ้น</p>
                      </div>
                      <div v-else class="space-y-1 text-xs text-gray-700">
                        <div
                          v-for="ev in getPendingEvaluationsForEmployee(emp)"
                          :key="ev.id"
                          class="flex items-center justify-between bg-gray-50 px-3 py-2 rounded"
                        >
                          <div>
                            <p class="text-sm text-gray-800">รอบ: {{ ev.period || 'ไม่ระบุรอบ' }}</p>
                            <p class="text-xs text-gray-500">สถานะ: {{ ev.status }}</p>
                          </div>
                          <button
                            type="button"
                            class="text-xs bg-blue-600 text-white px-3 py-1 rounded hover:bg-blue-700"
                            @click.stop="selectEvaluation(ev)"
                          >
                            ประเมิน
                          </button>
                        </div>
                      </div>
                    </div>

                    <div class="space-y-2">
                      <h3 class="font-medium text-gray-800 text-sm">ประเมินแล้ว</h3>
                      <div
                        v-if="getCompletedEvaluationsForEmployee(emp).length === 0"
                        class="text-xs text-gray-500"
                      >
                        ยังไม่มีรายการประเมินที่เสร็จสิ้น
                      </div>
                      <div
                        v-else
                        v-for="ev in getCompletedEvaluationsForEmployee(emp)"
                        :key="ev.id"
                        class="flex items-center justify-between bg-gray-50 px-3 py-2 rounded"
                      >
                        <div>
                          <p class="text-sm text-gray-800">รอบ: {{ ev.period || 'ไม่ระบุรอบ' }}</p>
                          <p class="text-xs text-gray-500">สถานะ: {{ ev.status }}</p>
                        </div>
                        <button
                          type="button"
                          class="text-xs bg-blue-600 text-white px-3 py-1 rounded hover:bg-blue-700"
                          @click.stop="selectEvaluation(ev)"
                        >
                          ประเมิน
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div v-if="selectedEvaluation" class="mt-6 bg-gray-50 p-6 rounded-lg border border-gray-200">
              <div class="mb-4">
                <h3 class="font-semibold text-xl text-gray-800">{{ selectedEvaluation.employee }}</h3>
                <p class="text-gray-600">ตำแหน่ง: {{ selectedEvaluation.position }}</p>
                <p class="text-sm text-gray-500">รอบ: {{ selectedEvaluation.period || 'ไม่ระบุรอบ' }}</p>
              </div>

              <div class="space-y-6">
                <div
                  v-for="criterion in visibleCriteriaForSelected"
                  :key="criterion.id"
                  class="bg-white p-4 rounded-lg"
                >
                  <h4 class="font-medium text-gray-800 mb-3">{{ criterion.name }}</h4>
                  <div
                    v-for="subItem in criterion.subItems"
                    :key="subItem.id"
                    class="mb-4 pb-4 border-b border-gray-200 last:border-0"
                  >
                    <p class="text-sm text-gray-700 mb-2">{{ subItem.name }}</p>
                    <div v-if="subItem.type === 'score'" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                      <div>
                        <label class="block text-xs text-gray-600 mb-1">ประเมินตนเอง (1-5)</label>
                        <input
                          type="number"
                          min="1"
                          max="5"
                          class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                          placeholder="คะแนน"
                          :value="getScoreValue('self', criterion.id, subItem.id)"
                          @input="
                            updateScoreValue(
                              'self',
                              criterion.id,
                              subItem.id,
                              Number(($event.target as HTMLInputElement).value || 0)
                            )
                          "
                        />
                      </div>
                      <div>
                        <label class="block text-xs text-gray-600 mb-1">ผู้บังคับบัญชา (1-5)</label>
                        <input
                          type="number"
                          min="1"
                          max="5"
                          class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
                          placeholder="คะแนน"
                          :value="getScoreValue('manager', criterion.id, subItem.id)"
                          @input="
                            updateScoreValue(
                              'manager',
                              criterion.id,
                              subItem.id,
                              Number(($event.target as HTMLInputElement).value || 0)
                            )
                          "
                        />
                      </div>
                    </div>
                    <div v-else class="flex items-center gap-2">
                      <button
                        type="button"
                        class="flex items-center gap-2 bg-gray-100 hover:bg-gray-200 px-4 py-2 rounded-lg text-sm"
                      >
                        อัพโหลดเอกสาร
                      </button>
                    </div>
                  </div>

                  <!-- เดิมใช้แสดง/อัพโหลดหลักฐาน ตอนนี้เปลี่ยนไปกำหนดช่วงวันที่แล้ว จึงตัดส่วนนี้ออก -->
                </div>
              </div>

              <div class="mt-6 flex gap-3">
                <button
                  type="button"
                  class="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700"
                  @click="saveCurrentScores"
                >
                  บันทึกการประเมิน
                </button>
                <button
                  type="button"
                  class="bg-green-600 text-white px-6 py-2 rounded-lg hover:bg-green-700"
                  @click="submitEvaluation"
                >
                  ส่งการประเมิน (mock)
                </button>
              </div>
            </div>
          </section>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

definePageMeta({
  middleware: 'auth'
})

const activeTab = ref<'periods' | 'criteria' | 'positions' | 'evaluate'>('periods')
const role = ref<string | null>(null)
const username = ref<string | null>(null)

const apiBase = 'http://localhost:3333'

const periods = ref<
  Array<{
    id: number
    name: string
    startDate: string
    endDate: string
    status: string
  }>
>([])

// helper แปลงวันที่จาก API ให้เป็นรูปแบบที่ input type="date" รองรับ (YYYY-MM-DD)
// ใช้ local date (ปี-เดือน-วัน) เพื่อเลี่ยงปัญหา timezone ทำให้วันที่คลาดเคลื่อน
const toInputDate = (value: any): string | null => {
  if (!value) return null
  try {
    const str = String(value)
    // ถ้าเป็นรูปแบบ YYYY-MM-DD อยู่แล้ว ให้คืนค่าตรง ๆ
    if (/^\d{4}-\d{2}-\d{2}$/.test(str)) return str

    const d = new Date(value)
    if (Number.isNaN(d.getTime())) return null

    const year = d.getFullYear()
    const month = String(d.getMonth() + 1).padStart(2, '0')
    const day = String(d.getDate()).padStart(2, '0')
    return `${year}-${month}-${day}`
  } catch {
    return null
  }
}

const saveCurrentScores = async () => {
  if (!selectedEvaluation.value) return
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return

  const evalId = selectedEvaluation.value.id
  const byCriteria = currentScores.value[evalId] || {}

  const payload: Array<{
    criteriaId: number
    subitemId: number
    selfScore: number | null
    managerScore: number | null
    note?: string | null
  }> = []

  Object.entries(byCriteria).forEach(([critIdStr, subMap]) => {
    const critId = Number(critIdStr)
    Object.entries(subMap as Record<number, { selfScore: number | null; managerScore: number | null; note?: string | null }>).forEach(
      ([subIdStr, val]) => {
        const subId = Number(subIdStr)
        payload.push({
          criteriaId: critId,
          subitemId: subId,
          selfScore: val.selfScore ?? null,
          managerScore: val.managerScore ?? null,
          note: val.note ?? null
        })
      }
    )
  })

  try {
    await $fetch(apiBase + `/api/evaluation/${evalId}/scores`, {
      method: 'POST',
      headers: { Authorization: `Bearer ${token}` },
      body: { scores: payload }
    })
  } catch (err) {
    console.error(err)
  }
}

// ปุ่มในแท็บตำแหน่ง: ใช้สำหรับ "บันทึก/เปิดให้ตำแหน่งนี้สามารถทำแบบประเมินได้"
// เงื่อนไขคือ ตำแหน่งนั้นต้องมีการเลือกหัวข้อประเมินอย่างน้อย 1 หัวข้อ
const canStartForPosition = (position: { name: string }) => {
  const critIds = positionCriteriaMap.value[position.name]
  return !!critIds && critIds.length > 0
}

const startEvaluationsForPosition = async (position: { name: string }) => {
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return

  const criteriaIds = positionCriteriaMap.value[position.name] || []
  const employeeIds = positionSelectedEmployees.value[position.name] || []
  if (!criteriaIds.length || !employeeIds.length) return

  try {
    await $fetch(apiBase + '/api/evaluation/position-config', {
      method: 'POST',
      headers: { Authorization: `Bearer ${token}` },
      body: {
        positionName: position.name,
        employeeIds,
        criteriaIds
      }
    })

    // หลังจากบันทึก mapping แล้ว ให้สร้าง evaluation ให้พนักงานที่เลือกทุกคนทันที
    const pos = positions.value.find(p => p.name === position.name)
    if (pos) {
      pos.employees
        .filter(e => employeeIds.includes(e.userId))
        .forEach(e => {
          // ใช้ startEvaluation เดิม ซึ่งจะผูกกับ period ที่ active ให้เอง
          startEvaluation({ name: position.name }, e.name)
        })
    }
  } catch (err) {
    console.error(err)
  }
}

const criteria = ref<
  Array<{
    id: number
    name: string
    description: string
    requiresEvidence: boolean
    periodStart: string | null
    periodEnd: string | null
    subItems: Array<{ id: number; name: string; type: 'score' | 'document' }>
  }>
>([])

const positions = ref<
  Array<{
    id: number
    name: string
    group: string
    employees: Array<{ userId: number; name: string }>
  }>
>([])

const evaluations = ref<
  Array<{
    id: number
    position: string
    employee: string
    period: string | undefined
    status: string
  }>
>([])

const editingPeriod = ref<number | null>(null)
const editingCriteria = ref<number | null>(null)

// สำหรับแท็บ "ดำเนินการประเมิน"
const expandedEmployee = ref<string | null>(null)
const selectedEvaluation = ref<{
  id: number
  position: string
  employee: string
  period: string | undefined
  status: string
} | null>(null)

// คะแนนต่อ evaluation/criteria/subitem ที่โหลดจาก/บันทึกไปยัง DB
const currentScores = ref<
  Record<
    number,
    Record<
      number,
      Record<
        number,
        {
          selfScore: number | null
          managerScore: number | null
          note?: string | null
        }
      >
    >
  >
>({})

const getScoreValue = (
  type: 'self' | 'manager',
  criteriaId: number,
  subitemId: number
): number | null => {
  if (!selectedEvaluation.value) return null
  const evalId = selectedEvaluation.value.id
  const byCriteria = currentScores.value[evalId]
  if (!byCriteria) return null
  const bySub = byCriteria[criteriaId]
  if (!bySub) return null
  const entry = bySub[subitemId]
  if (!entry) return null
  return type === 'self' ? entry.selfScore ?? null : entry.managerScore ?? null
}

const updateScoreValue = (
  type: 'self' | 'manager',
  criteriaId: number,
  subitemId: number,
  value: number
) => {
  if (!selectedEvaluation.value) return
  const evalId = selectedEvaluation.value.id
  if (!currentScores.value[evalId]) currentScores.value[evalId] = {}
  if (!currentScores.value[evalId][criteriaId]) currentScores.value[evalId][criteriaId] = {}
  if (!currentScores.value[evalId][criteriaId][subitemId]) {
    currentScores.value[evalId][criteriaId][subitemId] = {
      selfScore: null,
      managerScore: null,
      note: null
    }
  }

  if (type === 'self') {
    currentScores.value[evalId][criteriaId][subitemId].selfScore = value
  } else {
    currentScores.value[evalId][criteriaId][subitemId].managerScore = value
  }
}

const tabs = [
  { id: 'periods', label: 'ระยะเวลาการประเมิน', icon: '📅' },
  { id: 'criteria', label: 'หัวข้อการประเมิน', icon: '📄' },
  { id: 'positions', label: 'ตำแหน่งและบุคลากร', icon: '👥' },
  { id: 'evaluate', label: 'ดำเนินการประเมิน', icon: '✏️' }
] as const

// การแสดง/เลือกระยะเวลาภายในแท็บหัวข้อการประเมิน (UI อย่างเดียว)
const showCriteriaPeriods = ref(false)
const selectedCriteriaPeriodId = ref<number | null>(null)

// mapping ตำแหน่ง -> ชุดหัวข้อการประเมินที่ใช้ (id ของ criteria)
const positionCriteriaMap = ref<Record<string, number[]>>({})

// mapping userId -> ชุดหัวข้อการประเมินที่อนุญาตสำหรับพนักงานคนนั้น (มาจาก DB)
const employeeCriteriaMap = ref<Record<number, number[]>>({})

const togglePositionCriteria = (positionName: string, criteriaId: number, checked: boolean) => {
  const current = positionCriteriaMap.value[positionName] || []
  if (checked) {
    if (!current.includes(criteriaId)) {
      positionCriteriaMap.value[positionName] = [...current, criteriaId]
    }
  } else {
    positionCriteriaMap.value[positionName] = current.filter(id => id !== criteriaId)
  }
}

// รายการหัวข้อที่กำหนดช่วงวันที่ เพื่อนำมาแสดงในแท็บระยะเวลาการประเมิน พร้อมสถานะช่วงเวลา
const criteriaWithPeriod = computed(() => {
  const today = new Date()
  const todayYmd = new Date(
    today.getFullYear(),
    today.getMonth(),
    today.getDate()
  ).getTime()

  return criteria.value
    .filter(c => c.periodStart && c.periodEnd)
    .map(c => {
      const start = new Date(c.periodStart as string)
      const end = new Date(c.periodEnd as string)
      const startYmd = new Date(start.getFullYear(), start.getMonth(), start.getDate()).getTime()
      const endYmd = new Date(end.getFullYear(), end.getMonth(), end.getDate()).getTime()

      let status: 'upcoming' | 'active' | 'past' = 'upcoming'
      if (todayYmd < startYmd) status = 'upcoming'
      else if (todayYmd > endYmd) status = 'past'
      else status = 'active'

      return {
        id: c.id,
        name: c.name,
        periodStart: c.periodStart,
        periodEnd: c.periodEnd,
        status
      }
    })
})

// หัวข้อการประเมินที่ไม่ได้กำหนดช่วงวันที่
const criteriaWithoutPeriod = computed(() =>
  criteria.value.filter(c => !c.periodStart && !c.periodEnd)
)

onMounted(async () => {
  if (typeof window !== 'undefined') {
    role.value = localStorage.getItem('role')
    username.value = localStorage.getItem('username')

    const token = localStorage.getItem('token')
    if (!token) return

    try {
      // โหลด evaluations
      const evalData = await $fetch<any[]>(apiBase + '/api/evaluation', {
        headers: { Authorization: `Bearer ${token}` }
      })
      evaluations.value = evalData.map(e => ({
        id: e.id,
        employee: e.employee,
        position: e.position,
        period: e.period,
        status: e.status
      }))

      // โหลด periods จริง
      const periodData = await $fetch<any[]>(apiBase + '/api/evaluation/periods', {
        headers: { Authorization: `Bearer ${token}` }
      })
      periods.value = periodData.map(p => ({
        id: p.id,
        name: p.name,
        startDate: p.startDate,
        endDate: p.endDate,
        status: p.status
      }))

      // โหลด criteria + subItems จริง
      const criteriaData = await $fetch<any[]>(apiBase + '/api/evaluation/criteria', {
        headers: { Authorization: `Bearer ${token}` }
      })
      criteria.value = criteriaData.map(c => ({
        id: c.id,
        name: c.name,
        description: c.description || '',
        requiresEvidence: !!c.requiresEvidence,
        periodStart: toInputDate(c.periodStart),
        periodEnd: toInputDate(c.periodEnd),
        subItems: (c.subItems || []).map((s: any) => ({
          id: s.id,
          name: s.name,
          type: s.type === 'document' ? 'document' : 'score'
        }))
      }))

      // โหลด positions + employees จาก profiles (ใช้ในแท็บตำแหน่ง และ evaluate)
      const posRows = await $fetch<any[]>(apiBase + '/api/evaluation/positions', {
        headers: { Authorization: `Bearer ${token}` }
      })
      const grouped: Record<string, Array<{ userId: number; name: string }>> = {}
      posRows.forEach(row => {
        const posName = row.position || 'ไม่ระบุตำแหน่ง'
        if (!grouped[posName]) grouped[posName] = []
        if (row.userId && row.employee) {
          grouped[posName].push({ userId: row.userId, name: row.employee })
        }
      })

      positions.value = Object.entries(grouped).map(([posName, emps], index) => ({
        id: index + 1,
        name: posName,
        group: 'ตำแหน่ง',
        employees: emps
      }))

      // โหลด mapping ที่เคยบันทึก (userId <-> criteriaId) มาตั้งค่าเริ่มต้นให้แท็บตำแหน่ง
      const mappings = await $fetch<Array<{ userId: number; criteriaId: number }>>(
        apiBase + '/api/evaluation/employee-criteria',
        {
          headers: { Authorization: `Bearer ${token}` }
        }
      )

      // สร้าง employeeCriteriaMap: userId -> [criteriaId]
      const empMap: Record<number, number[]> = {}
      mappings.forEach(m => {
        if (!empMap[m.userId]) empMap[m.userId] = []
        if (!empMap[m.userId].includes(m.criteriaId)) {
          empMap[m.userId].push(m.criteriaId)
        }
      })
      employeeCriteriaMap.value = empMap

      // เติม positionSelectedEmployees: ในแต่ละตำแหน่ง เลือกบุคลากรที่มี mapping อย่างน้อย 1
      const posSelected: Record<string, number[]> = {}
      positions.value.forEach(pos => {
        const userIdsInPos = pos.employees.map(e => e.userId)
        const selectedIds = userIdsInPos.filter(uid =>
          mappings.some(m => m.userId === uid)
        )
        if (selectedIds.length) {
          posSelected[pos.name] = selectedIds
        }
      })
      positionSelectedEmployees.value = posSelected

      // positionCriteriaMap: ใช้ทุก criteriaId ที่มี mapping ร่วมกันสำหรับตำแหน่งนั้น
      const posCriteria: Record<string, number[]> = {}
      positions.value.forEach(pos => {
        const userIdsInPos = pos.employees.map(e => e.userId)
        const critSet = new Set<number>()
        mappings.forEach(m => {
          if (userIdsInPos.includes(m.userId)) {
            critSet.add(m.criteriaId)
          }
        })
        if (critSet.size) {
          posCriteria[pos.name] = Array.from(critSet)
        }
      })
      positionCriteriaMap.value = posCriteria
    } catch (err) {
      console.error(err)
    }
  }
})

const filteredEvaluations = computed(() => {
  if (role.value === 'user' && username.value) {
    return evaluations.value.filter(e => e.employee === username.value)
  }
  return evaluations.value
})

const filteredEmployees = (employees: Array<{ userId: number; name: string }>) => {
  if (role.value === 'user' && username.value) {
    return employees.filter(e => e.name === username.value)
  }
  return employees
}

// การเลือกบุคลากรต่อหนึ่งตำแหน่ง (ใช้สำหรับเริ่มประเมินแบบหลายคนพร้อมกัน)
const positionSelectedEmployees = ref<Record<string, number[]>>({})

const isEmployeeSelected = (positionName: string, userId: number) => {
  return (positionSelectedEmployees.value[positionName] || []).includes(userId)
}

const togglePositionEmployee = (positionName: string, userId: number, checked: boolean) => {
  const current = positionSelectedEmployees.value[positionName] || []
  if (checked) {
    if (!current.includes(userId)) {
      positionSelectedEmployees.value[positionName] = [...current, userId]
    }
  } else {
    positionSelectedEmployees.value[positionName] = current.filter(id => id !== userId)
  }
}

const areAllEmployeesSelected = (
  positionName: string,
  employees: Array<{ userId: number; name: string }>
) => {
  const current = positionSelectedEmployees.value[positionName] || []
  if (!employees.length) return false
  return employees.every(e => current.includes(e.userId))
}

const toggleSelectAllEmployees = (
  positionName: string,
  employees: Array<{ userId: number; name: string }>,
  checked: boolean
) => {
  if (checked) {
    positionSelectedEmployees.value[positionName] = employees.map(e => e.userId)
  } else {
    positionSelectedEmployees.value[positionName] = []
  }
}

// พนักงานที่จะแสดงในแท็บ "ดำเนินการประเมิน"
// แสดงเฉพาะตำแหน่งที่มีการเลือกหัวข้อการประเมินอย่างน้อย 1 หัวข้อใน positionCriteriaMap
const visibleEmployees = computed(() => {
  const all: string[] = []
  positions.value.forEach(p => {
    const selectedCriteriaIds = positionCriteriaMap.value[p.name]
    if (!selectedCriteriaIds || selectedCriteriaIds.length === 0) {
      // ยังไม่ได้กำหนดหัวข้อประเมินให้ตำแหน่งนี้ -> ยังไม่สามารถทำแบบประเมินได้
      return
    }

    filteredEmployees(p.employees).forEach(emp => {
      if (!all.includes(emp.name)) all.push(emp.name)
    })
  })
  return all
})

const activePeriodComputed = computed(() => periods.value.find(p => p.status === 'active'))

const getEvaluationsForEmployee = (emp: string) => {
  return filteredEvaluations.value.filter(e => e.employee === emp)
}

const getPendingEvaluationsForEmployee = (emp: string) => {
  return getEvaluationsForEmployee(emp).filter(e => e.status !== 'completed')
}

const getCompletedEvaluationsForEmployee = (emp: string) => {
  return getEvaluationsForEmployee(emp).filter(e => e.status === 'completed')
}

const findPositionForEmployee = (emp: string) => {
  return (
    positions.value.find(p => p.employees.some(e => e.name === emp)) || null
  )
}

const pendingEmployees = computed(() => {
  return visibleEmployees.value.filter(emp => getCompletedEvaluationsForEmployee(emp).length === 0)
})

const completedEmployees = computed(() => {
  return visibleEmployees.value.filter(emp => getCompletedEvaluationsForEmployee(emp).length > 0)
})

const toggleEmployee = async (emp: string) => {
  const isSame = expandedEmployee.value === emp
  expandedEmployee.value = isSame ? null : emp

  if (isSame) return

  // ถ้ามี evaluation ที่ยังไม่เสร็จสิ้น หรือประเมินแล้ว ให้เลือกอันแรก
  let firstPending = getPendingEvaluationsForEmployee(emp)[0]
  if (!firstPending) {
    firstPending = getCompletedEvaluationsForEmployee(emp)[0]
  }

  // ถ้ายังไม่มี evaluation เลย แต่รู้ตำแหน่งของพนักงาน ให้สร้าง evaluation ใหม่อัตโนมัติ
  if (!firstPending) {
    const pos = findPositionForEmployee(emp)
    if (pos) {
      await startEvaluation({ name: pos.name }, emp)
      firstPending = getPendingEvaluationsForEmployee(emp)[0]
    }
  }

  if (firstPending) {
    selectEvaluation(firstPending)
  } else {
    selectedEvaluation.value = null
  }
}

const selectEvaluation = (ev: {
  id: number
  position: string
  employee: string
  period: string | undefined
  status: string
}) => {
  selectedEvaluation.value = ev
  loadScoresForEvaluation(ev.id)
}

const loadScoresForEvaluation = async (evaluationId: number) => {
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return

  try {
    const rows = await $fetch<
      Array<{
        criteriaId: number
        subitemId: number
        selfScore: number | null
        managerScore: number | null
        note?: string | null
      }>
    >(apiBase + `/api/evaluation/${evaluationId}/scores`, {
      headers: { Authorization: `Bearer ${token}` }
    })

    const byCriteria: Record<number, Record<number, { selfScore: number | null; managerScore: number | null; note?: string | null }>> = {}
    rows.forEach(r => {
      if (!byCriteria[r.criteriaId]) byCriteria[r.criteriaId] = {}
      byCriteria[r.criteriaId][r.subitemId] = {
        selfScore: r.selfScore,
        managerScore: r.managerScore,
        note: r.note ?? null
      }
    })

    currentScores.value[evaluationId] = byCriteria
  } catch (err) {
    console.error(err)
  }
}

// criteria ที่ใช้ในแบบฟอร์มประเมิน: ดึงจาก mapping ใน DB ต่อพนักงานแต่ละคน
const visibleCriteriaForSelected = computed(() => {
  if (!selectedEvaluation.value) return criteria.value

  const empName = selectedEvaluation.value.employee

  // ลองหา userId จาก positions ก่อน
  let userId: number | null = null
  for (const pos of positions.value) {
    const emp = pos.employees.find(e => e.name === empName)
    if (emp) {
      userId = emp.userId
      break
    }
  }

  // ถ้าหาไม่เจอ ให้ลองใช้ employeeCriteriaMap ตรวจสอบว่ามี userId ใดที่ชื่อตรงกับ empName
  if (!userId) {
    // ลองหา userId จาก employeeCriteriaMap keys (ถ้ามี mapping)
    const mappedIds = Object.keys(employeeCriteriaMap.value).map(Number)
    if (mappedIds.length > 0) {
      userId = mappedIds[0] // ใช้ userId แรกที่มี mapping
    }
  }

  if (!userId) return criteria.value

  const allowedIds = employeeCriteriaMap.value[userId]
  if (!allowedIds || allowedIds.length === 0) return criteria.value

  return criteria.value.filter(c => allowedIds.includes(c.id))
})

const addPeriod = async () => {
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return

  const now = new Date()
  const in90 = new Date(now.getTime() + 90 * 24 * 60 * 60 * 1000)

  try {
    const created = await $fetch<any>(apiBase + '/api/evaluation/periods', {
      method: 'POST',
      headers: { Authorization: `Bearer ${token}` },
      body: {
        name: 'การประเมินใหม่',
        startDate: now.toISOString().split('T')[0],
        endDate: in90.toISOString().split('T')[0],
        status: 'draft'
      }
    })

    periods.value.unshift({
      id: created.id,
      name: created.name,
      startDate: created.startDate,
      endDate: created.endDate,
      status: created.status
    })
    editingPeriod.value = created.id
  } catch (err) {
    console.error(err)
  }
}

const saveCriteria = async (criterion: {
  id: number
  name: string
  description: string
  requiresEvidence: boolean
  periodStart: string | null
  periodEnd: string | null
  subItems: Array<{ id: number; name: string; type: 'score' | 'document' }>
}) => {
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return

  try {
    const updated = await $fetch<any>(apiBase + `/api/evaluation/criteria/${criterion.id}`, {
      method: 'PUT',
      headers: { Authorization: `Bearer ${token}` },
      body: {
        name: criterion.name,
        description: criterion.description,
        requiresEvidence: criterion.requiresEvidence,
        periodStart: criterion.periodStart || null,
        periodEnd: criterion.periodEnd || null,
        subItems: criterion.subItems
      }
    })

    criteria.value = criteria.value.map(c =>
      c.id === criterion.id
        ? {
            id: updated.id,
            name: updated.name,
            description: updated.description || '',
            requiresEvidence: !!updated.requiresEvidence,
            periodStart: updated.periodStart || null,
            periodEnd: updated.periodEnd || null,
            subItems: (updated.subItems || []).map((s: any) => ({
              id: s.id,
              name: s.name,
              type: s.type === 'document' ? 'document' : 'score'
            }))
          }
        : c
    )

    editingCriteria.value = null
  } catch (err) {
    console.error(err)
  }
}

const deleteCriteria = async (id: number) => {
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return

  try {
    await $fetch(apiBase + `/api/evaluation/criteria/${id}`, {
      method: 'DELETE',
      headers: { Authorization: `Bearer ${token}` }
    })

    criteria.value = criteria.value.filter(c => c.id !== id)
    // ลบ id ของหัวข้อนี้ออกจากการตั้งค่าต่อทุกตำแหน่ง
    Object.keys(positionCriteriaMap.value).forEach(posName => {
      const current = positionCriteriaMap.value[posName] || []
      positionCriteriaMap.value[posName] = current.filter(cid => cid !== id)
    })
    if (editingCriteria.value === id) editingCriteria.value = null
  } catch (err) {
    console.error(err)
  }
}

const deletePeriod = async (id: number) => {
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return

  try {
    await $fetch(apiBase + `/api/evaluation/periods/${id}`, {
      method: 'DELETE',
      headers: { Authorization: `Bearer ${token}` }
    })

    periods.value = periods.value.filter(p => p.id !== id)
    if (editingPeriod.value === id) editingPeriod.value = null
  } catch (err) {
    console.error(err)
  }
}

const addCriteria = async () => {
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return

  try {
    const created = await $fetch<any>(apiBase + '/api/evaluation/criteria', {
      method: 'POST',
      headers: { Authorization: `Bearer ${token}` },
      body: {
        name: 'หัวข้อใหม่',
        description: '',
        requiresEvidence: false
      }
    })

    const newCriteria = {
      id: created.id,
      name: created.name,
      description: created.description || '',
      requiresEvidence: !!created.requiresEvidence,
      periodStart: created.periodStart || null,
      periodEnd: created.periodEnd || null,
      subItems: (created.subItems || []).map((s: any) => ({
        id: s.id,
        name: s.name,
        type: s.type === 'document' ? 'document' : 'score'
      }))
    }
    criteria.value.push(newCriteria)
    editingCriteria.value = newCriteria.id
  } catch (err) {
    console.error(err)
  }
}

const addSubItem = async (criteriaId: number) => {
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return

  try {
    const created = await $fetch<any>(apiBase + `/api/evaluation/criteria/${criteriaId}/subitems`, {
      method: 'POST',
      headers: { Authorization: `Bearer ${token}` },
      body: {
        name: 'หัวข้อย่อยใหม่',
        type: 'score'
      }
    })

    criteria.value = criteria.value.map(c => {
      if (c.id === criteriaId) {
        return {
          ...c,
          subItems: [
            ...c.subItems,
            {
              id: created.id,
              name: created.name,
              type: created.type === 'document' ? 'document' : 'score'
            }
          ]
        }
      }
      return c
    })
  } catch (err) {
    console.error(err)
  }
}

const deleteSubItem = (criteriaId: number, subId: number) => {
  criteria.value = criteria.value.map(c => {
    if (c.id === criteriaId) {
      return {
        ...c,
        subItems: c.subItems.filter(s => s.id !== subId)
      }
    }
    return c
  })
}

const startEvaluation = async (position: { name: string }, employee: string) => {
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return

  // เลือก period ที่จะใช้ผูกกับการประเมิน
  // 1) ถ้ามี active ให้ใช้ตัวนั้น
  // 2) ถ้าไม่มี active แต่มีรายการ ให้ใช้ตัวแรก
  // 3) ถ้ายังไม่มีเลย ให้สร้าง period ใหม่แบบอัตโนมัติ (active) แล้วใช้ตัวนั้น
  let activePeriod =
    periods.value.find(p => p.status === 'active') || periods.value[0] || null

  if (!activePeriod) {
    const now = new Date()
    const in90 = new Date(now.getTime() + 90 * 24 * 60 * 60 * 1000)

    try {
      const created = await $fetch<any>(apiBase + '/api/evaluation/periods', {
        method: 'POST',
        headers: { Authorization: `Bearer ${token}` },
        body: {
          name: 'รอบการประเมินอัตโนมัติ',
          startDate: now.toISOString().split('T')[0],
          endDate: in90.toISOString().split('T')[0],
          status: 'active'
        }
      })

      activePeriod = created
      periods.value.push({
        id: created.id,
        name: created.name,
        startDate: created.startDate,
        endDate: created.endDate,
        status: created.status
      })
    } catch (err) {
      console.error('Failed to auto-create period for evaluation', err)
      return
    }
  }

  try {
    const created = await $fetch<any>(apiBase + '/api/evaluation', {
      method: 'POST',
      headers: { Authorization: `Bearer ${token}` },
      body: {
        employee,
        position: position.name,
        periodId: activePeriod.id
      }
    })

    evaluations.value.push({
      id: created.id,
      employee: created.employee,
      position: created.position,
      period: created.period,
      status: created.status
    })
  } catch (err) {
    console.error(err)
  }
}

const submitEvaluation = async () => {
  if (!selectedEvaluation.value) return
  if (typeof window === 'undefined') return
  const token = localStorage.getItem('token')
  if (!token) return

  try {
    await $fetch(apiBase + `/api/evaluation/${selectedEvaluation.value.id}/submit`, {
      method: 'PUT',
      headers: { Authorization: `Bearer ${token}` }
    })

    evaluations.value = evaluations.value.map(e =>
      e.id === selectedEvaluation.value!.id ? { ...e, status: 'completed' } : e
    )

    selectedEvaluation.value = {
      ...selectedEvaluation.value,
      status: 'completed'
    }
  } catch (err) {
    console.error(err)
  }
  
}

// simple emoji icons used directly in template (no JSX components needed)
</script>
