<script setup>
import { ref, onMounted, computed } from 'vue'

const token = '5dc0faddd67d4bd655e074569c6111e8'

const paises = ref([])
const paisSelecionado = ref('')
const jogadores = ref([])
const carregando = ref(false)
const paginaAtual = ref(1)
const porPagina = 12

const bandeiraSelecionada = computed(() => {
  const pais = paises.value.find(p => p.name === paisSelecionado.value)
  return pais?.flag ?? null
})

const totalPaginas = computed(() => Math.ceil(jogadores.value.length / porPagina))

const jogadoresPagina = computed(() => {
  const inicio = (paginaAtual.value - 1) * porPagina
  return jogadores.value.slice(inicio, inicio + porPagina)
})

const carregarPaises = async () => {
  try {
    const response = await fetch('https://v3.football.api-sports.io/teams/countries', {
      method: 'GET',
      headers: { 'x-apisports-key': token }
    })
    const dados = await response.json()
    paises.value = dados.response
  } catch (erro) {
    console.error('Erro ao carregar países:', erro)
  }
}

const carregarFigurinhas = async () => {
  if (!paisSelecionado.value) return
  carregando.value = true
  jogadores.value = []
  paginaAtual.value = 1
  try {
    const responseTime = await fetch(
      `https://v3.football.api-sports.io/teams?name=${paisSelecionado.value}`,
      { method: 'GET', headers: { 'x-apisports-key': token } }
    )
    const dadosTime = await responseTime.json()
    const teamId = dadosTime.response[0].team.id

    const responseElenco = await fetch(
      `https://v3.football.api-sports.io/players/squads?team=${teamId}`,
      { method: 'GET', headers: { 'x-apisports-key': token } }
    )
    const dadosElenco = await responseElenco.json()
    jogadores.value = dadosElenco.response[0].players
  } catch (erro) {
    console.error('Erro ao carregar jogadores:', erro)
  } finally {
    carregando.value = false
  }
}

const proximaPagina = () => {
  if (paginaAtual.value < totalPaginas.value) paginaAtual.value++
}

const paginaAnterior = () => {
  if (paginaAtual.value > 1) paginaAtual.value--
}

onMounted(() => {
  carregarPaises()
})
</script>

