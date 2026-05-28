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
    </aside>

    <!-- Main content -->
    <main class="main">

      <!-- Page header -->
      <div class="page-header">
        <h1 class="page-title">ALBUM DE FIGURINHAS: SELEÇÕES DO MUNDO</h1>
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
              <div class="album-sub">Álbum de Figurinhas</div>
              <div class="album-title">{{ paisSelecionado }}</div>
            </div>
          </div>
          <div class="album-header-right">
            <div class="counter-badge">
              Figurinhas Coletadas: {{ jogadores.length }} / {{ jogadores.length }}
            </div>
          </div>
        </div>

        <!-- Team banner inside album -->
        <div class="team-banner">
          <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="team-banner-flag" alt="Bandeira" />
          <div class="team-badge">
            <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="team-badge-img" alt="Escudo" />
          </div>
          <div class="team-banner-name">SELEÇÃO {{ paisSelecionado.toUpperCase() }}</div>
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
                <img :src="jogador.photo" :alt="jogador.name" class="sticker-photo" @error="e => e.target.style.opacity = '0'" />
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
  --green:        #1a7a2e;
  --green-dark:   #145c22;
  --green-light:  #22a03c;
  --gold:         #f5c518;
  --gold-dark:    #c9a00a;
  --bg:           #e8e4dc;
  --bg-album:     #f0ece3;
  --bg-sidebar:   #1c2333;
  --text-dark:    #1a1a2e;
  --text-mid:     #444;
  --white:        #ffffff;
  --shadow:       0 4px 20px rgba(0,0,0,0.15);
  --radius:       10px;
}

html, body {
  min-height: 100vh;
  background: var(--bg);
  font-family: 'Barlow', sans-serif;
  color: var(--text-dark);
}

/* ─── Layout ─── */
.app {
  display: flex;
  min-height: 100vh;
}

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
  border-bottom: 1px solid rgba(255,255,255,0.1);
}

.logo-icon { font-size: 28px; }

.logo-text {
  display: flex;
  flex-direction: column;
}

.logo-main {
  font-family: 'Oswald', sans-serif;
  font-size: 18px;
  font-weight: 700;
  color: var(--white);
  letter-spacing: 1px;
}

.logo-sub {
  font-size: 11px;
  color: var(--gold);
  letter-spacing: 2px;
  text-transform: uppercase;
}

.sidebar-nav {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.nav-item {
  padding: 10px 14px;
  border-radius: 8px;
  color: rgba(255,255,255,0.6);
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s, color 0.2s;
  text-decoration: none;
}

.nav-item:hover { background: rgba(255,255,255,0.08); color: white; }
.nav-item.active { background: var(--green); color: white; }

.sidebar-select-label {
  font-size: 10px;
  letter-spacing: 2px;
  color: rgba(255,255,255,0.35);
  font-weight: 600;
  text-transform: uppercase;
}

.select-wrapper {
  position: relative;
}

.select-pais {
  width: 100%;
  padding: 10px 32px 10px 12px;
  border-radius: 8px;
  border: 1px solid rgba(255,255,255,0.15);
  background: rgba(255,255,255,0.08);
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

.select-pais:focus { border-color: var(--green-light); }
.select-pais option { background: #1c2333; color: white; }

.select-chevron {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: rgba(255,255,255,0.5);
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
  border: 1px solid rgba(255,255,255,0.08);
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
  color: var(--gold);
  font-weight: 500;
}

/* ─── Main ─── */
.main {
  flex: 1;
  padding: 28px 32px;
  overflow-y: auto;
  background: var(--bg);
}

.page-header {
  margin-bottom: 24px;
}

.page-title {
  font-family: 'Oswald', sans-serif;
  font-size: 22px;
  font-weight: 700;
  color: var(--text-dark);
  letter-spacing: 1px;
  text-transform: uppercase;
}

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
  border: 3px solid rgba(26,122,46,0.2);
  border-top-color: var(--green);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }

/* ─── Empty state ─── */
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

.empty-album p {
  color: var(--text-mid);
  font-size: 16px;
  line-height: 1.5;
}

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
  border-bottom: 2px solid #e0dbd0;
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
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
}

.album-sub {
  font-size: 11px;
  color: #888;
  text-transform: uppercase;
  letter-spacing: 1.5px;
  font-weight: 600;
}

.album-title {
  font-family: 'Oswald', sans-serif;
  font-size: 22px;
  font-weight: 700;
  color: var(--text-dark);
  letter-spacing: 1px;
  text-transform: uppercase;
}

.counter-badge {
  background: var(--green);
  color: white;
  font-family: 'Barlow Condensed', sans-serif;
  font-size: 14px;
  font-weight: 600;
  letter-spacing: 0.5px;
  padding: 8px 16px;
  border-radius: 6px;
}

/* Team banner */
.team-banner {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 20px 28px;
  background: linear-gradient(135deg, var(--green-dark) 0%, var(--green) 60%, var(--green-light) 100%);
  position: relative;
  overflow: hidden;
}

.team-banner::before {
  content: '';
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    45deg,
    transparent,
    transparent 10px,
    rgba(255,255,255,0.03) 10px,
    rgba(255,255,255,0.03) 20px
  );
}

