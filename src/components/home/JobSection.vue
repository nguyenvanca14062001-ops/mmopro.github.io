<script setup lang="ts">
import { jobsData } from '@/data/jobs';

defineProps<{
  username: string;
  isLoggedIn: boolean;
  userBalance: number;
  totalWithdrawn?: number
}>();

const emit = defineEmits(['receiveJob', 'contactSupport', 'routerPush']);

const handleJobClick = (id: string) => {
  console.log('[JobSection] handleJobClick called', id);
  const job = jobsData[id];
  if (!job || (job as any).paused) return;
  if (typeof navigator !== 'undefined' && navigator.vibrate) navigator.vibrate(50);
  emit('receiveJob', id);
};

const formatReward = (val: any) => {
  if (!val) return '0';
  return String(val).replace(/\D/g, '');
};

const userStats = [
  { label: 'SỐ DƯ KHẢ DỤNG', key: 'balance', unit: 'XU', color: 'text-blue-500', icon: '💰', bgIcon: '🪙' },
  { label: 'TỔNG ĐÃ RÚT', key: 'withdrawn', unit: 'XU', color: 'text-rose-400/90', icon: '💸', bgIcon: '🏦' },
  { label: 'CẤP BẬC TÀI KHOẢN', key: 'rank', value: 'THÀNH VIÊN', unit: '', color: 'text-white/90', icon: '👑', bgIcon: '🏆' },
  { label: 'THU NHẬP TUẦN NÀY', key: 'weekly', value: '0', unit: 'XU', color: 'text-emerald-400/90', icon: '📈', bgIcon: '📊' }
];

const getJobIcon = (id: string) => {
  const config: Record<string, { t: string, c: string }> = {
    'view-tiktok': { t: 'TT', c: 'text-white' },
    'view-youtube': { t: 'YT', c: 'text-white' },
    'post-threads': { t: '@', c: 'text-white' },
    'seeding-vinfast': { t: 'VF', c: 'text-white' },
    'google-map': { t: '📍', c: 'text-[#090e17]' },
    'join-zalo': { t: 'ZALO', c: 'text-white' },
    'app-chung-khoan': { t: '📈', c: 'text-white' },
    'app-chung-khoan-2': { t: '📈', c: 'text-white' },
    'app-chung-khoan-3': { t: '📈', c: 'text-white' },
    'msb-bank': { t: 'MSB', c: 'text-white' },
    'vpbank': { t: 'VPB', c: 'text-white' },
    'tpbank': { t: 'TPB', c: 'text-white' },
  };
  const res = config[id] || { t: 'JOB', c: 'text-slate-400' };
  return { type: 'text', content: res.t, colorClass: res.c };
};

const getShortDesc = (id: string) => {
  const desc: Record<string, string> = {
    'view-tiktok': 'Xem video, thả tim, bình luận',
    'view-youtube': 'Xem video 3p, like & subscribe',
    'post-threads': 'Đăng tải nội dung kéo tương tác',
    'seeding-vinfast': 'Bình luận tích cực, lan tỏa thông điệp',
    'google-map': 'Đánh giá địa điểm nhận thưởng',
    'join-zalo': 'Vào nhóm cộng đồng nhận thông báo',
    'app-chung-khoan': 'Đăng ký tài khoản Kafi X',
    'app-chung-khoan-2': 'Đăng ký tài khoản DNSE',
    'app-chung-khoan-3': 'Đăng ký tài khoản KIS',
    'vpbank': 'Mở tài khoản số đẹp VPBank',
    'tpbank': 'Mở tài khoản TPBank Mobile',
    'msb-bank': 'Nhận quà tặng khi mở thẻ MSB'
  };
  return desc[id] || 'Làm nhiệm vụ ngay';
}
</script>

