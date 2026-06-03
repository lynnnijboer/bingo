<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { Head } from '@inertiajs/vue3'

interface Task {
    s: string
    t: string
    d: string
    p: 1 | 2 | 3 | 5
}

interface CellState {
    done: boolean
    photo?: string
}

interface AppState {
    cells: Record<number, CellState>
    lines: string[]
}

const TASKS: Task[] = [
    { s: 'Eerste Altbier', t: 'Drink je eerste Altbier', d: 'Het ritueel begint. Een vers getapt Altbier in de eerste kroeg.', p: 1 },
    { s: 'Groepsselfie', t: 'Groepsselfie vóór de eerste kroeg', d: 'Hele team op de foto voordat het feest losbarst.', p: 1 },
    { s: '"Prost!"', t: 'Proost en zeg "Prost!"', d: 'Hef het glas en toost luid in het Duits met het hele team.', p: 1 },
    { s: 'Bestel in het Duits', t: 'Bestel je drankje volledig in het Duits', d: 'Geen Nederlands, geen Engels. "Ein Alt, bitte!"', p: 2 },
    { s: 'Rijn op de foto', t: 'Foto met de Rijn op de achtergrond', d: 'Loop naar de Rheinuferpromenade en leg het water vast.', p: 1 },
    { s: 'Currywurst', t: 'Eet een currywurst', d: 'Klassieke tussenstop. Met of zonder Pommes.', p: 2 },
    { s: 'Killepitsch', t: 'Drink een Killepitsch', d: 'De donkerrode kruidenlikeur uit Düsseldorf. Eén shot telt.', p: 2 },
    { s: 'Köbes op de foto', t: 'Foto met een echte Köbes', d: 'De kelner in het blauwe schort. Vraag het netjes.', p: 3 },
    { s: 'Originele brouwerij', t: 'Bezoek een originele brouwerij', d: 'Uerige, Füchschen, Schlüssel of Schumacher — kies er één.', p: 2 },
    { s: 'Radslag', t: 'Doe een radslag voor de Radschläger', d: 'Het symbool van de stad. Wie durft het rad te slaan?', p: 3 },
    { s: 'Mosterd proeven', t: 'Proef Düsseldorfer mosterd', d: 'De pittige lokale Senf (Löwensenf). Vies gezicht = bonus.', p: 2 },
    { s: 'Rheinturm', t: 'Foto bij de Rheinturm', d: 'De toren met de grootste lichtklok ter wereld in beeld.', p: 2 },
    { s: 'Meezingen', t: 'Zing mee met een lied in de kroeg', d: 'Volume omhoog. Tekst kennen is optioneel.', p: 3 },
    { s: 'Fortuna spotten', t: 'Spot een Fortuna Düsseldorf-shirt', d: 'Rood-wit van de lokale club. Op straat of in de kroeg.', p: 2 },
    { s: '3 soorten Alt', t: 'Drink 3 verschillende Altbier', d: 'Drie merken op één avond. Houd het bij.', p: 3 },
    { s: 'Königsallee', t: 'Foto op de Königsallee (Kö)', d: 'De chique winkelboulevard met de gracht in het midden.', p: 1 },
    { s: 'Rondje gekregen', t: 'Krijg een rondje van een onbekende', d: 'Maak een praatje. Charmeer een drankje los.', p: 5 },
    { s: 'Dans op straat', t: 'Dans op straat in de Altstadt', d: 'Geen podium nodig. De stoep is de dansvloer.', p: 3 },
    { s: 'Toost met local', t: 'Toost met een echte Düsseldorfer', d: 'Vind een geboren en getogen local en klink samen.', p: 3 },
    { s: '5 min Duits', t: 'Praat 5 minuten lang alleen Duits', d: 'Geen woord Nederlands of Engels. Het team controleert.', p: 5 },
    { s: 'Gratis Alt', t: 'Versier een gratis Altbier van de Köbes', d: 'Praat, charmeer of zing — maar betaal niet.', p: 5 },
    { s: 'Karaoke', t: 'Doe karaoke of zing op een podium', d: 'Het podium op, microfoon vast. Pure moed.', p: 5 },
    { s: 'Foto met vreemden', t: 'Groepsfoto met onbekenden', d: 'Maak nieuwe vrienden en zet ze op de foto.', p: 3 },
    { s: '5 bierviltjes', t: 'Verzamel 5 bierviltjes', d: 'Vijf verschillende kroegen, vijf viltjes. Bewijs ze.', p: 2 },
    { s: 'Langste toog', t: 'Tel 10 kroegen op één rij', d: 'De "langste toog ter wereld" — tel 10 kroegen naast elkaar.', p: 3 },
]

const KEY = 'ddorf_bingo_v1'

function loadState(): AppState {
    try {
        const raw = localStorage.getItem(KEY)
        if (raw) {
            const o = JSON.parse(raw)
            if (o && o.cells) return o
        }
    } catch {
        // ignore
    }
    return { cells: {}, lines: [] }
}

const state = ref<AppState>(loadState())

function saveState() {
    try {
        localStorage.setItem(KEY, JSON.stringify(state.value))
    } catch {
        showToast('Opslag vol — foto te groot')
    }
}

