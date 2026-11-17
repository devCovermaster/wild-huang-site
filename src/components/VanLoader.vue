<template>
  <div
    class="van-loader"
    :class="[`theme-${theme}`, { 'is-compact': compact }]"
    role="status"
    aria-live="polite"
  >
    <div class="van-scene">
      <svg
        class="van-svg"
        viewBox="0 0 360 170"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <defs>
          <!-- 手绘素描微抖动滤镜，还原钢笔速写质感 -->
          <filter id="sketch-filter" x="-10%" y="-10%" width="120%" height="120%">
            <feTurbulence
              type="fractalNoise"
              baseFrequency="0.04"
              numOctaves="3"
              result="noise"
            />
            <feDisplacementMap
              in="SourceGraphic"
              in2="noise"
              scale="1.35"
              xChannelSelector="R"
              yChannelSelector="G"
            />
          </filter>
        </defs>

        <g filter="url(#sketch-filter)">
          <!-- 远景：飞鸟与流云 -->
          <g class="sky-elements">
            <path
              class="bird bird-1"
              d="M68 38 C72 34, 76 34, 79 38 C82 34, 86 34, 90 38"
              stroke="currentColor"
              stroke-width="1.6"
              stroke-linecap="round"
            />
            <path
              class="bird bird-2"
              d="M96 28 C99 25, 102 25, 104 28 C106 25, 109 25, 112 28"
              stroke="currentColor"
              stroke-width="1.4"
              stroke-linecap="round"
            />
          </g>

          <!-- 远景掠过的松树与帐篷剪影（循环向后移动） -->
          <g class="passing-scenery">
            <!-- 小松树 -->
            <path
              d="M315 128 L315 116 M315 100 L307 113 L323 113 Z M315 107 L305 120 L325 120 Z"
              stroke="currentColor"
              stroke-width="1.6"
              stroke-linecap="round"
              stroke-linejoin="round"
              opacity="0.45"
            />
            <!-- 远方小山丘/仙人掌草丛 -->
            <path
              d="M46 128 L43 119 M46 128 L46 117 M46 128 L50 120"
              stroke="currentColor"
              stroke-width="1.5"
              stroke-linecap="round"
              opacity="0.4"
            />
          </g>

          <!-- 车后卷曲的尾气与清风线条（复刻参考图1的手绘卷曲线） -->
          <g class="wind-swirls">
            <path
              class="wind-line wind-1"
              d="M112 94 C94 96, 82 84, 65 86 C48 88, 36 96, 26 91 C19 87, 22 78, 30 80 C36 82, 34 89, 29 90"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              fill="none"
            />
            <path
              class="wind-line wind-2"
              d="M118 105 C102 108, 92 100, 78 102 C65 104, 60 95, 67 93 C72 91, 74 97, 69 99"
              stroke="currentColor"
              stroke-width="1.8"
              stroke-linecap="round"
              fill="none"
            />
            <circle class="puff puff-1" cx="102" cy="112" r="2.2" fill="currentColor" />
            <circle class="puff puff-2" cx="88" cy="110" r="1.6" fill="currentColor" />
          </g>

          <!-- 露营车主体（随引擎节奏轻微颠簸） -->
          <g class="camper-van">
            <!-- 车顶冲浪板与尾鳍（带温暖复古配色点缀） -->
            <g class="surfboard-group">
              <!-- 下层冲浪板 -->
              <path
                d="M142 54 C162 47, 212 46, 240 52 C244 53, 244 56, 239 56 C210 55, 165 56, 142 57 C138 57, 138 55, 142 54 Z"
                class="surfboard-fill"
                stroke="currentColor"
                stroke-width="2.1"
                stroke-linejoin="round"
              />
              <!-- 冲浪板尾鳍 1 -->
              <path
                d="M202 48 C206 38, 214 35, 216 36 C217 37, 213 44, 212 48"
                class="surfboard-fill"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
              <!-- 冲浪板尾鳍 2 -->
              <path
                d="M222 50 C226 42, 233 40, 235 41 C236 42, 232 47, 230 51"
                class="surfboard-fill"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
              <!-- 车顶行李架绑带 -->
              <path
                d="M168 48 L170 61 M198 47 L199 60 M222 49 L224 61"
                stroke="currentColor"
                stroke-width="2.6"
                stroke-linecap="round"
              />
            </g>

            <!-- 露营车外壳轮廓 -->
            <path
              d="M138 117
                 L138 82
                 C138 67, 145 60, 160 60
                 L234 60
                 C248 60, 256 67, 261 82
                 L266 98
                 C267 103, 267 110, 266 116
                 L248 116
                 C246 105, 236 102, 226 102
                 C216 102, 208 106, 206 117
                 L178 117
                 C176 106, 168 102, 158 102
                 C148 102, 141 106, 139 117 Z"
              class="van-body-fill"
              stroke="currentColor"
              stroke-width="2.5"
              stroke-linecap="round"
              stroke-linejoin="round"
            />

            <!-- 车顶双色分界横线与腰线 -->
            <path
              d="M142 66 L252 66"
              stroke="currentColor"
              stroke-width="1.8"
              stroke-linecap="round"
            />
            <path
              d="M138 91 L252 91"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
            />

            <!-- 车门分割竖线与前脸弧线 -->
            <path
              d="M182 66 L182 116 M215 66 L215 114"
              stroke="currentColor"
              stroke-width="1.9"
              stroke-linecap="round"
            />
            <path
              d="M215 91 C220 98, 226 101, 252 101 C258 101, 260 107, 257 111"
              stroke="currentColor"
              stroke-width="1.9"
              stroke-linecap="round"
              fill="none"
            />

            <!-- 三扇复古车窗（带速写斜线阴影与小窗帘感） -->
            <!-- 后窗 -->
            <path
              d="M145 72 L176 72 L176 86 L143 86 C143 80, 144 75, 145 72 Z"
              class="van-window"
              stroke="currentColor"
              stroke-width="2"
              stroke-linejoin="round"
            />
            <path
              d="M150 74 L146 84 M156 74 L152 84 M168 74 L164 84"
              stroke="currentColor"
              stroke-width="1.4"
              stroke-linecap="round"
              opacity="0.55"
            />

            <!-- 中窗 -->
            <rect
              x="187"
              y="72"
              width="23"
              height="14"
              rx="2"
              class="van-window"
              stroke="currentColor"
              stroke-width="2"
            />
            <path
              d="M194 74 L190 84 M201 74 L197 84"
              stroke="currentColor"
              stroke-width="1.4"
              stroke-linecap="round"
              opacity="0.55"
            />

            <!-- 前驾驶窗 -->
            <path
              d="M221 72 L244 72 C247 76, 249 81, 250 86 L221 86 Z"
              class="van-window"
              stroke="currentColor"
              stroke-width="2"
              stroke-linejoin="round"
            />
            <path
              d="M228 74 L224 84 M235 74 L231 84"
              stroke="currentColor"
              stroke-width="1.4"
              stroke-linecap="round"
              opacity="0.55"
            />

            <!-- 后侧散热百叶窗 -->
            <path
              d="M148 96 L159 96 M148 99.5 L159 99.5 M148 103 L159 103 M148 95.5 L148 103.5 M159 95.5 L159 103.5"
              stroke="currentColor"
              stroke-width="1.4"
              stroke-linecap="round"
            />

            <!-- 车门小把手 -->
            <path
              d="M204 96 L209 96 M221 96 L226 96"
              stroke="currentColor"
              stroke-width="2.1"
              stroke-linecap="round"
            />

            <!-- 复古圆头灯 & 光束 -->
            <ellipse
              cx="267"
              cy="101"
              rx="3.2"
              ry="5.5"
              class="van-body-fill"
              stroke="currentColor"
              stroke-width="2.2"
            />
            <path
              class="headlight-beam"
              d="M273 98 L308 88 L308 112 L273 104 Z"
              fill="currentColor"
              opacity="0.08"
            />

            <!-- 前后保险杠 -->
            <path
              d="M132 117 C132 114, 136 113, 141 114 L141 120 L134 120 C132 120, 132 118, 132 117 Z"
              class="van-body-fill"
              stroke="currentColor"
              stroke-width="2.2"
              stroke-linejoin="round"
            />
            <path
              d="M258 116 L269 116 C272 116, 273 121, 269 121 L256 121 Z"
              class="van-body-fill"
              stroke="currentColor"
              stroke-width="2.2"
              stroke-linejoin="round"
            />
          </g>

          <!-- 前后车轮（独立旋转动画） -->
          <!-- 后轮 -->
          <g class="wheel wheel-rear">
            <circle
              cx="158"
              cy="120"
              r="11.5"
              fill="currentColor"
            />
            <circle
              cx="158"
              cy="120"
              r="5.5"
              class="wheel-hub"
            />
            <path
              d="M158 113 L158 127 M151 120 L165 120"
              stroke="currentColor"
              stroke-width="1.5"
              opacity="0.45"
            />
          </g>

          <!-- 前轮 -->
          <g class="wheel wheel-front">
            <circle
              cx="236"
              cy="120"
              r="11.5"
              fill="currentColor"
            />
            <circle
              cx="236"
              cy="120"
              r="5.5"
              class="wheel-hub"
            />
            <path
              d="M236 113 L236 127 M229 120 L243 120"
              stroke="currentColor"
              stroke-width="1.5"
              opacity="0.45"
            />
          </g>

          <!-- 地面速写公路线与车底阴影 -->
          <ellipse cx="198" cy="132" rx="56" ry="2.6" fill="currentColor" opacity="0.7" />
          <path
            d="M40 132 L320 132"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
          />
          <path
            class="road-dashes"
            d="M22 135 L35 135 M75 136 L98 136 M280 135 L305 135 M326 132 L338 132"
            stroke="currentColor"
            stroke-width="1.4"
            stroke-linecap="round"
            opacity="0.55"
          />
        </g>
      </svg>
    </div>

    <!-- 手写感文案与进度条 -->
    <div class="van-caption">
      <p class="van-slogan">{{ text }}</p>
      <div v-if="showProgress" class="van-progress-wrap">
        <div class="van-progress-track">
          <div class="van-progress-bar" :style="{ width: `${progress}%` }"></div>
        </div>
        <span class="van-progress-num">{{ Math.round(progress) }}%</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
