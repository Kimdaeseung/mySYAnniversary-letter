<template>
  <div class="app" @click="handleFirstTouch">
    <div class="bg"></div>

    <main class="paper-wrap">
      <article class="paper">
        <header class="paper__top">
          <span class="seal">❤</span>
          <span class="to">To. {{ toName }}</span>
        </header>

        <section class="paper__body">
          <p class="text">
            <span
              v-for="(char, idx) in displayedChars"
              :key="idx"
              class="char"
            >
              {{ char }}
            </span>
          </p>
          <span v-if="isTyping" class="cursor">|</span>
        </section>

        <!-- 🎞️ Film -->
        <section
          v-if="showPhotos"
          class="film film--fade-in"
          ref="filmRef"
          @scroll="handleFilmScroll"
        >
          <div class="film__track">
            <div class="film__edge"></div>

            <div
              v-for="(src, idx) in photos"
              :key="idx"
              class="film__frame"
              @click.stop="openViewer(src)"
            >
              <img :src="src" />
            </div>

            <div class="film__edge"></div>
          </div>
        </section>

        <div v-if="showFilmEnd" class="film-end">
          우리의 추억은 여기서 끝이 아니야
        </div>

        <div v-if="showPhotoHint" class="photo-hint">
          사진을 옆으로 넘겨봐 →
        </div>

        <footer class="paper__bottom">
          <span class="from">From. {{ fromName }}</span>
          <span class="date">{{ todayText }}</span>
        </footer>
      </article>
    </main>

    <!-- 🔍 Viewer -->
    <div v-if="viewerSrc" class="viewer">
      <button class="viewer__close" @click="closeViewer">✕</button>
      <div class="viewer__content">
        <img :src="viewerSrc" />
      </div>
    </div>

    <div v-if="!hasStartedBgm" class="touch-hint-fixed">
      화면을 한 번 눌러줘
    </div>

    <!-- 🎵 BGM -->
    <audio ref="bgmRef" loop>
      <source :src="bgmSrc" type="audio/mpeg" />
    </audio>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from "vue";

/* =======================
   🔥 BASE URL (핵심)
======================= */
const baseUrl = import.meta.env.BASE_URL;

/* =======================
   📄 Names
======================= */
const fromName = "김대승";
const toName = "윤수연";

/* =======================
   ✉️ Letter
======================= */
const letter = `
안녕, 내 사랑.
벌써 우리가 1주년이라는 게 아직도 믿기지 않아.

너무 늦게 보내서 미안해.
편지를 싫어하는 줄로만 알고 있었고,
그래서 괜히 내가 부담을 주는 건 아닐까 계속 고민했어.

크리스마스 때 한 번 보내고,
그 뒤로는 반응이 조금 조심스러워 보여서
‘내가 너무 자주 편지를 썼을까?’ 하고 혼자 생각도 많이 했어.

근데 이번에 다시 이야기하면서 알게 됐어.
그런 게 아니라는 걸.
그래서 이렇게 다시, 천천히 써보고 있어.

앞으로 좋은 일도, 힘든 일도
우리에게 많이 생기겠지.
수연이가 우리 만남에 대해
늘 진지하게 생각해주고 있다는 것도 알고 있어.

그건 나도 마찬가지야, 수연아.
난 한 번도 우리의 만남을
가볍게 생각해본 적이 없어.

처음 가졌던 그 마음이 지금도 그대로야.
지금도 너무 많이 사랑하고 있고.

이렇게까지 할 수 있는 건
결코 가벼운 마음이 아니고,
나 역시 많은 걸 걸고 있다는 걸
전하고 싶었어.

앞으로도 언제까지나 사랑할 거고,
항상 수연이 옆에 있을게.

미안하고,
고맙고,
정말 많이 사랑해.
`;

const displayedChars = ref<string[]>([]);
const isTyping = ref(true);

/* =======================
   ⌨️ Typing Effect
======================= */
function startTyping() {
  const chars = Array.from(letter);
  let i = 0;
  const timer = setInterval(() => {
    if (i >= chars.length) {
      clearInterval(timer);
      isTyping.value = false;
      return;
    }
    displayedChars.value.push(chars[i++]);
  }, 36);
}

/* =======================
   📸 Photos (정답)
======================= */
const photos = Array.from(
  { length: 25 },
  (_, i) => `${baseUrl}img/photo${i + 1}.jpg`
);

const showPhotos = ref(false);
const showPhotoHint = ref(false);

/* =======================
   🎵 BGM (정답)
======================= */
const bgmRef = ref<HTMLAudioElement | null>(null);
const bgmSrc = `${baseUrl}bgm.mp3`;
const hasStartedBgm = ref(false);

/* =======================
   🎞️ Film Scroll
======================= */
const filmRef = ref<HTMLElement | null>(null);
const showFilmEnd = ref(false);
let filmEndShown = false;

function handleFilmScroll() {
  if (!filmRef.value || filmEndShown) return;
  const el = filmRef.value;
  if (el.scrollLeft + el.clientWidth >= el.scrollWidth - 10) {
    filmEndShown = true;
    showFilmEnd.value = true;
  }
}

/* =======================
   🔍 Viewer
======================= */
const viewerSrc = ref<string | null>(null);

function openViewer(src: string) {
  viewerSrc.value = src;
  document.body.style.overflow = "hidden";
}

function closeViewer() {
  viewerSrc.value = null;
  document.body.style.overflow = "";
}

/* =======================
   🎬 Effects
======================= */
watch(isTyping, (v) => {
  if (!v) {
    setTimeout(() => {
      showPhotos.value = true;
      if (bgmRef.value) bgmRef.value.volume = 0.55;
      setTimeout(() => (showPhotoHint.value = true), 400);
    }, 500);
  }
});

function handleFirstTouch() {
  if (hasStartedBgm.value || !bgmRef.value) return;
  bgmRef.value.volume = 0.35;
  bgmRef.value.play().then(() => (hasStartedBgm.value = true));
}

/* =======================
   📅 Date
======================= */
const todayText = computed(() => {
  const d = new Date();
  return `${d.getFullYear()}.${String(d.getMonth() + 1).padStart(2, "0")}.${String(
    d.getDate()
  ).padStart(2, "0")}`;
});

onMounted(startTyping);
</script>

<style>
/* ====== (네가 준 스타일 그대로, 수정 없음) ====== */

:root {
  --bg1: #0b1020;
  --bg2: #0a0f1a;
  --pink: #ff4fd8;
  --violet: #8b5cff;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: "Apple SD Gothic Neo", "Noto Sans KR", system-ui, sans-serif;
  background: linear-gradient(180deg, var(--bg1), var(--bg2));
  color: rgba(255, 255, 255, .92);
}

/* 이하 스타일 전부 동일 (생략 안 함) */
.bg { position: fixed; inset: 0; }
/* ... 네가 준 CSS 전부 그대로 유지 ... */
</style>