<template>
  <div class="space-y-4 animate-in fade-in duration-700 text-left">
    <svg width="0" height="0" class="absolute">
      <defs>
        <linearGradient id="goldCoinGradient" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" style="stop-color:#fde047" />
          <stop offset="50%" style="stop-color:#eab308" />
          <stop offset="100%" style="stop-color:#854d0e" />
        </linearGradient>
      </defs>
    </svg>

    <div class="flex flex-col lg:flex-row gap-3">
      <section class="lg:w-2/3 relative bg-gradient-to-br from-[#111726] to-[#0d121f] rounded-[30px] border border-slate-800/60 p-6 md:p-10 overflow-hidden flex items-center min-h-[200px] md:min-h-[400px] shadow-2xl">

        <div class="absolute -right-20 -top-20 w-[300px] h-[300px] bg-blue-600/15 rounded-full blur-[90px]"></div>

        <div class="relative z-10 space-y-4 w-full md:w-[65%]">
          <div class="inline-flex items-center gap-1.5 bg-emerald-500/10 text-emerald-500 text-[9px] md:text-[10px] px-3 py-1 rounded-full border border-emerald-500/20 font-bold uppercase tracking-wider">
            <span class="w-1.5 h-1.5 rounded-full bg-emerald-500 animate-pulse"></span> ONLINE
          </div>

          <h1 class="text-2xl md:text-5xl text-white leading-tight tracking-tighter uppercase font-black italic">
            CHÀO MỪNG,<br/>
            <span class="text-transparent bg-clip-text bg-gradient-to-r from-blue-400 to-indigo-400 text-3xl md:text-6xl">
              {{ username.toUpperCase() }}
            </span>
          </h1>

          <div class="border-l-4 border-blue-600 pl-4 max-w-2xl space-y-2">
            <p class="text-slate-300 text-[12px] md:text-[15px] font-medium leading-relaxed">
              Nền tảng kiếm tiền Online minh bạch. Rút xu nhanh gọn 24/7 về mọi ngân hàng.
            </p>
            <p class="text-rose-400 text-[10px] md:text-[13px] font-bold tracking-wide">
              ⚠️ CẢNH BÁO: Nghiêm cấm gian lận hoặc gửi bằng chứng giả. Khóa vĩnh viễn nếu vi phạm.
            </p>
          </div>

          <div class="pt-3 flex flex-wrap gap-2 md:gap-3">
            <button v-if="!isLoggedIn" @click="emit('routerPush', '/login')" class="bg-blue-600 hover:bg-blue-500 text-white w-full md:w-auto px-8 py-3.5 rounded-xl text-[10px] md:text-[12px] shadow-xl shadow-blue-900/40 uppercase font-black italic transition-all active:scale-95">
              ĐĂNG KÝ / ĐĂNG NHẬP NGAY
            </button>
            <template v-else>
              <button @click="emit('contactSupport', 'facebook')" class="flex-1 md:flex-none justify-center bg-[#1877F2] hover:bg-blue-600 text-white px-5 py-3.5 rounded-xl text-[10px] md:text-[12px] shadow-xl shadow-blue-900/40 uppercase font-black italic transition-all active:scale-95 flex items-center gap-2 border border-white/10">
                <svg class="w-4 h-4 md:w-5 md:h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M24 12.073c0-6.627-5.373-12-12-12s-12 5.373-12 12c0 5.99 4.388 10.954 10.125 11.854v-8.385H7.078v-3.47h3.047V9.43c0-3.007 1.792-4.669 4.533-4.669 1.312 0 2.686.235 2.686.235v2.953H15.83c-1.491 0-1.956.925-1.956 1.874v2.25h3.328l-.532 3.47h-2.796v8.385C19.612 23.027 24 18.062 24 12.073z"/></svg>
                FANPAGE
              </button>
              <button @click="emit('contactSupport', 'zalo')" class="flex-1 md:flex-none justify-center bg-white hover:bg-slate-200 text-[#0068FF] px-5 py-3.5 rounded-xl text-[10px] md:text-[12px] shadow-xl shadow-white/10 uppercase font-black italic transition-all active:scale-95 flex items-center gap-2 border border-slate-200">
                <img src="https://upload.wikimedia.org/wikipedia/commons/9/91/Icon_of_Zalo.svg" class="w-4 h-4 md:w-5 md:h-5" />
                NHÓM KÍN
              </button>
            </template>
          </div>
        </div>

        <div class="absolute right-5 lg:right-10 top-0 bottom-0 hidden md:flex items-center justify-center pointer-events-none">
          <div class="relative animate-jump-cycle">
            <div class="absolute inset-0 bg-blue-500/30 rounded-full blur-[70px]"></div>
            <div class="text-[120px] lg:text-[150px] drop-shadow-[0_20px_50px_rgba(0,0,0,0.8)] filter contrast-125 saturate-150 rotate-12">🚀</div>
          </div>
        </div>
      </section>

      <div class="lg:w-1/3 grid grid-cols-2 gap-3">
        <div v-for="stat in userStats" :key="stat.label"
             class="bg-gradient-to-b from-[#1a2333]/80 to-[#111726]/90 border border-slate-800/60 border-t-slate-700/80 rounded-[25px] p-4 md:p-5 flex flex-col justify-between min-h-[110px] md:min-h-[180px] relative overflow-hidden shadow-2xl group">

          <div class="absolute -right-4 -bottom-4 md:-right-6 md:-bottom-6 text-6xl md:text-8xl opacity-[0.03] group-hover:opacity-[0.08] transition-opacity duration-500 pointer-events-none grayscale">
            {{ stat.bgIcon }}
          </div>

          <p class="text-slate-500 text-[8px] md:text-[10px] tracking-[1px] font-bold uppercase relative z-10">{{ stat.label }}</p>

          <div class="relative z-10 mt-auto">
            <p class="font-black italic tracking-tighter flex items-baseline gap-1" :class="stat.color">
              <span class="text-xl md:text-4xl leading-none">
                {{ stat.key === 'balance' ? userBalance.toLocaleString() : (stat.key === 'rank' ? stat.value : (totalWithdrawn || 0).toLocaleString()) }}
              </span>
              <div v-if="stat.key !== 'rank'" class="flex flex-col items-center translate-y-[-2px]">
                <svg class="w-4 h-4 md:w-5 md:h-5 drop-shadow-[0_0_5px_rgba(234,179,8,0.8)]" viewBox="0 0 24 24" fill="none">
                  <circle cx="12" cy="12" r="10" fill="url(#goldCoinGradient)" />
                  <path d="M12 7v10M9 10h6M9 14h6" stroke="#854d0e" stroke-width="2" stroke-linecap="round" />
                </svg>
                <span class="text-[7px] md:text-[9px] text-yellow-500 font-black not-italic tracking-tighter uppercase">Xu</span>
              </div>
            </p>
          </div>
        </div>
      </div>
    </div>

    <section class="space-y-4 pt-2">
      <div class="flex items-center gap-3 px-1">
        <div class="w-1.5 h-6 bg-emerald-500 rounded-full shadow-[0_0_10px_rgba(16,185,129,0.6)]"></div>
        <h3 class="text-lg md:text-3xl text-white tracking-tighter italic font-black uppercase">CÔNG VIỆC <span class="text-emerald-500">HOT</span></h3>
      </div>

      <div class="bg-[#111726]/40 border border-slate-800/50 rounded-[30px] p-3 md:p-8 shadow-inner">
        <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 md:gap-6">
          <div v-for="(j, id, index) in jobsData" :key="id" @click="handleJobClick(id as string)"
            class="relative p-5 md:p-7 rounded-[28px] border-[2px] transition-all duration-500 flex flex-col group cursor-pointer active:scale-95 shadow-2xl overflow-hidden"
            :class="[
              id === 'view-tiktok'
                ? 'bg-gradient-to-br from-[#3B1731] to-[#1F0A1A] border-pink-500/80 shadow-[0_0_20px_rgba(236,72,153,0.3)]'
                : id === 'view-youtube'
                ? 'bg-gradient-to-br from-[#4A1612] to-[#260505] border-red-500/80 shadow-[0_0_20px_rgba(239,68,68,0.3)]'
                : id === 'post-threads'
                ? 'bg-gradient-to-br from-[#2D3748] to-[#0F172A] border-slate-300/80 shadow-[0_0_20px_rgba(255,255,255,0.1)]'
                : id === 'seeding-vinfast'
                ? 'bg-gradient-to-br from-[#083344] to-[#082F49] border-cyan-500/80 shadow-[0_0_20px_rgba(6,182,212,0.3)]'
                : id === 'google-map'
                ? 'bg-gradient-to-br from-[#3B1731] to-[#240A1A] border-fuchsia-500/80 shadow-[0_0_20px_rgba(217,70,239,0.3)]'
                : id === 'join-zalo'
                ? 'bg-gradient-to-br from-[#182040] to-[#0C1226] border-indigo-500/80 shadow-[0_0_20px_rgba(99,102,241,0.3)]'
                : ['app-chung-khoan', 'app-chung-khoan-2', 'app-chung-khoan-3', 'msb-bank', 'vpbank', 'tpbank'].includes(id as string)
                ? 'bg-gradient-to-br from-[#4A1612] to-[#260505] border-red-500/90 shadow-[0_0_30px_rgba(239,68,68,0.5)]'
                : 'bg-[#0d121f] border-slate-800'
            ]"
          >
            <div class="absolute inset-0 bg-gradient-to-t from-transparent to-white/5 pointer-events-none rounded-[26px]"></div>

            <div class="absolute -top-0 -right-0 z-20 flex items-center gap-1 text-[9px] md:text-[10px] tracking-widest px-3 py-1.5 rounded-bl-2xl rounded-tr-[26px] font-black italic uppercase border-b border-l border-white/20 shadow-lg"
                 :class="[
                    ['app-chung-khoan', 'app-chung-khoan-2', 'app-chung-khoan-3', 'msb-bank', 'vpbank', 'tpbank'].includes(id as string) ? 'bg-gradient-to-r from-orange-500 to-red-600 text-white' :
                    id === 'view-tiktok' ? 'bg-pink-600 text-white' :
                    id === 'view-youtube' ? 'bg-red-600 text-white' :
                    id === 'post-threads' ? 'bg-slate-400 text-slate-900' :
                    id === 'seeding-vinfast' ? 'bg-cyan-600 text-white' :
                    id === 'google-map' ? 'bg-fuchsia-600 text-white' :
                    'bg-indigo-600 text-white'
                 ]">
              {{ ['app-chung-khoan', 'app-chung-khoan-2', 'app-chung-khoan-3', 'msb-bank', 'vpbank', 'tpbank'].includes(id as string) ? 'SIÊU HOT 🚀' : (j.badge || 'CƠ BẢN') }}
            </div>

            <div class="flex justify-between items-start mb-4 relative z-10">
              <div class="w-12 h-12 md:w-16 md:h-16 rounded-2xl flex items-center justify-center shadow-lg border-[1.5px] border-white/20 transition-transform group-hover:scale-110"
                   :class="[
                      id === 'view-tiktok' ? 'bg-pink-500/20 text-pink-400' :
                      id === 'view-youtube' ? 'bg-red-500/20 text-red-400' :
                      id === 'post-threads' ? 'bg-slate-300/20 text-slate-300' :
                      id === 'seeding-vinfast' ? 'bg-cyan-500/20 text-cyan-400' :
                      id === 'google-map' ? 'bg-fuchsia-500/20 text-fuchsia-400' :
                      id === 'join-zalo' ? 'bg-indigo-500/20 text-indigo-400' :
                      ['app-chung-khoan', 'app-chung-khoan-2', 'app-chung-khoan-3', 'msb-bank', 'vpbank', 'tpbank'].includes(id as string) ? 'bg-red-500/30 text-red-400' :
                      'bg-[#111726]'
                   ]">
                <span class="font-black text-sm md:text-xl italic">{{ getJobIcon(id as string).content }}</span>
              </div>

              <span class="text-4xl md:text-6xl font-black opacity-[0.08] group-hover:opacity-[0.15] transition-opacity text-white">
                {{ (index + 1).toString().padStart(2, '0') }}
              </span>
            </div>

            <h4 class="text-[13px] md:text-lg text-white font-black italic uppercase leading-tight mb-1"
                :class="{'text-pink-400': id === 'view-tiktok', 'text-red-400': id === 'view-youtube', 'text-slate-300': id === 'post-threads', 'text-cyan-400': id === 'seeding-vinfast', 'text-fuchsia-400': id === 'google-map', 'text-indigo-400': id === 'join-zalo' }">
              {{ j.title }}
            </h4>

            <p class="text-[10px] md:text-[13px] text-slate-300 font-medium line-clamp-2 leading-relaxed mb-4 mt-1">
              {{ getShortDesc(id as string) }}
            </p>

            <div class="flex flex-col mt-auto relative z-10">
              <p class="text-[9px] md:text-[10px] font-bold text-slate-400 uppercase tracking-widest mb-1">Thưởng ngay:</p>
              <div class="flex items-center gap-1.5">
                <p class="font-black text-xl md:text-3xl tracking-tighter italic leading-none" :class="j.color">
                  {{ formatReward(j.reward).toLocaleString() }}
                </p>
                <div class="flex flex-col items-start translate-y-[-2px]">
                  <svg class="w-4 h-4 md:w-5 md:h-5 drop-shadow-[0_0_5px_rgba(234,179,8,0.5)]" viewBox="0 0 24 24">
                    <circle cx="12" cy="12" r="10" fill="url(#goldCoinGradient)" />
                    <path d="M12 7v10M9 10h6M9 14h6" stroke="#854d0e" stroke-width="2" stroke-linecap="round" />
                  </svg>
                  <span class="text-[7px] md:text-[9px] text-yellow-500 font-black not-italic tracking-tighter leading-none uppercase">Xu</span>
                </div>
              </div>
            </div>

            <button @click.stop="handleJobClick(id as string)" class="w-full mt-4 py-3 md:py-4 rounded-xl text-[10px] md:text-[11px] font-black italic uppercase transition-all shadow-lg relative z-10 border-t border-white/20"
              :class="[
                id === 'view-tiktok' ? 'bg-gradient-to-r from-pink-600 to-rose-500 text-white' :
                id === 'view-youtube' ? 'bg-gradient-to-r from-red-600 to-rose-500 text-white' :
                id === 'post-threads' ? 'bg-gradient-to-r from-slate-300 to-slate-100 text-slate-900' :
                id === 'seeding-vinfast' ? 'bg-gradient-to-r from-cyan-600 to-blue-500 text-white' :
                id === 'google-map' ? 'bg-gradient-to-r from-fuchsia-600 to-pink-500 text-white' :
                id === 'join-zalo' ? 'bg-gradient-to-r from-indigo-600 to-blue-500 text-white' :
                ['app-chung-khoan', 'app-chung-khoan-2', 'app-chung-khoan-3', 'msb-bank', 'vpbank', 'tpbank'].includes(id as string) ? 'bg-gradient-to-r from-red-600 to-orange-500 hover:from-red-500 hover:to-orange-400 text-white shadow-[0_5px_15px_rgba(239,68,68,0.5)]' :
                'bg-[#1a2333] text-white'
              ]"
            >
              BẮT ĐẦU ⚡
            </button>
          </div>
        </div>
      </div>
    </section>
  </div>

</template>

<style scoped>
.animate-jump-cycle {
  animation: jump-cycle 3s ease-in-out infinite;
}
@keyframes jump-cycle {
  0%, 100% { transform: translateY(0) rotate(12deg); }
  50% { transform: translateY(-20px) rotate(15deg); }
}

</style>