interface Props {
  theme?: 'light' | 'dark'
  compact?: boolean
  text?: string
  progress?: number
  showProgress?: boolean
}

withDefaults(defineProps<Props>(), {
  theme: 'light',
  compact: false,
  text: 'WANDERING IN THE WILD...',
  progress: 0,
  showProgress: false
})
</script>

<style scoped>
.van-loader {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  user-select: none;
  padding: 24px;
}

.van-loader.theme-light {
  color: #181818;
  --van-body-bg: #fafafa;
  --van-window-bg: #222222;
  --van-surf-bg: #e6b89c;
  --van-hub-bg: #fafafa;
}

.van-loader.theme-dark {
  color: #f3f3ee;
  --van-body-bg: #161618;
  --van-window-bg: #f3f3ee;
  --van-surf-bg: #d98a5f;
  --van-hub-bg: #161618;
}

.van-scene {
  width: 300px;
  max-width: 82vw;
}

.van-loader.is-compact .van-scene {
  width: 230px;
}

.van-svg {
  width: 100%;
  height: auto;
  overflow: visible;
  display: block;
}

.van-body-fill {
  fill: var(--van-body-bg);
}

.van-window {
  fill: var(--van-window-bg);
  fill-opacity: 0.88;
}

.surfboard-fill {
  fill: var(--van-surf-bg);
}

