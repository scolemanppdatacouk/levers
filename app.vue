<template>
  <div class="min-h-screen flex flex-col bg-[#F1F1F1] font-[Arial,sans-serif]">

    <!-- ── Header ── -->
    <header class="bg-[#005159] px-12 py-5 flex items-center justify-between shadow-lg">
      <div class="flex items-center gap-4">
        <img
          src="https://pictureperfectdata.co.uk/wp-content/uploads/2025/01/picture-perfect-data-logo-white.svg"
          class="h-12"
          alt="Picture Perfect Data"
          @error="logoError = true"
        />
        <div v-if="logoError">
          <div class="text-[#BFCD00] font-bold text-xl tracking-wide">Picture Perfect Data</div>
          <div class="text-[#00A1A3] text-xs tracking-widest uppercase">For better decisions</div>
        </div>
      </div>

      <div class="text-center mr-8">
        <h1 class="text-3xl font-bold text-white tracking-tight">Workforce Planning Model</h1>
        <p class="text-[#BFCD00] text-sm italic mt-1">"Doing more, with less" becomes a reality when you're making better decisions</p>
      </div>

      <div class="flex items-start gap-8">
        <!-- Weekly Task Demand -->
        <div class="text-right">
          <div class="text-[#99A9B5] text-xs uppercase tracking-widest mb-1">Weekly Task Demand</div>
          <div class="flex items-center justify-end gap-3 my-1">
            <button @click="demand = Math.max(1_000, demand - 1_000)"
              class="w-8 h-8 rounded-full bg-[#BFCD00] text-[#005159] text-xl font-bold flex items-center justify-center hover:bg-white transition-colors select-none">−</button>
            <input
              type="number"
              :value="demand / 1000"
              @change="demand = Math.max(1, Math.round(Number(($event.target as HTMLInputElement).value))) * 1_000"
              class="w-20 text-center text-3xl font-bold text-[#BFCD00] bg-transparent border-b-2 border-[#BFCD00] focus:outline-none focus:border-white [appearance:textfield] [&::-webkit-inner-spin-button]:appearance-none"
            /><span class="text-[#BFCD00] text-3xl font-bold">K</span>
            <button @click="demand = demand + 1_000"
              class="w-8 h-8 rounded-full bg-[#BFCD00] text-[#005159] text-xl font-bold flex items-center justify-center hover:bg-white transition-colors select-none">+</button>
          </div>
          <div class="text-[#99A9B5] text-xs">tasks / week</div>
        </div>

        <!-- Optimum Util % -->
        <div class="text-right">
          <div class="text-[#99A9B5] text-xs uppercase tracking-widest mb-1">Optimum Util %</div>
          <input type="number" v-model.number="optimumUtil" min="1" max="100"
            class="w-16 text-center text-3xl font-bold text-[#BFCD00] bg-transparent border-b-2 border-[#BFCD00] focus:outline-none focus:border-white [appearance:textfield] [&::-webkit-inner-spin-button]:appearance-none" />
          <div class="text-[#99A9B5] text-xs mt-1">% threshold</div>
        </div>

        <!-- Optimum Tasks/hr -->
        <div class="text-right">
          <div class="text-[#99A9B5] text-xs uppercase tracking-widest mb-1">Optimum Tasks/hr</div>
          <input type="number" v-model.number="optimumProductivity" min="10" max="499"
            class="w-16 text-center text-3xl font-bold text-[#BFCD00] bg-transparent border-b-2 border-[#BFCD00] focus:outline-none focus:border-white [appearance:textfield] [&::-webkit-inner-spin-button]:appearance-none" />
          <div class="text-[#99A9B5] text-xs mt-1">tasks/hr threshold</div>
        </div>
      </div>
    </header>

    <!-- ── Main ── -->
    <main class="flex-1 px-12 py-6 flex flex-col gap-5">

      <!-- KPI Row -->
      <div class="grid grid-cols-5 gap-5">

        <!-- Operatives -->
        <div class="bg-white rounded-xl p-6 shadow flex flex-col items-center border-t-4 border-[#BFCD00]">
          <div class="text-xs uppercase tracking-widest text-[#99A9B5] mb-2">Operatives</div>
          <div class="flex items-baseline gap-2 my-2">
            <span class="text-5xl font-bold text-[#005159]">{{ Number.isInteger(operatives) ? operatives : operatives.toFixed(1) }}</span>
            <span class="text-xl text-[#99A9B5]">({{ Number.isInteger(OPTIMUM_OPS) ? OPTIMUM_OPS : OPTIMUM_OPS.toFixed(1) }})</span>
          </div>
          <div class="flex items-center gap-3 mt-2">
            <button
              @click="changeOperatives(-1)"
              class="w-11 h-11 rounded-full bg-[#005159] text-white text-2xl flex items-center justify-center hover:bg-[#00A1A3] transition-colors active:scale-95 select-none"
            >−</button>
            <button
              @click="changeOperatives(1)"
              class="w-11 h-11 rounded-full bg-[#005159] text-white text-2xl flex items-center justify-center hover:bg-[#00A1A3] transition-colors active:scale-95 select-none"
            >+</button>
          </div>
          <div class="text-[#99A9B5] text-xs mt-3">{{ HOURS_PER_WEEK }} hrs/wk each</div>
        </div>

        <!-- Hours card -->
        <div class="bg-white rounded-xl p-6 shadow flex flex-col items-center border-t-4 border-[#00A1A3]">
          <div class="text-xs uppercase tracking-widest text-[#99A9B5] mb-1">Hours Available</div>
          <div class="text-4xl font-bold text-[#005159] mb-3">{{ hoursAvailable.toLocaleString() }}</div>
          <div class="w-full h-px bg-[#E5E7EB] mb-3"></div>
          <div class="w-full flex justify-around">
            <div class="flex flex-col items-center">
              <div class="text-2xl font-bold text-[#005159]">{{ Math.round(hoursAvailable * 0.9).toLocaleString() }}</div>
              <div class="text-[#99A9B5] text-xs mt-1">optimum (90%)</div>
            </div>
            <div class="w-px bg-[#E5E7EB]"></div>
            <div class="flex flex-col items-center">
              <div class="text-2xl font-bold text-[#005159]">{{ Math.round(hoursUtilised).toLocaleString() }}</div>
              <div class="text-[#99A9B5] text-xs mt-1">utilised ({{ utilisation }}%)</div>
            </div>
          </div>
        </div>

        <!-- Hours Wasted -->
        <div class="bg-white rounded-xl p-6 shadow flex flex-col items-center border-t-4"
          :class="{ 'border-red-500': wastedTier === 'red', 'border-orange-400': wastedTier === 'orange', 'border-[#00A1A3]': wastedTier === 'base' }">
          <div class="text-xs uppercase tracking-widest text-[#99A9B5] mb-3">Hours Wasted</div>
          <div class="text-5xl font-bold my-2"
            :class="{ 'text-red-500': wastedTier === 'red', 'text-orange-500': wastedTier === 'orange', 'text-[#005159]': wastedTier === 'base' }">
            {{ Math.abs(Math.round(hoursWasted)).toLocaleString() }}
          </div>
          <div class="text-[#99A9B5] text-sm">hrs / week</div>
          <div class="text-xs mt-3"
            :class="{ 'text-red-400': wastedTier === 'red', 'text-orange-400': wastedTier === 'orange', 'text-[#00A1A3]': wastedTier === 'base' }">
            {{ hoursWasted > 0 ? 'surplus capacity' : hoursWasted < 0 ? 'hours short' : 'exact match' }}
          </div>
        </div>

        <!-- Tasks Completed -->
        <div class="bg-white rounded-xl p-6 shadow flex flex-col items-center border-t-4 border-[#00A1A3]">
          <div class="text-xs uppercase tracking-widest text-[#99A9B5] mb-2">Task Capacity</div>
          <div class="text-5xl font-bold text-[#005159] my-2">
            {{ formatNumber(tasksCompleted) }}<span v-if="surplusCapacity" class="relative group text-orange-500 ml-1 cursor-default">+
              <span class="absolute bottom-full left-1/2 -translate-x-1/2 mb-2 hidden group-hover:block bg-[#005159] text-white text-xs rounded-lg px-3 py-2 whitespace-nowrap shadow-lg z-10">
                Optimum capacity: {{ formatNumber(optimumTasks) }} tasks<br>
                <span class="text-[#99A9B5]">{{ operatives }} ops × 40hrs × {{ optimumUtil }}% × {{ optimumProductivity }}/hr</span>
              </span>
            </span>
          </div>
          <div class="text-[#99A9B5] text-sm">tasks / week</div>
          <div v-if="qualityFactor < 1" class="text-orange-500 text-xs mt-2 font-semibold text-center">
            {{ Math.round(qualityFactor * 100) }}% quality — {{ formatNumber(effectiveTasks) }} effective
          </div>
          <div v-else class="text-[#99A9B5] text-xs mt-3">{{ Math.round(hoursUtilised).toLocaleString() }}hrs × {{ productivity }}/hr</div>
        </div>

        <!-- SLA Status -->
        <div class="bg-white rounded-xl p-6 shadow flex flex-col items-center border-t-4" :class="badge.borderClass">
          <div class="text-xs uppercase tracking-widest text-[#99A9B5] mb-2">SLA Achieved</div>
          <div class="text-5xl font-bold my-2" :class="badge.textClass">{{ sla.toFixed(1) }}%</div>
          <div class="px-3 py-1 rounded-full text-xs font-bold mt-1" :class="badge.pillClass">{{ badge.label }}</div>
          <div class="text-[#99A9B5] text-xs mt-3">
            {{ formatNumber(tasksCompleted) }} / {{ formatNumber(demand) }}
          </div>
        </div>

      </div>

      <!-- ── Banner ── -->
      <div class="bg-[#005159] rounded-xl px-10 py-4 shadow text-center">
        <p class="text-white text-lg italic">You can ensure people are kept busy, or that they work more efficiently, or add in more people. Which would you prefer?</p>
      </div>

      <!-- ── Utilisation Slider ── -->
      <div class="bg-white rounded-xl px-10 py-7 shadow" :class="{ 'ring-2 ring-orange-400': utilisation > optimumUtil }">
        <div class="flex items-start justify-between mb-5">
          <div>
            <a href="https://tableau.ppdata.co.uk/#/site/Intralogistex2026/views/2026IntraLogistexShowcase/Utilisation" target="_blank" class="text-xs uppercase tracking-[0.2em] text-[#005159] font-bold mb-1 hover:text-[#00A1A3] hover:underline cursor-pointer transition-colors">Utilisation ↗</a>
            <div class="text-sm text-[#99A9B5]">% of available hours actively worked</div>
          </div>
          <div class="text-right">
            <span class="text-6xl font-bold" :class="utilisation > optimumUtil ? 'text-orange-500' : 'text-[#005159]'">{{ utilisation }}</span>
            <span class="text-2xl text-[#99A9B5] ml-1">%</span>
          </div>
        </div>
        <input
          type="range"
          class="ppd-slider"
          min="0" max="100" step="1"
          :value="utilisation"
          ref="utilisationSliderEl"
          @input="utilisation = Number(($event.target as HTMLInputElement).value)"
        />
        <div class="relative h-5 mt-2 text-xs">
          <span class="absolute left-0 text-[#99A9B5]">0%</span>
          <span class="absolute -translate-x-1/2 text-[#99A9B5]" style="left:25%">25%</span>
          <span class="absolute -translate-x-1/2 text-[#99A9B5]" style="left:50%">50%</span>
          <span class="absolute -translate-x-1/2 text-[#99A9B5]" style="left:75%">75%</span>
          <span class="absolute -translate-x-1/2 text-orange-400 font-semibold" :style="{ left: optimumUtil + '%' }">{{ optimumUtil }}%⚠</span>
          <span class="absolute right-0 text-[#99A9B5]">100%</span>
        </div>
        <div v-if="utilisation > optimumUtil" class="mt-3 flex items-center gap-2 text-orange-500 text-sm font-semibold">
          <span>⚠ Overworked — quality degrading</span>
          <span class="bg-orange-100 text-orange-600 px-2 py-0.5 rounded-full text-xs font-bold">−{{ utilisationPenalty }}% quality</span>
        </div>
      </div>

      <!-- ── Productivity Slider ── -->
      <div class="bg-white rounded-xl px-10 py-7 shadow" :class="{ 'ring-2 ring-orange-400': productivity > optimumProductivity }">
        <div class="flex items-start justify-between mb-5">
          <div>
            <a href="https://tableau.ppdata.co.uk/#/site/Intralogistex2026/views/2026IntraLogistexShowcase/WarehouseSelect" target="_blank" class="text-xs uppercase tracking-[0.2em] text-[#005159] font-bold mb-1 hover:text-[#00A1A3] hover:underline cursor-pointer transition-colors">Productivity ↗</a>
            <div class="text-sm text-[#99A9B5]">tasks completed per operative per hour</div>
          </div>
          <div class="text-right">
            <span class="text-6xl font-bold" :class="productivity > optimumProductivity ? 'text-orange-500' : 'text-[#005159]'">{{ productivity }}</span>
            <span class="text-2xl text-[#99A9B5] ml-1">tasks / hr</span>
          </div>
        </div>
        <input
          type="range"
          class="ppd-slider"
          min="10" max="500" step="5"
          :value="productivity"
          ref="productivitySliderEl"
          @input="onProductivityChange(Number(($event.target as HTMLInputElement).value))"
        />
        <div class="relative h-5 mt-2 text-xs">
          <span class="absolute left-0 text-[#99A9B5]">10</span>
          <span class="absolute -translate-x-1/2 text-[#99A9B5]" style="left:25%">133</span>
          <span class="absolute -translate-x-1/2 text-[#99A9B5]" style="left:50%">255</span>
          <span class="absolute -translate-x-1/2 text-orange-400 font-semibold"
            :style="{ left: productivityThresholdPct }">{{ optimumProductivity }}⚠</span>
          <span class="absolute right-0 text-[#99A9B5]">500</span>
        </div>
        <div v-if="productivity > optimumProductivity" class="mt-3 flex items-center gap-2 text-orange-500 text-sm font-semibold">
          <span>⚠ Rushing — error rate rising</span>
          <span class="bg-orange-100 text-orange-600 px-2 py-0.5 rounded-full text-xs font-bold">−{{ productivityPenalty }}% quality</span>
        </div>
      </div>

      <!-- ── SLA% Slider ── -->
      <div class="bg-white rounded-xl px-10 py-7 shadow">
        <div class="flex items-start justify-between mb-5">
          <div>
            <a href="https://tableau.ppdata.co.uk/#/site/Intralogistex2026/views/2026IntraLogistexShowcase/SLAPerformance" target="_blank" class="text-xs uppercase tracking-[0.2em] text-[#005159] font-bold mb-1 hover:text-[#00A1A3] hover:underline cursor-pointer transition-colors">SLA Target ↗</a>
            <div class="text-sm text-[#99A9B5]">
              set a target % of weekly demand — adjusts number of operatives to achieve it
              <span v-if="qualityFactor < 1" class="text-orange-500 font-semibold ml-2">
                (quality factor: {{ Math.round(qualityFactor * 100) }}%)
              </span>
            </div>
          </div>
          <div class="flex items-center gap-5">
            <span class="px-5 py-2 rounded-full text-sm font-bold" :class="badge.pillClass">{{ badge.label }}</span>
            <div class="text-right">
              <span class="text-6xl font-bold" :class="badge.textClass">{{ sla.toFixed(1) }}</span>
              <span class="text-2xl text-[#99A9B5] ml-1">%</span>
            </div>
          </div>
        </div>
        <input
          type="range"
          class="ppd-slider"
          min="0" max="100" step="0.5"
          :value="sla"
          ref="slaSliderEl"
          @input="onSlaChange(Number(($event.target as HTMLInputElement).value))"
        />
        <div class="flex justify-between text-xs text-[#99A9B5] mt-2 px-1">
          <span>0%</span><span>25%</span><span>50%</span><span>75%</span><span>100%</span>
        </div>
      </div>

    </main>

    <!-- ── Footer ── -->
    <footer class="bg-[#005159] text-[#99A9B5] text-center py-3 text-xs tracking-wide">
      Picture Perfect Data Ltd &nbsp;|&nbsp; Pure Offices, 4100 Park Approach, Leeds, LS15 8GB
      &nbsp;|&nbsp; pictureperfectdata.co.uk &nbsp;|&nbsp; 0113 483 0870
    </footer>

  </div>