<template>
  <div class="app">

    <!-- Sidebar -->
    <aside class="sidebar">
      <div class="sidebar-logo">
        <div class="logo-icon">⚽</div>
        <div class="logo-text">
          <span class="logo-main">FutÁlbum</span>
          <span class="logo-sub">Copa 2026</span>
        </div>
      </div>

      <nav class="sidebar-nav">
        <a class="nav-item active">📖 Álbuns</a>
        <a class="nav-item">🛒 Mercado</a>
        <a class="nav-item">👤 Perfil</a>
      </nav>

      <div class="sidebar-select-label">SELECIONE UM TIME:</div>

      <div class="select-wrapper">
        <select v-model="paisSelecionado" @change="carregarFigurinhas" class="select-pais">
          <option value="">Escolha...</option>
          <option v-for="pais in paises" :key="pais.name" :value="pais.name">
            {{ pais.name }}
          </option>
        </select>
        <span class="select-chevron">▾</span>
      </div>

      <div v-if="paisSelecionado && !carregando" class="sidebar-team-info">
        <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="sidebar-flag" alt="Bandeira" />
        <span class="sidebar-team-name">{{ paisSelecionado }}</span>
        <span class="sidebar-count">{{ jogadores.length }} jogadores</span>
      </div>

      <!-- Copa 2026 badge -->
      <div class="copa-badge">
        <div class="copa-badge-stripe red"></div>
        <div class="copa-badge-stripe blue"></div>
        <div class="copa-badge-text">
          <span>🏆</span>
          <span>FIFA WORLD CUP</span>
          <span class="copa-year">2026</span>
        </div>
        <div class="copa-badge-stripe blue"></div>
        <div class="copa-badge-stripe red"></div>
      </div>
    </aside>

    <!-- Main content -->
    <main class="main">

      <div class="page-header">
        <h1 class="page-title">ALBUM DE FIGURINHAS: SELEÇÕES DO MUNDO</h1>
        <div class="header-stripes">
          <span class="stripe red"></span>
          <span class="stripe blue"></span>
          <span class="stripe white"></span>
        </div>
      </div>

      <!-- Loading -->
      <div v-if="carregando" class="loading-area">
        <div class="spinner"></div>
        <p>Carregando figurinhas...</p>
      </div>

      <!-- Empty state -->
      <div v-else-if="!paisSelecionado" class="empty-state">
        <div class="empty-album">
          <div class="empty-icon">📋</div>
          <p>Selecione uma seleção na barra lateral para ver as figurinhas</p>
        </div>
      </div>

      <!-- Album -->
      <div v-else class="album">

        <!-- Album header -->
        <div class="album-header">
          <div class="album-header-left">
            <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="album-flag" alt="Bandeira" />
            <div class="album-title-block">
              <div class="album-sub">ÁLBUM DE FIGURINHAS</div>
              <div class="album-title">{{ paisSelecionado.toUpperCase() }}</div>
            </div>
          </div>
          <div class="album-header-right">
            <div class="counter-badge">
              Figurinhas Coletadas: {{ jogadores.length }} / {{ jogadores.length }}
            </div>
          </div>
        </div>

        <!-- Team banner -->
        <div class="team-banner">
          <div class="banner-left">
            <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="team-banner-flag" alt="Bandeira" />
            <div class="team-badge">
              <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="team-badge-img" alt="Escudo" />
            </div>
          </div>
          <div class="team-banner-name">SELEÇÃO {{ paisSelecionado.toUpperCase() }}</div>
          <div class="banner-deco">
            <span>2026</span>
          </div>
        </div>

        <!-- Stickers grid -->
        <div class="stickers-grid">
          <div
            class="sticker"
            v-for="(jogador, index) in jogadoresPagina"
            :key="jogador.id"
            :style="{ '--i': index }"
          >
            <div class="sticker-inner">
              <div class="sticker-top">
                <span class="sticker-num">{{ (paginaAtual - 1) * porPagina + index + 1 }}</span>
                <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="sticker-flag" alt="País" />
              </div>
              <div class="sticker-photo-wrap">
                <img
                  :src="jogador.photo"
                  :alt="jogador.name"
                  class="sticker-photo"
                  @error="e => e.target.style.opacity = '0'"
                />
              </div>
              <div class="sticker-info">
                <div class="sticker-name">{{ jogador.name }}</div>
                <div class="sticker-position">{{ jogador.position }}</div>
              </div>
            </div>
          </div>
        </div>

        <!-- Pagination -->
        <div class="pagination">
          <button class="btn-page" @click="paginaAnterior" :disabled="paginaAtual === 1">
            ← ANTERIOR
          </button>
          <span class="page-indicator">PÁGINA {{ paginaAtual }} DE {{ totalPaginas }}</span>
          <button class="btn-page btn-next" @click="proximaPagina" :disabled="paginaAtual === totalPaginas">
            PRÓXIMA →
          </button>
        </div>

      </div>
    </main>

  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Oswald:wght@400;500;600;700&family=Barlow+Condensed:wght@400;500;600;700&family=Barlow:wght@400;500&display=swap');

*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

:root {
  /* Cores oficiais Copa 2026 */
  --red:          #C8102E;
  --red-dark:     #a00d24;
  --blue:         #003DA5;
  --blue-dark:    #002d7a;
  --blue-light:   #1a56c4;
  --green:        #00a650;
  --green-dark:   #007a3d;
  --white:        #ffffff;

  --bg:           #edeae3;
  --bg-album:     #f2efe8;
  --bg-sidebar:   #111827;
  --text-dark:    #0f1923;
  --text-mid:     #555;
  --shadow:       0 4px 24px rgba(0,0,0,0.13);
  --radius:       10px;
}