.wheel-hub {
  fill: var(--van-hub-bg);
}

/* 露营车行驶颠簸动画 */
.camper-van {
  transform-origin: 200px 110px;
  animation: van-chug 0.65s ease-in-out infinite alternate;
}

@keyframes van-chug {
  0% {
    transform: translateY(0px) rotate(-0.4deg);
  }
  50% {
    transform: translateY(-2.4px) rotate(0.5deg);
  }
  100% {
    transform: translateY(0.6px) rotate(-0.2deg);
  }
}

/* 车轮旋转动画 */
.wheel-rear {
  transform-origin: 158px 120px;
  animation: wheel-spin 0.7s linear infinite;
}

.wheel-front {
  transform-origin: 236px 120px;
  animation: wheel-spin 0.7s linear infinite;
}

@keyframes wheel-spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* 尾气与微风流动动画 */
.wind-1 {
  stroke-dasharray: 140;
  stroke-dashoffset: 0;
  animation: wind-flow 1.4s ease-in-out infinite;
}

.wind-2 {
  stroke-dasharray: 90;
  stroke-dashoffset: 0;
  animation: wind-flow 1.4s ease-in-out infinite 0.25s;
}

@keyframes wind-flow {
  0% {
    opacity: 0.2;
    transform: translateX(6px);
  }
  50% {
    opacity: 0.95;
    transform: translateX(-3px);
  }
  100% {
    opacity: 0.2;
    transform: translateX(-10px);
  }
}

.puff-1 {
  animation: puff-float 1.1s ease-out infinite;
}

.puff-2 {
  animation: puff-float 1.1s ease-out infinite 0.4s;
}

@keyframes puff-float {
  0% {
    opacity: 0.85;
    transform: translate(0, 0) scale(0.8);
  }
  100% {
    opacity: 0;
    transform: translate(-18px, -6px) scale(1.5);
  }
}

/* 远景掠过动画 */
.passing-scenery {
  animation: scenery-pass 2.8s linear infinite;
}

@keyframes scenery-pass {
  0% {
    transform: translateX(30px);
    opacity: 0;
  }
  15% {
    opacity: 1;
  }
  85% {
    opacity: 1;
  }
  100% {
    transform: translateX(-260px);
    opacity: 0;
  }
}

.bird-1 {
  animation: bird-bob 1.8s ease-in-out infinite alternate;
}

.bird-2 {
  animation: bird-bob 1.8s ease-in-out infinite alternate-reverse;
}

@keyframes bird-bob {
  from {
    transform: translateY(0px);
  }
  to {
    transform: translateY(-4px);
  }
}

/* 底部文字与进度条 */
.van-caption {
  margin-top: 8px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.van-slogan {
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 3px;
  text-transform: uppercase;
  opacity: 0.78;
  margin: 0;
}

.van-progress-wrap {
  display: flex;
  align-items: center;
  gap: 10px;
}

.van-progress-track {
  width: 140px;
  height: 2px;
  background: rgba(128, 128, 128, 0.22);
  border-radius: 999px;
  overflow: hidden;
  position: relative;
}

.van-progress-bar {
  height: 100%;
  background: currentColor;
  transition: width 0.25s ease-out;
}

.van-progress-num {
  font-size: 11px;
  font-variant-numeric: tabular-nums;
  letter-spacing: 1px;
  opacity: 0.65;
  min-width: 34px;
}
</style>