</template>

<script setup lang="ts">
// ── Constants ──────────────────────────────────────────────────────────────
const HOURS_PER_WEEK = 40

// ── State ──────────────────────────────────────────────────────────────────
const demand            = ref(352_000)  // weekly task demand, editable in steps of 1,000
const operatives        = ref(50)
const utilisation       = ref(55)       // % of hours actually worked
const productivity      = ref(255)      // tasks per operative per hour
const optimumUtil       = ref(90)       // optimum utilisation % threshold
const optimumProductivity = ref(380)    // optimum tasks/hr threshold
const logoError         = ref(false)

// Optimum headcount reacts to demand, optimumUtil and optimumProductivity
const OPTIMUM_OPS = computed(() => Math.ceil((demand.value / (HOURS_PER_WEEK * (optimumUtil.value / 100) * optimumProductivity.value)) * 2) / 2)

// ── Slider gradients — computed so Vue re-renders when optimum thresholds change ──
const utilisationGradient  = computed(() => sliderGradient(utilisation.value, 0, 100, optimumUtil.value))
const productivityGradient = computed(() => sliderGradient(productivity.value, 10, 500, optimumProductivity.value))
// Position of the productivity threshold marker as a % of the track
const productivityThresholdPct = computed(() => ((optimumProductivity.value - 10) / (500 - 10) * 100).toFixed(1) + '%')