.team-banner-flag {
  width: 80px;
  height: 54px;
  object-fit: cover;
  border-radius: 5px;
  box-shadow: 0 3px 12px rgba(0,0,0,0.4);
  border: 2px solid rgba(255,255,255,0.3);
  position: relative;
  z-index: 1;
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
  position: relative;
  z-index: 1;
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

/* ─── Stickers grid ─── */
.stickers-grid {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 14px;
  padding: 24px;
  background: #ece8df;
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
  box-shadow: 0 2px 8px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.08);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  cursor: pointer;
  border: 1px solid rgba(0,0,0,0.06);
}

.sticker-inner:hover {
  transform: translateY(-4px) scale(1.04);
  box-shadow: 0 8px 24px rgba(0,0,0,0.18);
}

.sticker-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 5px 7px 3px;
  background: var(--green);
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

.sticker-photo-wrap {
  width: 100%;
  aspect-ratio: 1;
  overflow: hidden;
  background: linear-gradient(180deg, #e8f5e9 0%, #c8e6c9 100%);
  display: flex;
  align-items: center;
  justify-content: center;
}

.sticker-photo {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.sticker-photo-wrap::after {
  content: '👤';
  font-size: 40px;
  opacity: 0.3;
  position: absolute;
}

.sticker-info {
  padding: 6px 6px 8px;
  text-align: center;
  background: white;
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

.sticker-position {
  font-size: 9px;
  font-weight: 600;
  letter-spacing: 1px;
  text-transform: uppercase;
  color: white;
  background: var(--green-dark);
  padding: 2px 6px;
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
  border-top: 2px solid #e0dbd0;
}

.btn-page {
  font-family: 'Barlow Condensed', sans-serif;
  font-size: 13px;
  font-weight: 700;
  letter-spacing: 1px;
  padding: 8px 20px;
  border-radius: 6px;
  border: 2px solid var(--green);
  background: transparent;
  color: var(--green);
  cursor: pointer;
  transition: background 0.2s, color 0.2s;
}

.btn-page:hover:not(:disabled) {
  background: var(--green);
  color: white;
}

.btn-page:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}

.btn-next {
  background: var(--green);
  color: white;
}

.btn-next:hover:not(:disabled) {
  background: var(--green-dark);
  border-color: var(--green-dark);
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
  .main { padding: 16px; }
  .stickers-grid { grid-template-columns: repeat(3, 1fr); gap: 10px; padding: 14px; }
  .team-banner-name { font-size: 20px; }
}

@media (max-width: 480px) {
  .stickers-grid { grid-template-columns: repeat(2, 1fr); }
}
</style>