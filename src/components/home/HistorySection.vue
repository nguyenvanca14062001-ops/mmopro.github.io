<script setup lang="ts">
// Props nhận dữ liệu từ App.vue truyền xuống
defineProps<{
  myReports: any[]
  isLoggedIn: boolean
  isDataLoading: boolean
}>()

const formatNumber = (num: number) => {
  return num.toLocaleString('vi-VN')
}
</script>

<template>
  <div class="space-y-6">
    <!-- ĐỊNH NGHĨA GRADIENT CHO ĐỒNG XU (PHẢI CÓ ĐỂ HIỆN MÀU VÀNG) -->
    <svg width="0" height="0" class="absolute">
      <defs>
        <linearGradient id="historyGoldCoin" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" style="stop-color:#fde047" />
          <stop offset="50%" style="stop-color:#eab308" />
          <stop offset="100%" style="stop-color:#854d0e" />
        </linearGradient>
      </defs>
    </svg>

    <div class="flex items-center gap-3 mb-8">
      <div class="w-1.5 h-8 bg-blue-600 rounded-full shadow-[0_0_15px_rgba(37,99,235,0.5)]"></div>
      <h2 class="text-2xl md:text-3xl text-white font-black italic uppercase tracking-tighter">
        LỊCH SỬ <span class="text-blue-500">HOẠT ĐỘNG</span>
      </h2>
    </div>

    <!-- NẾU CHƯA ĐĂNG NHẬP -->
    <div v-if="!isLoggedIn" class="bg-[#111726] border border-slate-800 rounded-[30px] p-12 text-center">
      <p class="text-slate-500 font-bold italic uppercase tracking-widest text-xs">Vui lòng đăng nhập để xem lịch sử của bạn</p>
    </div>

    <!-- NẾU ĐANG TẢI DỮ LIỆU -->
    <div v-else-if="isDataLoading" class="text-center py-10">
      <div class="w-8 h-8 border-4 border-blue-500 border-t-transparent rounded-full animate-spin mx-auto mb-4"></div>
      <p class="text-slate-500 text-[10px] font-black italic uppercase">Đang đồng bộ dữ liệu...</p>
    </div>

    <!-- DANH SÁCH LỊCH SỬ -->
    <div v-else class="space-y-4">
      <div v-for="item in myReports" :key="item.id" 
           class="group bg-[#111726]/50 border border-slate-800/50 hover:border-blue-500/30 p-5 md:p-6 rounded-[25px] flex items-center justify-between transition-all duration-300 shadow-lg relative overflow-hidden">
        
        <!-- Hiệu ứng sáng khi hover -->
        <div class="absolute inset-0 bg-blue-500/5 opacity-0 group-hover:opacity-100 transition-opacity"></div>

        <div class="relative z-10 flex flex-col gap-1">
          <span class="text-blue-500 text-[9px] font-black tracking-[2px] opacity-80">{{ item.displayTime }}</span>
          <h3 class="text-white text-xs md:text-sm font-black italic uppercase tracking-tight">
            {{ item.type === 'withdraw' ? '🏦 RÚT TIỀN VỀ VÍ' : item.jobName }}
          </h3>
          <p v-if="item.type === 'withdraw'" class="text-slate-500 text-[9px] normal-case font-bold opacity-70">
            {{ item.bankInfo }}
          </p>
        </div>

        <div class="relative z-10 flex items-center gap-4 md:gap-6">
          <!-- SỐ TIỀN VÀ BIỂU TƯỢNG XU -->
          <div class="flex items-center gap-2">
            <span :class="[
              'text-lg md:text-2xl font-black italic tracking-tighter',
              item.type === 'withdraw' ? 'text-rose-500' : 'text-emerald-400'
            ]">
              {{ item.type === 'withdraw' ? '-' : '+' }}{{ formatNumber(item.reward || item.amount || 0) }}
            </span>
            
            <div class="flex flex-col items-center translate-y-[-1px]">
              <svg class="w-5 h-5 md:w-6 md:h-6 drop-shadow-[0_0_8px_rgba(234,179,8,0.5)]" viewBox="0 0 24 24" fill="none">
                <circle cx="12" cy="12" r="10" fill="url(#historyGoldCoin)" />
                <path d="M12 7v10M9 10h6M9 14h6" stroke="#854d0e" stroke-width="2" stroke-linecap="round" />
              </svg>
              <span class="text-[8px] text-yellow-500 font-black not-italic leading-none mt-1">XU</span>
            </div>
          </div>

          <!-- TRẠNG THÁI -->
          <div class="min-w-[90px] text-right">
            <span v-if="item.status === 'approved' || item.status === 'completed'" 
                  class="px-3 py-1.5 bg-emerald-500/10 border border-emerald-500/20 text-emerald-500 text-[8px] md:text-[9px] font-black rounded-lg uppercase italic tracking-widest shadow-[0_0_15px_rgba(16,185,129,0.1)]">
              THÀNH CÔNG
            </span>
            <span v-else-if="item.status === 'pending'" 
                  class="px-3 py-1.5 bg-yellow-500/10 border border-yellow-500/20 text-yellow-500 text-[8px] md:text-[9px] font-black rounded-lg uppercase italic tracking-widest shadow-[0_0_15px_rgba(234,179,8,0.1)]">
              ĐANG CHỜ
            </span>
            <span v-else
                  class="px-3 py-1.5 bg-rose-500/10 border border-rose-500/20 text-rose-500 text-[8px] md:text-[9px] font-black rounded-lg uppercase italic tracking-widest shadow-[0_0_15px_rgba(239,68,68,0.1)]">
              BỊ TỪ CHỐI
            </span>
          </div>
        </div>
      </div>

      <!-- KHI KHÔNG CÓ DỮ LIỆU -->
      <div v-if="myReports.length === 0" class="text-center py-20 bg-[#111726]/30 rounded-[30px] border border-dashed border-slate-800">
        <p class="text-slate-600 font-black italic uppercase text-[10px] tracking-[4px]">Chưa có hoạt động nào được ghi lại</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Hiệu ứng trượt nhẹ khi xuất hiện */
.group {
  animation: slideIn 0.5s ease-out forwards;
}

@keyframes slideIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>