html, body {
  min-height: 100vh;
  background: var(--bg);
  font-family: 'Barlow', sans-serif;
  color: var(--text-dark);
}

/* ─── Layout ─── */
.app { display: flex; min-height: 100vh; }

/* ─── Sidebar ─── */
.sidebar {
  width: 240px;
  min-width: 240px;
  background: var(--bg-sidebar);
  padding: 24px 16px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.sidebar-logo {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-bottom: 20px;
  border-bottom: 1px solid rgba(255,255,255,0.08);
}

.logo-icon { font-size: 28px; }

.logo-text { display: flex; flex-direction: column; }

.logo-main {
  font-family: 'Oswald', sans-serif;
  font-size: 18px;
  font-weight: 700;
  color: var(--white);
  letter-spacing: 1px;
}

.logo-sub {
  font-size: 11px;
  color: var(--red);
  letter-spacing: 2px;
  text-transform: uppercase;
  font-weight: 600;
}

.sidebar-nav { display: flex; flex-direction: column; gap: 4px; }

.nav-item {
  padding: 10px 14px;
  border-radius: 8px;
  color: rgba(255,255,255,0.55);
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s, color 0.2s;
  text-decoration: none;
}

.nav-item:hover { background: rgba(255,255,255,0.07); color: white; }

.nav-item.active {
  background: linear-gradient(135deg, var(--red), var(--red-dark));
  color: white;
  font-weight: 600;
}

.sidebar-select-label {
  font-size: 10px;
  letter-spacing: 2px;
  color: rgba(255,255,255,0.3);
  font-weight: 600;
  text-transform: uppercase;
}

.select-wrapper { position: relative; }

.select-pais {
  width: 100%;
  padding: 10px 32px 10px 12px;
  border-radius: 8px;
  border: 1px solid rgba(255,255,255,0.12);
  background: rgba(255,255,255,0.07);
  color: white;
  font-family: 'Barlow', sans-serif;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  appearance: none;
  -webkit-appearance: none;
  outline: none;
  transition: border-color 0.2s;
}

.select-pais:focus { border-color: var(--blue-light); }
.select-pais option { background: #1a2540; color: white; }

.select-chevron {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: rgba(255,255,255,0.4);
  pointer-events: none;
}

.sidebar-team-info {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 6px;
  padding: 12px;
  background: rgba(255,255,255,0.05);
  border-radius: 8px;
  border: 1px solid rgba(255,255,255,0.07);
}

.sidebar-flag {
  width: 36px;
  height: 24px;
  object-fit: cover;
  border-radius: 3px;
}

.sidebar-team-name {
  font-family: 'Oswald', sans-serif;
  font-size: 16px;
  color: white;
  font-weight: 600;
  letter-spacing: 1px;
}

.sidebar-count {
  font-size: 12px;
  color: var(--blue-light);
  font-weight: 600;
}

/* Copa badge na sidebar */
.copa-badge {
  margin-top: auto;
  display: flex;
  flex-direction: column;
  align-items: stretch;
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid rgba(255,255,255,0.1);
}

.copa-badge-stripe {
  height: 4px;
}
.copa-badge-stripe.red  { background: var(--red); }
.copa-badge-stripe.blue { background: var(--green); }

.copa-badge-text {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  padding: 12px 8px;
  background: rgba(255,255,255,0.04);
  font-family: 'Oswald', sans-serif;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 2px;
  color: rgba(255,255,255,0.5);
  text-transform: uppercase;
}

.copa-year {
  font-size: 22px;
  color: white;
  letter-spacing: 3px;
}

/* ─── Main ─── */
.main {
  flex: 1;
  padding: 28px 32px;
  overflow-y: auto;
  background: var(--bg);
}

.page-header { margin-bottom: 24px; }

.page-title {
  font-family: 'Oswald', sans-serif;
  font-size: 22px;
  font-weight: 700;
  color: var(--text-dark);
  letter-spacing: 1px;
  text-transform: uppercase;
  margin-bottom: 8px;
}

.header-stripes { display: flex; gap: 4px; }

.stripe {
  height: 3px;
  border-radius: 2px;
}
.stripe.red   { width: 48px; background: var(--red); }
.stripe.blue  { width: 32px; background: var(--blue); }
.stripe.white { width: 16px; background: #00a650; }

/* ─── Loading ─── */
.loading-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 16px;
  min-height: 300px;
  color: var(--text-mid);
  font-size: 16px;
  font-weight: 500;
}

.spinner {
  width: 44px;
  height: 44px;
  border: 3px solid rgba(0,61,165,0.2);
  border-top-color: var(--blue);
  border-right-color: var(--red);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }

/* ─── Empty ─── */
.empty-state {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 400px;
}

.empty-album {
  text-align: center;
  background: var(--bg-album);
  border-radius: var(--radius);
  padding: 60px 40px;
  border: 2px dashed #ccc;
  max-width: 400px;
}

.empty-icon { font-size: 56px; margin-bottom: 16px; }
.empty-album p { color: var(--text-mid); font-size: 16px; line-height: 1.5; }

/* ─── Album ─── */
.album {
  background: var(--bg-album);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  overflow: hidden;
}

/* Album header */
.album-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 24px;
  background: white;
  border-bottom: 3px solid;
  border-image: linear-gradient(90deg, var(--red), var(--blue), #00a650) 1;
}

.album-header-left {
  display: flex;
  align-items: center;
  gap: 14px;
}

.album-flag {
  width: 52px;
  height: 35px;
  object-fit: cover;
  border-radius: 4px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.2);
}

.album-sub {
  font-size: 10px;
  color: #999;
  text-transform: uppercase;
  letter-spacing: 2px;
  font-weight: 600;
}

.album-title {
  font-family: 'Oswald', sans-serif;
  font-size: 22px;
  font-weight: 700;
  color: var(--text-dark);
  letter-spacing: 1px;
}

.counter-badge {
  background: linear-gradient(135deg, var(--blue), var(--blue-dark));
  color: white;
  font-family: 'Barlow Condensed', sans-serif;
  font-size: 14px;
  font-weight: 600;
  letter-spacing: 0.5px;
  padding: 8px 18px;
  border-radius: 6px;
}

/* Team banner */
.team-banner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 28px;
  background: linear-gradient(135deg, var(--blue-dark) 0%, var(--blue) 40%, var(--green) 70%, var(--red) 100%);
  position: relative;
  overflow: hidden;
}