// ── Derived metrics ────────────────────────────────────────────────────────
const hoursAvailable  = computed(() => operatives.value * HOURS_PER_WEEK)
const hoursUtilised   = computed(() => hoursAvailable.value * (utilisation.value / 100))
const operativesNeeded = computed(() => demand.value / (HOURS_PER_WEEK * (optimumUtil.value / 100) * productivity.value))
const hoursWasted      = computed(() => (operatives.value - OPTIMUM_OPS.value) * HOURS_PER_WEEK * (optimumUtil.value / 100))
const wastedTier       = computed(() => {
  const w = hoursWasted.value
  const redAt = OPTIMUM_OPS.value * HOURS_PER_WEEK * (optimumUtil.value / 100) * 0.1
  if (w >= redAt) return 'red'
  if (w > 40)     return 'orange'
  return 'base'
})
const rawThroughput    = computed(() => hoursUtilised.value * productivity.value)
const tasksCompleted   = computed(() => Math.min(rawThroughput.value, demand.value))
const surplusCapacity  = computed(() => rawThroughput.value > demand.value)
const optimumTasks     = computed(() => operatives.value * HOURS_PER_WEEK * (optimumUtil.value / 100) * optimumProductivity.value)

// ── Quality degradation ────────────────────────────────────────────────────
// Above optimumUtil% utilisation people start making mistakes — up to 30% quality loss at 100%
const utilisationDegFactor = computed(() => {
  if (utilisation.value <= optimumUtil.value) return 1
  return 1 - ((utilisation.value - optimumUtil.value) / (100 - optimumUtil.value)) * 0.30
})
// Above optimumProductivity tasks/hr people start rushing — up to 25% quality loss at 500
const productivityDegFactor = computed(() => {
  if (productivity.value <= optimumProductivity.value) return 1
  return 1 - ((productivity.value - optimumProductivity.value) / (500 - optimumProductivity.value)) * 0.25
})
// Quality degrades from overwork (util > 90%) and rushing (productivity > 380)
const qualityFactor   = computed(() => utilisationDegFactor.value * productivityDegFactor.value)
const effectiveTasks  = computed(() => tasksCompleted.value * qualityFactor.value)
// SLA = capacity coverage × quality factor
// 100% when demand is met and util ≤ 90%; degrades if under-capacity or util > 90%
const sla             = computed(() => Math.min(tasksCompleted.value / demand.value, 1) * qualityFactor.value * 100)

