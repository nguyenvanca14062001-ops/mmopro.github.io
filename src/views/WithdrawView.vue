<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'
import { auth, db } from '@/firebase'
import { collection, addDoc, doc, updateDoc, onSnapshot, query, where, getDocs } from "firebase/firestore"
import { onAuthStateChanged } from "firebase/auth"

const router = useRouter()
const amount = ref<number | null>(null)
const bankInfo = ref('')
const isLoading = ref(false)
const userBalance = ref(0)
const currentUser = ref<any>(null)
const hasPendingWithdraw = ref(false)
const approvedJobsCount = ref(0)

// BIẾN CHO POPUP
const showConfirmModal = ref(false)

// DANH SÁCH MỐC RÚT THEO YÊU CẦU MỚI NHẤT
const withdrawOptions = [250000, 300000, 500000, 650000, 800000, 1000000, 2000000]

// FORMAT SỐ ĐẸP
const formatNumber = (num: number) => {
  return Math.floor(num).toLocaleString('vi-VN') 
}

const selectAmount = (val: number) => {
  amount.value = val
}

// LOGIC TẠO LỊCH SỬ RÚT TIỀN ẢO
const fakeWithdrawals = ref<any[]>([])
let intervalId: any = null

const hoHo = ['Nguyễn', 'Trần', 'Lê', 'Phạm', 'Hoàng', 'Phan', 'Vũ', 'Võ', 'Đặng', 'Bùi']
const tenTen = ['Thành', 'Hoa', 'Linh', 'Tùng', 'Hùng', 'Oanh', 'Trang', 'Nam', 'Việt', 'Đức']

const generateFakeWithdraw = () => {
  const ho = hoHo[Math.floor(Math.random() * hoHo.length)]
  const ten = tenTen[Math.floor(Math.random() * tenTen.length)]
  const name = `${ho} *** ${ten}`
  
  const mocs = [250000, 300000, 500000, 650000, 800000, 1000000, 2000000]
  const randomAmount = mocs[Math.floor(Math.random() * mocs.length)]
  
  const times = ['Vừa xong', '1 phút trước', '3 phút trước', '5 phút trước', '10 phút trước']
  const randomTime = times[Math.floor(Math.random() * times.length)]

  return { id: Date.now() + Math.random(), name, amount: randomAmount, time: randomTime }
}

const initFakeList = () => {
  for (let i = 0; i < 4; i++) {
    fakeWithdrawals.value.push(generateFakeWithdraw())
  }
}

const startFakeLoop = () => {
  intervalId = setInterval(() => {
    fakeWithdrawals.value.unshift(generateFakeWithdraw())
    if (fakeWithdrawals.value.length > 4) {
      fakeWithdrawals.value.pop()
    }
  }, 4000)
}

onMounted(() => {
  initFakeList()
  startFakeLoop()

  onAuthStateChanged(auth, async (user) => {
    if (user) {
      currentUser.value = user
      onSnapshot(doc(db, "users", user.uid), (docSnap) => {
        if (docSnap.exists()) {
          userBalance.value = docSnap.data().balance || 0
          hasPendingWithdraw.value = docSnap.data().hasPendingWithdraw || false
        }
      })

      // ĐẾM SỐ LƯỢNG JOB ĐÃ HOÀN THÀNH (APPROVED)
      try {
        const q = query(collection(db, "reports"), where("uid", "==", user.uid), where("status", "==", "approved"));
        const querySnapshot = await getDocs(q);
        approvedJobsCount.value = querySnapshot.size;
      } catch (err) {
        console.error("Lỗi khi đếm số job:", err);
      }

    } else {
      router.push('/login')
    }
  })
})

onUnmounted(() => {
  if (intervalId) clearInterval(intervalId)
})

const triggerWithdraw = () => {
  if (hasPendingWithdraw.value) {
    alert('Bạn đang có lệnh rút tiền chờ xử lý. Vui lòng đợi Admin duyệt trước khi tạo lệnh mới!')
    return
  }

  if (!amount.value) {
    alert('Vui lòng chọn số XU muốn rút!')
    return
  }

  if (amount.value > userBalance.value) {
    alert('Số dư XU không đủ!')
    return
  }

  if (!bankInfo.value.trim() || bankInfo.value.length < 10) {
    alert('Vui lòng nhập đầy đủ thông tin nhận tiền (Tên Ngân hàng - STK - Tên Chủ Thẻ)!')
    return
  }

  showConfirmModal.value = true
}

