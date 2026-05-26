<script setup>
import { ref, onMounted, computed } from 'vue'

const token = '5dc0faddd67d4bd655e074569c6111e8'

const paises = ref([])
const paisSelecionado = ref('')
const jogadores = ref([])
const carregando = ref(false)
const erroApi = ref('')

const bandeiraSelecionada = computed(() => {
  const pais = paises.value.find(p => p.name === paisSelecionado.value)
  return pais?.flag ?? null
})

const carregarPaises = async () => {
  erroApi.value = ''
  try {
    const response = await fetch(
      'https://v3.football.api-sports.io/teams/countries',
      { method: 'GET', headers: { 'x-apisports-key': token } }
    )
    const dados = await response.json()
    if (dados.response && dados.response.length > 0) {
      paises.value = dados.response
    } else {
      erroApi.value = 'Nenhum país retornado pela API.'
    }
  } catch (erro) {
    erroApi.value = 'Erro ao conectar com a API.'
    console.error('Erro ao carregar países:', erro)
  }
}

const carregarFigurinhas = async () => {
  if (!paisSelecionado.value) return
  carregando.value = true
  jogadores.value = []
  erroApi.value = ''
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
    erroApi.value = 'Erro ao carregar jogadores.'
    console.error('Erro ao carregar jogadores:', erro)
  } finally {
    carregando.value = false
  }
}

const fakeStatSeeded = (id, offset) => {
  const seed = (id * 9301 + offset * 49297) % 233280
  return 55 + Math.floor((seed / 233280) * 44)
}

const fakeShirt = (id) => ((id * 7 + 3) % 23) + 1

onMounted(() => { carregarPaises() })
</script>

