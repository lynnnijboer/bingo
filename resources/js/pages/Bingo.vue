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

interface OtherTeam {
    id: string
    name: string
    pts: number
}

interface LeaderboardRow {
    id: string
    name: string
    pts: number
    isYou: boolean
    done?: number
    bingos?: number
}

type Screen = 'home' | 'board' | 'leaderboard' | 'planning'

interface AppState {
    cells: Record<number, CellState>
    lines: string[]
    team: string
    others: OtherTeam[]
    screen: Screen
}

interface PlanItem {
    t: string
    a: string
    v?: string
    hl?: boolean
}

interface PlanDay {
    day: string
    items: PlanItem[]
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

const PLAN: PlanDay[] = [
    {
        day: 'Vrijdag',
        items: [
            { t: '16:00–16:30', a: 'Aankomst team', v: 'Numa Düsseldorf Leo' },
            { t: '18:30', a: 'Avondeten', v: 'Pizzeria Romantica' },
            { t: '21:30', a: 'Beerpongbar Düsseldorf' },
            { t: '00:00', a: 'Düsseldorfse clubs verkennen' },
        ],
    },
    {
        day: 'Zaterdag',
        items: [
            { t: '11:30', a: 'Lunchen' },
            { t: '13:00', a: 'Bingo door Düsseldorf', hl: true },
            { t: '15:30', a: 'Vrije tijd + klaarmaken' },
            { t: '17:00', a: 'Avondeten', v: 'Louisiana Düsseldorf Altstadt' },
            { t: '19:00', a: 'WK kijken', v: 'The Irish Pub Bei Fatty' },
            { t: '21:30', a: 'Pubgolf' },
        ],
    },
    {
        day: 'Zondag',
        items: [
            { t: '11:00', a: 'Uitchecken & KWW' },
            { t: '13:00', a: 'Brak naar de osso' },
        ],
    },
]

const KEY = 'ddorf_bingo_v1'
const VALID_SCREENS: Screen[] = ['home', 'board', 'leaderboard', 'planning']

function loadState(): AppState {
    try {
        const raw = localStorage.getItem(KEY)
        if (raw) {
            const o = JSON.parse(raw)
            if (o && o.cells) {
                return {
                    cells: o.cells,
                    lines: Array.isArray(o.lines) ? o.lines : [],
                    team: typeof o.team === 'string' ? o.team : '',
                    others: Array.isArray(o.others) ? o.others : [],
                    screen: VALID_SCREENS.includes(o.screen) ? o.screen : 'home',
                }
            }
        }
    } catch {}
    return { cells: {}, lines: [], team: '', others: [], screen: 'home' }
}

const state = ref<AppState>(loadState())

function saveState() {
    try {
        localStorage.setItem(KEY, JSON.stringify(state.value))
    } catch {
        showToast('Opslag vol — foto te groot')
    }
}

const LINES = (() => {
    const lines: { id: string; name: string; idx: number[] }[] = []
    const rows = Math.floor(TASKS.length / 3)
    for (let r = 0; r < rows; r++) {
        lines.push({ id: `r${r}`, name: `Rij ${r + 1}`, idx: [0, 1, 2].map((c) => r * 3 + c) })
    }
    return lines
})()

state.value.lines = state.value.lines.filter(
    (id) => LINES.some((l) => l.id === id && l.idx.every((i) => !!(state.value.cells[i]?.done))),
)

const maxPts = TASKS.reduce((a, t) => a + t.p, 0)

const completedCount = computed(() => TASKS.reduce((n, _, i) => n + (state.value.cells[i]?.done ? 1 : 0), 0))
const totalPts = computed(() => TASKS.reduce((pts, t, i) => pts + (state.value.cells[i]?.done ? t.p : 0), 0))
const bingoCount = computed(() => state.value.lines.length)
const progressPct = computed(() => `${(completedCount.value / 25) * 100}%`)

const winningCellSet = computed(() => {
    const set = new Set<number>()
    LINES.forEach((l) => { if (state.value.lines.includes(l.id)) l.idx.forEach((i) => set.add(i)) })
    return set
})

// Screen routing
const currentScreen = computed(() => state.value.screen)

function go(screen: Screen) {
    state.value.screen = screen
    saveState()
    window.scrollTo(0, 0)
}

// Team
const teamName = computed(() => state.value.team.trim() || 'Jouw team')

const teamInputValue = computed({
    get: () => state.value.team,
    set: (val: string) => { state.value.team = val; saveState() },
})

const homeBoardSub = computed(() =>
    completedCount.value > 0
        ? `${completedCount.value}/25 voltooid · ${totalPts.value} pt`
        : 'Speel de 25 opdrachten',
)

const homeLbSub = computed(() =>
    state.value.others.length > 0
        ? `${state.value.others.length + 1} teams in de strijd`
        : 'Vergelijk de teams',
)

// Leaderboard
const leaderboardRows = computed<LeaderboardRow[]>(() => {
    const you: LeaderboardRow = {
        id: '__you',
        name: teamName.value,
        pts: totalPts.value,
        isYou: true,
        done: completedCount.value,
        bingos: state.value.lines.length,
    }
    const others: LeaderboardRow[] = state.value.others.map((o) => ({ id: o.id, name: o.name, pts: o.pts, isYou: false }))
    return [you, ...others].sort((a, b) => b.pts - a.pts || (a.isYou ? -1 : 1))
})

// Task sheet
const taskSheetOpen = ref(false)
const activeIdx = ref<number | null>(null)
const activeTask = computed(() => (activeIdx.value !== null ? TASKS[activeIdx.value] : null))
const activeCellState = computed(() => (activeIdx.value !== null ? (state.value.cells[activeIdx.value] ?? null) : null))
const captureLabel = computed(() => (activeCellState.value?.done ? 'Andere foto maken' : 'Foto maken & voltooien'))

function openTask(i: number) { activeIdx.value = i; taskSheetOpen.value = true }
function closeTask() { taskSheetOpen.value = false; activeIdx.value = null }

const fileInputRef = ref<HTMLInputElement | null>(null)
function triggerCapture() { fileInputRef.value?.click() }

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
        setTimeout(() => { if (!bingoOpen.value) closeTask() }, 650)
    })
    input.value = ''
}

