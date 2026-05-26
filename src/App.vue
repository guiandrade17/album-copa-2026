<script setup>
import { ref, onMounted, computed } from 'vue'

const token = '5dc0faddd67d4bd655e074569c6111e8'

const paises = ref([])
const paisSelecionado = ref('')
const jogadores = ref([])
const carregando = ref(false)

const bandeiraSelecionada = computed(() => {
  const pais = paises.value.find(p => p.name === paisSelecionado.value)
  return pais?.flag ?? null
})

const carregarPaises = async () => {
  try {
    const response = await fetch(
      'https://v3.football.api-sports.io/teams/countries',
      {
        method: 'GET',
        headers: {
          'x-apisports-key': token
        }
      }
    )
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

  try {
    const responseTime = await fetch(
      `https://v3.football.api-sports.io/teams?name=${paisSelecionado.value}`,
      {
        method: 'GET',
        headers: {
          'x-apisports-key': token
        }
      }
    )
    const dadosTime = await responseTime.json()
    const teamId = dadosTime.response[0].team.id

    const responseElenco = await fetch(
      `https://v3.football.api-sports.io/players/squads?team=${teamId}`,
      {
        method: 'GET',
        headers: {
          'x-apisports-key': token
        }
      }
    )
    const dadosElenco = await responseElenco.json()
    jogadores.value = dadosElenco.response[0].players
  } catch (erro) {
    console.error('Erro ao carregar jogadores:', erro)
  } finally {
    carregando.value = false
  }
}

// Gera stats fictícias mas determinísticas por jogador
const fakeStatSeeded = (id, offset) => {
  const seed = (id * 9301 + offset * 49297) % 233280
  return 55 + Math.floor((seed / 233280) * 44)
}

onMounted(() => {
  carregarPaises()
})
</script>

<template>
  <!-- Navbar -->
  <nav class="navbar">
    <div class="navbar-brand">
      <div class="brand-icon">⚽</div>
      <span class="brand-name">FutÁlbum</span>
    </div>
    <div class="navbar-links">
      <a href="#" class="nav-link active">Álbuns</a>
      <a href="#" class="nav-link">Mercado</a>
      <a href="#" class="nav-link">Perfil</a>
    </div>
  </nav>

  <!-- Corpo principal -->
  <div class="album-wrapper">

    <!-- Sidebar de seleção -->
    <aside class="sidebar">
      <p class="sidebar-label">SELECIONE UM TIME:</p>

      <div class="select-box">
        <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="select-flag" />
        <span v-else class="select-globe">🌍</span>
        <select v-model="paisSelecionado" @change="carregarFigurinhas" class="select-pais">
          <option value="">Selecione...</option>
          <option v-for="pais in paises" :key="pais.name" :value="pais.name">
            {{ pais.name }}
          </option>
        </select>
        <span class="select-chevron">▾</span>
      </div>

      <!-- Info do time selecionado na sidebar -->
      <div v-if="paisSelecionado && !carregando" class="team-card">
        <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="team-flag" />
        <div class="team-info">
          <span class="team-label">SELEÇÃO</span>
          <span class="team-name">{{ paisSelecionado }}</span>
          <span class="team-count" v-if="jogadores.length">{{ jogadores.length }} jogadores</span>
        </div>
      </div>

      <!-- Legenda de stats -->
      <div class="stats-legend" v-if="jogadores.length">
        <p class="legend-title">ATRIBUTOS</p>
        <div class="legend-items">
          <span class="leg outfield">RIT</span>
          <span class="leg outfield">FIN</span>
          <span class="leg outfield">PAS</span>
          <span class="leg outfield">CON</span>
          <span class="leg outfield">DEF</span>
          <span class="leg outfield">FÍS</span>
        </div>
        <div class="legend-items" style="margin-top:6px">
          <span class="leg gk">ELA</span>
          <span class="leg gk">MAN</span>
          <span class="leg gk">CHU</span>
          <span class="leg gk">REF</span>
          <span class="leg gk">VEL</span>
          <span class="leg gk">POS</span>
        </div>
        <p class="legend-note">Amarelo = goleiro</p>
      </div>
    </aside>

    <!-- Área do álbum -->
    <main class="album-main">

      <!-- Cabeçalho do álbum -->
      <div class="album-header">
        <h1 class="album-title">ÁLBUM DE FIGURINHAS: SELEÇÕES DO MUNDO</h1>
        <div v-if="paisSelecionado && !carregando" class="album-subtitle">
          Álbum de Figurinhas – {{ paisSelecionado }}
        </div>
      </div>

      <!-- Banner da seleção no álbum -->
      <div v-if="paisSelecionado && !carregando" class="album-team-banner">
        <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="banner-flag" />
        <div class="banner-badge">
          <span class="banner-country">{{ paisSelecionado }}</span>
          <span class="banner-label">SELEÇÃO NACIONAL</span>
        </div>
        <div v-if="jogadores.length" class="banner-collected">
          Figurinhas Coletadas: {{ jogadores.length }} / {{ jogadores.length }}
        </div>
      </div>

      <!-- Loading -->
      <div v-if="carregando" class="loading-area">
        <div class="spinner"></div>
        <p>Carregando elenco...</p>
      </div>

      <!-- Estado vazio -->
      <div v-if="!paisSelecionado && !carregando" class="empty-state">
        <div class="empty-icon">📋</div>
        <p>Escolha uma seleção para ver os jogadores</p>
      </div>

      <!-- Grade de figurinhas -->
      <div class="sticker-grid">
        <div
          class="sticker"
          v-for="(jogador, index) in jogadores"
          :key="jogador.id"
          :style="{ '--delay': index * 0.04 + 's' }"
          :class="{ 'is-gk': jogador.position === 'Goalkeeper' }"
        >
          <!-- Topo colorido da figurinha -->
          <div class="sticker-top">
            <span class="sticker-num">{{ index + 1 }}</span>
            <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="sticker-flag" />
          </div>

          <!-- Foto -->
          <div class="sticker-photo-wrap">
            <img :src="jogador.photo" :alt="jogador.name" class="sticker-photo" />
          </div>

          <!-- Nome e posição -->
          <div class="sticker-body">
            <div class="sticker-name">{{ jogador.name }}</div>
            <div class="sticker-pos">{{ jogador.position }}</div>
          </div>

          <!-- Stats de jogador de linha -->
          <div class="sticker-stats" v-if="jogador.position !== 'Goalkeeper'">
            <div class="stat"><span class="stat-label">RIT</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 0) }}</span></div>
            <div class="stat"><span class="stat-label">FIN</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 1) }}</span></div>
            <div class="stat"><span class="stat-label">PAS</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 2) }}</span></div>
            <div class="stat"><span class="stat-label">CON</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 3) }}</span></div>
            <div class="stat"><span class="stat-label">DEF</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 4) }}</span></div>
            <div class="stat"><span class="stat-label">FÍS</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 5) }}</span></div>
          </div>

          <!-- Stats de goleiro -->
          <div class="sticker-stats gk-stats" v-else>
            <div class="stat"><span class="stat-label">ELA</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 0) }}</span></div>
            <div class="stat"><span class="stat-label">MAN</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 1) }}</span></div>
            <div class="stat"><span class="stat-label">CHU</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 2) }}</span></div>
            <div class="stat"><span class="stat-label">REF</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 3) }}</span></div>
            <div class="stat"><span class="stat-label">VEL</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 4) }}</span></div>
            <div class="stat"><span class="stat-label">POS</span><span class="stat-val">{{ fakeStatSeeded(jogador.id, 5) }}</span></div>
          </div>
        </div>
      </div>

    </main>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Anton&family=Barlow+Condensed:wght@400;600;700&family=Barlow:wght@400;500;600&display=swap');