<template>
  <div class="root">

    <!-- ══════════ NAVBAR ══════════ -->
    <nav class="navbar">
      <div class="nb-brand">
        <div class="nb-logo">
          <span class="nb-ball">⚽</span>
          <span class="nb-fa">FA</span>
        </div>
        <div class="nb-divider"></div>
        <span class="nb-site-name">FutÁlbum</span>
      </div>

      <div class="nb-links">
        <a href="#" class="nb-link">Início</a>
        <a href="#" class="nb-link nb-active">Álbuns</a>
        <a href="#" class="nb-link">Mercado</a>
        <a href="#" class="nb-link">Perfil</a>
      </div>

      <div class="nb-right">
        <div class="nb-search">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
          <input type="text" placeholder="Buscar..." class="nb-search-input" />
        </div>
      </div>
    </nav>

    <!-- ══════════ HERO AREA ══════════ -->
    <div class="hero">
      <div class="hero-inner">
        <div class="hero-text">
          <span class="hero-badge">FIFA WORLD CUP 2026</span>
          <h1 class="hero-title">Álbum de Figurinhas</h1>
          <p class="hero-sub">Selecione uma seleção e monte seu álbum oficial</p>
        </div>

        <!-- Select de país -->
        <div class="hero-select-block">
          <label class="hero-select-label">SELECIONE UMA SELEÇÃO</label>
          <div class="hero-select-wrap">
            <div class="hero-select-flag">
              <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="hsel-flag-img" />
              <svg v-else width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="#6b7a99" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="2" y1="12" x2="22" y2="12"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/></svg>
            </div>
            <select v-model="paisSelecionado" @change="carregarFigurinhas" class="hsel">
              <option value="">{{ paises.length ? 'Escolha uma seleção...' : 'Carregando países...' }}</option>
              <option v-for="pais in paises" :key="pais.name" :value="pais.name">
                {{ pais.name }}
              </option>
            </select>
            <svg class="hsel-arrow" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="6 9 12 15 18 9"/></svg>
          </div>
          <p v-if="erroApi" class="api-erro">⚠️ {{ erroApi }}</p>
          <p v-if="paises.length" class="paises-ok">{{ paises.length }} seleções disponíveis</p>
        </div>
      </div>
    </div>

    <!-- ══════════ ALBUM CONTAINER ══════════ -->
    <div class="album-outer">

      <!-- Banner da seleção escolhida -->
      <transition name="slide-down">
        <div class="team-banner" v-if="paisSelecionado && !carregando">
          <div class="tb-left">
            <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="tb-flag" />
            <div class="tb-shield">
              <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="tb-shield-img" />
            </div>
            <div class="tb-info">
              <span class="tb-tag">SELEÇÃO NACIONAL</span>
              <span class="tb-name">{{ paisSelecionado }}</span>
            </div>
          </div>
          <div class="tb-right" v-if="jogadores.length">
            <div class="tb-stat-pill">
              <span class="tb-stat-num">{{ jogadores.length }}</span>
              <span class="tb-stat-label">Jogadores</span>
            </div>
            <div class="tb-collected">
              Figurinhas Coletadas &nbsp;<strong>{{ jogadores.length }}/{{ jogadores.length }}</strong>
            </div>
          </div>
        </div>
      </transition>

      <!-- Loading -->
      <div v-if="carregando" class="loading">
        <div class="loading-ring">
          <div class="loading-ring-inner"></div>
          <span class="loading-ball">⚽</span>
        </div>
        <p class="loading-txt">Carregando elenco...</p>
      </div>

      <!-- Empty state -->
      <div v-if="!paisSelecionado && !carregando" class="empty">
        <div class="empty-pages">
          <div class="empty-page ep1"></div>
          <div class="empty-page ep2"></div>
          <div class="empty-page ep3"></div>
        </div>
        <p class="empty-txt">Escolha uma seleção para preencher seu álbum</p>
        <p class="empty-hint">↑ Use o seletor acima</p>
      </div>

      <!-- ══ GRADE DE FIGURINHAS ══ -->
      <div class="sticker-grid" v-if="jogadores.length && !carregando">
        <div
          class="sticker"
          v-for="(jogador, index) in jogadores"
          :key="jogador.id"
          :style="{ '--i': index }"
          :class="{ 'st-gk': jogador.position === 'Goalkeeper' }"
        >
          <!-- Topo colorido + bandeira -->
          <div class="st-top">
            <span class="st-num-top">{{ fakeShirt(jogador.id) }}</span>
            <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="st-flag" />
          </div>

          <!-- Área da foto com degradê Panini -->
          <div class="st-photo-wrap">
            <img :src="jogador.photo" :alt="jogador.name" class="st-photo" loading="lazy" />
          </div>

          <!-- Nome + posição -->
          <div class="st-mid">
            <span class="st-name">{{ jogador.name }}</span>
            <span class="st-pos">{{ jogador.position }}</span>
          </div>

          <!-- Divider dourado -->
          <div class="st-divider"></div>

          <!-- Stats jogadores de linha -->
          <div class="st-stats" v-if="jogador.position !== 'Goalkeeper'">
            <div class="st-s" v-for="(lb, i) in ['RIT','FIN','PAS','CON','DEF','FÍS']" :key="lb">
              <span class="st-sv">{{ fakeStatSeeded(jogador.id, i) }}</span>
              <span class="st-sl">{{ lb }}</span>
            </div>
          </div>

          <!-- Stats goleiro -->
          <div class="st-stats st-stats-gk" v-else>
            <div class="st-s" v-for="(lb, i) in ['ELA','MAN','CHU','REF','VEL','POS']" :key="lb">
              <span class="st-sv">{{ fakeStatSeeded(jogador.id, i) }}</span>
              <span class="st-sl">{{ lb }}</span>
            </div>
          </div>

        </div>
      </div>
      <!-- fim grid -->

    </div>
    <!-- fim album-outer -->

    <footer class="footer" v-if="jogadores.length">
      <span>Termos de Uso</span>
      <span class="ft-dot">·</span>
      <span>Política de Privacidade</span>
      <span class="ft-dot">·</span>
      <span>Contato</span>
      <span class="ft-dot">·</span>
      <span>© 2026 FutÁlbum</span>
    </footer>

  </div>
</template>

<style>
/* ────────────────────────────────────────
   IMPORTS
──────────────────────────────────────── */
@import url('https://fonts.googleapis.com/css2?family=Anton&family=Barlow+Condensed:wght@400;600;700&family=Barlow:wght@400;500;600;700&display=swap');

