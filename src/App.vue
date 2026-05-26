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
        headers: { 'x-apisports-key': token }
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

// Stats fictícias mas determinísticas por jogador
const fakeStatSeeded = (id, offset) => {
  const seed = (id * 9301 + offset * 49297) % 233280
  return 55 + Math.floor((seed / 233280) * 44)
}

// Número de camisa simulado
const fakeShirt = (id) => {
  return ((id * 7 + 3) % 23) + 1
}

onMounted(() => {
  carregarPaises()
})
</script>

<template>
  <div class="page">

    <!-- ── NAVBAR ── -->
    <nav class="navbar">
      <div class="nav-left">
        <div class="nav-logo">
          <span class="logo-icon">⚽</span>
          <span class="logo-fa">FA</span>
        </div>
        <a href="#" class="nav-link">Início</a>
        <a href="#" class="nav-link active">Álbuns</a>
        <a href="#" class="nav-link">Mercado</a>
        <a href="#" class="nav-link">Perfil</a>
      </div>
      <div class="nav-right">
        <div class="search-box">
          <span class="search-icon">🔍</span>
          <input type="text" placeholder="Busca..." class="search-input" />
        </div>
      </div>
    </nav>

    <!-- ── CONTEÚDO ── -->
    <div class="content">

      <!-- Cabeçalho + select -->
      <div class="top-bar">
        <h1 class="page-title">ÁLBUM DE FIGURINHAS: SELEÇÕES DO MUNDO</h1>
        <div class="select-area">
          <span class="select-label">SELECIONE UM TIME:</span>
          <div class="select-wrap">
            <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="sel-flag" />
            <span v-else class="sel-globe">🌍</span>
            <select v-model="paisSelecionado" @change="carregarFigurinhas" class="sel">
              <option value="">Selecione...</option>
              <option v-for="pais in paises" :key="pais.name" :value="pais.name">
                {{ pais.name }}
              </option>
            </select>
            <span class="sel-arrow">▾</span>
          </div>
        </div>
      </div>

      <!-- Álbum box -->
      <div class="album-box">

        <!-- Subtítulo -->
        <div class="album-sub" v-if="paisSelecionado">
          Álbum de Figurinhas – {{ paisSelecionado }}
        </div>
        <div class="album-sub" v-else>Álbum de Figurinhas</div>

        <!-- Banner da seleção -->
        <div class="team-banner" v-if="paisSelecionado && !carregando">
          <div class="banner-left">
            <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="banner-flag" />
            <div class="banner-shield">
              <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="shield-img" />
            </div>
            <div class="banner-text">
              <span class="banner-country-name">SELEÇÃO {{ paisSelecionado.toUpperCase() }}</span>
              <span class="banner-country-sub">{{ paisSelecionado }}</span>
            </div>
          </div>
          <div class="banner-right" v-if="jogadores.length">
            <div class="collected-box">
              Figurinhas Coletadas: <strong>{{ jogadores.length }} / {{ jogadores.length }}</strong>
            </div>
          </div>
        </div>

        <!-- Loading -->
        <div v-if="carregando" class="loading-area">
          <div class="spinner"></div>
          <p>Carregando elenco...</p>
        </div>

        <!-- Empty -->
        <div v-if="!paisSelecionado && !carregando" class="empty-state">
          <span class="empty-icon">📋</span>
          <p>Escolha uma seleção acima para ver os jogadores</p>
        </div>

        <!-- ── GRADE DE FIGURINHAS ── -->
        <div class="sticker-grid" v-if="jogadores.length && !carregando">
          <div
            class="sticker"
            v-for="(jogador, index) in jogadores"
            :key="jogador.id"
            :style="{ '--i': index }"
            :class="{ 'sticker-gk': jogador.position === 'Goalkeeper' }"
          >
            <!-- Faixa superior colorida com bandeira -->
            <div class="st-header">
              <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="st-flag" />
            </div>

            <!-- Foto do jogador -->
            <div class="st-photo-area">
              <img :src="jogador.photo" :alt="jogador.name" class="st-photo" />
            </div>

            <!-- Rodapé da figurinha -->
            <div class="st-footer">
              <div class="st-num">{{ fakeShirt(jogador.id) }}</div>
              <div class="st-info">
                <div class="st-name">{{ jogador.name }}</div>
                <div class="st-pos">{{ jogador.position }}</div>
              </div>
              <div class="st-dot">
                <div class="dot-circle"></div>
              </div>
            </div>

            <!-- Stats na parte de baixo -->
            <div class="st-stats" v-if="jogador.position !== 'Goalkeeper'">
              <div class="st-stat" v-for="(s, i) in ['RIT','FIN','PAS','CON','DEF','FÍS']" :key="s">
                <span class="st-stat-v">{{ fakeStatSeeded(jogador.id, i) }}</span>
                <span class="st-stat-l">{{ s }}</span>
              </div>
            </div>
            <div class="st-stats st-stats-gk" v-else>
              <div class="st-stat" v-for="(s, i) in ['ELA','MAN','CHU','REF','VEL','POS']" :key="s">
                <span class="st-stat-v">{{ fakeStatSeeded(jogador.id, i) }}</span>
                <span class="st-stat-l">{{ s }}</span>
              </div>
            </div>

          </div>
        </div>
        <!-- fim grid -->

      </div>
      <!-- fim album-box -->

    </div>
    <!-- fim content -->

    <!-- FOOTER -->
    <footer class="footer" v-if="jogadores.length">
      Termos de Uso | Política de Privacidade | Contato | © 2026 FutÁlbum
    </footer>

  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Anton&family=Barlow+Condensed:ital,wght@0,400;0,600;0,700;1,700&family=Barlow:wght@400;500;600;700&display=swap');