*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

html, body {
  min-height: 100vh;
  background: #e8edf2;
  font-family: 'Barlow', sans-serif;
  color: #1a2035;
}

/* ═══ NAVBAR ═══ */
.navbar {
  background: #0f1f3d;
  display: flex;
  align-items: center;
  gap: 40px;
  padding: 0 32px;
  height: 56px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.35);
  position: sticky;
  top: 0;
  z-index: 100;
}

.navbar-brand {
  display: flex;
  align-items: center;
  gap: 10px;
}

.brand-icon { font-size: 22px; }

.brand-name {
  font-family: 'Anton', sans-serif;
  font-size: 22px;
  color: #f5c518;
  letter-spacing: 1px;
}

.navbar-links { display: flex; gap: 4px; }

.nav-link {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 600;
  font-size: 15px;
  letter-spacing: 1px;
  color: rgba(255,255,255,0.65);
  text-decoration: none;
  padding: 6px 16px;
  border-radius: 6px;
  transition: color .2s, background .2s;
}

.nav-link:hover, .nav-link.active {
  color: #fff;
  background: rgba(245,197,24,0.15);
}

.nav-link.active { color: #f5c518; }

/* ═══ LAYOUT ═══ */
.album-wrapper {
  display: flex;
  max-width: 1400px;
  margin: 0 auto;
  padding: 28px 20px 60px;
  gap: 24px;
  align-items: flex-start;
}

/* ═══ SIDEBAR ═══ */
.sidebar {
  width: 220px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.sidebar-label {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 11px;
  letter-spacing: 2px;
  color: #6b7a99;
}

.select-box {
  position: relative;
  display: flex;
  align-items: center;
}

.select-flag, .select-globe {
  position: absolute;
  left: 10px;
  width: 26px;
  height: 18px;
  object-fit: cover;
  border-radius: 3px;
  font-size: 18px;
  pointer-events: none;
  z-index: 2;
}

.select-pais {
  width: 100%;
  padding: 10px 34px 10px 44px;
  border: 1.5px solid #c8d0df;
  border-radius: 10px;
  background: #fff;
  color: #1a2035;
  font-family: 'Barlow', sans-serif;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  appearance: none;
  outline: none;
  transition: border-color .2s, box-shadow .2s;
}

.select-pais:focus {
  border-color: #1a5fd4;
  box-shadow: 0 0 0 3px rgba(26,95,212,0.15);
}

.select-chevron {
  position: absolute;
  right: 10px;
  color: #6b7a99;
  pointer-events: none;
  font-size: 14px;
}

/* Team card na sidebar */
.team-card {
  background: #fff;
  border: 1.5px solid #c8d0df;
  border-radius: 12px;
  padding: 14px;
  display: flex;
  align-items: center;
  gap: 12px;
}

.team-flag {
  width: 44px;
  height: 30px;
  object-fit: cover;
  border-radius: 5px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.18);
  flex-shrink: 0;
}

.team-info { display: flex; flex-direction: column; gap: 2px; }

.team-label {
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 2px;
  color: #6b7a99;
}

.team-name {
  font-family: 'Anton', sans-serif;
  font-size: 17px;
  color: #0f1f3d;
  letter-spacing: 0.5px;
}

.team-count {
  font-size: 11px;
  color: #6b7a99;
  font-weight: 500;
}

/* Legenda de stats */
.stats-legend {
  background: #fff;
  border: 1.5px solid #c8d0df;
  border-radius: 12px;
  padding: 14px;
}

.legend-title {
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 2px;
  color: #6b7a99;
  margin-bottom: 8px;
}

.legend-items { display: flex; flex-wrap: wrap; gap: 4px; }

.leg {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 10px;
  letter-spacing: 1px;
  padding: 3px 7px;
  border-radius: 4px;
}

.leg.outfield { background: #1a5fd4; color: #fff; }
.leg.gk       { background: #f5c518; color: #0f1f3d; }

.legend-note {
  font-size: 10px;
  color: #9aa3b8;
  margin-top: 8px;
}

/* ═══ MAIN ═══ */
.album-main { flex: 1; min-width: 0; }

.album-header { margin-bottom: 20px; }

.album-title {
  font-family: 'Anton', sans-serif;
  font-size: clamp(20px, 3vw, 28px);
  color: #0f1f3d;
  letter-spacing: 1px;
}

.album-subtitle {
  font-family: 'Barlow Condensed', sans-serif;
  font-size: 15px;
  font-weight: 600;
  color: #6b7a99;
  margin-top: 4px;
  letter-spacing: 1px;
}

/* Banner da seleção */
.album-team-banner {
  display: flex;
  align-items: center;
  gap: 18px;
  background: #fff;
  border: 1.5px solid #c8d0df;
  border-left: 5px solid #f5c518;
  border-radius: 12px;
  padding: 14px 20px;
  margin-bottom: 22px;
}

.banner-flag {
  width: 60px;
  height: 40px;
  object-fit: cover;
  border-radius: 6px;
  box-shadow: 0 3px 10px rgba(0,0,0,0.2);
  border: 1px solid rgba(0,0,0,0.1);
}

.banner-badge { display: flex; flex-direction: column; }

.banner-country {
  font-family: 'Anton', sans-serif;
  font-size: 22px;
  color: #0f1f3d;
  letter-spacing: 1px;
}

.banner-label {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 2.5px;
  color: #6b7a99;
}

.banner-collected {
  margin-left: auto;
  background: #0f1f3d;
  color: #f5c518;
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 13px;
  letter-spacing: 1px;
  padding: 6px 14px;
  border-radius: 8px;
}

/* Loading */
.loading-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  padding: 80px 0;
  color: #6b7a99;
  font-weight: 600;
  letter-spacing: 1px;
  font-size: 14px;
}

.spinner {
  width: 44px;
  height: 44px;
  border: 3px solid #c8d0df;
  border-top-color: #1a5fd4;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }

/* Empty state */
.empty-state {
  text-align: center;
  padding: 80px 20px;
  color: #9aa3b8;
  font-size: 16px;
  font-weight: 600;
}

.empty-icon { font-size: 50px; margin-bottom: 16px; opacity: 0.5; }

/* ═══ FIGURINHAS ═══ */
.sticker-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(148px, 1fr));
  gap: 16px;
}

.sticker {
  background: #fff;
  border-radius: 12px;
  border: 1.5px solid #c8d0df;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  cursor: pointer;
  animation: sticker-in 0.4s ease both;
  animation-delay: var(--delay);
  transition: transform .3s ease, box-shadow .3s ease;
}

.sticker:hover {
  transform: translateY(-6px) scale(1.03);
  box-shadow: 0 12px 30px rgba(0,0,0,0.18);
  border-color: #1a5fd4;
}

@keyframes sticker-in {
  from { opacity: 0; transform: translateY(20px) scale(0.95); }
  to   { opacity: 1; transform: translateY(0) scale(1); }
}

/* Topo colorido */
.sticker-top {
  background: linear-gradient(135deg, #1a3a7a 0%, #1a5fd4 100%);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 6px 8px;
}

.is-gk .sticker-top {
  background: linear-gradient(135deg, #7a5500 0%, #c89600 100%);
}

.sticker-num {
  font-family: 'Anton', sans-serif;
  font-size: 14px;
  color: rgba(255,255,255,0.75);
  letter-spacing: 0.5px;
}

.sticker-flag {
  width: 26px;
  height: 17px;
  object-fit: cover;
  border-radius: 3px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.35);
  border: 1px solid rgba(255,255,255,0.3);
}

/* Foto */
.sticker-photo-wrap {
  background: linear-gradient(180deg, #d6e4f7 0%, #eef3fb 100%);
  display: flex;
  align-items: flex-end;
  justify-content: center;
  height: 110px;
  overflow: hidden;
}

.is-gk .sticker-photo-wrap {
  background: linear-gradient(180deg, #fdf0c2 0%, #fdf7e3 100%);
}

.sticker-photo {
  width: 90px;
  height: 90px;
  object-fit: cover;
  border-radius: 50%;
  border: 3px solid #fff;
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
  margin-bottom: 8px;
}

/* Nome e posição */
.sticker-body {
  padding: 8px 8px 4px;
  text-align: center;
  border-bottom: 1px solid #eef0f5;
}

.sticker-name {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 13px;
  color: #0f1f3d;
  letter-spacing: 0.5px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.sticker-pos {
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 1.5px;
  color: #6b7a99;
  text-transform: uppercase;
  margin-top: 2px;
}

/* Stats */
.sticker-stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  padding: 6px 6px 8px;
  gap: 4px;
}

.stat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1px;
}

.stat-label {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 8px;
  letter-spacing: 1px;
  color: #fff;
  background: #1a5fd4;
  border-radius: 3px;
  padding: 1px 4px;
  line-height: 1.4;
}

.gk-stats .stat-label {
  background: #c89600;
  color: #0f1f3d;
}

.stat-val {
  font-family: 'Anton', sans-serif;
  font-size: 14px;
  color: #0f1f3d;
  line-height: 1.2;
}

/* ═══ RESPONSIVO ═══ */
@media (max-width: 900px) {
  .album-wrapper { flex-direction: column; }
  .sidebar { width: 100%; flex-direction: row; flex-wrap: wrap; }
  .stats-legend { display: none; }
  .sticker-grid { grid-template-columns: repeat(auto-fill, minmax(130px, 1fr)); }
}

@media (max-width: 500px) {
  .sticker-grid { grid-template-columns: repeat(3, 1fr); gap: 10px; }
  .album-wrapper { padding: 16px 12px 40px; }
}
</style>