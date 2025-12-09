<template>
  <main
    class="min-h-screen px-4 md:px-6 lg:px-10 py-12 flex flex-col gap-8 text-slate-100"
  >
    <section class="flex flex-col gap-3 max-w-3xl">
      <div
        class="inline-flex w-fit items-center rounded-full bg-white/10 px-3 py-1 text-[11px] font-semibold tracking-[0.12em] text-slate-200"
      >
        2025년 우리아이들 송년회
      </div>
      <h1 class="text-3xl md:text-4xl font-semibold leading-tight">
        날짜 눌러라. 저장해라.
      </h1>
      <p class="text-slate-300 text-base leading-relaxed">
        이름을 적고 가능한 날짜 눌러서 저장해라 개쉐이들아
      </p>
      <div class="flex flex-wrap items-center gap-3 text-sm text-slate-200">
        <span>총 인원: {{ participantCount }}명</span>
        <span class="text-slate-300">최다 일치: {{ bestCountLabel }}</span>
        <span v-if="isLoading" class="text-sky-300">불러오는 중...</span>
      </div>
    </section>

    <section class="grid grid-cols-1 gap-5 lg:grid-cols-[2fr_1fr]">
      <div class="overflow-x-auto pb-1">
        <div
          class="min-w-[720px] sm:min-w-full rounded-2xl border border-white/10 bg-white/5 p-4 md:p-6 shadow-[0_20px_60px_rgba(0,0,0,0.35)] backdrop-blur-sm"
        >
          <div class="mb-4 flex items-center justify-between gap-4">
            <div>
              <p class="text-xs uppercase tracking-[0.22em] text-sky-300">
                December
              </p>
              <h2 class="mt-1 text-2xl font-semibold">2025년 12월</h2>
            </div>
            <div class="flex items-center gap-3 text-xs text-slate-200">
              <span
                class="inline-flex h-3.5 w-3.5 rounded-full bg-gradient-to-r from-sky-300 to-indigo-400"
              />
              내 선택
              <span
                class="inline-flex h-3.5 w-3.5 rounded-full bg-gradient-to-r from-amber-200 to-rose-400"
              />
              다수 가능
            </div>
          </div>

          <div class="flex flex-col gap-3">
            <div
              class="grid grid-cols-7 text-center text-[11px] uppercase tracking-wide text-slate-400"
            >
              <span v-for="w in weekdayLabels" :key="w">{{ w }}</span>
            </div>
            <div class="grid grid-cols-7 gap-1.5 md:gap-2">
              <button
                v-for="cell in calendarCells"
                :key="cell.key"
                class="group relative flex min-h-[92px] flex-col gap-2 rounded-xl border border-white/10 bg-white/5 p-3 text-left text-sm transition duration-150 hover:-translate-y-0.5 hover:border-sky-300/70"
                :class="{
                  'pointer-events-none cursor-not-allowed bg-white/5 border-transparent text-slate-500 opacity-40':
                    !cell.isCurrentMonth,
                  'bg-gradient-to-br from-sky-300/20 via-indigo-700/40 to-slate-900/90 border-sky-300/80 shadow-xl shadow-sky-500/20':
                    cell.day && selectedDays.has(cell.day),
                  'bg-gradient-to-br from-amber-200/15 via-rose-800/40 to-slate-900/90 border-amber-200/60':
                    cell.day &&
                    dayCounts[cell.day] >= 3 &&
                    !selectedDays.has(cell.day),
                }"
                :disabled="!cell.isCurrentMonth"
                @click="cell.day && toggleDay(cell.day)"
              >
                <span class="text-base font-semibold text-slate-100">
                  {{ cell.label }}
                </span>
                <span
                  v-if="cell.day && dayCounts[cell.day]"
                  class="text-[12px]"
                  :class="
                    maxAvailableCount &&
                    dayCounts[cell.day] === maxAvailableCount
                      ? 'text-amber-300'
                      : 'text-sky-200'
                  "
                >
                  {{ dayCounts[cell.day] }}명 가능
                </span>
                <div
                  v-if="cell.day && dayNames[cell.day]?.length"
                  class="flex gap-1 overflow-x-auto pr-1"
                >
                  <span
                    v-for="person in dayNames[cell.day]"
                    :key="person"
                    class="rounded-md px-2 py-1 text-[11px] font-semibold whitespace-nowrap"
                    :class="
                      person === currentName
                        ? 'bg-gradient-to-r from-amber-200 to-rose-400 text-slate-950 shadow-sm'
                        : 'bg-white/10 text-slate-100'
                    "
                  >
                    {{ person }}
                  </span>
                </div>
              </button>
            </div>
          </div>
        </div>
      </div>

      <aside
        class="rounded-2xl border border-white/10 bg-white/5 p-4 md:p-5 shadow-[0_16px_40px_rgba(0,0,0,0.35)] backdrop-blur-sm flex flex-col gap-5"
      >
        <div class="flex flex-col gap-2">
          <p class="text-sm text-slate-300">이름</p>
          <input
            v-model="nameInput"
            type="text"
            placeholder="예: 잉좆"
            class="w-full rounded-xl border border-white/15 bg-white/5 px-3 py-3 text-base text-slate-100 placeholder:text-slate-500 outline-none transition focus:border-sky-300/80 focus:ring-2 focus:ring-sky-300/30"
          />
        </div>

        <div class="flex flex-col gap-3 md:flex-row">
          <button
            class="flex-1 rounded-xl bg-gradient-to-r from-sky-300 to-indigo-500 px-4 py-3 text-sm font-semibold text-slate-950 shadow-lg shadow-sky-500/20 transition hover:brightness-105 disabled:cursor-not-allowed disabled:opacity-60"
            :disabled="isLoading"
            @click="saveAvailability"
          >
            {{ isLoading ? "저장 중..." : "가능 일정 저장" }}
          </button>
          <button
            class="flex-1 rounded-xl bg-gradient-to-r from-rose-200 to-orange-400 px-4 py-3 text-sm font-semibold text-slate-950 shadow-lg shadow-rose-500/20 transition hover:brightness-105 disabled:cursor-not-allowed disabled:opacity-60"
            :disabled="isLoading"
            @click="deleteAvailability"
          >
            삭제
          </button>
          <button
            class="flex-1 rounded-xl border border-white/15 bg-white/5 px-4 py-3 text-sm font-semibold text-slate-100 transition hover:border-sky-300/70 hover:text-sky-200"
            @click="resetSelection"
          >
            선택 초기화
          </button>
        </div>

        <div class="flex flex-col gap-2">
          <p class="text-sm text-slate-300">내 선택</p>
          <div class="flex flex-wrap gap-2">
            <span
              v-for="d in [...selectedDays].sort((a, b) => a - b)"
              :key="d"
              class="rounded-lg bg-white/10 px-3 py-2 text-xs text-slate-100"
            >
              12월 {{ d }}일
            </span>
            <span
              v-if="!selectedDays.size"
              class="rounded-lg bg-white/5 px-3 py-2 text-xs text-slate-400"
            >
              아직 선택 없음
            </span>
          </div>
        </div>

        <div class="flex flex-col gap-3">
          <p class="text-sm text-slate-300">겹치는 날</p>
          <div class="flex flex-col gap-2">
            <div
              v-for="day in bestDays"
              :key="day.day"
              class="flex flex-col gap-2 rounded-xl border border-white/10 bg-white/5 px-3 py-3"
            >
              <div class="flex items-center gap-2">
                <p class="text-sm font-semibold">12월 {{ day.day }}일</p>
                <span
                  class="ml-auto rounded-lg bg-white/10 px-2.5 py-1 text-xs text-slate-100"
                >
                  {{ day.names.length }}명
                </span>
              </div>
              <div class="flex flex-wrap gap-1">
                <span
                  v-for="person in day.names"
                  :key="person"
                  class="rounded-md px-2 py-1 text-[11px] font-semibold whitespace-nowrap"
                  :class="
                    person === currentName
                      ? 'bg-gradient-to-r from-amber-200 to-rose-400 text-slate-950 shadow-sm'
                      : 'bg-white/10 text-slate-100'
                  "
                >
                  {{ person }}
                </span>
              </div>
            </div>
            <p
              v-if="!bestDays.length"
              class="rounded-lg bg-white/5 px-3 py-2 text-xs text-slate-400"
            >
              아직 아무도 일정을 넣지 않았어요.
            </p>
          </div>
        </div>
      </aside>
    </section>
  </main>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from "vue";
