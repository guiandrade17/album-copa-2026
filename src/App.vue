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
  <!-- Partículas decorativas de fundo -->
  <div class="bg-orbs">
    <div class="orb orb-1"></div>
    <div class="orb orb-2"></div>
    <div class="orb orb-3"></div>
  </div>

  <div class="container">

    <!-- Cabeçalho -->
    <header class="header">
      <div class="header-badge">FIFA WORLD CUP</div>
      <h1 class="titulo">
        <span class="titulo-icon">⚽</span>
        Álbum da Copa
        <span class="titulo-ano">2026</span>
      </h1>
      <p class="subtitulo">Selecione uma seleção e monte seu álbum</p>
    </header>

    <!-- Seletor de país -->
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

    <!-- Nome da seleção selecionada -->
    <transition name="fade">
      <div v-if="paisSelecionado && !carregando" class="selecionada-banner">
        <span class="selecionada-label">SELEÇÃO</span>
        <span class="selecionada-nome">{{ paisSelecionado }}</span>
        <span class="selecionada-count" v-if="jogadores.length">
          {{ jogadores.length }} jogadores
        </span>
      </div>
    </transition>

    <!-- Loading Spinner -->
    <transition name="fade">
      <div v-if="carregando" class="loading-area">
        <div class="spinner">
          <div class="spinner-inner"></div>
          <div class="spinner-bola">⚽</div>
        </div>
        <p class="loading-texto">Carregando elenco...</p>
      </div>
    </transition>

    <!-- Grid de cards -->
    <transition-group name="card-list" tag="div" class="grid">
      <div
        class="card"
        v-for="(jogador, index) in jogadores"
        :key="jogador.id"
        :style="{ '--delay': index * 0.05 + 's' }"
      >
        <!-- Número da figurinha -->
        <div class="card-numero">#{{ index + 1 }}</div>

        <!-- Foto -->
        <div class="card-foto-wrapper">
          <img
            :src="jogador.photo"
            :alt="jogador.name"
            class="card-foto"
          />
          <div class="card-foto-brilho"></div>
        </div>

        <!-- Info -->
        <div class="card-info">
          <h2 class="card-nome">{{ jogador.name }}</h2>
          <span class="card-posicao">{{ jogador.position }}</span>
        </div>

        <!-- Efeito holográfico no hover -->
        <div class="card-holo"></div>
      </div>
    </transition-group>

    <!-- Estado vazio -->
    <transition name="fade">
      <div v-if="!paisSelecionado && !carregando" class="empty-state">
        <div class="empty-icon">📋</div>
        <p>Escolha uma seleção acima para ver os jogadores</p>
      </div>
    </transition>

  </div>
</template>

<style>
/* ─── Imports de fontes ─── */
@import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Rajdhani:wght@400;600;700&display=swap');

/* ─── Reset e variáveis globais ─── */
*, *::before, *::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --gold:       #f0c040;
  --gold-light: #ffe07a;
  --gold-dark:  #b8860b;
  --azul-deep:  #050d1a;
  --azul-mid:   #0a1f3c;
  --azul-card:  rgba(10, 30, 65, 0.7);
  --verde:      #00e676;
  --branco:     #ffffff;
  --cinza:      rgba(255,255,255,0.55);
  --radius-lg:  20px;
  --radius-md:  14px;
  --radius-sm:  8px;
  --shadow-card: 0 20px 50px rgba(0,0,0,0.5);
  --font-display: 'Bebas Neue', sans-serif;
  --font-body:    'Rajdhani', sans-serif;
  --transition:   0.35s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

html, body {
  min-height: 100vh;
  background: var(--azul-deep);
  font-family: var(--font-body);
  overflow-x: hidden;
}

/* ─── Fundo com orbs animadas ─── */
.bg-orbs {
  position: fixed;
  inset: 0;
  z-index: 0;
  pointer-events: none;
  overflow: hidden;
  background: radial-gradient(ellipse at 20% 10%, #0d2b5e 0%, transparent 60%),
              radial-gradient(ellipse at 80% 80%, #051630 0%, #050d1a 100%);
}

.orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.25;
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
  0%, 100% { transform: translateY(0px) scale(1); }
  50%       { transform: translateY(-40px) scale(1.08); }
}

/* ─── Container principal ─── */
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
  font-family: var(--font-body);
  font-weight: 700;
  font-size: 11px;
  letter-spacing: 4px;
  color: var(--gold);
  border: 1px solid var(--gold-dark);
  padding: 5px 18px;
  border-radius: 30px;
  margin-bottom: 18px;
  background: rgba(240,192,64,0.08);
  text-transform: uppercase;
}

.titulo {
  font-family: var(--font-display);
  font-size: clamp(52px, 9vw, 100px);
  color: var(--branco);
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
  color: var(--gold);
  -webkit-text-stroke: 1px var(--gold-dark);
}

.subtitulo {
  margin-top: 14px;
  color: var(--cinza);
  font-size: 17px;
  font-weight: 400;
  letter-spacing: 1px;
}

/* ─── Select personalizado ─── */
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
  border-radius: var(--radius-lg);
  border: 1px solid rgba(240,192,64,0.35);
  background: rgba(10, 25, 55, 0.85);
  backdrop-filter: blur(16px);
  color: var(--branco);
  font-family: var(--font-body);
  font-size: 17px;
  font-weight: 600;
  cursor: pointer;
  appearance: none;
  -webkit-appearance: none;
  outline: none;
  transition: border-color var(--transition), box-shadow var(--transition);
}