// Bingo line definitions — rows of 3
const LINES = (() => {
    const lines: { id: string; name: string; idx: number[] }[] = []
    const rows = Math.floor(TASKS.length / 3)
    for (let r = 0; r < rows; r++) {
        lines.push({ id: `r${r}`, name: `Rij ${r + 1}`, idx: [0, 1, 2].map((c) => r * 3 + c) })
    }
    return lines
})()

// Sanitize persisted lines
state.value.lines = state.value.lines.filter(
    (id) => LINES.some((l) => l.id === id && l.idx.every((i) => !!(state.value.cells[i]?.done))),
)

const maxPts = TASKS.reduce((a, t) => a + t.p, 0)

const completedCount = computed(() =>
    TASKS.reduce((n, _, i) => n + (state.value.cells[i]?.done ? 1 : 0), 0),
)

const totalPts = computed(() =>
    TASKS.reduce((pts, t, i) => pts + (state.value.cells[i]?.done ? t.p : 0), 0),
)

const bingoCount = computed(() => state.value.lines.length)

const progressPct = computed(() => `${(completedCount.value / 25) * 100}%`)

const winningCellSet = computed(() => {
    const set = new Set<number>()
    LINES.forEach((l) => {
        if (state.value.lines.includes(l.id)) l.idx.forEach((i) => set.add(i))
    })
    return set
})

// Task detail sheet
const taskSheetOpen = ref(false)
const activeIdx = ref<number | null>(null)

const activeTask = computed(() => (activeIdx.value !== null ? TASKS[activeIdx.value] : null))

const activeCellState = computed(() =>
    activeIdx.value !== null ? (state.value.cells[activeIdx.value] ?? null) : null,
)

const captureLabel = computed(() =>
    activeCellState.value?.done ? 'Andere foto maken' : 'Foto maken & voltooien',
)

function openTask(i: number) {
    activeIdx.value = i
    taskSheetOpen.value = true
}

function closeTask() {
    taskSheetOpen.value = false
    activeIdx.value = null
}

const fileInputRef = ref<HTMLInputElement | null>(null)

function triggerCapture() {
    fileInputRef.value?.click()
}

function onFileChange(e: Event) {
    const input = e.target as HTMLInputElement
    const f = input.files?.[0]
    if (!f) return
    resizeImage(f, (dataUrl) => {
        const i = activeIdx.value
        if (i === null) return
        state.value.cells[i] = { done: true, photo: dataUrl }
        saveState()
        showToast(`<b>+${TASKS[i].p}</b> ${TASKS[i].s} voltooid`)
        checkBingo()
        setTimeout(() => {
            if (!bingoOpen.value) closeTask()
        }, 650)
    })
    input.value = ''
}

function resizeImage(file: File, cb: (dataUrl: string) => void) {
    const img = new Image()
    const url = URL.createObjectURL(file)
    img.onload = () => {
        const max = 720
        let { width: w, height: h } = img
        if (w > h && w > max) {
            h = Math.round((h * max) / w)
            w = max
        } else if (h >= w && h > max) {
            w = Math.round((w * max) / h)
            h = max
        }
        const cv = document.createElement('canvas')
        cv.width = w
        cv.height = h
        cv.getContext('2d')!.drawImage(img, 0, 0, w, h)
        URL.revokeObjectURL(url)
        cb(cv.toDataURL('image/jpeg', 0.72))
    }
    img.onerror = () => {
        URL.revokeObjectURL(url)
        showToast('Kon foto niet laden')
    }
    img.src = url
}

function undoTask() {
    const i = activeIdx.value
    if (i === null) return
    const newCells = { ...state.value.cells }
    delete newCells[i]
    state.value.cells = newCells
    state.value.lines = LINES.filter(
        (l) => state.value.lines.includes(l.id) && l.idx.every((j) => !!(state.value.cells[j]?.done)),
    ).map((l) => l.id)
    saveState()
    closeTask()
    showToast('Opdracht teruggezet')
}

// Bingo detection
const bingoOpen = ref(false)
const bingoCelebLines = ref<{ id: string; name: string }[]>([])

const bingoKindText = computed(() => {
    const lines = bingoCelebLines.value
    if (!lines.length) return ''
    return lines.length > 1 ? `${lines.length} lijnen tegelijk` : lines[0].name
})

const bingoPtsText = computed(() => {
    const lines = bingoCelebLines.value
    if (!lines.length) return ''
    return lines.length > 1 ? 'Wat een team.' : `Drie op een rij — ${lines[0].name.toLowerCase()}`
})

function isDone(i: number) {
    return !!(state.value.cells[i]?.done)
}

function lineComplete(line: { idx: number[] }) {
    return line.idx.every(isDone)
}

function checkBingo() {
    const newly = LINES.filter((l) => lineComplete(l) && !state.value.lines.includes(l.id))
    if (newly.length) {
        newly.forEach((l) => state.value.lines.push(l.id))
        saveState()
        celebrate(newly)
    }
}

function celebrate(lines: typeof LINES) {
    closeTask()
    bingoCelebLines.value = lines
    bingoOpen.value = true
    fireConfetti(2600)
    if (navigator.vibrate) navigator.vibrate([40, 60, 40, 60, 120])
}