const handleConfirmWithdraw = async () => {
  if (isLoading.value || !currentUser.value || !amount.value) return
  isLoading.value = true

  try {
    const withdrawalRef = collection(db, "withdrawals")
    const realMoneyVND = Math.floor(amount.value / 12)

    await addDoc(withdrawalRef, {
      uid: currentUser.value.uid,
      amount: amount.value,            
      realMoney: realMoneyVND,        
      bankInfo: bankInfo.value,
      status: 'pending',
      createdAt: new Date()
    })

    const userRef = doc(db, "users", currentUser.value.uid)
    await updateDoc(userRef, {
      balance: userBalance.value - amount.value,
      hasPendingWithdraw: true
    })

    alert('Gửi lệnh rút tiền thành công! Vui lòng chờ Admin duyệt.')
    router.push('/')
  } catch (error) {
    console.error("Lỗi khi rút tiền: ", error)
    alert('Có lỗi xảy ra, vui lòng thử lại sau.')
  } finally {
    isLoading.value = false
    showConfirmModal.value = false
  }
}
</script>

<template>
  <div class="min-h-screen bg-[#090e17] text-slate-300 font-sans p-4 md:p-10 font-black uppercase italic tracking-tighter pb-36">
    
    <button @click="router.back()" class="flex items-center gap-2 text-slate-500 hover:text-white transition-colors mb-6 md:mb-10 group">
      <svg class="w-5 h-5 group-hover:-translate-x-1 transition-transform" fill="none" stroke="currentColor" stroke-width="3" viewBox="0 0 24 24"><path d="M15 19l-7-7 7-7" /></svg>
      <span class="tracking-widest text-[10px]">QUAY LẠI</span>
    </button>

    <div class="max-w-xl mx-auto space-y-6">
      
      <div class="bg-[#111726] border border-slate-800 rounded-[30px] p-6 md:p-8 shadow-2xl relative overflow-hidden">
        
        <div class="text-center mb-8 relative z-10">
          <div class="w-16 h-16 bg-gradient-to-tr from-yellow-400 to-orange-500 rounded-2xl mx-auto flex items-center justify-center text-3xl mb-4 shadow-[0_0_20px_rgba(234,179,8,0.3)]">
            <svg class="w-8 h-8 drop-shadow-md" viewBox="0 0 24 24" fill="none">
              <circle cx="12" cy="12" r="10" fill="#fde047" />
              <path d="M12 7v10M9 10h6M9 14h6" stroke="#854d0e" stroke-width="2" stroke-linecap="round" />
            </svg>
          </div>
          <h1 class="text-3xl text-white">RÚT TIỀN</h1>
          <p class="text-yellow-500 mt-2 text-xs tracking-widest">SỐ DƯ KHẢ DỤNG: {{ formatNumber(userBalance) }} XU</p>
        </div>

        <div class="space-y-6 relative z-10">
          <div>
            <label class="block text-blue-500 text-[10px] tracking-widest mb-3">CHỌN SỐ XU MUỐN RÚT</label>
            <div class="grid grid-cols-2 gap-3">
              <button 
                v-for="opt in withdrawOptions" :key="opt"
                @click="selectAmount(opt)"
                :class="[
                  'py-3 rounded-[14px] border-2 transition-all text-xs md:text-sm',
                  amount === opt ? 'bg-yellow-500/10 border-yellow-500 text-yellow-400 shadow-[0_0_15px_rgba(234,179,8,0.2)]' : 'bg-[#0d121f] border-slate-800 text-slate-500 hover:border-slate-600'
                ]"
              >
                {{ formatNumber(opt) }} XU
              </button>
            </div>
          </div>

          <div>
            <label class="block text-blue-500 text-[10px] tracking-widest mb-3">THÔNG TIN NHẬN TIỀN (VNĐ)</label>
            <textarea 
              v-model="bankInfo" 
              rows="3"
              placeholder="VD: MB BANK - 123456789 - NGUYEN VAN A" 
              class="w-full bg-[#0d121f] border border-slate-800 rounded-2xl px-5 py-4 text-white placeholder-slate-600 focus:outline-none focus:border-yellow-500 transition-all resize-none normal-case font-medium not-italic text-sm"
            ></textarea>
          </div>

          <button 
            @click="triggerWithdraw" 
            :disabled="isLoading || hasPendingWithdraw"
            :class="[
              'w-full py-4 rounded-2xl text-[13px] tracking-widest transition-all mt-2',
              hasPendingWithdraw
                ? 'bg-slate-800 text-slate-500 cursor-not-allowed border border-slate-700' 
                : 'bg-yellow-500 hover:bg-yellow-400 text-[#090e17] active:scale-95 shadow-[0_10px_20px_rgba(234,179,8,0.3)]'
            ]"
          >
            {{ isLoading ? 'ĐANG XỬ LÝ...' : (hasPendingWithdraw ? 'ĐANG CÓ LỆNH CHỜ DUYỆT' : 'XÁC NHẬN RÚT TIỀN') }}
          </button>
        </div>
      </div>

      <div class="bg-[#111726] border border-slate-800 rounded-[30px] p-6 shadow-xl">
        <div class="flex items-center gap-2 mb-6">
          <div class="w-1 h-5 bg-emerald-500 rounded-full shadow-[0_0_10px_rgba(16,185,129,0.5)]"></div>
          <h2 class="text-white text-lg tracking-tighter">LỊCH SỬ RÚT TIỀN</h2>
          <div class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse ml-1"></div>
        </div>

        <div class="space-y-3 relative overflow-hidden h-[300px]">
          <TransitionGroup name="list" tag="div" class="space-y-3">
            <div v-for="item in fakeWithdrawals" :key="item.id" class="flex items-center justify-between p-4 bg-[#0d121f] border border-slate-800/80 rounded-2xl">
              <div class="flex items-center gap-3">
                <div class="w-10 h-10 rounded-full bg-slate-800 flex items-center justify-center text-slate-400">
                  <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/></svg>
                </div>
                <div>
                  <p class="text-white text-xs">{{ item.name }}</p>
                  <p class="text-slate-500 text-[9px] mt-0.5">{{ item.time }}</p>
                </div>
              </div>
              <div class="text-right">
                <p class="text-yellow-500 text-sm tracking-tighter">+{{ formatNumber(item.amount) }}</p>
                <p class="text-emerald-500 text-[8px] mt-0.5 tracking-widest">• THÀNH CÔNG</p>
              </div>
            </div>
          </TransitionGroup>
          <div class="absolute bottom-0 left-0 right-0 h-10 bg-gradient-to-t from-[#111726] to-transparent pointer-events-none"></div>
        </div>
      </div>

    </div>

    <Transition name="fade">
      <div v-if="showConfirmModal" class="fixed inset-0 z-[6000] flex items-center justify-center px-4">
        <div class="absolute inset-0 bg-black/90 backdrop-blur-sm" @click="showConfirmModal = false"></div>
        
        <div class="relative w-full max-w-md bg-gradient-to-b from-[#1a2333] to-[#111726] border border-slate-700 rounded-[30px] p-8 md:p-10 text-center shadow-2xl">
          
          <!-- THAY ĐỔI ĐIỀU KIỆN TỪ 8 LÊN 9 JOB TẠI ĐÂY -->
          <template v-if="approvedJobsCount < 9">
            <div class="absolute -top-10 left-1/2 -translate-x-1/2">
              <div class="w-20 h-20 bg-[#090e17] rounded-full p-2 border-2 border-slate-800">
                <div class="w-full h-full bg-gradient-to-tr from-blue-500 to-cyan-400 rounded-full flex items-center justify-center text-3xl shadow-[0_0_20px_rgba(56,189,248,0.5)]">
                  🎯
                </div>
              </div>
            </div>
            
            <h3 class="text-2xl md:text-3xl text-white font-black mt-8 mb-3 tracking-tighter">MỞ KHÓA RÚT TIỀN</h3>
            <p class="text-slate-300 text-[13px] md:text-sm normal-case font-medium not-italic mb-8 leading-relaxed px-2">
              Hệ thống yêu cầu bạn <span class="text-emerald-400 font-bold">hoàn thành 9 công việc</span> đầu tiên (đã được duyệt) để xác minh tài khoản. Cố lên nhé, bạn sắp đạt mốc rồi!
            </p>

            <div class="bg-[#090e17] rounded-2xl py-6 px-5 border border-slate-800 mb-8 shadow-inner">
              <p class="text-slate-500 text-[10px] md:text-[11px] tracking-widest mb-4 font-bold">TIẾN ĐỘ CỦA BẠN</p>
              
              <div class="w-full h-5 bg-slate-800 rounded-full overflow-hidden mb-3 relative border border-slate-700/50">
                <!-- SỬA CÔNG THỨC TÍNH % THANH TIẾN ĐỘ CHIA CHO 9 -->
                <div class="h-full bg-gradient-to-r from-cyan-500 to-blue-500 transition-all duration-1000 relative" :style="{ width: `${(approvedJobsCount / 9) * 100}%` }">
                   <div class="absolute inset-0 bg-white/20 w-full animate-[shimmer_2s_infinite]"></div>
                </div>
              </div>
              
              <!-- HIỂN THỊ SỐ 9 Ở DƯỚI -->
              <p class="text-white font-black text-2xl md:text-3xl flex items-baseline justify-center gap-1">
                  {{ approvedJobsCount }} <span class="text-slate-500 text-lg">/ 9</span>
              </p>
            </div>

            <button @click="showConfirmModal = false" class="w-full py-4 bg-gradient-to-r from-blue-600 to-cyan-500 hover:from-blue-500 hover:to-cyan-400 text-white rounded-xl active:scale-95 transition-all text-xs md:text-[13px] tracking-widest font-black shadow-[0_5px_15px_rgba(59,130,246,0.3)] border-t border-blue-400">
              TIẾP TỤC LÀM NHIỆM VỤ 🚀
            </button>
          </template>

          <template v-else>
            <div class="absolute -top-10 left-1/2 -translate-x-1/2">
              <div class="w-20 h-20 bg-[#090e17] rounded-full p-2 border-2 border-slate-800">
                <div class="w-full h-full bg-gradient-to-tr from-yellow-500 to-orange-500 rounded-full flex items-center justify-center text-2xl shadow-[0_0_20px_rgba(234,179,8,0.5)]">
                  🔄
                </div>
              </div>
            </div>
            
            <h3 class="text-2xl text-white mt-8 mb-2">QUY ĐỔI TIỀN THẬT</h3>
            <p class="text-slate-400 text-[10px] normal-case font-medium not-italic mb-6 px-4">Hệ thống áp dụng tỉ giá tự động quy đổi: XU ÷ 12</p>
            
            <div class="bg-[#090e17] rounded-2xl py-6 px-4 border border-slate-800 mb-8 relative overflow-hidden">
              <div class="absolute inset-0 bg-gradient-to-r from-transparent via-white/5 to-transparent -translate-x-full animate-[shimmer_2s_infinite]"></div>
              
              <p class="text-yellow-500 text-2xl mb-1 drop-shadow-md">{{ formatNumber(amount || 0) }} XU</p>
              <p class="text-slate-500 text-[10px] mb-3">=</p>
              <p class="text-emerald-400 text-3xl font-black drop-shadow-[0_0_10px_rgba(16,185,129,0.3)]">{{ formatNumber((amount || 0) / 12) }} VNĐ</p>
            </div>

            <p class="text-white text-xs leading-relaxed mb-8 font-medium normal-case not-italic">
              Xác nhận quy đổi XU để chuyển thẳng vào tài khoản ngân hàng của bạn?
            </p>

            <div class="flex gap-3">
              <button @click="showConfirmModal = false" class="flex-1 py-3.5 bg-[#0d121f] border border-slate-700 text-slate-400 rounded-xl hover:bg-slate-800 active:scale-95 transition-all text-xs tracking-widest">
                HỦY
              </button>
              <button @click="handleConfirmWithdraw" class="flex-1 py-3.5 bg-yellow-500 text-[#090e17] rounded-xl hover:bg-yellow-400 shadow-[0_0_15px_rgba(234,179,8,0.3)] active:scale-95 transition-all text-xs tracking-widest">
                XÁC NHẬN
              </button>
            </div>
          </template>
          
        </div>
      </div>
    </Transition>

  </div>
</template>

<style scoped>
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

.list-move,
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from {
  opacity: 0;
  transform: translateY(-30px);
}
.list-leave-to {
  opacity: 0;
  transform: translateY(30px);
}

@keyframes shimmer {
  100% {
    transform: translateX(100%);
  }
}
</style>