import { createClient, type SupabaseClient } from "@supabase/supabase-js";
import { useRuntimeConfig } from "#app";
import { useHead } from "#imports";

const year = 2025;
const month = 11; // December (0-based)
const weekdayLabels = ["일", "월", "화", "수", "목", "금", "토"];

type CalendarCell = {
  key: string;
  label: number;
  isCurrentMonth: boolean;
  day?: number;
};

type AvailabilityByName = Record<string, Set<number>>;
type AvailabilityByDay = Record<number, Set<string>>;
type AvailabilityRow = { name: string; days: number[] };

const runtimeConfig = useRuntimeConfig();
const siteUrl = runtimeConfig.public.siteUrl;
const ogImage = siteUrl
  ? `${siteUrl.replace(/\/$/, "")}/og-calendar.png`
  : "/og-calendar.png";

useHead({
  title: "2025년 우리아이들 송년회 캘린더",
  meta: [
    {
      property: "og:title",
      content: "2025년 우리아이들 송년회 캘린더",
    },
    {
      property: "og:description",
      content: "날짜 눌러서 저장해라. 다 같이 맞는 날 찾아보자!",
    },
    {
      property: "og:image",
      content: ogImage,
    },
    {
      property: "og:type",
      content: "website",
    },
    {
      property: "og:url",
      content: siteUrl || "https://main.d3jjn14xt8mkxt.amplifyapp.com",
    },
    {
      property: "og:site_name",
      content: "우리아이들 송년회 캘린더",
    },
  ],
  link: siteUrl
    ? [
        {
          rel: "canonical",
          href: siteUrl,
        },
      ]
    : [],
});