// Confetti
const confettiCanvasRef = ref<HTMLCanvasElement | null>(null)
let confettiParts: {
    x: number
    y: number
    r: number
    c: string
    vy: number
    vx: number
    rot: number
    vr: number
    sh: 'rect' | 'circ'
}[] = []
let confettiRafId: number | null = null
let confettiUntil = 0
const CONFETTI_COLORS = ['#30d158', '#ffffff', '#0a84ff', '#ff9f0a', '#ff375f', '#a1a1a6']

function sizeCanvas() {
    if (!confettiCanvasRef.value) return
    confettiCanvasRef.value.width = window.innerWidth
    confettiCanvasRef.value.height = window.innerHeight
}

function fireConfetti(ms: number) {
    const cv = confettiCanvasRef.value
    if (!cv) return
    confettiUntil = Date.now() + ms
    for (let i = 0; i < 150; i++) {
        confettiParts.push({
            x: Math.random() * cv.width,
            y: -20 - Math.random() * cv.height * 0.4,
            r: 5 + Math.random() * 6,
            c: CONFETTI_COLORS[i % CONFETTI_COLORS.length],
            vy: 3 + Math.random() * 5,
            vx: -2 + Math.random() * 4,
            rot: Math.random() * Math.PI,
            vr: -0.2 + Math.random() * 0.4,
            sh: Math.random() > 0.5 ? 'rect' : 'circ',
        })
    }
    if (!confettiRafId) confettiLoop()
}

function confettiLoop() {
    const cv = confettiCanvasRef.value
    if (!cv) return
    const ctx = cv.getContext('2d')!
    ctx.clearRect(0, 0, cv.width, cv.height)

    if (Date.now() < confettiUntil && confettiParts.length < 400 && Math.random() < 0.6) {
        for (let i = 0; i < 6; i++) {
            confettiParts.push({
                x: Math.random() * cv.width,
                y: -20,
                r: 5 + Math.random() * 6,
                c: CONFETTI_COLORS[(Math.random() * CONFETTI_COLORS.length) | 0],
                vy: 3 + Math.random() * 5,
                vx: -2 + Math.random() * 4,
                rot: Math.random() * 6,
                vr: -0.2 + Math.random() * 0.4,
                sh: Math.random() > 0.5 ? 'rect' : 'circ',
            })
        }
    }

    confettiParts.forEach((p) => {
        p.x += p.vx
        p.y += p.vy
        p.vy += 0.06
        p.rot += p.vr
        p.vx *= 0.995
        ctx.save()
        ctx.translate(p.x, p.y)
        ctx.rotate(p.rot)
        ctx.fillStyle = p.c
        if (p.sh === 'rect') {
            ctx.fillRect(-p.r / 2, -p.r / 2, p.r, p.r * 0.6)
        } else {
            ctx.beginPath()
            ctx.arc(0, 0, p.r / 2, 0, Math.PI * 2)
            ctx.fill()
        }
        ctx.restore()
    })

    confettiParts = confettiParts.filter((p) => p.y < cv.height + 30)

    if (confettiParts.length) {
        confettiRafId = requestAnimationFrame(confettiLoop)
    } else {
        ctx.clearRect(0, 0, cv.width, cv.height)
        confettiRafId = null
    }
}

// Menu
const menuOpen = ref(false)
let deferredInstallPrompt: any = null

function onBeforeInstallPrompt(e: Event) {
    e.preventDefault()
    deferredInstallPrompt = e
}

async function installApp() {
    if (deferredInstallPrompt) {
        deferredInstallPrompt.prompt()
        await deferredInstallPrompt.userChoice
        deferredInstallPrompt = null
        menuOpen.value = false
    } else {
        showToast('Gebruik "Deel → Zet op beginscherm"')
    }
}

function confirmReset() {
    if (confirm('Alles wissen? Alle voltooide opdrachten en foto\'s worden verwijderd.')) {
        state.value = { cells: {}, lines: [] }
        saveState()
        menuOpen.value = false
        showToast('Bord gereset')
    }
}

// Toast
const toastHtml = ref('')
const toastVisible = ref(false)
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(html: string) {
    toastHtml.value = html
    toastVisible.value = true
    if (toastTimer) clearTimeout(toastTimer)
    toastTimer = setTimeout(() => {
        toastVisible.value = false
    }, 2200)
}

// PWA icons + manifest injected at runtime
function makeIcon(size: number) {
    const cv = document.createElement('canvas')
    cv.width = cv.height = size
    const g = cv.getContext('2d')!
    g.fillStyle = '#000000'
    g.fillRect(0, 0, size, size)
    g.fillStyle = '#30d158'
    g.font = `700 ${Math.round(size * 0.56)}px -apple-system, "SF Pro Display", system-ui, sans-serif`
    g.textAlign = 'center'
    g.textBaseline = 'middle'
    g.fillText('B', size / 2, size * 0.55)
    return cv.toDataURL('image/png')
}