.select-pais:focus {
  border-color: var(--gold);
  box-shadow: 0 0 0 3px rgba(240,192,64,0.2), 0 8px 30px rgba(0,0,0,0.4);
}

.select-pais option {
  background: #0d1f40;
  color: var(--branco);
}

.select-arrow {
  position: absolute;
  right: 18px;
  color: var(--gold);
  font-size: 20px;
  pointer-events: none;
}

/* ─── Banner da seleção escolhida ─── */
.selecionada-banner {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin-bottom: 40px;
  padding: 14px 28px;
  border-radius: var(--radius-md);
  background: linear-gradient(135deg, rgba(240,192,64,0.12), rgba(240,192,64,0.04));
  border: 1px solid rgba(240,192,64,0.25);
  max-width: 500px;
  margin-left: auto;
  margin-right: auto;
  flex-wrap: wrap;
}

.selecionada-label {
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 3px;
  color: var(--gold-dark);
}

.selecionada-nome {
  font-family: var(--font-display);
  font-size: 28px;
  color: var(--gold);
  letter-spacing: 2px;
}

.selecionada-count {
  font-size: 13px;
  color: var(--cinza);
  background: rgba(255,255,255,0.07);
  padding: 3px 10px;
  border-radius: 20px;
}

/* ─── Spinner de loading ─── */
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
  border: 3px solid rgba(240,192,64,0.15);
  border-top-color: var(--gold);
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
  color: var(--cinza);
  font-size: 16px;
  letter-spacing: 2px;
  text-transform: uppercase;
  font-weight: 600;
}

/* ─── Estado vazio ─── */
.empty-state {
  text-align: center;
  padding: 80px 20px;
  color: rgba(255,255,255,0.25);
  font-size: 18px;
  font-weight: 600;
  letter-spacing: 0.5px;
}

.empty-icon {
  font-size: 60px;
  margin-bottom: 20px;
  opacity: 0.4;
}

/* ─── Grid de cards ─── */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 24px;
}

/* ─── Card da figurinha ─── */
.card {
  position: relative;
  background: var(--azul-card);
  backdrop-filter: blur(18px);
  border-radius: var(--radius-lg);
  border: 1px solid rgba(255,255,255,0.1);
  padding: 28px 20px 22px;
  text-align: center;
  overflow: hidden;
  cursor: pointer;
  transition:
    transform var(--transition),
    box-shadow var(--transition),
    border-color var(--transition);
  animation: card-entrada 0.5s ease both;
  animation-delay: var(--delay);
}

@keyframes card-entrada {
  from {
    opacity: 0;
    transform: translateY(30px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

/* Faixa dourada no topo do card */
.card::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 3px;
  background: linear-gradient(90deg, transparent, var(--gold), transparent);
  opacity: 0;
  transition: opacity var(--transition);
}

.card:hover {
  transform: translateY(-12px) scale(1.02);
  box-shadow: 0 28px 60px rgba(0,0,0,0.6), 0 0 30px rgba(240,192,64,0.15);
  border-color: rgba(240,192,64,0.4);
}

.card:hover::before {
  opacity: 1;
}

/* Efeito holográfico no hover */
.card-holo {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    135deg,
    transparent 0%,
    rgba(255,255,255,0.04) 40%,
    rgba(240,192,64,0.06) 60%,
    transparent 100%
  );
  opacity: 0;
  transition: opacity var(--transition);
  pointer-events: none;
}

.card:hover .card-holo {
  opacity: 1;
}

/* Número da figurinha */
.card-numero {
  position: absolute;
  top: 12px;
  left: 14px;
  font-family: var(--font-display);
  font-size: 13px;
  color: var(--gold-dark);
  letter-spacing: 1px;
}

/* Foto do jogador */
.card-foto-wrapper {
  position: relative;
  width: 110px;
  height: 110px;
  margin: 0 auto 18px;
}

.card-foto {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 50%;
  border: 3px solid rgba(240,192,64,0.5);
  transition: border-color var(--transition), box-shadow var(--transition);
  position: relative;
  z-index: 1;
  display: block;
}

.card:hover .card-foto {
  border-color: var(--gold);
  box-shadow: 0 0 20px rgba(240,192,64,0.4);
}

/* Brilho atrás da foto */
.card-foto-brilho {
  position: absolute;
  inset: -6px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(240,192,64,0.2), transparent 70%);
  opacity: 0;
  transition: opacity var(--transition);
}

.card:hover .card-foto-brilho {
  opacity: 1;
}

/* Info do jogador */
.card-info {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.card-nome {
  font-family: var(--font-display);
  font-size: 19px;
  color: var(--branco);
  letter-spacing: 1px;
  line-height: 1.15;
}

.card-posicao {
  display: inline-block;
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: var(--gold);
  background: rgba(240,192,64,0.1);
  border: 1px solid rgba(240,192,64,0.25);
  padding: 3px 10px;
  border-radius: 20px;
}

/* ─── Animações de transição (Vue) ─── */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease, transform 0.4s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}

.card-list-enter-active {
  transition: all 0.4s ease;
}

.card-list-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

/* ─── Responsividade ─── */
@media (max-width: 768px) {
  .container {
    padding: 28px 16px 60px;
  }

  .grid {
    grid-template-columns: repeat(auto-fill, minmax(155px, 1fr));
    gap: 16px;
  }

  .card {
    padding: 22px 14px 18px;
  }

  .card-foto-wrapper {
    width: 85px;
    height: 85px;
  }

  .card-nome {
    font-size: 15px;
  }

  .selecionada-banner {
    flex-direction: column;
    gap: 6px;
  }
}

@media (max-width: 480px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>