/* ── RESET ── */
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

html, body {
  height: 100%;
  background: #dde3ec;
  font-family: 'Barlow', sans-serif;
  color: #1a2035;
}

.page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* ── NAVBAR ── */
.navbar {
  background: #fff;
  border-bottom: 2px solid #e0e5ef;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 28px;
  height: 52px;
  position: sticky;
  top: 0;
  z-index: 200;
  box-shadow: 0 2px 8px rgba(0,0,0,0.07);
}

.nav-left {
  display: flex;
  align-items: center;
  gap: 4px;
}

.nav-logo {
  display: flex;
  align-items: center;
  gap: 5px;
  background: #1a3a7a;
  border-radius: 8px;
  padding: 5px 10px;
  margin-right: 14px;
}

.logo-icon { font-size: 16px; }

.logo-fa {
  font-family: 'Anton', sans-serif;
  font-size: 18px;
  color: #fff;
  letter-spacing: 1px;
}

.nav-link {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 14px;
  letter-spacing: 0.5px;
  color: #4a5578;
  text-decoration: none;
  padding: 6px 14px;
  border-radius: 6px;
  transition: color .2s, background .2s;
  text-transform: uppercase;
}

.nav-link:hover { color: #1a3a7a; background: #eef2fb; }

.nav-link.active {
  color: #1a3a7a;
  border-bottom: 3px solid #1a3a7a;
  border-radius: 0;
}

.search-box {
  display: flex;
  align-items: center;
  gap: 8px;
  background: #f0f3fa;
  border: 1.5px solid #d0d7e8;
  border-radius: 8px;
  padding: 6px 14px;
}

.search-icon { font-size: 14px; opacity: 0.5; }

.search-input {
  border: none;
  background: transparent;
  outline: none;
  font-family: 'Barlow', sans-serif;
  font-size: 13px;
  color: #1a2035;
  width: 140px;
}

.search-input::placeholder { color: #9aa3b8; }

/* ── CONTENT ── */
.content {
  flex: 1;
  padding: 20px 24px 0;
}

/* ── TOP BAR ── */
.top-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
  flex-wrap: wrap;
  gap: 12px;
}

.page-title {
  font-family: 'Anton', sans-serif;
  font-size: clamp(16px, 2.2vw, 24px);
  color: #0f1f3d;
  letter-spacing: 1px;
}

.select-area {
  display: flex;
  align-items: center;
  gap: 10px;
}

.select-label {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 11px;
  letter-spacing: 2px;
  color: #6b7a99;
  white-space: nowrap;
  text-transform: uppercase;
}

.select-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.sel-flag {
  position: absolute;
  left: 10px;
  width: 24px;
  height: 16px;
  object-fit: cover;
  border-radius: 3px;
  pointer-events: none;
  z-index: 2;
  box-shadow: 0 1px 4px rgba(0,0,0,0.2);
}

.sel-globe {
  position: absolute;
  left: 10px;
  font-size: 16px;
  pointer-events: none;
  z-index: 2;
}

.sel {
  padding: 9px 36px 9px 42px;
  border: 1.5px solid #c0cae0;
  border-radius: 8px;
  background: #fff;
  color: #1a2035;
  font-family: 'Barlow Condensed', sans-serif;
  font-size: 14px;
  font-weight: 700;
  letter-spacing: 0.5px;
  cursor: pointer;
  appearance: none;
  outline: none;
  min-width: 200px;
  transition: border-color .2s, box-shadow .2s;
}

.sel:focus {
  border-color: #1a5fd4;
  box-shadow: 0 0 0 3px rgba(26,95,212,0.12);
}

.sel-arrow {
  position: absolute;
  right: 12px;
  color: #6b7a99;
  pointer-events: none;
  font-size: 13px;
}

/* ── ALBUM BOX ── */
.album-box {
  background: #f5f7fc;
  border: 1.5px solid #d0d7e8;
  border-radius: 14px;
  padding: 18px 20px 24px;
  min-height: 60vh;
}

.album-sub {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 13px;
  letter-spacing: 1px;
  color: #6b7a99;
  margin-bottom: 14px;
  text-transform: uppercase;
}

/* ── BANNER DA SELEÇÃO ── */
.team-banner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #fff;
  border: 1.5px solid #d0d7e8;
  border-radius: 10px;
  padding: 14px 20px;
  margin-bottom: 20px;
  flex-wrap: wrap;
  gap: 12px;
}