function setupPWA() {
    const i192 = makeIcon(192)
    const i512 = makeIcon(512)

    const al = document.createElement('link')
    al.rel = 'apple-touch-icon'
    al.href = i192
    document.head.appendChild(al)

    const fav = document.createElement('link')
    fav.rel = 'icon'
    fav.href = makeIcon(64)
    document.head.appendChild(fav)

    const manifest = {
        name: 'Kroegentocht Bingo — Düsseldorf',
        short_name: 'Bingo',
        start_url: '.',
        display: 'standalone',
        orientation: 'portrait',
        background_color: '#000000',
        theme_color: '#000000',
        icons: [
            { src: i192, sizes: '192x192', type: 'image/png', purpose: 'any maskable' },
            { src: i512, sizes: '512x512', type: 'image/png', purpose: 'any maskable' },
        ],
    }

    const blob = new Blob([JSON.stringify(manifest)], { type: 'application/manifest+json' })
    const ml = document.createElement('link')
    ml.rel = 'manifest'
    ml.href = URL.createObjectURL(blob)
    document.head.appendChild(ml)
}

onMounted(() => {
    sizeCanvas()
    window.addEventListener('resize', sizeCanvas)
    window.addEventListener('beforeinstallprompt', onBeforeInstallPrompt)
    setupPWA()
})

onUnmounted(() => {
    window.removeEventListener('resize', sizeCanvas)
    window.removeEventListener('beforeinstallprompt', onBeforeInstallPrompt)
    if (confettiRafId) cancelAnimationFrame(confettiRafId)
    if (toastTimer) clearTimeout(toastTimer)
})
</script>

<template>
    <div class="bingo-root">
        <Head title="Kroegentocht Bingo — Düsseldorf">
            <meta name="theme-color" content="#000000" />
            <meta name="apple-mobile-web-app-capable" content="yes" />
            <meta name="mobile-web-app-capable" content="yes" />
            <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
            <meta name="apple-mobile-web-app-title" content="Bingo" />
            <meta
                name="description"
                content="Team kroegentocht bingo voor Düsseldorf — 25 opdrachten, punten, foto's en BINGO."
            />
        </Head>

        <canvas ref="confettiCanvasRef" class="b-confetti"></canvas>

        <div class="b-app">
            <header class="b-topbar">
                <div class="b-brandrow">
                    <div class="b-brand">
                        <span class="b-kicker">Düsseldorf · Altstadt</span>
                        <h1 class="b-title">Kroegentocht Bingo</h1>
                        <p class="b-subtitle">Verzamel punten. Maak foto's. Pak je bingo.</p>
                    </div>
                    <button class="b-menu-btn" aria-label="Menu" @click="menuOpen = true">
                        <svg viewBox="0 0 24 24" fill="currentColor">
                            <circle cx="5" cy="12" r="2" />
                            <circle cx="12" cy="12" r="2" />
                            <circle cx="19" cy="12" r="2" />
                        </svg>
                    </button>
                </div>

                <div class="b-scorecard">
                    <div class="b-scorerow">
                        <div class="b-scol b-accent">
                            <div class="b-num">{{ totalPts }}</div>
                            <div class="b-lbl">Punten</div>
                        </div>
                        <div class="b-scol">
                            <div class="b-num">{{ completedCount }}<em>/25</em></div>
                            <div class="b-lbl">Voltooid</div>
                        </div>
                        <div class="b-scol">
                            <div class="b-num">{{ bingoCount }}</div>
                            <div class="b-lbl">Bingo's</div>
                        </div>
                    </div>
                    <div class="b-progress">
                        <div class="b-pmeta">
                            <span>{{ completedCount }} van 25 opdrachten</span>
                            <span>{{ totalPts }} / {{ maxPts }} pt</span>
                        </div>
                        <div class="b-bar">
                            <i :style="{ width: progressPct }"></i>
                        </div>
                    </div>
                </div>
            </header>

            <div class="b-legend">
                <span><i class="b-dot" style="background: var(--b-p1)"></i> 1 pt</span>
                <span><i class="b-dot" style="background: var(--b-p2)"></i> 2 pt</span>
                <span><i class="b-dot" style="background: var(--b-p3)"></i> 3 pt</span>
                <span><i class="b-dot" style="background: var(--b-p5)"></i> 5 pt</span>
            </div>

            <div class="b-grid">
                <div
                    v-for="(task, i) in TASKS"
                    :key="i"
                    :class="[
                        'b-cell',
                        `b-p${task.p}`,
                        { 'b-done': !!state.cells[i]?.done, 'b-win': winningCellSet.has(i) },
                    ]"
                    @click="openTask(i)"
                >
                    <div
                        v-if="state.cells[i]?.photo"
                        class="b-photo"
                        :style="{ backgroundImage: `url('${state.cells[i].photo}')` }"
                    ></div>
                    <div class="b-ct">{{ task.s }}</div>
                    <div class="b-check">
                        <svg
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="3"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        >
                            <path d="M20 6 9 17l-5-5" />
                        </svg>
                    </div>
                    <div class="b-badge">{{ task.p }}</div>
                </div>
            </div>
        </div>

        <!-- Task detail sheet -->
        <div class="b-scrim" :class="{ open: taskSheetOpen }" @click.self="closeTask">
            <div class="b-sheet">
                <div class="b-grip"></div>
                <span v-if="activeTask" class="b-tag" :class="`b-p${activeTask.p}`">
                    {{ activeTask.p }}{{ activeTask.p === 1 ? ' punt' : ' punten' }}
                </span>
                <h2>{{ activeTask?.t }}</h2>
                <p class="b-desc">{{ activeTask?.d }}</p>
                <div
                    class="b-proof"
                    :class="{ 'b-has': !!activeCellState?.photo }"
                    :style="
                        activeCellState?.photo
                            ? { backgroundImage: `url('${activeCellState.photo}')` }
                            : {}
                    "
                >
                    <div class="b-ph">
                        <svg
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="1.5"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        >
                            <path
                                d="M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3Z"
                            />
                            <circle cx="12" cy="13" r="3.5" />
                        </svg>
                        <span>Maak of upload een foto als bewijs</span>
                        <small>De opdracht telt mee zodra je een foto toevoegt</small>
                    </div>
                    <button class="b-retake" @click="triggerCapture">
                        <svg
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="1.7"
                            stroke-linecap="round"
                        >
                            <path d="M23 4v6h-6M1 20v-6h6" />
                            <path d="M3.5 9a9 9 0 0 1 15-3.4L23 10M1 14l4.5 4.4A9 9 0 0 0 20.5 15" />
                        </svg>
                        Opnieuw
                    </button>
                </div>
                <input
                    ref="fileInputRef"
                    type="file"
                    accept="image/*"
                    capture="environment"
                    class="b-hidden-file"
                    @change="onFileChange"
                />
                <div class="b-actions">
                    <button class="b-btn b-btn-primary" @click="triggerCapture">
                        <svg
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="1.8"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        >
                            <path
                                d="M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3Z"
                            />
                            <circle cx="12" cy="13" r="3.5" />
                        </svg>
                        {{ captureLabel }}
                    </button>
                    <button v-if="activeCellState?.done" class="b-btn b-btn-danger" @click="undoTask">
                        Voltooiing ongedaan maken
                    </button>
                    <button class="b-btn b-btn-ghost" @click="closeTask">Sluiten</button>
                </div>
            </div>
        </div>

        <!-- Menu sheet -->
        <div class="b-scrim" :class="{ open: menuOpen }" @click.self="menuOpen = false">
            <div class="b-sheet b-msheet">
                <div class="b-grip"></div>
                <h2>Menu</h2>
                <p class="b-desc" style="margin-bottom: 8px">
                    Alles wordt op dit toestel bewaard. Geen account, werkt offline.
                </p>
                <div class="b-row" @click="installApp">
                    <span class="b-ic">
                        <svg
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="1.7"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        >
                            <path d="M12 3v12m0 0 4-4m-4 4-4-4" />
                            <path d="M4 17v2a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-2" />
                        </svg>
                    </span>
                    <div class="b-tx">
                        <b>Installeer als app</b><small>Voeg toe aan je beginscherm</small>
                    </div>
                    <span class="b-chev">
                        <svg
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="2"
                            stroke-linecap="round"
                        >
                            <path d="m9 18 6-6-6-6" />
                        </svg>
                    </span>
                </div>
                <div class="b-row b-danger" @click="confirmReset">
                    <span class="b-ic">
                        <svg
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="1.7"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        >
                            <path
                                d="M3 6h18M8 6V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2m3 0v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6"
                            />
                        </svg>
                    </span>
                    <div class="b-tx">
                        <b>Alles wissen</b><small>Reset het bord en verwijder alle foto's</small>
                    </div>
                    <span class="b-chev">
                        <svg
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="2"
                            stroke-linecap="round"
                        >
                            <path d="m9 18 6-6-6-6" />
                        </svg>
                    </span>
                </div>
                <div class="b-actions" style="margin-top: 20px">
                    <button class="b-btn b-btn-ghost" @click="menuOpen = false">Gereed</button>
                </div>
            </div>
        </div>

        <!-- BINGO celebration overlay -->
        <div class="b-bingo" :class="{ open: bingoOpen }">
            <div class="b-bingo-word">Bingo!</div>
            <div class="b-bingo-kind">{{ bingoKindText }}</div>
            <div class="b-bingo-pts">{{ bingoPtsText }}</div>
            <button class="b-bingo-close" @click="bingoOpen = false">Doorgaan met spelen</button>
        </div>

        <!-- Toast -->
        <div class="b-toast" :class="{ show: toastVisible }" v-html="toastHtml"></div>
    </div>