.team-banner::before {
  content: '';
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    -45deg,
    transparent,
    transparent 12px,
    rgba(255,255,255,0.03) 12px,
    rgba(255,255,255,0.03) 24px
  );
}

.banner-left {
  display: flex;
  align-items: center;
  gap: 16px;
  position: relative;
  z-index: 1;
}

.team-banner-flag {
  width: 80px;
  height: 54px;
  object-fit: cover;
  border-radius: 5px;
  box-shadow: 0 3px 12px rgba(0,0,0,0.4);
  border: 2px solid rgba(255,255,255,0.3);
}

.team-badge {
  width: 60px;
  height: 60px;
  background: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  box-shadow: 0 3px 12px rgba(0,0,0,0.3);
}

.team-badge-img {
  width: 44px;
  height: 30px;
  object-fit: cover;
  border-radius: 3px;
}

.team-banner-name {
  font-family: 'Oswald', sans-serif;
  font-size: 28px;
  font-weight: 700;
  color: white;
  letter-spacing: 3px;
  text-shadow: 0 2px 8px rgba(0,0,0,0.4);
  position: relative;
  z-index: 1;
}

.banner-deco {
  font-family: 'Oswald', sans-serif;
  font-size: 48px;
  font-weight: 700;
  color: rgba(255,255,255,0.12);
  letter-spacing: 4px;
  position: relative;
  z-index: 1;
  user-select: none;
}

/* ─── Stickers grid ─── */
.stickers-grid {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 14px;
  padding: 24px;
  background: #e6e2d9;
}

.sticker {
  animation: sticker-in 0.35s ease both;
  animation-delay: calc(var(--i) * 0.04s);
}