// Penalty % shown on each slider when in danger zone
const utilisationPenalty = computed(() => Math.round((1 - utilisationDegFactor.value) * 100))
const productivityPenalty = computed(() => Math.round((1 - productivityDegFactor.value) * 100))

// ── SLA status badge ───────────────────────────────────────────────────────
const badge = computed(() => {
  const s = sla.value
  if (s >= 100) return {
    label: 'EXCELLENT',
    textClass: 'text-[#005159]',
    borderClass: 'border-[#BFCD00]',
    pillClass: 'bg-[#BFCD00] text-[#3C3950]',
  }
  if (s > 98) return {
    label: 'WARNING',
    textClass: 'text-orange-500',
    borderClass: 'border-orange-400',
    pillClass: 'bg-orange-400 text-white',
  }
  return {
    label: 'AT RISK',
    textClass: 'text-red-500',
    borderClass: 'border-red-500',
    pillClass: 'bg-red-500 text-white',
  }
})

// ── Slider DOM refs — background set imperatively so it always repaints ────
const utilisationSliderEl  = ref<HTMLInputElement | null>(null)
const productivitySliderEl = ref<HTMLInputElement | null>(null)
const slaSliderEl          = ref<HTMLInputElement | null>(null)

watchEffect(() => {
  if (utilisationSliderEl.value)
    utilisationSliderEl.value.style.background = utilisationGradient.value
}, { flush: 'post' })