const supabaseClient: SupabaseClient | null =
  runtimeConfig.public.supabaseUrl && runtimeConfig.public.supabaseAnonKey
    ? createClient(
        runtimeConfig.public.supabaseUrl,
        runtimeConfig.public.supabaseAnonKey,
      )
    : null;

const calendarCells = computed<CalendarCell[]>(() => {
  const firstDay = new Date(year, month, 1).getDay();
  const totalDays = new Date(year, month + 1, 0).getDate();
  const prevMonthDays = new Date(year, month, 0).getDate();

  const leading = firstDay;
  const trailing = (7 - ((leading + totalDays) % 7)) % 7;

  const cells: CalendarCell[] = [];

  for (let i = leading - 1; i >= 0; i -= 1) {
    const label = prevMonthDays - i;
    cells.push({
      key: `prev-${label}`,
      label,
      isCurrentMonth: false,
    });
  }

  for (let day = 1; day <= totalDays; day += 1) {
    cells.push({
      key: `current-${day}`,
      label: day,
      isCurrentMonth: true,
      day,
    });
  }

  for (let day = 1; day <= trailing; day += 1) {
    cells.push({
      key: `next-${day}`,
      label: day,
      isCurrentMonth: false,
    });
  }

  return cells;
});

const nameInput = ref("");
const currentName = computed(() => nameInput.value.trim());
const selectedDays = ref<Set<number>>(new Set());
const availabilityByName = ref<AvailabilityByName>({});
const availabilityByDay = ref<AvailabilityByDay>({});
const isLoading = ref(false);

const dayCounts = computed<Record<number, number>>(() => {
  const entries: Record<number, number> = {};
  for (const [day, names] of Object.entries(availabilityByDay.value)) {
    entries[Number(day)] = names.size;
  }
  return entries;
});

const maxAvailableCount = computed(() => {
  const counts = Object.values(dayCounts.value);
  if (!counts.length) return 0;
  return Math.max(...counts);
});

const dayNames = computed<Record<number, string[]>>(() => {
  const entries: Record<number, string[]> = {};
  for (const [day, names] of Object.entries(availabilityByDay.value)) {
    entries[Number(day)] = [...names];
  }
  return entries;
});

const participantCount = computed(() =>
  Object.keys(availabilityByName.value).length,
);

const bestDays = computed(() => {
  const arr = Object.entries(availabilityByDay.value).map(([day, names]) => ({
    day: Number(day),
    names: [...names],
  }));

  arr.sort((a, b) => b.names.length - a.names.length || a.day - b.day);
  return arr.slice(0, 5);
});

