<script setup>
import { computed, onMounted, onUnmounted, reactive } from "vue";

const screens = [
  {
    id: "incident",
    label: "Инцидент",
    subtitle: "Очередь, SOP, видео и журнал",
    icon: ["M10.3 3.9 1.8 18a2 2 0 0 0 1.7 3h17a2 2 0 0 0 1.7-3L13.7 3.9a2 2 0 0 0-3.4 0Z", "M12 9v4", "M12 17h.01"],
  },
  {
    id: "events",
    label: "События",
    subtitle: "Поиск, фильтры и таймлайн",
    icon: ["M4 6h16", "M7 12h10", "M10 18h4"],
  },
  {
    id: "archive",
    label: "Архив",
    subtitle: "Завершенные инциденты и отчеты",
    icon: ["M3 8h18v13H3z", "M1 3h22v5H1z", "M10 13h4"],
  },
  {
    id: "map",
    label: "Карта",
    subtitle: "Объекты, этажи, камеры и датчики",
    icon: ["M9 18 3 21V6l6-3 6 3 6-3v15l-6 3-6-3Z", "M9 3v15", "M15 6v15"],
  },
  {
    id: "vms",
    label: "Видео",
    subtitle: "Верификация и доказательства",
    icon: ["M1 5h15v14H1z", "M23 7l-7 5 7 5z"],
  },
  {
    id: "supervisor",
    label: "Supervisor",
    subtitle: "Нагрузка, SLA и смена",
    icon: ["M3 21h18", "M7 17v-6", "M12 17V7", "M17 17v-9"],
  },
];

const state = reactive({
  activeScreen: "incident",
  selectedIncident: "INC-2481",
  incidentListVisible: true,
  sidebarCollapsed: true,
  incidentSearch: "",
  incidentSeverity: "all",
  incidentPreset: "all",
  eventSearch: "",
  eventPriority: "all",
  eventStatus: "all",
  selectedEventId: null,
  activeEventPanelId: "events-1",
  fullscreenEventPanelId: null,
  eventPanels: [
    { id: "events-1", title: "Все события", system: "all", source: "all", priority: "all", status: "all", query: "", selectedEventId: null, filtersOpen: false },
  ],
  nextEventPanelIndex: 2,
  archiveSearch: "",
  archiveRange: "week",
  archiveSeverity: "all",
  archiveOwner: "all",
  archiveFrom: "2026-06-27",
  archiveTo: "2026-07-03",
  cameraSearch: "",
  cameraZone: "all",
  cameraStatus: "all",
  selectedCameraIds: ["cam-12", "cam-14", "cam-18", "cam-21"],
  quadSlotCount: 4,
  fullscreenScreen: null,
  videoMode: "live",
  archiveTime: "12:31:08",
  focusedCameraId: "cam-12",
  mapPreviewCameraId: "cam-12",
  incidentContextVisible: false,
  incidentComment: "",
  sopStepIndex: 2,
});

const severityOptions = [
  { id: "all", label: "Все" },
  { id: "critical", label: "Critical" },
  { id: "high", label: "High" },
  { id: "medium", label: "Medium" },
  { id: "low", label: "Low" },
];

const incidentPresets = [
  { id: "all", label: "Все открытые" },
  { id: "mine", label: "Мои" },
  { id: "sla", label: "SLA риск" },
  { id: "handover", label: "На передачу" },
];

const eventPriorityOptions = ["all", "HIGH", "MED", "LOW"];
const eventStatusOptions = ["all", "Новое", "Связано", "В работе", "Закрыто", "Ложное"];
const eventPanelTemplates = [
  { title: "Все события", system: "all" },
  { title: "СКУД", system: "СКУД" },
  { title: "VMS", system: "VMS" },
  { title: "Датчики", system: "Датчик" },
];
const archiveRangeOptions = [
  { id: "today", label: "Сегодня" },
  { id: "week", label: "7 дней" },
  { id: "month", label: "Месяц" },
  { id: "custom", label: "Период" },
  { id: "all", label: "Все" },
];

const sopSteps = [
  {
    title: "Принять инцидент оператором",
    detail: "Зафиксировать ответственного, время принятия и источник первичного события.",
  },
  {
    title: "Проверить видео за 30 секунд до тревоги",
    detail:
      "Сравнить live и архивный фрагмент, отметить наличие человека, транспорта или ложного срабатывания. Проверить соседние камеры, время начала движения и направление перемещения объекта до срабатывания датчика.",
    image: "Camera 12 / -00:30",
  },
  {
    title: "Связаться с постом охраны",
    detail: "Передать локацию, тип инцидента и запросить подтверждение обстановки на месте.",
  },
  {
    title: "Заблокировать соседние двери",
    detail: "Выполнить команду только после проверки, что блокировка не мешает эвакуации.",
  },
  {
    title: "Добавить комментарий и результат проверки",
    detail:
      "Описать наблюдения оператора, приложить ключевой кадр или фото с места, указать выполненные команды и причину закрытия, эскалации или передачи следующей смене. Комментарий должен быть достаточным для отчета и последующего разбора инцидента.",
    image: "Фото поста / акт проверки",
  },
];

const incidentCommands = [
  { id: "unlock-door", label: "Открыть дверь D12", tone: "normal" },
  { id: "lock-zone", label: "Заблокировать зону 2B", tone: "danger" },
  { id: "ptz-preset", label: "PTZ на Door D12", tone: "normal" },
  { id: "call-guard", label: "Вызвать пост охраны", tone: "normal" },
  { id: "create-task", label: "Создать заявку технику", tone: "normal" },
];

const incidents = [
  {
    id: "INC-2481",
    title: "Взлом двери D12",
    severity: "critical",
    time: "00:03:42",
    location: "Здание A / этаж 2 / зона 2B",
    source: "СКУД + движение + камера 12",
    owner: "Иванов",
    status: "В работе",
    slaRisk: true,
    handover: false,
  },
  {
    id: "INC-2480",
    title: "Дым в техпомещении",
    severity: "high",
    time: "00:08:11",
    location: "Здание B / цоколь",
    source: "Пожарный датчик",
    owner: "Петрова",
    status: "Эскалация",
    slaRisk: true,
    handover: true,
  },
  {
    id: "INC-2478",
    title: "Камера offline",
    severity: "medium",
    time: "00:21:04",
    location: "КПП-3 / въезд",
    source: "VMS health",
    owner: "Смена 2",
    status: "Назначено",
    slaRisk: false,
    handover: true,
  },
  {
    id: "INC-2476",
    title: "Доступ запрещен",
    severity: "medium",
    time: "00:34:28",
    location: "Турникет 4 / лобби",
    source: "СКУД",
    owner: "Иванов",
    status: "Проверка",
    slaRisk: false,
    handover: false,
  },
];