.banner-left {
  display: flex;
  align-items: center;
  gap: 14px;
}

.banner-flag {
  width: 72px;
  height: 48px;
  object-fit: cover;
  border-radius: 6px;
  border: 1px solid rgba(0,0,0,0.1);
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
}

.banner-shield {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background: #1a3a7a;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #d0d7e8;
}

.shield-img {
  width: 36px;
  height: 24px;
  object-fit: cover;
  border-radius: 3px;
}

.banner-text {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.banner-country-name {
  font-family: 'Anton', sans-serif;
  font-size: 20px;
  color: #0f1f3d;
  letter-spacing: 1px;
}

.banner-country-sub {
  font-family: 'Barlow Condensed', sans-serif;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 2px;
  color: #6b7a99;
  text-transform: uppercase;
}

.collected-box {
  background: #e8f0fe;
  border: 1.5px solid #b8cef8;
  border-radius: 8px;
  padding: 8px 18px;
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 14px;
  color: #1a3a7a;
  letter-spacing: 0.5px;
}

.collected-box strong { font-size: 16px; }

/* ── LOADING ── */
.loading-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
  padding: 70px 0;
  color: #6b7a99;
  font-weight: 600;
  font-size: 14px;
  letter-spacing: 1px;
  text-transform: uppercase;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #d0d7e8;
  border-top-color: #1a5fd4;
  border-radius: 50%;
  animation: spin .8s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }

/* ── EMPTY ── */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  padding: 70px 0;
  color: #9aa3b8;
  font-size: 15px;
  font-weight: 600;
}

.empty-icon { font-size: 48px; opacity: 0.45; }

/* ══════════════════════════════
   FIGURINHAS — inspiradas na imagem
══════════════════════════════ */
.sticker-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(112px, 1fr));
  gap: 10px;
}

.sticker {
  background: #fff;
  border-radius: 8px;
  border: 1.5px solid #c8d2e4;
  overflow: hidden;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1), 0 1px 2px rgba(0,0,0,0.06);
  cursor: pointer;
  animation: pop-in .35s ease both;
  animation-delay: calc(var(--i) * 0.03s);
  transition: transform .25s ease, box-shadow .25s ease;
  display: flex;
  flex-direction: column;
}

.sticker:hover {
  transform: translateY(-5px) scale(1.04);
  box-shadow: 0 10px 24px rgba(0,0,0,0.18);
  border-color: #1a5fd4;
  z-index: 10;
}