</template>

<style>
/* =====================================================
   Bingo page — all styles namespaced under .bingo-root
   ===================================================== */
.bingo-root {
    --b-bg: #000000;
    --b-bg-soft: #0a0a0c;
    --b-surface: #1c1c1e;
    --b-surface-2: #2c2c2e;
    --b-hair: rgba(255, 255, 255, 0.1);
    --b-hair-soft: rgba(255, 255, 255, 0.06);
    --b-txt: #f5f5f7;
    --b-txt-2: #a1a1a6;
    --b-txt-3: #6e6e73;
    --b-accent: #30d158;
    --b-accent-press: #28b84c;
    --b-p1: #98989d;
    --b-p2: #0a84ff;
    --b-p3: #ff9f0a;
    --b-p5: #ff375f;
    --b-font: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Helvetica Neue',
        Helvetica, Arial, sans-serif;

    background: var(--b-bg);
    color: var(--b-txt);
    font-family: var(--b-font);
    font-size: 16px;
    min-height: 100vh;
    min-height: 100dvh;
    -webkit-font-smoothing: antialiased;
    overscroll-behavior-y: none;
    letter-spacing: -0.01em;
    padding: 0 0 env(safe-area-inset-bottom);
    box-sizing: border-box;
}

.bingo-root *,
.bingo-root *::before,
.bingo-root *::after {
    box-sizing: border-box;
    -webkit-tap-highlight-color: transparent;
}