watchEffect(() => {
  if (productivitySliderEl.value)
    productivitySliderEl.value.style.background = productivityGradient.value
}, { flush: 'post' })

watchEffect(() => {
  if (slaSliderEl.value)
    slaSliderEl.value.style.background = sliderGradient(sla.value, 0, 100)
}, { flush: 'post' })

// ── Helpers ────────────────────────────────────────────────────────────────
function sliderGradient(value: number, min: number, max: number, warnAt?: number): string {
  const total = max - min
  const pct = ((value - min) / total) * 100
  if (warnAt === undefined || value <= warnAt) {
    return `linear-gradient(to right, #00A1A3 0%, #00A1A3 ${pct}%, #E5E7EB ${pct}%, #E5E7EB 100%)`
  }
  const warnPct = ((warnAt - min) / total) * 100
  return `linear-gradient(to right, #00A1A3 0%, #00A1A3 ${warnPct}%, #F97316 ${warnPct}%, #F97316 ${pct}%, #E5E7EB ${pct}%, #E5E7EB 100%)`
}

function formatNumber(n: number): string {
  if (n >= 1_000_000) return (n / 1_000_000).toFixed(2) + 'M'
  if (n >= 1_000)     return Math.round(n / 1_000) + 'K'
  return Math.round(n).toLocaleString()
}

