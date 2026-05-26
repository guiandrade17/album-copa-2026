<script setup>
import { ref, onMounted } from 'vue'

const token = '5dc0faddd67d4bd655e074569c6111e8'

const paises = ref([])
const paisSelecionado = ref('')
const jogadores = ref([])
const carregando = ref(false)

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

onMounted(() => {
  carregarPaises()
})
</script>

<template>
  <div class="bg-orbs">
    <div class="orb orb-1"></div>
    <div class="orb orb-2"></div>
    <div class="orb orb-3"></div>
  </div>

  <div class="container">

    <header class="header">
      <div class="header-badge">FIFA WORLD CUP</div>
      <h1 class="titulo">
        <span class="titulo-icon">⚽</span>
        Álbum da Copa
        <span class="titulo-ano">2026</span>
      </h1>
      <p class="subtitulo">Selecione uma seleção e monte seu álbum</p>
    </header>

    <div class="select-wrapper">
      <div class="select-icon">🌍</div>
      <select
        v-model="paisSelecionado"
        @change="carregarFigurinhas"
        class="select-pais"
      >
        <option value="">Selecione uma seleção...</option>
        <option
          v-for="pais in paises"
          :key="pais.name"
          :value="pais.name"
        >
          {{ pais.name }}
        </option>
      </select>
      <div class="select-arrow">▾</div>
    </div>

    <transition name="fade">
      <div v-if="paisSelecionado && !carregando" class="selecionada-banner">
        <span class="selecionada-label">SELEÇÃO</span>
        <span class="selecionada-nome">{{ paisSelecionado }}</span>
        <span class="selecionada-count" v-if="jogadores.length">
          {{ jogadores.length }} jogadores
        </span>
      </div>
    </transition>

    <transition name="fade">
      <div v-if="carregando" class="loading-area">
        <div class="spinner">
          <div class="spinner-inner"></div>
          <div class="spinner-bola">⚽</div>
        </div>
        <p class="loading-texto">Carregando elenco...</p>
      </div>
    </transition>

    <div class="grid">
      <div
        class="card"
        v-for="(jogador, index) in jogadores"
        :key="jogador.id"
        :style="{ '--delay': index * 0.05 + 's' }"
      >
        <div class="card-num">#{{ index + 1 }}</div>

        <div class="foto-wrapper">
          <div class="foto-ring"></div>
          <div class="foto-bg"></div>
          <div class="foto">
            <img :src="jogador.photo" :alt="jogador.name" />
          </div>
        </div>

        <div class="card-nome">{{ jogador.name }}</div>
        <div class="card-posicao">{{ jogador.position }}</div>

        <div class="divider"></div>
      </div>
    </div>

    <transition name="fade">
      <div v-if="!paisSelecionado && !carregando" class="empty-state">
        <div class="empty-icon">📋</div>
        <p>Escolha uma seleção acima para ver os jogadores</p>
      </div>
    </transition>

  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Rajdhani:wght@500;600;700&display=swap');

/* ─── Reset ─── */
*, *::before, *::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  min-height: 100vh;
  background: #07111f;
  font-family: 'Rajdhani', sans-serif;
  overflow-x: hidden;
}