/* Confetti canvas */
.bingo-root .b-confetti {
    position: fixed;
    inset: 0;
    z-index: 79;
    pointer-events: none;
}

/* App container */
.bingo-root .b-app {
    max-width: 540px;
    margin: 0 auto;
    padding: 0 18px 120px;
}

/* ---- Header ---- */
.bingo-root .b-topbar {
    position: sticky;
    top: 0;
    z-index: 30;
    margin: 0 -18px 18px;
    padding: calc(env(safe-area-inset-top) + 16px) 20px 16px;
    background: rgba(0, 0, 0, 0.62);
    backdrop-filter: saturate(160%) blur(22px);
    -webkit-backdrop-filter: saturate(160%) blur(22px);
    border-bottom: 1px solid var(--b-hair-soft);
}

.bingo-root .b-brandrow {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 12px;
}

.bingo-root .b-brand {
    display: flex;
    flex-direction: column;
}

.bingo-root .b-kicker {
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 0.01em;
    color: var(--b-accent);
    margin-bottom: 5px;
}

.bingo-root .b-title {
    margin: 0;
    font-size: clamp(28px, 8vw, 34px);
    font-weight: 700;
    line-height: 1.02;
    letter-spacing: -0.03em;
    color: var(--b-txt);
}

.bingo-root .b-subtitle {
    margin: 7px 0 0;
    font-size: 14px;
    font-weight: 400;
    color: var(--b-txt-2);
    letter-spacing: -0.01em;
}

.bingo-root .b-menu-btn {
    flex: none;
    width: 40px;
    height: 40px;
    border-radius: 999px;
    border: none;
    background: var(--b-surface-2);
    color: var(--b-txt);
    display: grid;
    place-items: center;
    cursor: pointer;
    transition: background 0.2s;
}

.bingo-root .b-menu-btn:active {
    background: #3a3a3c;
}

.bingo-root .b-menu-btn svg {
    width: 20px;
    height: 20px;
}

/* ---- Score card ---- */
.bingo-root .b-scorecard {
    margin-top: 18px;
    background: var(--b-surface);
    border-radius: 20px;
    padding: 4px 4px 16px;
}

.bingo-root .b-scorerow {
    display: flex;
}

.bingo-root .b-scol {
    flex: 1;
    padding: 18px 8px 14px;
    text-align: center;
    position: relative;
}

.bingo-root .b-scol + .b-scol::before {
    content: '';
    position: absolute;
    left: 0;
    top: 18px;
    bottom: 14px;
    width: 1px;
    background: var(--b-hair);
}

.bingo-root .b-num {
    font-size: 34px;
    font-weight: 700;
    line-height: 1;
    letter-spacing: -0.03em;
    font-variant-numeric: tabular-nums;
}

.bingo-root .b-scol.b-accent .b-num {
    color: var(--b-accent);
}

.bingo-root .b-num em {
    font-style: normal;
    color: var(--b-txt-3);
    font-weight: 600;
    font-size: 19px;
}

.bingo-root .b-lbl {
    margin-top: 7px;
    font-size: 12px;
    font-weight: 500;
    color: var(--b-txt-2);
    letter-spacing: 0;
}

.bingo-root .b-progress {
    margin: 2px 14px 0;
}

.bingo-root .b-pmeta {
    display: flex;
    justify-content: space-between;
    font-size: 12px;
    color: var(--b-txt-3);
    margin-bottom: 8px;
    font-variant-numeric: tabular-nums;
}

.bingo-root .b-bar {
    height: 7px;
    border-radius: 999px;
    background: var(--b-surface-2);
    overflow: hidden;
}

.bingo-root .b-bar i {
    display: block;
    height: 100%;
    border-radius: 999px;
    background: var(--b-accent);
    transition: width 0.6s cubic-bezier(0.22, 0.61, 0.36, 1);
}

/* ---- Legend ---- */
.bingo-root .b-legend {
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
    margin: 20px 2px 2px;
}

.bingo-root .b-legend span {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    font-size: 12px;
    color: var(--b-txt-2);
    font-weight: 500;
    background: var(--b-surface);
    padding: 6px 11px;
    border-radius: 999px;
}

.bingo-root .b-dot {
    width: 8px;
    height: 8px;
    border-radius: 999px;
    display: inline-block;
}

/* ---- Grid ---- */
.bingo-root .b-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
    margin-top: 12px;
}

.bingo-root .b-cell {
    position: relative;
    aspect-ratio: 1;
    border-radius: 20px;
    background: var(--b-surface);
    border: 1px solid transparent;
    padding: 14px;
    cursor: pointer;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    transition: transform 0.18s cubic-bezier(0.22, 0.61, 0.36, 1), background 0.3s ease;
}

.bingo-root .b-cell:active {
    transform: scale(0.95);
}

.bingo-root .b-ct {
    font-size: 14.5px;
    line-height: 1.26;
    font-weight: 600;
    color: var(--b-txt);
    text-wrap: balance;
    overflow: hidden;
    letter-spacing: -0.015em;
}