const eventRows = [
  {
    id: "evt-1",
    time: "12:31:08",
    priority: "HIGH",
    type: "Взлом двери",
    source: "Door D12",
    status: "Новое",
    incident: "INC-2481",
    location: "Здание A / этаж 2",
    category: "СКУД",
    media: "Фото прохода D12",
    payload: ["doorId: D12", "cardId: 4421", "result: forced_open", "confidence: 0.96"],
  },
  {
    id: "evt-2",
    time: "12:30:55",
    priority: "MED",
    type: "Движение в зоне",
    source: "Motion 2B",
    status: "Связано",
    incident: "INC-2481",
    location: "Здание A / этаж 2",
    category: "Датчик",
    payload: ["zone: 2B", "sensor: PIR-2B-04", "duration: 18 сек", "threshold: exceeded"],
  },
  {
    id: "evt-3",
    time: "12:30:44",
    priority: "HIGH",
    type: "Видеоаналитика",
    source: "Camera 12",
    status: "Связано",
    incident: "INC-2481",
    location: "Здание A / этаж 2",
    category: "VMS",
    media: "Кадр Camera 12",
    payload: ["cameraId: cam-12", "rule: line_crossing", "object: person", "confidence: 0.89"],
  },
  {
    id: "evt-4",
    time: "12:30:30",
    priority: "LOW",
    type: "Потеря сигнала",
    source: "Camera 18",
    status: "В работе",
    incident: "INC-2478",
    location: "КПП-3",
    category: "VMS",
    payload: ["cameraId: cam-18", "health: offline", "lastFrame: 12:29:57", "reason: no_signal"],
  },
  {
    id: "evt-5",
    time: "12:28:19",
    priority: "MED",
    type: "Доступ запрещен",
    source: "Turnstile 4",
    status: "Закрыто",
    incident: "INC-2476",
    location: "Лобби",
    category: "СКУД",
    media: "Фото турникета",
    payload: ["readerId: T4", "cardId: 9810", "result: denied", "reason: expired"],
  },
  {
    id: "evt-6",
    time: "12:25:02",
    priority: "LOW",
    type: "Дверь открыта долго",
    source: "Door C04",
    status: "Ложное",
    incident: "INC-2475",
    location: "Здание C",
    category: "СКУД",
    payload: ["doorId: C04", "openDuration: 04:12", "operator: Иванов", "resolution: false_alarm"],
  },
];

const completedIncidents = [
  {
    id: "INC-2475",
    title: "Дверь открыта долго",
    severity: "low",
    openedAt: "2026-07-03 12:10",
    closedAt: "2026-07-03 12:25",
    location: "Здание C / коридор",
    owner: "Иванов",
    source: "СКУД",
    status: "Закрыт",
    closeReason: "Ложная тревога",
    report: "Готов",
    duration: "00:15:02",
  },
  {
    id: "INC-2468",
    title: "Доступ запрещен",
    severity: "medium",
    openedAt: "2026-07-02 18:44",
    closedAt: "2026-07-02 19:18",
    location: "Лобби / турникет 4",
    owner: "Петрова",
    source: "СКУД",
    status: "Закрыт",
    closeReason: "Проверено",
    report: "Черновик",
    duration: "00:34:12",
  },
  {
    id: "INC-2451",
    title: "Дым в техпомещении",
    severity: "high",
    openedAt: "2026-06-30 04:12",
    closedAt: "2026-06-30 05:06",
    location: "Здание B / цоколь",
    owner: "Смена 2",
    source: "Пожарный датчик + VMS",
    status: "Закрыт",
    closeReason: "Эскалация завершена",
    report: "Готов",
    duration: "00:54:01",
  },
  {
    id: "INC-2419",
    title: "Камера offline",
    severity: "medium",
    openedAt: "2026-06-22 09:02",
    closedAt: "2026-06-22 10:40",
    location: "КПП-3 / въезд",
    owner: "Иванов",
    source: "VMS health",
    status: "Закрыт",
    closeReason: "Техническая заявка",
    report: "Готов",
    duration: "01:38:33",
  },
];

const baseCameras = [
  { id: "cam-12", name: "Camera 12", zone: "Здание A / этаж 2", status: "online", tags: "коридор, Door D12" },
  { id: "cam-14", name: "Camera 14", zone: "Здание A / этаж 2", status: "online", tags: "лестница, зона 2B" },
  { id: "cam-18", name: "Camera 18", zone: "КПП-3", status: "offline", tags: "въезд, периметр" },
  { id: "cam-21", name: "Entrance 2", zone: "Здание A / вход", status: "online", tags: "вход, турникеты" },
  { id: "cam-31", name: "Parking East", zone: "Парковка", status: "online", tags: "парковка, шлагбаум" },
  { id: "cam-44", name: "Server Room", zone: "Здание A / серверная", status: "restricted", tags: "серверная, доступ" },
];

const cameras = [
  ...baseCameras,
  ...Array.from({ length: 26 }, (_, index) => {
    const number = index + 45;
    const zones = ["Здание A / этаж 1", "Здание B / склад", "Периметр", "Парковка", "КПП-1"];
    return {
      id: `cam-${number}`,
      name: `Camera ${number}`,
      zone: zones[index % zones.length],
      status: index % 11 === 0 ? "offline" : "online",
      tags: index % 2 === 0 ? "обзор, маршрут" : "периметр, доступ",
    };
  }),
];

const screenIds = new Set(screens.map((screen) => screen.id));

const selectedIncident = computed(
  () =>
    [...incidents, ...completedIncidents].find((incident) => incident.id === state.selectedIncident) ??
    incidents[0],
);

const filteredIncidents = computed(() => {
  const query = normalize(state.incidentSearch);

  return incidents.filter((incident) => {
    const matchesQuery =
      query === "" ||
      normalize(
        [
          incident.id,
          incident.title,
          incident.location,
          incident.source,
          incident.owner,
          incident.status,
        ].join(" "),
      ).includes(query);

    const matchesSeverity =
      state.incidentSeverity === "all" || incident.severity === state.incidentSeverity;

    const matchesPreset =
      state.incidentPreset === "all" ||
      (state.incidentPreset === "mine" && incident.owner === "Иванов") ||
      (state.incidentPreset === "sla" && incident.slaRisk) ||
      (state.incidentPreset === "handover" && incident.handover);

    return matchesQuery && matchesSeverity && matchesPreset;
  });
});

const filteredEvents = computed(() => {
  const query = normalize(state.eventSearch);

  return eventRows.filter((event) => {
    const matchesQuery =
      query === "" ||
      normalize(Object.values(event).join(" ")).includes(query);
    const matchesPriority = state.eventPriority === "all" || event.priority === state.eventPriority;
    const matchesStatus = state.eventStatus === "all" || event.status === state.eventStatus;

    return matchesQuery && matchesPriority && matchesStatus;
  });
});

const selectedIncidentEvents = computed(() =>
  eventRows.filter((event) => event.incident === state.selectedIncident),
);

const eventSystemOptions = computed(() => ["all", ...new Set(eventRows.map((event) => event.category))]);

const eventSourceOptions = computed(() => ["all", ...new Set(eventRows.map((event) => event.source))]);

const activeEventPanel = computed(
  () =>
    state.eventPanels.find((panel) => panel.id === state.activeEventPanelId) ??
    state.eventPanels[0] ??
    null,
);

const visibleEventPanels = computed(() => {
  if (!state.fullscreenEventPanelId) {
    return state.eventPanels;
  }

  return state.eventPanels.filter((panel) => panel.id === state.fullscreenEventPanelId);
});