function resizeImage(file: File, cb: (dataUrl: string) => void) {
    const img = new Image()
    const url = URL.createObjectURL(file)
    img.onload = () => {
        const max = 720
        let { width: w, height: h } = img
        if (w > h && w > max) { h = Math.round((h * max) / w); w = max }
        else if (h >= w && h > max) { w = Math.round((w * max) / h); h = max }
        const cv = document.createElement('canvas')
        cv.width = w; cv.height = h
        cv.getContext('2d')!.drawImage(img, 0, 0, w, h)
        URL.revokeObjectURL(url)
        cb(cv.toDataURL('image/jpeg', 0.72))
    }
    img.onerror = () => { URL.revokeObjectURL(url); showToast('Kon foto niet laden') }
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
    saveState(); closeTask(); showToast('Opdracht teruggezet')
}

// Bingo
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

function isDone(i: number) { return !!(state.value.cells[i]?.done) }
function lineComplete(line: { idx: number[] }) { return line.idx.every(isDone) }

function checkBingo() {
    const newly = LINES.filter((l) => lineComplete(l) && !state.value.lines.includes(l.id))
    if (newly.length) { newly.forEach((l) => state.value.lines.push(l.id)); saveState(); celebrate(newly) }
}

function celebrate(lines: typeof LINES) {
    closeTask(); bingoCelebLines.value = lines; bingoOpen.value = true
    fireConfetti(2600)
    if (navigator.vibrate) navigator.vibrate([40, 60, 40, 60, 120])
}

// Confetti
const confettiCanvasRef = ref<HTMLCanvasElement | null>(null)
let confettiParts: { x: number; y: number; r: number; c: string; vy: number; vx: number; rot: number; vr: number; sh: 'rect' | 'circ' }[] = []
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
        confettiParts.push({ x: Math.random() * cv.width, y: -20 - Math.random() * cv.height * 0.4, r: 5 + Math.random() * 6, c: CONFETTI_COLORS[i % CONFETTI_COLORS.length], vy: 3 + Math.random() * 5, vx: -2 + Math.random() * 4, rot: Math.random() * Math.PI, vr: -0.2 + Math.random() * 0.4, sh: Math.random() > 0.5 ? 'rect' : 'circ' })
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
            confettiParts.push({ x: Math.random() * cv.width, y: -20, r: 5 + Math.random() * 6, c: CONFETTI_COLORS[(Math.random() * CONFETTI_COLORS.length) | 0], vy: 3 + Math.random() * 5, vx: -2 + Math.random() * 4, rot: Math.random() * 6, vr: -0.2 + Math.random() * 0.4, sh: Math.random() > 0.5 ? 'rect' : 'circ' })
        }
    }
    confettiParts.forEach((p) => {
        p.x += p.vx; p.y += p.vy; p.vy += 0.06; p.rot += p.vr; p.vx *= 0.995
        ctx.save(); ctx.translate(p.x, p.y); ctx.rotate(p.rot); ctx.fillStyle = p.c
        if (p.sh === 'rect') ctx.fillRect(-p.r / 2, -p.r / 2, p.r, p.r * 0.6)
        else { ctx.beginPath(); ctx.arc(0, 0, p.r / 2, 0, Math.PI * 2); ctx.fill() }
        ctx.restore()
    })
    confettiParts = confettiParts.filter((p) => p.y < cv.height + 30)
    if (confettiParts.length) { confettiRafId = requestAnimationFrame(confettiLoop) }
    else { ctx.clearRect(0, 0, cv.width, cv.height); confettiRafId = null }
}

// Menu
const menuOpen = ref(false)
let deferredInstallPrompt: any = null

function onBeforeInstallPrompt(e: Event) { e.preventDefault(); deferredInstallPrompt = e }

async function installApp() {
    if (deferredInstallPrompt) {
        deferredInstallPrompt.prompt()
        await deferredInstallPrompt.userChoice
        deferredInstallPrompt = null; menuOpen.value = false
    } else {
        showToast('Gebruik "Deel → Zet op beginscherm"')
    }
}

function confirmReset() {
    if (confirm('Alles wissen? Alle voltooide opdrachten en foto\'s worden verwijderd. Teamnaam en ranglijst blijven staan.')) {
        state.value.cells = {}; state.value.lines = []
        saveState(); menuOpen.value = false; showToast('Bord gereset')
    }
}

// Toast
const toastHtml = ref('')
const toastVisible = ref(false)
let toastTimer: ReturnType<typeof setTimeout> | null = null

function showToast(html: string) {
    toastHtml.value = html; toastVisible.value = true
    if (toastTimer) clearTimeout(toastTimer)
    toastTimer = setTimeout(() => { toastVisible.value = false }, 2200)
}

// PWA
function makeIcon(size: number) {
    const cv = document.createElement('canvas')
    cv.width = cv.height = size
    const g = cv.getContext('2d')!
    g.fillStyle = '#000000'; g.fillRect(0, 0, size, size)
    g.fillStyle = '#30d158'
    g.font = `700 ${Math.round(size * 0.56)}px -apple-system, "SF Pro Display", system-ui, sans-serif`
    g.textAlign = 'center'; g.textBaseline = 'middle'
    g.fillText('B', size / 2, size * 0.55)
    return cv.toDataURL('image/png')
}