/* ────────────────────────────────────────
   RESET + BASE — garante 100vw sem margens
──────────────────────────────────────── */
html, body, #app {
  margin: 0 !important;
  padding: 0 !important;
  width: 100% !important;
  max-width: 100% !important;
  min-height: 100vh;
  background: #eaeff7;
  font-family: 'Barlow', sans-serif;
  color: #1a2035;
  overflow-x: hidden;
}

*, *::before, *::after {
  box-sizing: border-box;
}

.root {
  width: 100%;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* ────────────────────────────────────────
   NAVBAR
──────────────────────────────────────── */
.navbar {
  width: 100%;
  background: #0f1f3d;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 32px;
  height: 58px;
  position: sticky;
  top: 0;
  z-index: 500;
  box-shadow: 0 3px 16px rgba(0,0,0,0.35);
}

.nb-brand {
  display: flex;
  align-items: center;
  gap: 12px;
}

.nb-logo {
  display: flex;
  align-items: center;
  gap: 6px;
  background: linear-gradient(135deg, #1a4aaa, #2a6aee);
  border-radius: 8px;
  padding: 5px 11px;
}

.nb-ball { font-size: 15px; }

.nb-fa {
  font-family: 'Anton', sans-serif;
  font-size: 19px;
  color: #fff;
  letter-spacing: 1px;
}

.nb-divider {
  width: 1px;
  height: 22px;
  background: rgba(255,255,255,0.15);
}

.nb-site-name {
  font-family: 'Anton', sans-serif;
  font-size: 17px;
  color: rgba(255,255,255,0.75);
  letter-spacing: 1px;
}

.nb-links {
  display: flex;
  align-items: center;
  gap: 2px;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
}

.nb-link {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 13px;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  color: rgba(255,255,255,0.5);
  text-decoration: none;
  padding: 7px 16px;
  border-radius: 6px;
  transition: color .2s, background .2s;
}

.nb-link:hover { color: #fff; background: rgba(255,255,255,0.08); }

.nb-active {
  color: #fff !important;
  border-bottom: 2px solid #4a8fff;
  border-radius: 0;
}

.nb-search {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(255,255,255,0.08);
  border: 1px solid rgba(255,255,255,0.15);
  border-radius: 8px;
  padding: 7px 14px;
  color: rgba(255,255,255,0.5);
}

.nb-search-input {
  background: transparent;
  border: none;
  outline: none;
  color: #fff;
  font-family: 'Barlow', sans-serif;
  font-size: 13px;
  width: 130px;
}

.nb-search-input::placeholder { color: rgba(255,255,255,0.3); }

/* ────────────────────────────────────────
   HERO
──────────────────────────────────────── */
.hero {
  width: 100%;
  background: linear-gradient(135deg, #0f1f3d 0%, #1a3a7a 50%, #0f2a5e 100%);
  padding: 40px 40px 44px;
  position: relative;
  overflow: hidden;
}

.hero::before {
  content: '';
  position: absolute;
  inset: 0;
  background: url("data:image/svg+xml,%3Csvg width='60' height='60' viewBox='0 0 60 60' xmlns='http://www.w3.org/2000/svg'%3E%3Cg fill='none' fill-rule='evenodd'%3E%3Cg fill='%23ffffff' fill-opacity='0.03'%3E%3Cpath d='M36 34v-4h-2v4h-4v2h4v4h2v-4h4v-2h-4zm0-30V0h-2v4h-4v2h4v4h2V6h4V4h-4zM6 34v-4H4v4H0v2h4v4h2v-4h4v-2H6zM6 4V0H4v4H0v2h4v4h2V6h4V4H6z'/%3E%3C/g%3E%3C/g%3E%3C/svg%3E");
  pointer-events: none;
}

.hero::after {
  content: '⚽';
  position: absolute;
  right: 80px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 160px;
  opacity: 0.04;
  pointer-events: none;
}

.hero-inner {
  position: relative;
  z-index: 1;
  max-width: 1400px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 32px;
  flex-wrap: wrap;
}

.hero-badge {
  display: inline-block;
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 10px;
  letter-spacing: 3px;
  color: #f5c518;
  border: 1px solid rgba(245,197,24,0.4);
  background: rgba(245,197,24,0.1);
  padding: 4px 14px;
  border-radius: 20px;
  margin-bottom: 10px;
  text-transform: uppercase;
}

.hero-title {
  font-family: 'Anton', sans-serif;
  font-size: clamp(28px, 4vw, 48px);
  color: #fff;
  letter-spacing: 1px;
  line-height: 1;
  margin-bottom: 8px;
}

.hero-sub {
  color: rgba(255,255,255,0.5);
  font-size: 15px;
  font-weight: 500;
  letter-spacing: 0.3px;
}

/* Select no hero */
.hero-select-block {
  display: flex;
  flex-direction: column;
  gap: 8px;
  min-width: 280px;
}

.hero-select-label {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 10px;
  letter-spacing: 3px;
  color: rgba(255,255,255,0.5);
  text-transform: uppercase;
}

.hero-select-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.hero-select-flag {
  position: absolute;
  left: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  pointer-events: none;
  z-index: 2;
  width: 24px;
}

.hsel-flag-img {
  width: 26px;
  height: 17px;
  object-fit: cover;
  border-radius: 3px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.3);
}

.hsel {
  width: 100%;
  padding: 13px 40px 13px 46px;
  background: rgba(255,255,255,0.1);
  backdrop-filter: blur(10px);
  border: 1.5px solid rgba(255,255,255,0.2);
  border-radius: 10px;
  color: #fff;
  font-family: 'Barlow Condensed', sans-serif;
  font-size: 15px;
  font-weight: 700;
  letter-spacing: 0.5px;
  cursor: pointer;
  appearance: none;
  outline: none;
  transition: border-color .2s, background .2s, box-shadow .2s;
}

.hsel:focus {
  border-color: #4a8fff;
  background: rgba(255,255,255,0.15);
  box-shadow: 0 0 0 3px rgba(74,143,255,0.25);
}

.hsel option {
  background: #0f1f3d;
  color: #fff;
  font-family: 'Barlow', sans-serif;
  font-size: 14px;
}

.hsel-arrow {
  position: absolute;
  right: 12px;
  color: rgba(255,255,255,0.5);
  pointer-events: none;
}

.api-erro {
  font-size: 12px;
  color: #ff7070;
  font-weight: 600;
}

.paises-ok {
  font-size: 11px;
  color: rgba(255,255,255,0.35);
  font-weight: 500;
}

/* ────────────────────────────────────────
   ALBUM OUTER
──────────────────────────────────────── */
.album-outer {
  flex: 1;
  width: 100%;
  max-width: 100%;
  padding: 28px 32px 40px;
}

/* ── BANNER DA SELEÇÃO ── */
.team-banner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #fff;
  border-radius: 14px;
  border: 1.5px solid #d0d9ee;
  border-left: 6px solid #f5c518;
  padding: 16px 24px;
  margin-bottom: 24px;
  box-shadow: 0 4px 18px rgba(0,0,0,0.07);
  flex-wrap: wrap;
  gap: 14px;
}

.tb-left {
  display: flex;
  align-items: center;
  gap: 16px;
}

.tb-flag {
  width: 72px;
  height: 48px;
  object-fit: cover;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
  border: 1px solid rgba(0,0,0,0.08);
}

.tb-shield {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: linear-gradient(135deg, #0f1f3d, #1a3a7a);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  box-shadow: 0 3px 10px rgba(0,0,0,0.25);
  border: 2px solid rgba(255,255,255,0.1);
}

.tb-shield-img {
  width: 38px;
  height: 25px;
  object-fit: cover;
  border-radius: 3px;
}

.tb-info {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.tb-tag {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 10px;
  letter-spacing: 2.5px;
  color: #8a97b8;
  text-transform: uppercase;
}

.tb-name {
  font-family: 'Anton', sans-serif;
  font-size: 26px;
  color: #0f1f3d;
  letter-spacing: 1px;
}

.tb-right {
  display: flex;
  align-items: center;
  gap: 16px;
}

.tb-stat-pill {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #eef3fe;
  border: 1.5px solid #c8d8f8;
  border-radius: 10px;
  padding: 8px 18px;
}

.tb-stat-num {
  font-family: 'Anton', sans-serif;
  font-size: 24px;
  color: #1a3a7a;
  line-height: 1;
}

.tb-stat-label {
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 1.5px;
  color: #8a97b8;
  text-transform: uppercase;
}

.tb-collected {
  background: #0f1f3d;
  color: #f5c518;
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 13px;
  letter-spacing: 0.5px;
  padding: 10px 20px;
  border-radius: 10px;
  white-space: nowrap;
}

.tb-collected strong { font-size: 15px; }

/* ── LOADING ── */
.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 18px;
  padding: 80px 0;
}

.loading-ring {
  position: relative;
  width: 64px;
  height: 64px;
}

.loading-ring-inner {
  position: absolute;
  inset: 0;
  border: 3px solid rgba(26,95,212,0.15);
  border-top-color: #1a5fd4;
  border-radius: 50%;
  animation: spin .9s linear infinite;
}

.loading-ball {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 26px;
  animation: spin 1.8s linear infinite reverse;
}

@keyframes spin { to { transform: rotate(360deg); } }

.loading-txt {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 14px;
  letter-spacing: 2px;
  color: #8a97b8;
  text-transform: uppercase;
}

/* ── EMPTY STATE ── */
.empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
  padding: 80px 0;
}

.empty-pages {
  position: relative;
  width: 80px;
  height: 90px;
  margin-bottom: 8px;
}

.empty-page {
  position: absolute;
  width: 60px;
  height: 78px;
  background: #fff;
  border: 1.5px solid #d0d9ee;
  border-radius: 6px;
}

.ep1 { left: 0; top: 12px; transform: rotate(-8deg); opacity: 0.5; }
.ep2 { left: 10px; top: 6px; transform: rotate(-3deg); opacity: 0.7; }
.ep3 { left: 20px; top: 0; background: #eef3fe; border-color: #b8cef8; }

.empty-txt {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 16px;
  letter-spacing: 0.5px;
  color: #8a97b8;
}

.empty-hint {
  font-size: 13px;
  color: #b8c4d8;
  font-weight: 500;
}

/* ────────────────────────────────────────
   GRADE DE FIGURINHAS
──────────────────────────────────────── */
.sticker-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(118px, 1fr));
  gap: 12px;
}

/* ── FIGURINHA BASE ── */
.sticker {
  background: #fff;
  border-radius: 10px;
  border: 1.5px solid #c8d2e4;
  overflow: hidden;
  box-shadow: 0 3px 10px rgba(0,0,0,0.1), 0 1px 3px rgba(0,0,0,0.06);
  cursor: pointer;
  animation: pop-in .38s cubic-bezier(.22,.68,0,1.2) both;
  animation-delay: calc(var(--i) * 0.025s);
  transition: transform .28s ease, box-shadow .28s ease, border-color .2s;
  display: flex;
  flex-direction: column;
  position: relative;
}

.sticker:hover {
  transform: translateY(-8px) scale(1.04) rotate(0.5deg);
  box-shadow: 0 18px 40px rgba(0,0,0,0.2), 0 0 0 2px #4a8fff;
  border-color: #4a8fff;
  z-index: 20;
}

@keyframes pop-in {
  from { opacity: 0; transform: scale(0.82) translateY(14px); }
  to   { opacity: 1; transform: scale(1) translateY(0); }
}

/* Topo azul / verde para GK */
.st-top {
  background: linear-gradient(90deg, #0c2660 0%, #1a4aaa 100%);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 5px 7px;
  min-height: 26px;
}

.st-gk .st-top {
  background: linear-gradient(90deg, #1a4a1a 0%, #2a8a2a 100%);
}

.st-num-top {
  font-family: 'Anton', sans-serif;
  font-size: 14px;
  color: rgba(255,255,255,0.65);
  letter-spacing: 0.5px;
  line-height: 1;
}

.st-flag {
  width: 24px;
  height: 15px;
  object-fit: cover;
  border-radius: 2px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.35);
  border: 1px solid rgba(255,255,255,0.25);
}

/* Área da foto — degradê Panini */
.st-photo-wrap {
  background: linear-gradient(180deg, #b8d0ee 0%, #d8e8f8 40%, #eef4fc 75%, #f8faff 100%);
  display: flex;
  align-items: flex-end;
  justify-content: center;
  height: 112px;
  overflow: hidden;
  position: relative;
}

.st-gk .st-photo-wrap {
  background: linear-gradient(180deg, #a8d4a8 0%, #c8e8c8 40%, #e8f5e8 75%, #f5fdf5 100%);
}

.st-photo {
  width: 88px;
  height: 104px;
  object-fit: cover;
  object-position: top center;
  display: block;
  filter: drop-shadow(0 4px 8px rgba(0,0,0,0.2));
}

/* Nome + posição */
.st-mid {
  padding: 7px 8px 5px;
  display: flex;
  flex-direction: column;
  gap: 2px;
  background: #fff;
  border-top: 1px solid #edf1f8;
}

.st-name {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 11px;
  color: #0f1f3d;
  letter-spacing: 0.3px;
  text-transform: uppercase;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  line-height: 1.2;
}

.st-pos {
  font-size: 8.5px;
  font-weight: 600;
  letter-spacing: 1px;
  color: #8a97b8;
  text-transform: uppercase;
  line-height: 1.3;
}

/* Divider dourado */
.st-divider {
  height: 2px;
  background: linear-gradient(90deg, transparent, #f5c518 30%, #f5c518 70%, transparent);
  opacity: 0.7;
}

.st-gk .st-divider {
  background: linear-gradient(90deg, transparent, #2a8a2a 30%, #2a8a2a 70%, transparent);
}

/* Stats — faixa inferior */
.st-stats {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  background: linear-gradient(90deg, #0c2660, #1a4aaa);
  padding: 5px 4px 6px;
  gap: 1px;
}

.st-stats-gk {
  background: linear-gradient(90deg, #1a4a1a, #2a8a2a);
}

.st-s {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0;
}

.st-sv {
  font-family: 'Anton', sans-serif;
  font-size: 12px;
  color: #fff;
  line-height: 1.1;
}

.st-sl {
  font-family: 'Barlow Condensed', sans-serif;
  font-weight: 700;
  font-size: 6px;
  letter-spacing: 0.3px;
  color: rgba(255,255,255,0.55);
  text-transform: uppercase;
  line-height: 1.1;
}

/* ── TRANSIÇÕES ── */
.slide-down-enter-active, .slide-down-leave-active {
  transition: opacity .4s ease, transform .4s ease;
}
.slide-down-enter-from, .slide-down-leave-to {
  opacity: 0;
  transform: translateY(-12px);
}

/* ── FOOTER ── */
.footer {
  width: 100%;
  text-align: center;
  padding: 18px;
  font-size: 11px;
  color: #8a97b8;
  font-weight: 500;
  letter-spacing: 0.5px;
  border-top: 1px solid #d0d9ee;
  background: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.ft-dot { color: #c8d2e4; }

/* ────────────────────────────────────────
   RESPONSIVO
──────────────────────────────────────── */
@media (max-width: 900px) {
  .nb-links { position: static; transform: none; }
  .hero-inner { flex-direction: column; align-items: flex-start; }
  .hero-select-block { width: 100%; }
  .hsel { width: 100%; }
  .album-outer { padding: 20px 16px 32px; }
  .sticker-grid { grid-template-columns: repeat(auto-fill, minmax(100px, 1fr)); gap: 9px; }
  .st-photo-wrap { height: 96px; }
  .st-photo { width: 75px; height: 88px; }
}

@media (max-width: 600px) {
  .navbar { padding: 0 16px; }
  .nb-site-name { display: none; }
  .nb-search { display: none; }
  .hero { padding: 28px 20px 32px; }
  .sticker-grid { grid-template-columns: repeat(3, 1fr); gap: 8px; }
  .team-banner { padding: 12px 14px; }
  .tb-name { font-size: 20px; }
}
</style>