/* ─── Orbs de fundo ─── */
.bg-orbs {
  position: fixed;
  inset: 0;
  z-index: 0;
  pointer-events: none;
  overflow: hidden;
  background: radial-gradient(ellipse at 20% 10%, #0d2b5e 0%, transparent 60%),
              radial-gradient(ellipse at 80% 80%, #051630 0%, #07111f 100%);
}

.orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.22;
  animation: orb-float 12s ease-in-out infinite;
}

.orb-1 {
  width: 600px; height: 600px;
  top: -200px; left: -150px;
  background: radial-gradient(circle, #1a4a8f, transparent);
  animation-delay: 0s;
}

.orb-2 {
  width: 500px; height: 500px;
  top: 40%; right: -150px;
  background: radial-gradient(circle, #c8960a, transparent);
  animation-delay: -4s;
}

.orb-3 {
  width: 400px; height: 400px;
  bottom: -100px; left: 35%;
  background: radial-gradient(circle, #0f3d7a, transparent);
  animation-delay: -8s;
}

@keyframes orb-float {
  0%, 100% { transform: translateY(0px)   scale(1);    }
  50%       { transform: translateY(-40px) scale(1.08); }
}

/* ─── Container ─── */
.container {
  position: relative;
  z-index: 1;
  max-width: 1440px;
  margin: 0 auto;
  padding: 40px 28px 80px;
}

/* ─── Cabeçalho ─── */
.header {
  text-align: center;
  margin-bottom: 50px;
}

.header-badge {
  display: inline-block;
  font-family: 'Rajdhani', sans-serif;
  font-weight: 700;
  font-size: 11px;
  letter-spacing: 4px;
  color: #f0c040;
  border: 1px solid #b8860b;
  padding: 5px 18px;
  border-radius: 30px;
  margin-bottom: 18px;
  background: rgba(240, 192, 64, 0.08);
  text-transform: uppercase;
}

.titulo {
  font-family: 'Bebas Neue', sans-serif;
  font-size: clamp(52px, 9vw, 100px);
  color: #ffffff;
  line-height: 0.95;
  letter-spacing: 3px;
  text-transform: uppercase;
}

.titulo-icon {
  display: inline-block;
  animation: spin-slow 8s linear infinite;
  margin-right: 12px;
}

@keyframes spin-slow {
  from { transform: rotate(0deg); }
  to   { transform: rotate(360deg); }
}

.titulo-ano {
  color: #f0c040;
}

.subtitulo {
  margin-top: 14px;
  color: rgba(255, 255, 255, 0.5);
  font-size: 17px;
  font-weight: 500;
  letter-spacing: 1px;
}

/* ─── Select ─── */
.select-wrapper {
  position: relative;
  max-width: 560px;
  margin: 0 auto 36px;
  display: flex;
  align-items: center;
}

.select-icon {
  position: absolute;
  left: 18px;
  font-size: 22px;
  pointer-events: none;
  z-index: 2;
}

.select-pais {
  width: 100%;
  padding: 18px 50px 18px 54px;
  border-radius: 14px;
  border: 1px solid rgba(240, 192, 64, 0.35);
  background: rgba(10, 25, 55, 0.85);
  backdrop-filter: blur(16px);
  color: #ffffff;
  font-family: 'Rajdhani', sans-serif;
  font-size: 17px;
  font-weight: 600;
  cursor: pointer;
  appearance: none;
  -webkit-appearance: none;
  outline: none;
  transition: border-color 0.3s, box-shadow 0.3s;
}

.select-pais:focus {
  border-color: #f0c040;
  box-shadow: 0 0 0 3px rgba(240, 192, 64, 0.2), 0 8px 30px rgba(0, 0, 0, 0.4);
}

.select-pais option {
  background: #0d1f40;
  color: #ffffff;
}

.select-arrow {
  position: absolute;
  right: 18px;
  color: #f0c040;
  font-size: 20px;
  pointer-events: none;
}

/* ─── Banner da seleção ─── */
.selecionada-banner {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin-bottom: 40px;
  padding: 14px 28px;
  border-radius: 14px;
  background: linear-gradient(135deg, rgba(240, 192, 64, 0.1), rgba(240, 192, 64, 0.03));
  border: 1px solid rgba(240, 192, 64, 0.22);
  max-width: 500px;
  margin-left: auto;
  margin-right: auto;
  flex-wrap: wrap;
}

.selecionada-label {
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 3px;
  color: #b8860b;
}

.selecionada-nome {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 28px;
  color: #f0c040;
  letter-spacing: 2px;
}

.selecionada-count {
  font-size: 13px;
  color: rgba(255, 255, 255, 0.45);
  background: rgba(255, 255, 255, 0.07);
  padding: 3px 10px;
  border-radius: 20px;
}

/* ─── Loading ─── */
.loading-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 60px 0;
}

.spinner {
  position: relative;
  width: 72px;
  height: 72px;
}

.spinner-inner {
  position: absolute;
  inset: 0;
  border: 3px solid rgba(240, 192, 64, 0.15);
  border-top-color: #f0c040;
  border-radius: 50%;
  animation: spin 0.9s linear infinite;
}

.spinner-bola {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 30px;
  animation: spin 1.8s linear infinite reverse;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.loading-texto {
  color: rgba(255, 255, 255, 0.45);
  font-size: 16px;
  letter-spacing: 2px;
  text-transform: uppercase;
  font-weight: 600;
}

/* ─── Estado vazio ─── */
.empty-state {
  text-align: center;
  padding: 80px 20px;
  color: rgba(255, 255, 255, 0.2);
  font-size: 18px;
  font-weight: 600;
  letter-spacing: 0.5px;
}

.empty-icon {
  font-size: 60px;
  margin-bottom: 20px;
  opacity: 0.35;
}

/* ─── Grid ─── */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 22px;
}

/* ─── Card base ─── */
.card {
  position: relative;
  background: linear-gradient(145deg, #0d1e3e 0%, #091529 100%);
  border-radius: 18px;
  border: 1px solid rgba(255, 255, 255, 0.08);
  padding: 22px 16px 18px;
  text-align: center;
  overflow: hidden;
  cursor: pointer;
  animation: card-entrada 0.45s ease both;
  animation-delay: var(--delay);
  transition: transform 0.35s cubic-bezier(0.25, 0.46, 0.45, 0.94),
              box-shadow 0.35s ease;
}

@keyframes card-entrada {
  from { opacity: 0; transform: translateY(28px) scale(0.94); }
  to   { opacity: 1; transform: translateY(0)    scale(1);    }
}

/* Faixa dourada topo */
.card::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 3px;
  background: linear-gradient(90deg, transparent, #f0c040, transparent);
  opacity: 0;
  transition: opacity 0.35s;
}

/* Reflexo holográfico */
.card::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(
    135deg,
    transparent 30%,
    rgba(255, 255, 255, 0.04) 50%,
    rgba(240, 192, 64, 0.06) 70%,
    transparent 100%
  );
  opacity: 0;
  transition: opacity 0.35s;
  pointer-events: none;
}

.card:hover {
  transform: translateY(-12px) scale(1.03);
  box-shadow: 0 28px 60px rgba(0, 0, 0, 0.7),
              0 0 40px rgba(240, 192, 64, 0.18);
}

.card:hover::before { opacity: 1; }
.card:hover::after  { opacity: 1; }

/* Número da figurinha */
.card-num {
  position: absolute;
  top: 11px;
  left: 13px;
  font-family: 'Bebas Neue', sans-serif;
  font-size: 13px;
  color: rgba(240, 192, 64, 0.5);
  letter-spacing: 1px;
}

/* ─── Foto ─── */
.foto-wrapper {
  position: relative;
  width: 96px;
  height: 96px;
  margin: 0 auto 16px;
}

.foto-ring {
  position: absolute;
  inset: -4px;
  border-radius: 50%;
  background: conic-gradient(#f0c040 0deg, #b8860b 120deg, #f0c040 240deg, #b8860b 360deg);
  animation: ring-spin 6s linear infinite;
  opacity: 0;
  transition: opacity 0.4s;
}

.card:hover .foto-ring { opacity: 1; }

@keyframes ring-spin {
  to { transform: rotate(360deg); }
}

.foto-bg {
  position: absolute;
  inset: 0;
  border-radius: 50%;
  background: #091529;
  margin: 2px;
}

.foto {
  position: absolute;
  inset: 3px;
  border-radius: 50%;
  overflow: hidden;
  border: 2px solid rgba(240, 192, 64, 0.4);
  transition: border-color 0.35s, box-shadow 0.35s;
}

.card:hover .foto {
  border-color: #f0c040;
  box-shadow: 0 0 18px rgba(240, 192, 64, 0.45);
}

.foto img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

/* ─── Info ─── */
.card-nome {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 20px;
  color: #ffffff;
  letter-spacing: 1.5px;
  line-height: 1.1;
  margin-bottom: 8px;
}

.card-posicao {
  display: inline-block;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 2.5px;
  text-transform: uppercase;
  color: #f0c040;
  background: rgba(240, 192, 64, 0.1);
  border: 1px solid rgba(240, 192, 64, 0.28);
  padding: 3px 11px;
  border-radius: 20px;
  margin-bottom: 12px;
}

.divider {
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.09), transparent);
  margin-top: 12px;
}

/* ─── Transições Vue ─── */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease, transform 0.4s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}

/* ─── Responsividade ─── */
@media (max-width: 768px) {
  .container { padding: 28px 16px 60px; }
  .grid { grid-template-columns: repeat(auto-fill, minmax(155px, 1fr)); gap: 14px; }
  .foto-wrapper { width: 80px; height: 80px; }
  .card-nome { font-size: 16px; }
  .selecionada-banner { flex-direction: column; gap: 6px; }
}

@media (max-width: 480px) {
  .grid { grid-template-columns: repeat(2, 1fr); }
}
</style>