.bingo-root .b-badge {
    position: absolute;
    left: 12px;
    bottom: 12px;
    min-width: 23px;
    height: 23px;
    padding: 0 6px;
    border-radius: 999px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    font-size: 13px;
    font-weight: 700;
    color: #fff;
    line-height: 1;
    font-variant-numeric: tabular-nums;
}

.bingo-root .b-p1 .b-badge {
    background: var(--b-p1);
    color: #1c1c1e;
}
.bingo-root .b-p2 .b-badge {
    background: var(--b-p2);
}
.bingo-root .b-p3 .b-badge {
    background: var(--b-p3);
    color: #1c1c1e;
}
.bingo-root .b-p5 .b-badge {
    background: var(--b-p5);
}

/* Completed cell */
.bingo-root .b-cell.b-done {
    background: var(--b-surface-2);
}

.bingo-root .b-photo {
    position: absolute;
    inset: 0;
    background-size: cover;
    background-position: center;
    opacity: 0.42;
    transition: opacity 0.45s;
}

.bingo-root .b-cell.b-done::after {
    content: '';
    position: absolute;
    inset: 0;
    background: linear-gradient(180deg, rgba(0, 0, 0, 0.1), rgba(0, 0, 0, 0.72));
}

.bingo-root .b-check {
    position: absolute;
    top: 11px;
    right: 11px;
    z-index: 3;
    width: 25px;
    height: 25px;
    border-radius: 999px;
    background: var(--b-accent);
    display: none;
    place-items: center;
}

.bingo-root .b-cell.b-done .b-check {
    display: grid;
}

.bingo-root .b-check svg {
    width: 14px;
    height: 14px;
    color: #fff;
}

.bingo-root .b-cell.b-done .b-ct {
    position: relative;
    z-index: 3;
    color: #fff;
    opacity: 0.96;
}

.bingo-root .b-cell.b-done .b-badge {
    z-index: 3;
}

/* Winning line highlight */
.bingo-root .b-cell.b-win {
    box-shadow: inset 0 0 0 2px var(--b-accent);
}

.bingo-root .b-cell.b-win .b-photo {
    opacity: 0.5;
}

/* ---- Sheets / modals ---- */
.bingo-root .b-scrim {
    position: fixed;
    inset: 0;
    z-index: 60;
    background: rgba(0, 0, 0, 0.5);
    backdrop-filter: blur(2px);
    -webkit-backdrop-filter: blur(2px);
    display: none;
    align-items: flex-end;
    justify-content: center;
}

.bingo-root .b-scrim.open {
    display: flex;
}

.bingo-root .b-sheet {
    width: 100%;
    max-width: 540px;
    background: #161618;
    border: 1px solid var(--b-hair-soft);
    border-bottom: none;
    border-radius: 26px 26px 0 0;
    padding: 8px 20px calc(env(safe-area-inset-bottom) + 24px);
    transform: translateY(100%);
    transition: transform 0.44s cubic-bezier(0.22, 0.61, 0.36, 1);
    max-height: 92vh;
    overflow: auto;
}

.bingo-root .b-scrim.open .b-sheet {
    transform: translateY(0);
}

.bingo-root .b-grip {
    width: 38px;
    height: 5px;
    border-radius: 99px;
    background: var(--b-surface-2);
    margin: 8px auto 18px;
}

.bingo-root .b-tag {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    font-size: 12px;
    font-weight: 600;
    padding: 6px 13px;
    border-radius: 999px;
    color: #fff;
    margin-bottom: 16px;
    letter-spacing: 0;
}

.bingo-root .b-tag.b-p1 {
    background: var(--b-p1);
    color: #1c1c1e;
}
.bingo-root .b-tag.b-p2 {
    background: var(--b-p2);
}
.bingo-root .b-tag.b-p3 {
    background: var(--b-p3);
    color: #1c1c1e;
}
.bingo-root .b-tag.b-p5 {
    background: var(--b-p5);
}

.bingo-root .b-sheet h2 {
    font-weight: 700;
    font-size: 25px;
    line-height: 1.15;
    margin: 0 0 9px;
    letter-spacing: -0.03em;
    color: var(--b-txt);
}

.bingo-root .b-desc {
    color: var(--b-txt-2);
    font-size: 15px;
    line-height: 1.5;
    margin: 0 0 22px;
    letter-spacing: -0.01em;
}

/* Photo proof area */
.bingo-root .b-proof {
    border-radius: 18px;
    aspect-ratio: 16 / 11;
    display: grid;
    place-items: center;
    text-align: center;
    background: var(--b-surface);
    background-size: cover;
    background-position: center;
    position: relative;
    overflow: hidden;
    margin-bottom: 18px;
}

.bingo-root .b-ph {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
    padding: 18px;
}

.bingo-root .b-ph svg {
    width: 30px;
    height: 30px;
    color: var(--b-txt-3);
}

.bingo-root .b-ph span {
    font-size: 14px;
    font-weight: 500;
    color: var(--b-txt-2);
}

.bingo-root .b-ph small {
    font-size: 12px;
    color: var(--b-txt-3);
}

.bingo-root .b-proof.b-has .b-ph {
    display: none;
}