const bestCountLabel = computed(() => {
  if (!bestDays.value.length) return "아직 없음";
  return `${bestDays.value[0].names.length}명`;
});

function toggleDay(day: number) {
  const next = new Set(selectedDays.value);
  if (next.has(day)) {
    next.delete(day);
  } else {
    next.add(day);
  }
  selectedDays.value = next;
}

function resetSelection() {
  selectedDays.value = new Set();
}

async function fetchAvailability() {
  if (!supabaseClient) return;

  isLoading.value = true;
  const { data, error } = await supabaseClient
    .from("availabilities")
    .select("name, days");
  isLoading.value = false;

  if (error) {
    console.error(error);
    window.alert("니 폰 허접이라 데이터 못불러옴 ㅋ");
    return;
  }

  const nameMap: AvailabilityByName = {};
  const dayMap: AvailabilityByDay = {};

  data?.forEach((row: AvailabilityRow) => {
    const daysSet = new Set<number>(row.days || []);
    nameMap[row.name] = daysSet;

    daysSet.forEach((day) => {
      if (!dayMap[day]) dayMap[day] = new Set();
      dayMap[day].add(row.name);
    });
  });

  availabilityByName.value = nameMap;
  availabilityByDay.value = dayMap;
}

async function saveAvailability() {
  const name = nameInput.value.trim();
  if (!name) {
    window.alert("이름은 엿바꿔먹었냐");
    return;
  }

  const previous = availabilityByName.value[name];
  const next = new Set(selectedDays.value);

  if (!supabaseClient) {
    window.alert("이건 뜨면 안되는 경고임 ㅅㅂ 뭔짓했냐");
    return;
  }

  const payload = { name, days: [...next].sort((a, b) => a - b) };

  isLoading.value = true;
  const { error } = await supabaseClient
    .from("availabilities")
    .upsert(payload);
  isLoading.value = false;

  if (error) {
    console.error(error);
    window.alert("니 폰 허접이라 저장 실패한듯 ㅋ");
    return;
  }

  if (previous) {
    previous.forEach((day) => {
      if (!next.has(day)) {
        const set = availabilityByDay.value[day];
        if (set) {
          set.delete(name);
          if (!set.size) delete availabilityByDay.value[day];
        }
      }
    });
  }

  next.forEach((day) => {
    if (!availabilityByDay.value[day]) {
      availabilityByDay.value[day] = new Set();
    }
    availabilityByDay.value[day].add(name);
  });

  availabilityByName.value[name] = next;
  availabilityByDay.value = { ...availabilityByDay.value };
  availabilityByName.value = { ...availabilityByName.value };
  window.alert(`${name}님의 가능 일정이 저장되었습니다.`);
}

async function deleteAvailability() {
  const name = nameInput.value.trim();
  if (!name) {
    window.alert("이름은 자꾸 어따 빼먹음?");
    return;
  }
  if (!supabaseClient) {
    window.alert("뜨면 안되는 경고인데 ㅅㅂ 뭔짓했냐");
    return;
  }
  const confirmed = window.confirm(`${name} <-- 이놈 일정 다 삭제해버린다?`);
  if (!confirmed) return;

  isLoading.value = true;
  const { error } = await supabaseClient
    .from("availabilities")
    .delete()
    .eq("name", name);
  isLoading.value = false;

  if (error) {
    console.error(error);
    window.alert("니 폰 허접이라 삭제못한듯 ㅋ");
    return;
  }

  const previous = availabilityByName.value[name];
  if (previous) {
    previous.forEach((day) => {
      const set = availabilityByDay.value[day];
      if (set) {
        set.delete(name);
        if (!set.size) delete availabilityByDay.value[day];
      }
    });
    delete availabilityByName.value[name];
  }

  availabilityByDay.value = { ...availabilityByDay.value };
  availabilityByName.value = { ...availabilityByName.value };
  selectedDays.value = new Set();
  fetchAvailability();
  window.alert(`${name} <-- 이새기 일정 삭제됨 ㅋ`);
}

onMounted(() => {
  fetchAvailability();
});
</script>