function setupPWA() {
    const i192 = makeIcon(192); const i512 = makeIcon(512)
    const al = document.createElement('link'); al.rel = 'apple-touch-icon'; al.href = i192; document.head.appendChild(al)
    const fav = document.createElement('link'); fav.rel = 'icon'; fav.href = makeIcon(64); document.head.appendChild(fav)
    const manifest = { name: 'Kroegentocht Bingo — Düsseldorf', short_name: 'Bingo', start_url: '.', display: 'standalone', orientation: 'portrait', background_color: '#000000', theme_color: '#000000', icons: [{ src: i192, sizes: '192x192', type: 'image/png', purpose: 'any maskable' }, { src: i512, sizes: '512x512', type: 'image/png', purpose: 'any maskable' }] }
    const blob = new Blob([JSON.stringify(manifest)], { type: 'application/manifest+json' })
    const ml = document.createElement('link'); ml.rel = 'manifest'; ml.href = URL.createObjectURL(blob); document.head.appendChild(ml)
}

// Team add/edit sheet
const teamSheetOpen = ref(false)
const editTeamId = ref<string | null>(null)
const otName = ref('')
const otPts = ref(0)

const isEditingTeam = computed(() => editTeamId.value !== null)
const teamSheetTitle = computed(() => (isEditingTeam.value ? 'Team bewerken' : 'Team toevoegen'))
const teamSheetDesc = computed(() => isEditingTeam.value ? 'Pas de naam of punten aan.' : 'Voeg een ander team toe en houd hun punten bij.')

function openTeamSheet(id?: string) {
    editTeamId.value = id ?? null
    const team = id ? state.value.others.find((x) => x.id === id) : null
    otName.value = team ? team.name : ''; otPts.value = team ? team.pts : 0
    teamSheetOpen.value = true
}

function closeTeamSheet() { teamSheetOpen.value = false; editTeamId.value = null }

function saveTeam() {
    const name = otName.value.trim()
    const pts = Math.max(0, otPts.value)
    if (!name) { showToast('Geef het team een naam'); return }
    const wasEditing = !!editTeamId.value
    if (editTeamId.value) {
        const team = state.value.others.find((x) => x.id === editTeamId.value)
        if (team) { team.name = name; team.pts = pts }
    } else {
        state.value.others.push({ id: 't' + Date.now().toString(36), name, pts })
    }
    saveState(); closeTeamSheet(); showToast(wasEditing ? 'Team bijgewerkt' : 'Team toegevoegd')
}