const filteredArchive = computed(() => {
  const query = normalize(state.archiveSearch);

  return completedIncidents.filter((incident) => {
    const closedDate = incident.closedAt.slice(0, 10);
    const matchesQuery =
      query === "" ||
      normalize(Object.values(incident).join(" ")).includes(query);
    const matchesSeverity =
      state.archiveSeverity === "all" || incident.severity === state.archiveSeverity;
    const matchesOwner = state.archiveOwner === "all" || incident.owner === state.archiveOwner;
    const matchesRange =
      state.archiveRange === "all" ||
      (state.archiveRange === "today" && closedDate === "2026-07-03") ||
      (state.archiveRange === "week" && closedDate >= "2026-06-27") ||
      (state.archiveRange === "month" && closedDate >= "2026-07-01") ||
      (state.archiveRange === "custom" &&
        closedDate >= state.archiveFrom &&
        closedDate <= state.archiveTo);

    return matchesQuery && matchesSeverity && matchesOwner && matchesRange;
  });
});

const archiveOwners = computed(() => [
  "all",
  ...new Set(completedIncidents.map((incident) => incident.owner)),
]);

const reportSummary = computed(() => {
  const ready = filteredArchive.value.filter((incident) => incident.report === "Готов").length;
  const drafts = filteredArchive.value.filter((incident) => incident.report === "Черновик").length;

  return { ready, drafts, total: filteredArchive.value.length };
});

const cameraZones = computed(() => ["all", ...new Set(cameras.map((camera) => camera.zone))]);

const filteredCameras = computed(() => {
  const query = normalize(state.cameraSearch);

  return cameras.filter((camera) => {
    const matchesQuery =
      query === "" || normalize(Object.values(camera).join(" ")).includes(query);
    const matchesZone = state.cameraZone === "all" || camera.zone === state.cameraZone;
    const matchesStatus = state.cameraStatus === "all" || camera.status === state.cameraStatus;

    return matchesQuery && matchesZone && matchesStatus;
  });
});

const quadratorSlots = computed(() => {
  const selected = state.selectedCameraIds
    .map((cameraId) => cameras.find((camera) => camera.id === cameraId))
    .filter(Boolean)
    .slice(0, state.quadSlotCount);

  return [...selected, ...Array(state.quadSlotCount - selected.length).fill(null)];
});

const quadColumnCount = computed(() => {
  if (state.quadSlotCount <= 4) return 2;
  if (state.quadSlotCount <= 9) return 3;
  if (state.quadSlotCount <= 16) return 4;
  return 8;
});

const mapPreviewCamera = computed(
  () => cameras.find((camera) => camera.id === state.mapPreviewCameraId) ?? cameras[0],
);

const mapDeviceEvents = computed(() =>
  eventRows.filter(
    (event) =>
      event.source === mapPreviewCamera.value.name ||
      event.payload?.some((item) => item.includes(mapPreviewCamera.value.id)) ||
      event.source === "Door D12",
  ),
);