.bingo-root .b-retake {
    position: absolute;
    right: 11px;
    bottom: 11px;
    display: none;
    font-size: 12px;
    font-weight: 600;
    padding: 8px 14px;
    border-radius: 999px;
    background: rgba(0, 0, 0, 0.6);
    color: #fff;
    border: none;
    backdrop-filter: blur(8px);
    align-items: center;
    gap: 6px;
    cursor: pointer;
    font-family: var(--b-font);
}

.bingo-root .b-proof.b-has .b-retake {
    display: inline-flex;
}

.bingo-root .b-retake svg {
    width: 13px;
    height: 13px;
}

/* Action buttons */
.bingo-root .b-actions {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.bingo-root .b-btn {
    width: 100%;
    border: none;
    border-radius: 999px;
    padding: 16px;
    font-family: var(--b-font);
    font-size: 16px;
    font-weight: 600;
    letter-spacing: -0.01em;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 9px;
    transition: transform 0.15s, background 0.2s, opacity 0.2s;
}

.bingo-root .b-btn:active {
    transform: scale(0.98);
}

.bingo-root .b-btn svg {
    width: 18px;
    height: 18px;
}

.bingo-root .b-btn-primary {
    background: var(--b-accent);
    color: #fff;
}

.bingo-root .b-btn-primary:active {
    background: var(--b-accent-press);
}

.bingo-root .b-btn-ghost {
    background: var(--b-surface-2);
    color: var(--b-txt);
}

.bingo-root .b-btn-danger {
    background: transparent;
    color: var(--b-p5);
}

.bingo-root .b-hidden-file {
    display: none;
}

/* Menu sheet */
.bingo-root .b-msheet h2 {
    margin: 2px 0 4px;
    font-size: 23px;
    color: var(--b-txt);
}

.bingo-root .b-row {
    display: flex;
    align-items: center;
    gap: 14px;
    padding: 16px 2px;
    border-bottom: 1px solid var(--b-hair-soft);
    cursor: pointer;
}

.bingo-root .b-row:last-of-type {
    border-bottom: none;
}

.bingo-root .b-ic {
    width: 38px;
    height: 38px;
    border-radius: 999px;
    background: var(--b-surface-2);
    display: grid;
    place-items: center;
    flex: none;
}

.bingo-root .b-ic svg {
    width: 19px;
    height: 19px;
    color: var(--b-accent);
}

.bingo-root .b-row.b-danger .b-ic svg {
    color: var(--b-p5);
}

.bingo-root .b-tx {
    flex: 1;
}

.bingo-root .b-tx b {
    display: block;
    font-size: 16px;
    font-weight: 600;
    color: var(--b-txt);
}

.bingo-root .b-tx small {
    display: block;
    font-size: 13px;
    color: var(--b-txt-3);
    margin-top: 2px;
}

.bingo-root .b-chev {
    color: var(--b-txt-3);
}

.bingo-root .b-chev svg {
    width: 18px;
    height: 18px;
}

/* ---- BINGO overlay ---- */
.bingo-root .b-bingo {
    position: fixed;
    inset: 0;
    z-index: 80;
    display: none;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: rgba(0, 0, 0, 0.72);
    backdrop-filter: blur(24px);
    -webkit-backdrop-filter: blur(24px);
    text-align: center;
    padding: 32px;
}

.bingo-root .b-bingo.open {
    display: flex;
}

.bingo-root .b-bingo-word {
    font-size: clamp(64px, 21vw, 108px);
    font-weight: 700;
    line-height: 0.9;
    color: var(--b-accent);
    letter-spacing: -0.045em;
    animation: bingo-pop 0.55s cubic-bezier(0.2, 0.9, 0.3, 1) both;
}

.bingo-root .b-bingo-kind {
    margin-top: 18px;
    font-size: 17px;
    font-weight: 600;
    color: var(--b-txt);
    letter-spacing: -0.01em;
}

.bingo-root .b-bingo-pts {
    margin-top: 7px;
    font-size: 15px;
    color: var(--b-txt-2);
}

.bingo-root .b-bingo-close {
    margin-top: 38px;
    background: var(--b-accent);
    color: #fff;
    border: none;
    border-radius: 999px;
    padding: 15px 40px;
    font-weight: 600;
    font-size: 16px;
    cursor: pointer;
    font-family: var(--b-font);
}

@keyframes bingo-pop {
    0% {
        transform: scale(0.6);
        opacity: 0;
    }
    60% {
        transform: scale(1.05);
    }
    100% {
        transform: scale(1);
        opacity: 1;
    }
}

/* ---- Toast ---- */
.bingo-root .b-toast {
    position: fixed;
    left: 50%;
    bottom: calc(env(safe-area-inset-bottom) + 24px);
    transform: translateX(-50%) translateY(20px);
    background: rgba(44, 44, 46, 0.92);
    backdrop-filter: blur(12px);
    border: 1px solid var(--b-hair);
    color: var(--b-txt);
    padding: 13px 20px;
    border-radius: 999px;
    font-size: 14px;
    font-weight: 500;
    z-index: 70;
    opacity: 0;
    transition: opacity 0.3s, transform 0.3s;
    box-shadow: 0 12px 40px rgba(0, 0, 0, 0.5);
    pointer-events: none;
    white-space: nowrap;
}

.bingo-root .b-toast.show {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
}

.bingo-root .b-toast b {
    color: var(--b-accent);
    font-weight: 700;
}
</style>