function deleteTeam() {
    if (editTeamId.value) {
        state.value.others = state.value.others.filter((x) => x.id !== editTeamId.value)
        saveState(); closeTeamSheet(); showToast('Team verwijderd')
    }
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
            <meta name="description" content="Team kroegentocht bingo voor Düsseldorf — 25 opdrachten, punten, foto's en BINGO." />
        </Head>

        <canvas ref="confettiCanvasRef" class="b-confetti"></canvas>

        <!-- ==================== HOME ==================== -->
        <section v-show="currentScreen === 'home'" class="b-screen">
            <div class="b-home">
                <div class="b-hero">
                    <h1 class="b-title">Kroegentocht<br />Bingo</h1>
                    <p class="b-subtitle">Verzamel punten. Maak foto's. Pak je bingo.</p>

                    <div class="b-field">
                        <label for="teamInput">Teamnaam</label>
                        <div class="b-inwrap">
                            <span class="b-field-ic">
                                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round">
                                    <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2" />
                                    <circle cx="9" cy="7" r="4" />
                                    <path d="M23 21v-2a4 4 0 0 0-3-3.87M16 3.13a4 4 0 0 1 0 7.75" />
                                </svg>
                            </span>
                            <input id="teamInput" v-model="teamInputValue" type="text" maxlength="28" placeholder="Bijv. De Altbier Bende" autocomplete="off" />
                        </div>
                    </div>

                    <div class="b-navcards">
                        <button class="b-navcard b-navcard-primary" @click="go('board')">
                            <span class="b-nic">
                                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
                                    <rect x="3" y="3" width="7" height="7" rx="1.5" /><rect x="14" y="3" width="7" height="7" rx="1.5" /><rect x="3" y="14" width="7" height="7" rx="1.5" /><rect x="14" y="14" width="7" height="7" rx="1.5" />
                                </svg>
                            </span>
                            <span class="b-ntx"><b>Naar het bord</b><small>{{ homeBoardSub }}</small></span>
                            <span class="b-narr"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="m9 18 6-6-6-6" /></svg></span>
                        </button>
                        <button class="b-navcard b-navcard-secondary" @click="go('leaderboard')">
                            <span class="b-nic">
                                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round">
                                    <path d="M8 21h8M12 17v4M7 4h10v5a5 5 0 0 1-10 0V4Z" /><path d="M17 5h3v2a3 3 0 0 1-3 3M7 5H4v2a3 3 0 0 0 3 3" />
                                </svg>
                            </span>
                            <span class="b-ntx"><b>Ranglijst</b><small>{{ homeLbSub }}</small></span>
                            <span class="b-narr"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="m9 18 6-6-6-6" /></svg></span>
                        </button>
                        <button class="b-navcard b-navcard-secondary" @click="go('planning')">
                            <span class="b-nic">
                                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round">
                                    <rect x="3" y="4.5" width="18" height="16" rx="2.5" /><path d="M3 9h18M8 2.5v4M16 2.5v4" />
                                </svg>
                            </span>
                            <span class="b-ntx"><b>Planning</b><small>Het weekendprogramma</small></span>
                            <span class="b-narr"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="m9 18 6-6-6-6" /></svg></span>
                        </button>
                    </div>
                </div>
                <p class="b-foot">Alles wordt op dit toestel bewaard.<br />Geen account · werkt offline.</p>
            </div>
        </section>

        <!-- ==================== BOARD ==================== -->
        <section v-show="currentScreen === 'board'" class="b-screen">
            <div class="b-app">
                <header class="b-topbar">
                    <div class="b-navrow">
                        <span class="b-teamchip"><i class="b-dotg"></i><span>{{ teamName }}</span></span>
                    </div>
                    <div class="b-brandrow">
                        <div class="b-brand">
                            <h1 class="b-title">Kroegentocht Bingo</h1>
                            <p class="b-subtitle">Verzamel punten. Maak foto's. Pak je bingo.</p>
                        </div>
                        <button class="b-menu-btn" aria-label="Menu" @click="menuOpen = true">
                            <svg viewBox="0 0 24 24" fill="currentColor"><circle cx="5" cy="12" r="2" /><circle cx="12" cy="12" r="2" /><circle cx="19" cy="12" r="2" /></svg>
                        </button>
                    </div>
                    <div class="b-scorecard">
                        <div class="b-scorerow">
                            <div class="b-scol b-accent"><div class="b-num">{{ totalPts }}</div><div class="b-lbl">Punten</div></div>
                            <div class="b-scol"><div class="b-num">{{ completedCount }}<em>/25</em></div><div class="b-lbl">Voltooid</div></div>
                            <div class="b-scol"><div class="b-num">{{ bingoCount }}</div><div class="b-lbl">Bingo's</div></div>
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
                        :class="['b-cell', `b-p${task.p}`, { 'b-done': !!state.cells[i]?.done, 'b-win': winningCellSet.has(i) }]"
                        @click="openTask(i)"
                    >
                        <div v-if="state.cells[i]?.photo" class="b-photo" :style="{ backgroundImage: `url('${state.cells[i].photo}')` }"></div>
                        <div class="b-ct">{{ task.s }}</div>
                        <div class="b-check">
                            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><path d="M20 6 9 17l-5-5" /></svg>
                        </div>
                        <div class="b-badge">{{ task.p }}</div>
                    </div>
                </div>
            </div>
        </section>

        <!-- ==================== LEADERBOARD ==================== -->
        <section v-show="currentScreen === 'leaderboard'" class="b-screen">
            <div class="b-lb">
                <div class="b-navrow">
                    <span class="b-teamchip"><i class="b-dotg"></i><span>{{ teamName }}</span></span>
                </div>
                <span class="b-kicker">Düsseldorf · Altstadt</span>
                <h1 class="b-lbtitle">Ranglijst</h1>
                <p class="b-lbsub">Jouw team telt automatisch mee. Voeg andere teams toe en houd hun punten bij.</p>

                <div class="b-lblist">
                    <div
                        v-for="(row, i) in leaderboardRows"
                        :key="row.id"
                        :class="['b-lbrow', { 'b-lbrow-you': row.isYou, 'b-lbrow-top1': i === 0 }]"
                        @click="row.isYou ? go('board') : openTeamSheet(row.id)"
                    >
                        <div class="b-rank">{{ i + 1 }}</div>
                        <div class="b-who">
                            <span v-if="row.isYou" class="b-youtag">JOUW TEAM</span>
                            <b>{{ row.name }}</b>
                            <small v-if="row.isYou">{{ row.done }}/25 voltooid · {{ row.bingos }} bingo{{ row.bingos === 1 ? '' : "'s" }}</small>
                            <small v-else>Tik om punten te bewerken</small>
                        </div>
                        <div class="b-ptsbox"><b>{{ row.pts }}</b><small>punten</small></div>
                    </div>
                </div>

                <p v-if="state.others.length === 0" class="b-lbempty">
                    Nog geen andere teams. Voeg er een toe om de stand te vergelijken.
                </p>

                <div class="b-actions" style="margin-top: 22px">
                    <button class="b-btn b-btn-ghost" @click="openTeamSheet()">
                        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round"><path d="M12 5v14M5 12h14" /></svg>
                        Team toevoegen
                    </button>
                </div>
            </div>
        </section>

        <!-- ==================== PLANNING ==================== -->
        <section v-show="currentScreen === 'planning'" class="b-screen">
            <div class="b-plan">
                <span class="b-kicker">Teamweekend · Avanti Wilskracht Vrouwen 1</span>
                <h1 class="b-lbtitle">Planning</h1>
                <p class="b-lbsub">Het volledige weekendprogramma in Düsseldorf.</p>

                <div v-for="group in PLAN" :key="group.day" class="b-daygroup">
                    <div class="b-dayhead">
                        <h2>{{ group.day }}</h2>
                        <span class="b-ln"></span>
                        <span class="b-cnt">{{ group.items.length }} blokken</span>
                    </div>
                    <div class="b-daycard">
                        <div
                            v-for="(item, j) in group.items"
                            :key="j"
                            :class="['b-pitem', { 'b-pitem-hl': item.hl }]"
                            @click="item.hl ? go('board') : undefined"
                        >
                            <div class="b-ptime">{{ item.t }}</div>
                            <div class="b-pbody">
                                <b>{{ item.a }}</b>
                                <small v-if="item.v">{{ item.v }}</small>
                                <span v-if="item.hl" class="b-ptag">
                                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
                                        <rect x="3" y="3" width="7" height="7" rx="1.5" /><rect x="14" y="3" width="7" height="7" rx="1.5" /><rect x="3" y="14" width="7" height="7" rx="1.5" /><rect x="14" y="14" width="7" height="7" rx="1.5" />
                                    </svg>
                                    Open het bingobord
                                </span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- Task detail sheet -->
        <div class="b-scrim" :class="{ open: taskSheetOpen }" @click.self="closeTask">
            <div class="b-sheet">
                <div class="b-grip"></div>
                <span v-if="activeTask" class="b-tag" :class="`b-p${activeTask.p}`">
                    {{ activeTask.p }}{{ activeTask.p === 1 ? ' punt' : ' punten' }}
                </span>
                <h2>{{ activeTask?.t }}</h2>
                <p class="b-desc">{{ activeTask?.d }}</p>
                <div class="b-proof" :class="{ 'b-has': !!activeCellState?.photo }" :style="activeCellState?.photo ? { backgroundImage: `url('${activeCellState.photo}')` } : {}">
                    <div class="b-ph">
                        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3Z" /><circle cx="12" cy="13" r="3.5" /></svg>
                        <span>Maak of upload een foto als bewijs</span>
                        <small>De opdracht telt mee zodra je een foto toevoegt</small>
                    </div>
                    <button class="b-retake" @click="triggerCapture">
                        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round"><path d="M23 4v6h-6M1 20v-6h6" /><path d="M3.5 9a9 9 0 0 1 15-3.4L23 10M1 14l4.5 4.4A9 9 0 0 0 20.5 15" /></svg>
                        Opnieuw
                    </button>
                </div>
                <input ref="fileInputRef" type="file" accept="image/*" capture="environment" class="b-hidden-file" @change="onFileChange" />
                <div class="b-actions">
                    <button class="b-btn b-btn-primary" @click="triggerCapture">
                        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3Z" /><circle cx="12" cy="13" r="3.5" /></svg>
                        {{ captureLabel }}
                    </button>
                    <button v-if="activeCellState?.done" class="b-btn b-btn-danger" @click="undoTask">Voltooiing ongedaan maken</button>
                    <button class="b-btn b-btn-ghost" @click="closeTask">Sluiten</button>
                </div>
            </div>
        </div>

        <!-- Menu sheet -->
        <div class="b-scrim" :class="{ open: menuOpen }" @click.self="menuOpen = false">
            <div class="b-sheet b-msheet">
                <div class="b-grip"></div>
                <h2>Menu</h2>
                <p class="b-desc" style="margin-bottom: 8px">Alles wordt op dit toestel bewaard. Geen account, werkt offline.</p>
                <div class="b-row" @click="installApp">
                    <span class="b-ic"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round"><path d="M12 3v12m0 0 4-4m-4 4-4-4" /><path d="M4 17v2a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-2" /></svg></span>
                    <div class="b-tx"><b>Installeer als app</b><small>Voeg toe aan je beginscherm</small></div>
                    <span class="b-chev"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="m9 18 6-6-6-6" /></svg></span>
                </div>
                <div class="b-row b-danger" @click="confirmReset">
                    <span class="b-ic"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round"><path d="M3 6h18M8 6V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2m3 0v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6" /></svg></span>
                    <div class="b-tx"><b>Alles wissen</b><small>Reset het bord en verwijder alle foto's</small></div>
                    <span class="b-chev"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="m9 18 6-6-6-6" /></svg></span>
                </div>
                <div class="b-actions" style="margin-top: 20px">
                    <button class="b-btn b-btn-ghost" @click="menuOpen = false">Gereed</button>
                </div>
            </div>
        </div>

        <!-- Team add/edit sheet -->
        <div class="b-scrim" :class="{ open: teamSheetOpen }" @click.self="closeTeamSheet">
            <div class="b-sheet">
                <div class="b-grip"></div>
                <h2>{{ teamSheetTitle }}</h2>
                <p class="b-desc" style="margin-bottom: 18px">{{ teamSheetDesc }}</p>
                <div class="b-field" style="margin-top: 0">
                    <label for="otName">Teamnaam</label>
                    <div class="b-inwrap">
                        <input id="otName" v-model="otName" type="text" maxlength="28" placeholder="Bijv. De Radschläger" autocomplete="off" />
                    </div>
                </div>
                <div class="b-field">
                    <label>Punten</label>
                    <div class="b-numrow">
                        <div class="b-stepper">
                            <button type="button" aria-label="Minder" @click="otPts = Math.max(0, otPts - 1)">−</button>
                            <input v-model.number="otPts" type="number" inputmode="numeric" />
                            <button type="button" aria-label="Meer" @click="otPts = Math.max(0, otPts + 1)">+</button>
                        </div>
                    </div>
                </div>
                <div class="b-actions" style="margin-top: 22px">
                    <button class="b-btn b-btn-primary" @click="saveTeam">Opslaan</button>
                    <button v-if="isEditingTeam" class="b-btn b-btn-danger" @click="deleteTeam">Team verwijderen</button>
                    <button class="b-btn b-btn-ghost" @click="closeTeamSheet">Annuleren</button>
                </div>
            </div>
        </div>

        <!-- BINGO overlay -->
        <div class="b-bingo" :class="{ open: bingoOpen }">
            <div class="b-bingo-word">Bingo!</div>
            <div class="b-bingo-kind">{{ bingoKindText }}</div>
            <div class="b-bingo-pts">{{ bingoPtsText }}</div>
            <button class="b-bingo-close" @click="bingoOpen = false">Doorgaan met spelen</button>
        </div>

        <!-- Bottom tab bar -->
        <nav class="b-tabbar">
            <div class="b-tabbar-inner">
                <button :class="['b-tab', { active: currentScreen === 'home' }]" @click="go('home')">
                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round"><path d="M3 10.5 12 3l9 7.5" /><path d="M5 9.5V20a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1V9.5" /></svg>
                    <span>Home</span>
                </button>
                <button :class="['b-tab', { active: currentScreen === 'board' }]" @click="go('board')">
                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7" rx="1.5" /><rect x="14" y="3" width="7" height="7" rx="1.5" /><rect x="3" y="14" width="7" height="7" rx="1.5" /><rect x="14" y="14" width="7" height="7" rx="1.5" /></svg>
                    <span>Bord</span>
                </button>
                <button :class="['b-tab', { active: currentScreen === 'leaderboard' }]" @click="go('leaderboard')">
                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round"><path d="M8 21h8M12 17v4M7 4h10v5a5 5 0 0 1-10 0V4Z" /><path d="M17 5h3v2a3 3 0 0 1-3 3M7 5H4v2a3 3 0 0 0 3 3" /></svg>
                    <span>Ranglijst</span>
                </button>
                <button :class="['b-tab', { active: currentScreen === 'planning' }]" @click="go('planning')">
                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="4.5" width="18" height="16" rx="2.5" /><path d="M3 9h18M8 2.5v4M16 2.5v4" /></svg>
                    <span>Planning</span>
                </button>
            </div>
        </nav>

        <div class="b-toast" :class="{ show: toastVisible }" v-html="toastHtml"></div>
    </div>
</template>

<style>
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
    --b-font: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Helvetica Neue', Helvetica, Arial, sans-serif;

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

.bingo-root .b-confetti {
    position: fixed;
    inset: 0;
    z-index: 79;
    pointer-events: none;
}

/* ---- Screen wrapper ---- */
.bingo-root .b-screen {
    display: block;
}

/* ---- Shared kicker/title/subtitle ---- */
.bingo-root .b-kicker {
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 0.01em;
    color: var(--b-accent);
    margin-bottom: 5px;
    display: block;
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

/* ---- Home screen ---- */
.bingo-root .b-home {
    max-width: 540px;
    margin: 0 auto;
    min-height: 100dvh;
    display: flex;
    flex-direction: column;
    padding: calc(env(safe-area-inset-top) + 46px) 24px calc(env(safe-area-inset-bottom) + 96px);
}

.bingo-root .b-hero {
    display: flex;
    flex-direction: column;
}

.bingo-root .b-home .b-kicker {
    font-size: 13px;
}

.bingo-root .b-home .b-title {
    font-size: clamp(42px, 13vw, 58px);
    line-height: 0.96;
    margin-top: 7px;
}

.bingo-root .b-home .b-subtitle {
    font-size: 16px;
    margin-top: 13px;
    max-width: 26ch;
    line-height: 1.4;
}

.bingo-root .b-field {
    margin-top: 34px;
}

.bingo-root .b-field label {
    display: block;
    font-size: 12px;
    font-weight: 600;
    color: var(--b-txt-2);
    margin: 0 0 9px 3px;
}

.bingo-root .b-inwrap {
    display: flex;
    align-items: center;
    gap: 11px;
    background: var(--b-surface);
    border-radius: 16px;
    padding: 0 16px;
    border: 1.5px solid transparent;
    transition: border-color 0.25s;
}

.bingo-root .b-inwrap:focus-within {
    border-color: var(--b-accent);
}

.bingo-root .b-field-ic {
    color: var(--b-txt-3);
    flex: none;
}

.bingo-root .b-field-ic svg {
    width: 20px;
    height: 20px;
    display: block;
}

.bingo-root .b-inwrap input {
    flex: 1;
    min-width: 0;
    background: none;
    border: none;
    outline: none;
    color: var(--b-txt);
    font-family: var(--b-font);
    font-size: 17px;
    font-weight: 600;
    padding: 16px 0;
    letter-spacing: -0.01em;
}

.bingo-root .b-inwrap input::placeholder {
    color: var(--b-txt-3);
    font-weight: 500;
}

.bingo-root .b-navcards {
    display: flex;
    flex-direction: column;
    gap: 12px;
    margin-top: 18px;
}

.bingo-root .b-navcard {
    display: flex;
    align-items: center;
    gap: 16px;
    background: var(--b-surface);
    border: none;
    border-radius: 18px;
    padding: 18px 20px;
    cursor: pointer;
    text-align: left;
    width: 100%;
    font-family: var(--b-font);
    color: var(--b-txt);
    transition: transform 0.15s, background 0.2s;
}

.bingo-root .b-navcard:active {
    transform: scale(0.98);
    background: var(--b-surface-2);
}

.bingo-root .b-nic {
    width: 50px;
    height: 50px;
    border-radius: 14px;
    display: grid;
    place-items: center;
    flex: none;
}

.bingo-root .b-nic svg {
    width: 24px;
    height: 24px;
}

.bingo-root .b-navcard-primary .b-nic {
    background: var(--b-accent);
    color: #fff;
}

.bingo-root .b-navcard-secondary .b-nic {
    background: var(--b-surface-2);
    color: var(--b-accent);
}

.bingo-root .b-ntx {
    flex: 1;
}

.bingo-root .b-ntx b {
    display: block;
    font-size: 18px;
    font-weight: 700;
    letter-spacing: -0.02em;
}

.bingo-root .b-ntx small {
    display: block;
    font-size: 13.5px;
    color: var(--b-txt-2);
    margin-top: 3px;
    font-weight: 400;
}

.bingo-root .b-narr {
    color: var(--b-txt-3);
}

.bingo-root .b-narr svg {
    width: 20px;
    height: 20px;
    display: block;
}

.bingo-root .b-foot {
    margin-top: 26px;
    text-align: center;
    font-size: 12px;
    color: var(--b-txt-3);
    line-height: 1.5;
}

/* ---- Team chip + navrow ---- */
.bingo-root .b-navrow {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    gap: 10px;
    margin-bottom: 14px;
}

.bingo-root .b-teamchip {
    display: inline-flex;
    align-items: center;
    gap: 7px;
    background: var(--b-surface-2);
    border-radius: 999px;
    padding: 7px 14px;
    font-size: 13px;
    font-weight: 600;
    color: var(--b-txt);
    max-width: 55%;
}

.bingo-root .b-teamchip span {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

.bingo-root .b-dotg {
    width: 7px;
    height: 7px;
    border-radius: 999px;
    background: var(--b-accent);
    flex: none;
    display: inline-block;
}

/* ---- Board ---- */
.bingo-root .b-app {
    max-width: 540px;
    margin: 0 auto;
    padding: 0 18px 120px;
}

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

.bingo-root .b-menu-btn:active { background: #3a3a3c; }
.bingo-root .b-menu-btn svg { width: 20px; height: 20px; }

.bingo-root .b-scorecard {
    margin-top: 18px;
    background: var(--b-surface);
    border-radius: 20px;
    padding: 4px;
}

.bingo-root .b-scorerow { display: flex; }

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

.bingo-root .b-scol.b-accent .b-num { color: var(--b-accent); }

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

.bingo-root .b-progress { margin: 2px 14px 0; }

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

.bingo-root .b-cell:active { transform: scale(0.95); }

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

.bingo-root .b-p1 .b-badge { background: var(--b-p1); color: #1c1c1e; }
.bingo-root .b-p2 .b-badge { background: var(--b-p2); }
.bingo-root .b-p3 .b-badge { background: var(--b-p3); color: #1c1c1e; }
.bingo-root .b-p5 .b-badge { background: var(--b-p5); }

.bingo-root .b-cell.b-done { background: var(--b-surface-2); }

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

.bingo-root .b-cell.b-done .b-check { display: grid; }
.bingo-root .b-check svg { width: 14px; height: 14px; color: #fff; }
.bingo-root .b-cell.b-done .b-ct { position: relative; z-index: 3; color: #fff; opacity: 0.96; }
.bingo-root .b-cell.b-done .b-badge { z-index: 3; }
.bingo-root .b-cell.b-win { box-shadow: inset 0 0 0 2px var(--b-accent); }
.bingo-root .b-cell.b-win .b-photo { opacity: 0.5; }

/* ---- Sheets ---- */
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

.bingo-root .b-scrim.open { display: flex; }

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

.bingo-root .b-scrim.open .b-sheet { transform: translateY(0); }

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

.bingo-root .b-tag.b-p1 { background: var(--b-p1); color: #1c1c1e; }
.bingo-root .b-tag.b-p2 { background: var(--b-p2); }
.bingo-root .b-tag.b-p3 { background: var(--b-p3); color: #1c1c1e; }
.bingo-root .b-tag.b-p5 { background: var(--b-p5); }

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

.bingo-root .b-ph { display: flex; flex-direction: column; align-items: center; gap: 10px; padding: 18px; }
.bingo-root .b-ph svg { width: 30px; height: 30px; color: var(--b-txt-3); }
.bingo-root .b-ph span { font-size: 14px; font-weight: 500; color: var(--b-txt-2); }
.bingo-root .b-ph small { font-size: 12px; color: var(--b-txt-3); }
.bingo-root .b-proof.b-has .b-ph { display: none; }

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

.bingo-root .b-proof.b-has .b-retake { display: inline-flex; }
.bingo-root .b-retake svg { width: 13px; height: 13px; }

.bingo-root .b-actions { display: flex; flex-direction: column; gap: 10px; }

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

.bingo-root .b-btn:active { transform: scale(0.98); }
.bingo-root .b-btn svg { width: 18px; height: 18px; }
.bingo-root .b-btn-primary { background: var(--b-accent); color: #fff; }
.bingo-root .b-btn-primary:active { background: var(--b-accent-press); }
.bingo-root .b-btn-ghost { background: var(--b-surface-2); color: var(--b-txt); }
.bingo-root .b-btn-danger { background: transparent; color: var(--b-p5); }
.bingo-root .b-hidden-file { display: none; }

.bingo-root .b-msheet h2 { margin: 2px 0 4px; font-size: 23px; color: var(--b-txt); }

.bingo-root .b-row {
    display: flex;
    align-items: center;
    gap: 14px;
    padding: 16px 2px;
    border-bottom: 1px solid var(--b-hair-soft);
    cursor: pointer;
}

.bingo-root .b-row:last-of-type { border-bottom: none; }

.bingo-root .b-ic {
    width: 38px;
    height: 38px;
    border-radius: 999px;
    background: var(--b-surface-2);
    display: grid;
    place-items: center;
    flex: none;
}

.bingo-root .b-ic svg { width: 19px; height: 19px; color: var(--b-accent); }
.bingo-root .b-row.b-danger .b-ic svg { color: var(--b-p5); }
.bingo-root .b-tx { flex: 1; }
.bingo-root .b-tx b { display: block; font-size: 16px; font-weight: 600; color: var(--b-txt); }
.bingo-root .b-tx small { display: block; font-size: 13px; color: var(--b-txt-3); margin-top: 2px; }
.bingo-root .b-chev { color: var(--b-txt-3); }
.bingo-root .b-chev svg { width: 18px; height: 18px; }

/* ---- Points stepper ---- */
.bingo-root .b-numrow { display: flex; align-items: center; gap: 12px; }

.bingo-root .b-stepper {
    display: flex;
    align-items: center;
    background: var(--b-surface);
    border-radius: 14px;
    overflow: hidden;
}

.bingo-root .b-stepper button {
    width: 54px;
    height: 56px;
    border: none;
    background: none;
    color: var(--b-accent);
    font-size: 26px;
    font-weight: 400;
    cursor: pointer;
    font-family: var(--b-font);
}

.bingo-root .b-stepper button:active { background: var(--b-surface-2); }

.bingo-root .b-stepper input {
    width: 74px;
    text-align: center;
    background: none;
    border: none;
    outline: none;
    color: var(--b-txt);
    font-size: 22px;
    font-weight: 700;
    font-family: var(--b-font);
    font-variant-numeric: tabular-nums;
}

.bingo-root .b-stepper input::-webkit-outer-spin-button,
.bingo-root .b-stepper input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }

/* ---- Leaderboard ---- */
.bingo-root .b-lb {
    max-width: 540px;
    margin: 0 auto;
    padding: calc(env(safe-area-inset-top) + 16px) 18px calc(96px + env(safe-area-inset-bottom));
}

.bingo-root .b-lbtitle {
    margin: 6px 0 2px;
    font-size: clamp(30px, 9vw, 38px);
    font-weight: 700;
    letter-spacing: -0.03em;
    color: var(--b-txt);
}

.bingo-root .b-lbsub {
    margin: 0 0 6px;
    font-size: 14px;
    color: var(--b-txt-2);
}

.bingo-root .b-lblist {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin-top: 18px;
}

.bingo-root .b-lbrow {
    display: flex;
    align-items: center;
    gap: 14px;
    background: var(--b-surface);
    border-radius: 18px;
    padding: 15px 18px;
    border: 1.5px solid transparent;
    cursor: pointer;
    transition: background 0.2s;
}

.bingo-root .b-lbrow-you {
    border-color: var(--b-accent);
    background: rgba(48, 209, 88, 0.09);
    cursor: default;
}

.bingo-root .b-rank {
    font-size: 19px;
    font-weight: 700;
    color: var(--b-txt-3);
    min-width: 26px;
    text-align: center;
    font-variant-numeric: tabular-nums;
}

.bingo-root .b-lbrow-top1 .b-rank { color: #ffd60a; }

.bingo-root .b-who { flex: 1; min-width: 0; }

.bingo-root .b-who b {
    font-size: 16.5px;
    font-weight: 600;
    letter-spacing: -0.02em;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    display: block;
    color: var(--b-txt);
}

.bingo-root .b-who small {
    display: block;
    font-size: 12.5px;
    color: var(--b-txt-2);
    margin-top: 2px;
}

.bingo-root .b-youtag {
    display: inline-block;
    font-size: 10px;
    font-weight: 700;
    background: var(--b-accent);
    color: #fff;
    padding: 2px 8px;
    border-radius: 999px;
    margin-bottom: 4px;
    letter-spacing: 0.03em;
}

.bingo-root .b-ptsbox { text-align: right; flex: none; }

.bingo-root .b-ptsbox b {
    font-size: 23px;
    font-weight: 700;
    color: var(--b-accent);
    font-variant-numeric: tabular-nums;
    letter-spacing: -0.02em;
    display: block;
}

.bingo-root .b-ptsbox small {
    display: block;
    font-size: 11px;
    color: var(--b-txt-3);
    margin-top: 1px;
}

.bingo-root .b-lbempty {
    text-align: center;
    color: var(--b-txt-3);
    font-size: 14px;
    padding: 14px 20px 4px;
}

/* ---- Planning ---- */
.bingo-root .b-plan {
    max-width: 540px;
    margin: 0 auto;
    padding: calc(env(safe-area-inset-top) + 16px) 18px calc(96px + env(safe-area-inset-bottom));
}

.bingo-root .b-daygroup { margin-top: 26px; }

.bingo-root .b-dayhead {
    display: flex;
    align-items: center;
    gap: 13px;
    margin-bottom: 11px;
    padding: 0 2px;
}

.bingo-root .b-dayhead h2 {
    font-size: 23px;
    font-weight: 700;
    letter-spacing: -0.03em;
    margin: 0;
    color: var(--b-txt);
}

.bingo-root .b-ln {
    flex: 1;
    height: 1px;
    background: var(--b-hair);
}

.bingo-root .b-cnt {
    font-size: 12px;
    font-weight: 500;
    color: var(--b-txt-3);
}

.bingo-root .b-daycard {
    background: var(--b-surface);
    border-radius: 18px;
    padding: 2px 18px;
}

.bingo-root .b-pitem {
    display: flex;
    gap: 14px;
    padding: 15px 0;
    border-bottom: 1px solid var(--b-hair-soft);
    align-items: flex-start;
}

.bingo-root .b-pitem:last-child { border-bottom: none; }

.bingo-root .b-ptime {
    flex: none;
    width: 84px;
    font-size: 14.5px;
    font-weight: 600;
    color: var(--b-accent);
    font-variant-numeric: tabular-nums;
    letter-spacing: -0.02em;
    padding-top: 1px;
}

.bingo-root .b-pbody { flex: 1; min-width: 0; }

.bingo-root .b-pbody b {
    font-size: 16px;
    font-weight: 600;
    letter-spacing: -0.01em;
    color: var(--b-txt);
    display: block;
}

.bingo-root .b-pbody small {
    display: block;
    font-size: 13.5px;
    color: var(--b-txt-2);
    margin-top: 3px;
    line-height: 1.35;
}

.bingo-root .b-pitem-hl {
    margin: 6px -12px;
    padding: 14px 12px;
    background: rgba(48, 209, 88, 0.1);
    border: 1px solid rgba(48, 209, 88, 0.32);
    border-radius: 14px;
    cursor: pointer;
}

.bingo-root .b-pitem-hl .b-pbody b { color: var(--b-accent); }

.bingo-root .b-ptag {
    display: inline-flex;
    align-items: center;
    gap: 4px;
    margin-top: 7px;
    font-size: 11px;
    font-weight: 700;
    color: var(--b-accent);
    letter-spacing: 0.02em;
}

.bingo-root .b-ptag svg { width: 13px; height: 13px; }

/* ---- Bottom tab bar ---- */
.bingo-root .b-tabbar {
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 55;
    display: flex;
    justify-content: center;
    background: rgba(0, 0, 0, 0.72);
    backdrop-filter: saturate(170%) blur(22px);
    -webkit-backdrop-filter: saturate(170%) blur(22px);
    border-top: 1px solid var(--b-hair-soft);
    padding-bottom: env(safe-area-inset-bottom);
}

.bingo-root .b-tabbar-inner {
    display: flex;
    width: 100%;
    max-width: 540px;
}

.bingo-root .b-tab {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 4px;
    padding: 9px 4px 8px;
    min-height: 56px;
    background: none;
    border: none;
    cursor: pointer;
    color: var(--b-txt-3);
    font-family: var(--b-font);
    transition: color 0.2s;
}

.bingo-root .b-tab svg { width: 25px; height: 25px; }
.bingo-root .b-tab span { font-size: 11px; font-weight: 600; letter-spacing: -0.01em; }
.bingo-root .b-tab.active { color: var(--b-accent); }
.bingo-root .b-tab:active { opacity: 0.55; }

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

.bingo-root .b-bingo.open { display: flex; }

.bingo-root .b-bingo-word {
    font-size: clamp(64px, 21vw, 108px);
    font-weight: 700;
    line-height: 0.9;
    color: var(--b-accent);
    letter-spacing: -0.045em;
    animation: bingo-pop 0.55s cubic-bezier(0.2, 0.9, 0.3, 1) both;
}

.bingo-root .b-bingo-kind { margin-top: 18px; font-size: 17px; font-weight: 600; color: var(--b-txt); letter-spacing: -0.01em; }
.bingo-root .b-bingo-pts { margin-top: 7px; font-size: 15px; color: var(--b-txt-2); }

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
    0% { transform: scale(0.6); opacity: 0; }
    60% { transform: scale(1.05); }
    100% { transform: scale(1); opacity: 1; }
}

/* ---- Toast ---- */
.bingo-root .b-toast {
    position: fixed;
    left: 50%;
    bottom: calc(env(safe-area-inset-bottom) + 80px);
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

.bingo-root .b-toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }
.bingo-root .b-toast b { color: var(--b-accent); font-weight: 700; }
</style>