function screenFromRoute() {
  if (typeof window === "undefined") return null;

  const route = window.location.hash.replace(/^#\/?/, "");

  return screenIds.has(route) ? route : null;
}

function setActiveScreen(screenId, updateRoute = true) {
  if (!screenIds.has(screenId)) return;

  state.activeScreen = screenId;

  if (updateRoute && typeof window !== "undefined") {
    const nextHash = `#/${screenId}`;

    if (window.location.hash !== nextHash) {
      window.location.hash = nextHash;
    }
  }
}

function syncScreenFromRoute() {
  const routeScreen = screenFromRoute();

  if (routeScreen) {
    state.activeScreen = routeScreen;
  }
}

function normalize(value) {
  return String(value).trim().toLowerCase();
}

function selectIncident(id) {
  state.selectedIncident = id;
  state.incidentListVisible = false;
  state.sopStepIndex = 2;
  state.incidentComment = "";
  setActiveScreen("incident");
}

function openIncidentFromEvent(event) {
  selectIncident(event.incident);
}

function showIncidentList() {
  state.incidentListVisible = true;
}

function selectEvent(event, panel = null) {
  state.selectedEventId = event.id;
  state.selectedIncident = event.incident;

  if (panel) {
    state.activeEventPanelId = panel.id;
    panel.selectedEventId = event.id;
    return;
  }

  if (state.eventPanels[0]) {
    state.activeEventPanelId = state.eventPanels[0].id;
    state.eventPanels[0].selectedEventId = event.id;
  }
}

function addEventPanel(template = eventPanelTemplates[0]) {
  if (state.eventPanels.length >= 4) {
    return;
  }

  const panelId = `events-${state.nextEventPanelIndex}`;

  state.eventPanels.push({
    id: panelId,
    title: template.title,
    system: template.system,
    source: "all",
    priority: "all",
    status: "all",
    query: "",
    selectedEventId: null,
    filtersOpen: false,
  });
  state.activeEventPanelId = panelId;
  state.nextEventPanelIndex += 1;
}

function removeEventPanel(panelId) {
  if (state.eventPanels.length === 1) {
    return;
  }

  const index = state.eventPanels.findIndex((panel) => panel.id === panelId);

  if (index >= 0) {
    state.eventPanels.splice(index, 1);

    if (state.activeEventPanelId === panelId) {
      state.activeEventPanelId =
        state.eventPanels[Math.min(index, state.eventPanels.length - 1)]?.id ??
        state.eventPanels[0]?.id ??
        null;
    }

    if (state.fullscreenEventPanelId === panelId) {
      state.fullscreenEventPanelId = null;
    }
  }
}

function selectEventPanel(panelId) {
  state.activeEventPanelId = panelId;
}

function toggleEventPanelFullscreen(panelId) {
  state.fullscreenEventPanelId = state.fullscreenEventPanelId === panelId ? null : panelId;
  state.activeEventPanelId = panelId;
}

function closeEventDetail(panel) {
  if (state.selectedEventId === panel.selectedEventId) {
    state.selectedEventId = null;
  }

  panel.selectedEventId = null;
}

function filteredPanelEvents(panel) {
  const query = normalize(panel.query);

  return eventRows.filter((event) => {
    const matchesQuery = query === "" || normalize(Object.values(event).join(" ")).includes(query);
    const matchesSystem = panel.system === "all" || event.category === panel.system;
    const matchesSource = panel.source === "all" || event.source === panel.source;
    const matchesPriority = panel.priority === "all" || event.priority === panel.priority;
    const matchesStatus = panel.status === "all" || event.status === panel.status;

    return matchesQuery && matchesSystem && matchesSource && matchesPriority && matchesStatus;
  });
}

function selectedEventForPanel(panel) {
  if (!panel?.selectedEventId) {
    return null;
  }

  const events = filteredPanelEvents(panel);

  return events.find((event) => event.id === panel.selectedEventId) ?? null;
}

function confirmSopStep(index) {
  if (index <= state.sopStepIndex && state.sopStepIndex < sopSteps.length) {
    state.sopStepIndex = Math.max(state.sopStepIndex, index + 1);
  }
}

function finishIncident() {
  state.incidentComment = "";
  state.sopStepIndex = sopSteps.length;
  state.incidentListVisible = true;
  setActiveScreen("incident");
}

function toggleCamera(cameraId) {
  const index = state.selectedCameraIds.indexOf(cameraId);

  if (index >= 0) {
    state.selectedCameraIds.splice(index, 1);
    return;
  }

  if (state.selectedCameraIds.length >= state.quadSlotCount) {
    state.selectedCameraIds.shift();
  }

  state.selectedCameraIds.push(cameraId);
  state.focusedCameraId = cameraId;
}

function setQuadSlotCount(count) {
  state.quadSlotCount = count;
  state.selectedCameraIds.splice(count);
}

function openCameraFromMap(cameraId) {
  if (!state.selectedCameraIds.includes(cameraId)) {
    if (state.selectedCameraIds.length >= state.quadSlotCount) {
      state.selectedCameraIds.shift();
    }
    state.selectedCameraIds.push(cameraId);
  }
  state.focusedCameraId = cameraId;
  state.mapPreviewCameraId = cameraId;
}

function openMapFromCamera(cameraId) {
  state.focusedCameraId = cameraId;
  state.mapPreviewCameraId = cameraId;
  setActiveScreen("map");
}

function toggleFullscreen(screenId) {
  state.fullscreenScreen = state.fullscreenScreen === screenId ? null : screenId;
}

onMounted(() => {
  syncScreenFromRoute();
  window.addEventListener("hashchange", syncScreenFromRoute);
});

onUnmounted(() => {
  window.removeEventListener("hashchange", syncScreenFromRoute);
});
</script>

<template>
  <main
    class="app-shell"
    :class="{
      'sidebar-collapsed': true,
      'content-fullscreen': state.fullscreenScreen === state.activeScreen,
    }"
  >
    <aside class="sidebar" aria-label="Навигация по макетам">
      <div class="brand-block">
        <div class="brand-mark">PS</div>
        <div class="brand-copy">
          <p class="eyebrow">PSIM prototype</p>
          <h1>Операторский контур</h1>
        </div>
      </div>

      <nav class="screen-nav">
        <button
          v-for="screen in screens"
          :key="screen.id"
          class="nav-item"
          :class="{ 'nav-item-active': state.activeScreen === screen.id }"
          type="button"
          @click="setActiveScreen(screen.id)"
          :title="screen.label"
        >
          <svg class="nav-icon" viewBox="0 0 24 24" aria-hidden="true">
            <path v-for="path in screen.icon" :key="path" :d="path"></path>
          </svg>
          <span>{{ screen.label }}</span>
          <small>{{ screen.subtitle }}</small>
        </button>
      </nav>

      <section class="shift-card">
        <p class="eyebrow">Смена</p>
        <div class="shift-row">
          <strong>12 открытых</strong>
          <span>3 критичных</span>
        </div>
        <div class="load-line" aria-hidden="true">
          <span style="width: 72%"></span>
        </div>
      </section>
    </aside>

    <section class="workspace">
      <button
        v-if="state.fullscreenScreen === state.activeScreen"
        class="fullscreen-exit-button"
        type="button"
        @click="toggleFullscreen(state.activeScreen)"
      >
        Свернуть
      </button>

      <header class="topbar">
        <div class="topbar-pill">
          <div>
            <span>Command center / Северный кампус</span>
            <strong>{{ screens.find((screen) => screen.id === state.activeScreen)?.label }}</strong>
          </div>
          <button class="primary-button compact-create-button" type="button">Создать событие</button>
        </div>
      </header>

      <section v-if="state.activeScreen === 'monitoring'" class="screen monitoring-screen">
        <div class="main-grid">
          <section class="panel incident-queue">
            <div class="panel-header">
              <h3>Очередь инцидентов</h3>
              <span>{{ filteredIncidents.length }} / {{ incidents.length }}</span>
            </div>

            <div class="search-block">
              <label class="search-field">
                <span>Поиск</span>
                <input v-model="state.incidentSearch" type="search" placeholder="ID, зона, дверь, камера, оператор" />
              </label>
              <div class="segmented-control" aria-label="Быстрые представления инцидентов">
                <button
                  v-for="preset in incidentPresets"
                  :key="preset.id"
                  type="button"
                  :class="{ active: state.incidentPreset === preset.id }"
                  @click="state.incidentPreset = preset.id"
                >
                  {{ preset.label }}
                </button>
              </div>
              <div class="chip-row" aria-label="Фильтр критичности">
                <button
                  v-for="option in severityOptions"
                  :key="option.id"
                  type="button"
                  :class="{ active: state.incidentSeverity === option.id }"
                  @click="state.incidentSeverity = option.id"
                >
                  {{ option.label }}
                </button>
              </div>
            </div>

            <button
              v-for="incident in filteredIncidents"
              :key="incident.id"
              class="incident-card"
              :class="`severity-${incident.severity}`"
              type="button"
              @click="selectIncident(incident.id)"
            >
              <span class="severity-dot"></span>
              <span>
                <strong>{{ incident.title }}</strong>
                <small>{{ incident.location }}</small>
                <small>{{ incident.id }} · {{ incident.owner }} · {{ incident.status }}</small>
              </span>
              <em>{{ incident.time }}</em>
            </button>
            <div v-if="filteredIncidents.length === 0" class="empty-state">
              Нет инцидентов под выбранные фильтры
            </div>
          </section>

          <section class="panel map-panel">
            <div class="panel-header">
              <h3>Карта выбранного инцидента</h3>
              <span>INC-2481 / Здание A / этаж 2</span>
            </div>
            <div class="map-canvas compact-map">
              <div class="room room-a">Склад</div>
              <div class="room room-b">Коридор</div>
              <div class="room room-c">Пост</div>
              <button class="map-marker marker-critical" type="button" @click="selectIncident('INC-2481')">
                D12
              </button>
              <button class="camera camera-one" :class="{ 'camera-focused': state.focusedCameraId === 'cam-12' }" type="button" @click="openCameraFromMap('cam-12')">Cam 12</button>
              <button class="camera camera-two" :class="{ 'camera-focused': state.focusedCameraId === 'cam-14' }" type="button" @click="openCameraFromMap('cam-14')">Cam 14</button>
              <span class="fov fov-one"></span>
              <span class="route-line"></span>
            </div>
          </section>

          <section class="panel video-stack">
            <div class="panel-header">
              <h3>Камеры выбранного инцидента</h3>
              <span>Door D12 / 2 рядом</span>
            </div>
            <div class="video-tile large-video">Camera 12</div>
            <div class="video-grid">
              <div class="video-tile">Cam 14</div>
              <div class="video-tile warning-video">Cam 18 offline</div>
            </div>
          </section>
        </div>
      </section>

      <section
        v-else-if="state.activeScreen === 'incident'"
        class="screen incident-screen"
        :class="{
          'incident-list-hidden': !state.incidentListVisible,
          'incident-list-mode': state.incidentListVisible,
        }"
      >
        <section v-if="state.incidentListVisible" class="panel incident-queue incident-worklist">
          <div class="panel-header">
            <h3>Очередь инцидентов</h3>
            <span>{{ filteredIncidents.length }} / {{ incidents.length }}</span>
          </div>

          <div class="search-block">
            <label class="search-field">
              <span>Поиск</span>
              <input v-model="state.incidentSearch" type="search" placeholder="ID, зона, дверь, камера, оператор" />
            </label>
            <div class="segmented-control" aria-label="Быстрые представления инцидентов">
              <button
                v-for="preset in incidentPresets"
                :key="preset.id"
                type="button"
                :class="{ active: state.incidentPreset === preset.id }"
                @click="state.incidentPreset = preset.id"
              >
                {{ preset.label }}
              </button>
            </div>
            <div class="chip-row" aria-label="Фильтр критичности">
              <button
                v-for="option in severityOptions"
                :key="option.id"
                type="button"
                :class="{ active: state.incidentSeverity === option.id }"
                @click="state.incidentSeverity = option.id"
              >
                {{ option.label }}
              </button>
            </div>
          </div>

          <div class="incident-list-scroll">
            <button
              v-for="incident in filteredIncidents"
              :key="incident.id"
              class="incident-card"
              :class="[`severity-${incident.severity}`, { selected: selectedIncident.id === incident.id }]"
              type="button"
              @click="selectIncident(incident.id)"
            >
              <span class="severity-dot"></span>
              <span>
                <strong>{{ incident.title }}</strong>
                <small>{{ incident.location }}</small>
                <small>{{ incident.id }} · {{ incident.owner }} · {{ incident.status }}</small>
              </span>
              <em>{{ incident.time }}</em>
            </button>
            <div v-if="filteredIncidents.length === 0" class="empty-state">
              Нет инцидентов под выбранные фильтры
            </div>
          </div>
        </section>

        <div v-else class="incident-focus">
          <div class="incident-hero">
            <div>
              <p class="eyebrow">{{ selectedIncident.id }} / {{ selectedIncident.source }}</p>
              <h3>{{ selectedIncident.title }}</h3>
              <p>{{ selectedIncident.location }}</p>
            </div>
            <div class="incident-actions">
              <span class="timer">{{ selectedIncident.time }}</span>
              <button class="ghost-button" type="button" @click="showIncidentList">К списку</button>
              <button class="ghost-button" type="button" @click="state.incidentContextVisible = !state.incidentContextVisible">
                {{ state.incidentContextVisible ? "Скрыть контекст" : "Показать карту и видео" }}
              </button>
              <button class="ghost-button" type="button">Назначить</button>
              <button class="danger-button" type="button">Эскалировать</button>
            </div>
          </div>

          <div class="incident-layout" :class="{ 'context-visible': state.incidentContextVisible }">
            <section class="panel sop-panel">
              <div class="panel-header">
                <h3>Инструкция обработки</h3>
                <span>{{ selectedIncident.status }}</span>
              </div>
              <ol class="sop-list">
                <li
                  v-for="(step, index) in sopSteps"
                  :key="step.title"
                  :class="{ done: index < state.sopStepIndex, active: index === state.sopStepIndex }"
                >
                  <div class="sop-step-body">
                    <strong>{{ step.title }}</strong>
                    <span>{{ step.detail }}</span>
                    <div v-if="step.image" class="sop-step-media">
                      {{ step.image }}
                    </div>
                  </div>
                  <button
                    class="ghost-button compact-button"
                    type="button"
                    :disabled="index > state.sopStepIndex || index < state.sopStepIndex"
                    @click="confirmSopStep(index)"
                  >
                    {{ index < state.sopStepIndex ? "Готово" : "Подтвердить" }}
                  </button>
                </li>
              </ol>
              <div class="sop-actions">
                <button class="danger-button" type="button" @click="finishIncident">
                  Завершить инцидент
                </button>
              </div>
            </section>

            <section class="panel command-panel">
              <div class="panel-header">
                <h3>Команды</h3>
                <span>исполнение оператором</span>
              </div>
              <div class="command-grid">
                <button
                  v-for="command in incidentCommands"
                  :key="command.id"
                  :class="command.tone === 'danger' ? 'danger-button' : 'ghost-button'"
                  type="button"
                >
                  {{ command.label }}
                </button>
              </div>
            </section>

            <section v-if="state.incidentContextVisible" class="panel map-panel">
              <div class="panel-header">
                <h3>Контекст места</h3>
                <span>2 камеры рядом</span>
              </div>
              <div class="map-canvas incident-map">
                <div class="room room-a">Зона 2B</div>
                <div class="room room-b">Лестница</div>
                <button class="map-marker marker-critical">Door D12</button>
                <button class="camera camera-one" :class="{ 'camera-focused': state.focusedCameraId === 'cam-12' }" type="button" @click="openCameraFromMap('cam-12')">Cam 12</button>
                <span class="fov fov-one"></span>
              </div>
            </section>

            <section v-if="state.incidentContextVisible" class="panel video-stack">
              <div class="panel-header">
                <h3>Видео</h3>
                <span>pre/post alarm</span>
              </div>
              <div class="video-tile large-video">Camera 12 / -00:30</div>
              <div class="timeline">
                <span></span>
                <strong></strong>
              </div>
            </section>

            <section class="panel linked-events-panel">
              <div class="panel-header">
                <h3>Связанные события</h3>
                <span>{{ selectedIncidentEvents.length }} событий</span>
              </div>
              <div class="linked-event-list">
                <button
                  v-for="event in selectedIncidentEvents"
                  :key="`${event.time}-${event.source}`"
                  class="linked-event"
                  type="button"
                  @click="setActiveScreen('events')"
                >
                  <span>{{ event.time }}</span>
                  <mark :class="`priority-${event.priority.toLowerCase()}`">{{ event.priority }}</mark>
                  <strong>{{ event.type }}</strong>
                  <small>{{ event.category }} · {{ event.source }}</small>
                </button>
                <div v-if="selectedIncidentEvents.length === 0" class="empty-state">
                  Связанных событий пока нет
                </div>
              </div>
            </section>

            <section class="panel journal-panel">
              <div class="panel-header">
                <h3>Журнал</h3>
                <span>audit trail</span>
              </div>
              <div class="comment-box">
                <label class="search-field">
                  <span>Комментарий оператора</span>
                  <textarea
                    v-model="state.incidentComment"
                    placeholder="Опишите результат проверки, контакт с постом, причину закрытия или эскалации"
                  ></textarea>
                </label>
                <div class="comment-actions">
                  <button class="ghost-button" type="button" :disabled="state.incidentComment.trim() === ''">
                    Добавить комментарий
                  </button>
                  <button class="primary-button" type="button" :disabled="state.incidentComment.trim() === ''">
                    Добавить в отчет
                  </button>
                </div>
              </div>
              <ul class="activity-list">
                <li><strong>12:31</strong> Инцидент принят оператором Иванов</li>
                <li><strong>12:32</strong> Открыта Camera 12</li>
                <li><strong>12:33</strong> SOP шаг 2 отмечен выполненным</li>
                <li v-if="state.incidentComment.trim() !== ''">
                  <strong>Сейчас</strong> {{ state.incidentComment }}
                </li>
              </ul>
            </section>
          </div>
        </div>
      </section>

      <section v-else-if="state.activeScreen === 'events'" class="screen events-screen">
        <section class="panel event-workbench-panel">
          <div class="panel-header">
            <h3>Рабочие таблицы событий</h3>
            <div class="filter-pills">
              <span>{{ state.eventPanels.length }} таблиц</span>
              <span>до 4 одновременно</span>
              <span>фильтры независимы</span>
              <span>переход к инциденту</span>
            </div>
            <button class="ghost-button compact-button" type="button" @click="toggleFullscreen('events')">
              {{ state.fullscreenScreen === 'events' ? "Свернуть" : "На всю страницу" }}
            </button>
          </div>

          <div class="event-workbench-actions">
            <div class="event-add-actions">
              <span>Открыть таблицу:</span>
              <button
                v-for="template in eventPanelTemplates"
                :key="template.title"
                class="ghost-button"
                type="button"
                :disabled="state.eventPanels.length >= 4"
                @click="addEventPanel(template)"
              >
                + {{ template.title }}
              </button>
            </div>
            <div class="event-tab-strip" aria-label="Открытые таблицы событий">
              <button
                v-for="panel in state.eventPanels"
                :key="panel.id"
                class="event-tab"
                :class="{ active: activeEventPanel?.id === panel.id }"
                type="button"
                @click="selectEventPanel(panel.id)"
              >
                <strong>{{ panel.title }}</strong>
                <span>{{ filteredPanelEvents(panel).length }} событий</span>
                <em v-if="selectedEventForPanel(panel)">открыта карточка</em>
              </button>
            </div>
          </div>

          <div
            class="event-panel-stage"
            :class="{
              'event-panel-stage-fullscreen': state.fullscreenEventPanelId,
              'event-panel-stage-single': visibleEventPanels.length === 1,
            }"
          >
            <section
              v-for="panel in visibleEventPanels"
              :key="panel.id"
              class="event-panel"
              :class="{ 'event-panel-active': activeEventPanel?.id === panel.id }"
            >
              <div class="event-panel-title">
                <div>
                  <h4>{{ panel.title }}</h4>
                  <span>{{ filteredPanelEvents(panel).length }} событий</span>
                </div>
                <div class="event-panel-actions">
                  <button
                    class="ghost-button compact-button"
                    type="button"
                    @click="panel.filtersOpen = !panel.filtersOpen"
                  >
                    {{ panel.filtersOpen ? "Скрыть фильтры" : "Фильтры" }}
                  </button>
                  <button
                    v-if="panel.filtersOpen"
                    class="ghost-button compact-button"
                    type="button"
                    @click="panel.query = ''; panel.source = 'all'; panel.system = 'all'; panel.priority = 'all'; panel.status = 'all'"
                  >
                    Сброс
                  </button>
                  <button
                    class="ghost-button compact-button"
                    type="button"
                    @click="toggleEventPanelFullscreen(panel.id)"
                  >
                    {{ state.fullscreenEventPanelId === panel.id ? "Свернуть" : "Развернуть" }}
                  </button>
                  <button
                    class="ghost-button compact-button"
                    type="button"
                    :disabled="state.eventPanels.length === 1"
                    @click="removeEventPanel(panel.id)"
                  >
                    Закрыть
                  </button>
                </div>
              </div>

              <div v-if="panel.filtersOpen" class="event-panel-filters">
                <label class="search-field">
                  <span>Поиск</span>
                  <input v-model="panel.query" type="search" placeholder="INC, источник, зона" />
                </label>
                <label class="select-field">
                  <span>Система</span>
                  <select v-model="panel.system">
                    <option v-for="system in eventSystemOptions" :key="system" :value="system">
                      {{ system === "all" ? "Все" : system }}
                    </option>
                  </select>
                </label>
                <label class="select-field">
                  <span>Источник</span>
                  <select v-model="panel.source">
                    <option v-for="source in eventSourceOptions" :key="source" :value="source">
                      {{ source === "all" ? "Все" : source }}
                    </option>
                  </select>
                </label>
                <label class="select-field">
                  <span>Уровень</span>
                  <select v-model="panel.priority">
                    <option v-for="priority in eventPriorityOptions" :key="priority" :value="priority">
                      {{ priority === "all" ? "Все" : priority }}
                    </option>
                  </select>
                </label>
                <label class="select-field">
                  <span>Статус</span>
                  <select v-model="panel.status">
                    <option v-for="status in eventStatusOptions" :key="status" :value="status">
                      {{ status === "all" ? "Все" : status }}
                    </option>
                  </select>
                </label>
              </div>

              <div
                class="event-table-stage"
                :class="{ 'detail-open': selectedEventForPanel(panel) }"
              >
                <div v-if="selectedEventForPanel(panel)" class="event-panel-detail">
                  <div class="event-detail-header">
                    <div class="event-panel-detail-main">
                      <span class="event-time">{{ selectedEventForPanel(panel).time }}</span>
                      <strong>{{ selectedEventForPanel(panel).type }}</strong>
                      <small>
                        {{ selectedEventForPanel(panel).category }} ·
                        {{ selectedEventForPanel(panel).source }} ·
                        {{ selectedEventForPanel(panel).location }}
                      </small>
                    </div>
                    <button
                      class="icon-button detail-close-button"
                      type="button"
                      aria-label="Закрыть карточку события"
                      @click="closeEventDetail(panel)"
                    >
                      ×
                    </button>
                  </div>
                  <div class="event-detail-grid">
                    <span>
                      Приоритет
                      <strong>{{ selectedEventForPanel(panel).priority }}</strong>
                    </span>
                    <span>
                      Статус
                      <strong>{{ selectedEventForPanel(panel).status }}</strong>
                    </span>
                    <span>
                      Инцидент
                      <button
                        class="incident-number-button"
                        type="button"
                        @click="openIncidentFromEvent(selectedEventForPanel(panel))"
                      >
                        {{ selectedEventForPanel(panel).incident }}
                      </button>
                    </span>
                  </div>
                  <div class="event-photo-grid">
                    <div class="event-photo event-photo-main">
                      <strong>{{ selectedEventForPanel(panel).source }}</strong>
                      <small>кадр события</small>
                    </div>
                    <div class="event-photo">
                      <strong>до события</strong>
                      <small>-00:10</small>
                    </div>
                    <div class="event-photo">
                      <strong>после события</strong>
                      <small>+00:10</small>
                    </div>
                  </div>
                  <div class="event-panel-payload">
                    <code
                      v-for="item in selectedEventForPanel(panel).payload"
                      :key="`${panel.id}-${item}`"
                    >
                      {{ item }}
                    </code>
                  </div>
                  <div class="event-panel-controls">
                    <button class="ghost-button compact-button" type="button">Подтвердить</button>
                    <button class="ghost-button compact-button" type="button">Команда</button>
                    <button class="ghost-button compact-button" type="button">Открыть фото</button>
                    <button class="primary-button compact-button" type="button">Создать действие</button>
                  </div>
                </div>

                <div class="mini-event-table">
                  <div
                    v-for="event in filteredPanelEvents(panel)"
                    :key="`${panel.id}-${event.time}-${event.source}`"
                    class="mini-event-row"
                    :class="{ selected: selectedEventForPanel(panel)?.id === event.id }"
                    role="button"
                    tabindex="0"
                    @click="selectEvent(event, panel)"
                    @keydown.enter="selectEvent(event, panel)"
                    @keydown.space.prevent="selectEvent(event, panel)"
                  >
                    <span class="event-time">{{ event.time }}</span>
                    <mark :class="`priority-${event.priority.toLowerCase()}`">{{ event.priority }}</mark>
                    <span>
                      <strong>{{ event.type }}</strong>
                      <small>{{ event.category }} · {{ event.source }} · {{ event.location }}</small>
                    </span>
                    <button class="incident-number-button" type="button" @click.stop="openIncidentFromEvent(event)">
                      {{ event.incident }}
                    </button>
                  </div>
                  <div v-if="filteredPanelEvents(panel).length === 0" class="empty-state">
                    Нет событий
                  </div>
                </div>
              </div>
            </section>
          </div>
        </section>
      </section>

      <section v-else-if="state.activeScreen === 'archive'" class="screen archive-screen">
        <section class="panel archive-filter-panel">
          <div class="panel-header">
            <h3>Завершенные инциденты</h3>
            <div class="filter-pills">
              <span>{{ reportSummary.total }} найдено</span>
              <span>{{ reportSummary.ready }} отчетов готово</span>
              <span>{{ reportSummary.drafts }} черновиков</span>
            </div>
          </div>

          <div class="archive-filter-bar">
            <label class="search-field archive-search">
              <span>Поиск по архиву</span>
              <input v-model="state.archiveSearch" type="search" placeholder="ID, объект, причина, оператор" />
            </label>
            <label class="select-field">
              <span>Период</span>
              <select v-model="state.archiveRange">
                <option v-for="range in archiveRangeOptions" :key="range.id" :value="range.id">
                  {{ range.label }}
                </option>
              </select>
            </label>
            <label class="select-field">
              <span>Критичность</span>
              <select v-model="state.archiveSeverity">
                <option v-for="option in severityOptions" :key="option.id" :value="option.id">
                  {{ option.label }}
                </option>
              </select>
            </label>
            <label class="select-field">
              <span>Оператор</span>
              <select v-model="state.archiveOwner">
                <option v-for="owner in archiveOwners" :key="owner" :value="owner">
                  {{ owner === "all" ? "Все" : owner }}
                </option>
              </select>
            </label>
          </div>

          <div v-if="state.archiveRange === 'custom'" class="date-range">
            <label class="search-field">
              <span>С</span>
              <input v-model="state.archiveFrom" type="date" />
            </label>
            <label class="search-field">
              <span>По</span>
              <input v-model="state.archiveTo" type="date" />
            </label>
          </div>

          <div class="saved-views archive-actions">
            <button type="button" @click="state.archiveRange = 'today'; state.archiveSearch = ''">
              Закрыто сегодня
            </button>
            <button type="button" @click="state.archiveSearch = 'Ложная тревога'; state.archiveRange = 'all'">
              Ложные тревоги
            </button>
            <button type="button" @click="state.archiveSeverity = 'high'; state.archiveRange = 'all'">
              High для отчета
            </button>
            <button type="button">Сформировать отчет</button>
          </div>
        </section>

        <section class="panel archive-table-panel">
          <div class="archive-row archive-head">
            <span>Инцидент</span>
            <span>Закрыт</span>
            <span>Длительность</span>
            <span>Оператор</span>
            <span>Причина</span>
            <span>Отчет</span>
          </div>
          <button
            v-for="incident in filteredArchive"
            :key="incident.id"
            class="archive-row"
            type="button"
            @click="selectIncident(incident.id)"
          >
            <span>
              <strong>{{ incident.id }} · {{ incident.title }}</strong>
              <small>{{ incident.location }}</small>
            </span>
            <span>{{ incident.closedAt }}</span>
            <span>{{ incident.duration }}</span>
            <span>{{ incident.owner }}</span>
            <span>{{ incident.closeReason }}</span>
            <span>
              <mark :class="incident.report === 'Готов' ? 'report-ready' : 'report-draft'">
                {{ incident.report }}
              </mark>
            </span>
          </button>
          <div v-if="filteredArchive.length === 0" class="empty-state">
            В архиве нет инцидентов под выбранные фильтры
          </div>
        </section>
      </section>

      <section v-else-if="state.activeScreen === 'map'" class="screen map-screen">
        <section class="panel layer-panel">
          <div class="panel-header">
            <h3>Слои</h3>
            <span>этаж 2</span>
          </div>
          <label><input type="checkbox" checked /> Инциденты</label>
          <label><input type="checkbox" checked /> Камеры и FOV</label>
          <label><input type="checkbox" checked /> Двери и СКУД</label>
          <label><input type="checkbox" checked /> Датчики</label>
          <label><input type="checkbox" /> Патрули</label>
        </section>
        <section class="panel full-map-panel">
          <div class="panel-header">
            <h3>Здание A</h3>
            <div class="filter-pills">
              <span>Кампус</span>
              <span>Здание A</span>
              <span>Этаж 2</span>
            </div>
            <button class="ghost-button compact-button" type="button" @click="toggleFullscreen('map')">
              {{ state.fullscreenScreen === 'map' ? "Свернуть" : "На всю страницу" }}
            </button>
          </div>
          <div class="map-canvas full-map">
            <div class="room room-a">Зона 2A</div>
            <div class="room room-b">Зона 2B</div>
            <div class="room room-c">Пост охраны</div>
            <div class="room room-d">Серверная</div>
            <button class="map-marker marker-critical">D12</button>
            <button class="map-marker marker-high">Smoke</button>
            <button class="camera camera-one" :class="{ 'camera-focused': state.focusedCameraId === 'cam-12' }" type="button" @click="openCameraFromMap('cam-12')">Cam 12</button>
            <button class="camera camera-two" :class="{ 'camera-focused': state.focusedCameraId === 'cam-14' }" type="button" @click="openCameraFromMap('cam-14')">Cam 14</button>
            <button class="camera camera-three" :class="{ 'camera-focused': state.focusedCameraId === 'cam-18' }" type="button" @click="openCameraFromMap('cam-18')">Cam 18</button>
            <span class="fov fov-one"></span>
            <span class="fov fov-two"></span>
          </div>
        </section>
        <section class="panel object-panel">
          <div class="panel-header">
            <h3>Door D12</h3>
            <span>Alarm</span>
          </div>
          <p>Ближайшие камеры: Camera 12, Camera 14</p>
          <div class="device-info-grid">
            <span>Устройство <strong>{{ mapPreviewCamera.name }}</strong></span>
            <span>Статус <strong>{{ mapPreviewCamera.status }}</strong></span>
            <span>Зона <strong>{{ mapPreviewCamera.zone }}</strong></span>
            <span>Теги <strong>{{ mapPreviewCamera.tags }}</strong></span>
          </div>
          <div class="map-preview video-tile" :class="{ 'warning-video': mapPreviewCamera.status === 'offline' }">
            <strong>{{ mapPreviewCamera.name }}</strong>
            <small>{{ mapPreviewCamera.zone }}</small>
            <small>{{ mapPreviewCamera.status === "online" ? "Live stream" : "Нет сигнала" }}</small>
          </div>
          <div class="object-actions">
            <button class="ghost-button" type="button" @click="openCameraFromMap('cam-12')">Camera 12</button>
            <button class="ghost-button" type="button" @click="openCameraFromMap('cam-14')">Camera 14</button>
            <button class="ghost-button" type="button">История</button>
            <button class="primary-button" type="button">Открыть SOP</button>
          </div>
          <div class="device-command-panel">
            <strong>Команды устройству</strong>
            <div class="object-actions">
              <button class="ghost-button compact-button" type="button">PTZ preset</button>
              <button class="ghost-button compact-button" type="button">Открыть дверь</button>
              <button class="ghost-button compact-button" type="button">Запросить архив</button>
              <button class="danger-button compact-button" type="button">Блокировать</button>
            </div>
          </div>
          <div class="device-events">
            <strong>Последние события</strong>
            <button
              v-for="event in mapDeviceEvents"
              :key="`map-device-${event.id}`"
              type="button"
              @click="selectEvent(event); setActiveScreen('events')"
            >
              <span>{{ event.time }}</span>
              <small>{{ event.type }} · {{ event.source }}</small>
            </button>
          </div>
        </section>
      </section>

      <section v-else-if="state.activeScreen === 'vms'" class="screen vms-screen">
        <section class="panel camera-catalog">
          <div class="panel-header">
            <h3>Камеры</h3>
            <span>{{ filteredCameras.length }} / {{ cameras.length }}</span>
          </div>

          <div class="camera-filters">
            <label class="search-field">
              <span>Поиск камеры</span>
              <input v-model="state.cameraSearch" type="search" placeholder="Cam 12, вход, КПП, зона" />
            </label>
            <label class="select-field">
              <span>Зона</span>
              <select v-model="state.cameraZone">
                <option v-for="zone in cameraZones" :key="zone" :value="zone">
                  {{ zone === "all" ? "Все зоны" : zone }}
                </option>
              </select>
            </label>
            <label class="select-field">
              <span>Статус</span>
              <select v-model="state.cameraStatus">
                <option value="all">Все</option>
                <option value="online">Online</option>
                <option value="offline">Offline</option>
                <option value="restricted">Restricted</option>
              </select>
            </label>
          </div>

          <div class="camera-list">
            <div
              v-for="cameraItem in filteredCameras"
              :key="cameraItem.id"
              class="camera-row"
              :class="{
                selected: state.selectedCameraIds.includes(cameraItem.id),
                offline: cameraItem.status === 'offline',
              }"
              role="button"
              tabindex="0"
              @click="toggleCamera(cameraItem.id)"
            >
              <span class="camera-status-dot"></span>
              <span>
                <strong>{{ cameraItem.name }}</strong>
                <small>{{ cameraItem.zone }} · {{ cameraItem.tags }}</small>
              </span>
              <em>{{ cameraItem.status }}</em>
              <button class="map-link-button" type="button" @click.stop="openMapFromCamera(cameraItem.id)">
                Карта
              </button>
            </div>
          </div>
        </section>

        <section class="panel vms-main">
          <div class="panel-header">
            <h3>Квадратор</h3>
            <div class="filter-pills">
              <span>{{ state.selectedCameraIds.length }} / {{ state.quadSlotCount }}</span>
              <span>{{ state.videoMode === "live" ? "Live" : `Архив ${state.archiveTime}` }}</span>
            </div>
            <button class="ghost-button compact-button" type="button" @click="toggleFullscreen('vms')">
              {{ state.fullscreenScreen === 'vms' ? "Свернуть" : "На всю страницу" }}
            </button>
          </div>

          <div class="quad-grid" :style="{ '--quad-columns': quadColumnCount }">
            <div
              v-for="(cameraItem, index) in quadratorSlots"
              :key="cameraItem?.id ?? `empty-${index}`"
              class="video-tile quad-tile"
              :class="{ 'warning-video': cameraItem?.status === 'offline', 'empty-video': !cameraItem }"
            >
              <template v-if="cameraItem">
                <strong>{{ cameraItem.name }}</strong>
                <small>{{ cameraItem.zone }}</small>
                <small>{{ state.videoMode === "live" ? "Live stream" : `Archive ${state.archiveTime}` }}</small>
                <button class="video-map-button" type="button" @click="openMapFromCamera(cameraItem.id)">
                  На карте
                </button>
              </template>
              <template v-else>
                Выберите камеру
              </template>
            </div>
          </div>

          <div class="vms-controls">
            <button class="ghost-button" type="button" @click="setQuadSlotCount(4)">2x2</button>
            <button class="ghost-button" type="button" @click="setQuadSlotCount(9)">3x3</button>
            <button class="ghost-button" type="button" @click="setQuadSlotCount(16)">4x4</button>
            <button class="ghost-button" type="button" @click="setQuadSlotCount(32)">32 камеры</button>
            <button class="ghost-button" type="button" @click="state.videoMode = state.videoMode === 'live' ? 'archive' : 'live'">
              {{ state.videoMode === "live" ? "Открыть архив" : "Вернуться Live" }}
            </button>
            <input v-if="state.videoMode === 'archive'" v-model="state.archiveTime" class="archive-time-input" type="time" step="1" />
            <button class="ghost-button" type="button">Bookmark</button>
            <button class="ghost-button" type="button">Snapshot</button>
            <button class="ghost-button" type="button">Evidence lock</button>
            <button class="primary-button" type="button">Экспорт фрагмента</button>
          </div>

          <div class="wide-timeline vms-timeline">
            <span class="timeline-event event-two">Alarm</span>
          </div>
        </section>
      </section>

      <section v-else class="screen supervisor-screen">
        <div class="kpi-grid">
          <section class="panel kpi-card">
            <span>MTTA</span>
            <strong>00:41</strong>
            <small>лучше нормы на 18%</small>
          </section>
          <section class="panel kpi-card">
            <span>Открыто</span>
            <strong>12</strong>
            <small>3 критичных</small>
          </section>
          <section class="panel kpi-card">
            <span>Ложные</span>
            <strong>22%</strong>
            <small>за последние 24 часа</small>
          </section>
          <section class="panel kpi-card">
            <span>SLA риск</span>
            <strong>4</strong>
            <small>2 инцидента старше 10 минут</small>
          </section>
        </div>

        <div class="supervisor-grid">
          <section class="panel supervisor-board">
            <div class="panel-header">
              <h3>Нагрузка операторов</h3>
              <span>SLA и handover</span>
            </div>
            <div class="operator-row">
              <span>Иванов</span>
              <div class="load-line"><span style="width: 82%"></span></div>
              <strong>5</strong>
            </div>
            <div class="operator-row">
              <span>Петрова</span>
              <div class="load-line"><span style="width: 54%"></span></div>
              <strong>3</strong>
            </div>
            <div class="operator-row">
              <span>Смена 2</span>
              <div class="load-line"><span style="width: 37%"></span></div>
              <strong>2</strong>
            </div>
          </section>

          <section class="panel analytics-panel">
            <div class="panel-header">
              <h3>Инциденты по источникам</h3>
              <span>24 часа</span>
            </div>
            <div class="bar-list">
              <div><span>СКУД</span><div class="load-line"><span style="width: 76%"></span></div><strong>18</strong></div>
              <div><span>VMS</span><div class="load-line"><span style="width: 52%"></span></div><strong>12</strong></div>
              <div><span>Датчики</span><div class="load-line"><span style="width: 38%"></span></div><strong>9</strong></div>
            </div>
          </section>

          <section class="panel analytics-panel">
            <div class="panel-header">
              <h3>Тренд по часам</h3>
              <span>закрыто / открыто</span>
            </div>
            <div class="sparkline-bars">
              <span style="height: 28%"></span>
              <span style="height: 42%"></span>
              <span style="height: 35%"></span>
              <span style="height: 68%"></span>
              <span style="height: 54%"></span>
              <span style="height: 82%"></span>
              <span style="height: 46%"></span>
              <span style="height: 63%"></span>
            </div>
          </section>

          <section class="panel analytics-panel">
            <div class="panel-header">
              <h3>Топ зон</h3>
              <span>по повторяемости</span>
            </div>
            <div class="zone-list">
              <button type="button">Здание A / этаж 2 <strong>11</strong></button>
              <button type="button">КПП-3 <strong>7</strong></button>
              <button type="button">Парковка <strong>5</strong></button>
            </div>
          </section>
        </div>
      </section>
    </section>
  </main>
</template>