@keyframes sticker-in {
  from { opacity: 0; transform: scale(0.88) translateY(10px); }
  to   { opacity: 1; transform: scale(1)    translateY(0);    }
}

.sticker-inner {
  background: white;
  border-radius: 6px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0,0,0,0.12);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  cursor: pointer;
  border: 1px solid rgba(0,0,0,0.06);
}

.sticker-inner:hover {
  transform: translateY(-4px) scale(1.04);
  box-shadow: 0 10px 28px rgba(0,0,0,0.18);
}

/* Topo da figurinha — degradê vermelho/azul Copa 2026 */
.sticker-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 5px 7px 4px;
  background: linear-gradient(90deg, var(--blue), #00a650, var(--red));
}

.sticker-num {
  font-family: 'Oswald', sans-serif;
  font-size: 13px;
  font-weight: 700;
  color: white;
}

.sticker-flag {
  width: 22px;
  height: 15px;
  object-fit: cover;
  border-radius: 2px;
}

/* Foto */
.sticker-photo-wrap {
  width: 100%;
  aspect-ratio: 1;
  overflow: hidden;
  background: linear-gradient(180deg, #dce8f5 0%, #b8d0f0 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.sticker-photo {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

/* Info */
.sticker-info {
  padding: 6px 6px 8px;
  text-align: center;
  background: white;
  border-top: 2px solid;
  border-image: linear-gradient(90deg, var(--blue), var(--red)) 1;
}

.sticker-name {
  font-family: 'Barlow Condensed', sans-serif;
  font-size: 11px;
  font-weight: 700;
  color: var(--text-dark);
  text-transform: uppercase;
  letter-spacing: 0.3px;
  line-height: 1.2;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Badge de posição — alterna vermelho/azul */
.sticker-position {
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 1px;
  text-transform: uppercase;
  color: white;
  background: var(--blue);
  padding: 2px 7px;
  border-radius: 3px;
  display: inline-block;
  margin-top: 4px;
}

/* ─── Pagination ─── */
.pagination {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 24px;
  background: white;
  border-top: 3px solid;
  border-image: linear-gradient(90deg, var(--red), var(--blue), #00a650) 1;
}

.btn-page {
  font-family: 'Barlow Condensed', sans-serif;
  font-size: 13px;
  font-weight: 700;
  letter-spacing: 1px;
  padding: 8px 20px;
  border-radius: 6px;
  border: 2px solid var(--blue);
  background: transparent;
  color: var(--blue);
  cursor: pointer;
  transition: background 0.2s, color 0.2s;
}

.btn-page:hover:not(:disabled) {
  background: var(--blue);
  color: white;
}

.btn-page:disabled { opacity: 0.3; cursor: not-allowed; }

.btn-next {
  background: linear-gradient(135deg, var(--red), var(--red-dark));
  color: white;
  border-color: var(--red);
}

.btn-next:hover:not(:disabled) {
  background: var(--red-dark);
  border-color: var(--red-dark);
}

.page-indicator {
  font-family: 'Oswald', sans-serif;
  font-size: 14px;
  font-weight: 600;
  color: var(--text-mid);
  letter-spacing: 1px;
}

/* ─── Responsividade ─── */
@media (max-width: 1100px) {
  .stickers-grid { grid-template-columns: repeat(4, 1fr); }
}

@media (max-width: 768px) {
  .app { flex-direction: column; }
  .sidebar { width: 100%; min-width: unset; flex-direction: row; flex-wrap: wrap; gap: 12px; padding: 16px; }
  .sidebar-nav { flex-direction: row; }
  .copa-badge { display: none; }
  .main { padding: 16px; }
  .stickers-grid { grid-template-columns: repeat(3, 1fr); gap: 10px; padding: 14px; }
  .team-banner-name { font-size: 20px; }
  .banner-deco { display: none; }
}

@media (max-width: 480px) {
  .stickers-grid { grid-template-columns: repeat(2, 1fr); }
}
</style>