// ── Productivity change — utilisation stays fixed, SLA updates
function onProductivityChange(newProductivity: number) {
  productivity.value = newProductivity
}

// ── Operatives change — utilisation stays fixed, SLA and tasks update
function changeOperatives(delta: number) {
  operatives.value = Math.max(1, operatives.value + delta)
}

function roundHalf(n: number): number {
  return Math.round(n * 2) / 2
}

// ── SLA slider drives operatives back-calculation ──────────────────────────
function onSlaChange(targetSla: number) {
  const requiredTasks = (targetSla / 100) * demand.value
  const requiredOps   = requiredTasks / (HOURS_PER_WEEK * (utilisation.value / 100) * productivity.value * qualityFactor.value)
  operatives.value    = Math.max(0.5, roundHalf(requiredOps))
}
</script>

<style>
/* ── Custom slider styling ── */
.ppd-slider {
  -webkit-appearance: none;
  appearance: none;
  width: 100%;
  height: 20px;
  border-radius: 10px;
  outline: none;
  cursor: pointer;
}

/* WebKit thumb */
.ppd-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: #005159;
  cursor: pointer;
  box-shadow: 0 4px 16px rgba(0, 81, 89, 0.4);
  border: 4px solid white;
  transition: background 0.15s, transform 0.1s;
}

.ppd-slider::-webkit-slider-thumb:hover {
  background: #BFCD00;
}

.ppd-slider::-webkit-slider-thumb:active {
  transform: scale(1.15);
  background: #BFCD00;
}

/* Firefox thumb */
.ppd-slider::-moz-range-thumb {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: #005159;
  cursor: pointer;
  border: 4px solid white;
  box-shadow: 0 4px 16px rgba(0, 81, 89, 0.4);
  transition: background 0.15s;
}

.ppd-slider::-moz-range-thumb:hover {
  background: #BFCD00;
}
</style>