@keyframes pop-in {
  from { opacity: 0; transform: scale(0.88) translateY(12px); }
  to   { opacity: 1; transform: scale(1) translateY(0); }
}

/* Faixa superior — azul escuro para jogadores, verde-dourado para GK */
.st-header {
  background: linear-gradient(90deg, #0f2a5e 60%, #1a5fd4 100%);
  height: 22px;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  padding: 0 6px;
}

.sticker-gk .st-header {
  background: linear-gradient(90deg, #1a4a1a 60%, #2d8c2d 100%);
}

.st-flag {
  width: 22px;
  height: 14px;
  object-fit: cover;
  border-radius: 2px;
  border: 1px solid rgba(255,255,255,0.4);
  box-shadow: 0 1px 3px rgba(0,0,0,0.3);
}

/* Foto — fundo degradê claro igual ao Panini */
.st-photo-area {
  background: linear-gradient(180deg, #c8d8f0 0%, #e8f0fa 55%, #f5f8fd 100%);
  display: flex;
  align-items: flex-end;
  justify-content: center;
  height: 108px;
  overflow: hidden;
  position: relative;
}

.sticker-gk .st-photo-area {
  background: linear-gradient(180deg, #c8e8c8 0%, #e8f5e8 55%, #f5fdf5 100%);
}

.st-photo {
  width: 84px;
  height: 96px;
  object-fit: cover;
  object-position: top center;
  display: block;
  /* sem borda circular — igual Panini: foto retangular recortada */
}

/* Rodapé branco com número, nome e posição */
.st-footer {
  background: #fff;
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 5px 6px 4px;
  border-top: 1px solid #e8ecf4;
}

.st-num {
  font-family: 'Anton', sans-serif;
  font-size: 18px;
  color: #0f1f3d;
  min-width: 20px;
  line-height: 1;
}

.st-info {
  flex: 1;
  min-width: 0;
}

.st-name {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 10px;
  color: #0f1f3d;
  letter-spacing: 0.3px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  text-transform: uppercase;
  line-height: 1.2;
}

.st-pos {
  font-family: 'Barlow', sans-serif;
  font-size: 8px;
  color: #6b7a99;
  font-weight: 600;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  line-height: 1.3;
}

/* Bolinha do clube (imitando ícone do clube no Panini) */
.st-dot { flex-shrink: 0; }

.dot-circle {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: linear-gradient(135deg, #1a3a7a, #3a7ad4);
  border: 1.5px solid #d0d7e8;
}

.sticker-gk .dot-circle {
  background: linear-gradient(135deg, #1a4a1a, #3a9a3a);
}

/* Stats — faixa inferior compacta */
.st-stats {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  background: #0f2a5e;
  padding: 4px 4px 5px;
  gap: 2px;
}

.st-stats-gk {
  background: #1a4a1a;
}

.st-stat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0;
}

.st-stat-v {
  font-family: 'Anton', sans-serif;
  font-size: 11px;
  color: #fff;
  line-height: 1.1;
}

.st-stat-l {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 6.5px;
  letter-spacing: 0.3px;
  color: rgba(255,255,255,0.6);
  text-transform: uppercase;
  line-height: 1;
}

/* ── FOOTER ── */
.footer {
  text-align: center;
  padding: 16px;
  font-size: 11px;
  color: #9aa3b8;
  font-weight: 500;
  letter-spacing: 0.3px;
  border-top: 1px solid #d0d7e8;
  margin-top: 24px;
}

/* ── RESPONSIVO ── */
@media (max-width: 768px) {
  .content { padding: 14px 12px 0; }
  .top-bar { flex-direction: column; align-items: flex-start; }
  .sticker-grid { grid-template-columns: repeat(auto-fill, minmax(96px, 1fr)); gap: 8px; }
  .st-photo-area { height: 90px; }
  .st-photo { width: 70px; height: 82px; }
  .st-stats { padding: 3px 2px 4px; }
  .st-stat-v { font-size: 10px; }
  .st-stat-l { font-size: 6px; }
}

@media (max-width: 480px) {
  .sticker-grid { grid-template-columns: repeat(3, 1fr); gap: 7px; }
  .navbar { padding: 0 14px; }
  .nav-link { font-size: 12px; padding: 5px 8px; }
}
</style>