<script setup>
// ──────────────────────────────────────────
//  Props recebidas do componente pai (App.vue)
// ──────────────────────────────────────────
const props = defineProps({
  jogador: {
    type: Object,
    required: true
  },
  numero: {
    type: Number,
    required: true
  },
  indice: {
    type: Number,
    required: true
  },
  bandeiraSelecionada: {
    type: String,
    default: null
  },
  paisSelecionado: {
    type: String,
    default: ''
  }
})

// ──────────────────────────────────────────
//  Lógica de stats (gerada localmente a partir
//  do nome do jogador — sem chamada à API)
// ──────────────────────────────────────────
const gerarStats = (jogador) => {
  const seed = jogador.name.split('').reduce((a, c) => a + c.charCodeAt(0), 0)
  const rng  = (min, max, off = 0) => {
    const v = ((seed + off) * 2654435761) >>> 0
    return min + (v % (max - min + 1))
  }
  const pos   = (jogador.position || '').toLowerCase()
  const isGk  = pos.includes('goalkeeper')
  const isDef = pos.includes('defender') || pos.includes('back')
  const isMid = pos.includes('midfielder')

  if (isGk) return [
    { s: 'ELA', v: rng(60, 92, 1) }, { s: 'MAN', v: rng(62, 93, 2) },
    { s: 'CHU', v: rng(45, 78, 3) }, { s: 'REF', v: rng(65, 95, 4) },
    { s: 'VEL', v: rng(55, 85, 5) }, { s: 'POS', v: rng(68, 95, 6) },
  ]
  return [
    { s: 'RIT', v: isDef ? rng(55, 80, 1) : rng(65, 95, 1) },
    { s: 'FIN', v: isDef ? rng(40, 70, 2) : isMid ? rng(55, 85, 2) : rng(72, 96, 2) },
    { s: 'PAS', v: isMid ? rng(75, 95, 3) : rng(58, 88, 3) },
    { s: 'CON', v: rng(60, 92, 4) },
    { s: 'DEF', v: isDef ? rng(72, 95, 5) : rng(38, 72, 5) },
    { s: 'FÍS', v: rng(62, 93, 6) },
  ]
}

const corStat = (v) => v >= 85 ? '#00d48c' : v >= 72 ? '#f5c842' : '#ff6b6b'

const mediaGeral = (jogador) => {
  const stats = gerarStats(jogador)
  return Math.round(stats.reduce((a, s) => a + s.v, 0) / stats.length)
}

const iniciais = (nome) =>
  nome.split(' ').map(w => w[0]).slice(0, 2).join('').toUpperCase()
</script>

<template>
  <div
    class="sticker"
    :style="{ '--i': indice, '--media': mediaGeral(jogador) }"
  >
    <!-- Número da figurinha -->
    <div class="stk-num">{{ numero }}</div>

    <!-- Brilho no hover -->
    <div class="stk-glare"></div>

    <!-- Cabeçalho: bandeira + bola -->
    <div class="stk-head">
      <div class="stk-head-l">
        <img
          v-if="bandeiraSelecionada"
          :src="bandeiraSelecionada"
          class="stk-flag"
          alt="País"
        />
        <span class="stk-country">{{ paisSelecionado.slice(0, 3).toUpperCase() }}</span>
      </div>
      <span class="stk-ball">⚽</span>
    </div>

    <!-- Foto do jogador com fallback de iniciais -->
    <div class="stk-photo-wrap">
      <div class="stk-photo-bg"></div>
      <div class="stk-photo-fallback">{{ iniciais(jogador.name) }}</div>
      <img
        :src="jogador.photo"
        :alt="jogador.name"
        class="stk-photo"
        @error="e => { e.target.style.display = 'none' }"
      />
      <div class="stk-photo-grad"></div>
      <div class="stk-overall">
        <span class="so-val">{{ mediaGeral(jogador) }}</span>
        <span class="so-lbl">OVR</span>
      </div>
    </div>

    <!-- Nome e posição -->
    <div class="stk-identity">
      <div class="stk-name">{{ jogador.name }}</div>
      <div class="stk-pos">{{ jogador.position || 'Jogador' }}</div>
    </div>

    <!-- Atributos com barras -->
    <div class="stk-attrs">
      <div class="attr" v-for="st in gerarStats(jogador)" :key="st.s">
        <span class="attr-s">{{ st.s }}</span>
        <div class="attr-bar-bg">
          <div
            class="attr-bar-fill"
            :style="{ width: st.v + '%', background: corStat(st.v) }"
          ></div>
        </div>
        <span class="attr-v" :style="{ color: corStat(st.v) }">{{ st.v }}</span>
      </div>
    </div>

    <!-- Rodapé -->
    <div class="stk-foot">
      <span>FIFA</span>
      <span class="stk-foot-dot"></span>
      <span>COPA 2026</span>
      <span class="stk-foot-dot"></span>
      <span>OFFICIAL</span>
    </div>
  </div>
</template>

<style scoped>
/* ══════════════════════════════════════════
   FIGURINHA CARD — estilos com scoped para
   não vazar para outros componentes
══════════════════════════════════════════ */

.sticker {
  position: relative;
  background: var(--bg3);
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid var(--border);
  box-shadow: var(--sh-md);
  cursor: pointer;
  display: flex;
  flex-direction: column;

  animation: stk-in 0.45s cubic-bezier(0.34, 1.56, 0.64, 1) both;
  animation-delay: calc(var(--i) * 0.05s);

  transition:
    transform 0.28s cubic-bezier(0.34, 1.56, 0.64, 1),
    box-shadow 0.28s ease,
    border-color 0.28s ease;
}

@keyframes stk-in {
  from { opacity: 0; transform: translateY(24px) scale(0.88); }
  to   { opacity: 1; transform: translateY(0)    scale(1); }
}

.sticker:hover {
  transform: translateY(-8px) scale(1.04) rotate(0.4deg);
  box-shadow:
    0 24px 48px rgba(0, 0, 0, 0.7),
    0 0 0 1px rgba(37, 99, 235, 0.45),
    0 0 30px rgba(37, 99, 235, 0.15);
  border-color: rgba(37, 99, 235, 0.5);
  z-index: 5;
}

/* Número flutuante */
.stk-num {
  position: absolute;
  top: -7px; left: -7px;
  z-index: 10;
  width: 24px; height: 24px;
  background: linear-gradient(135deg, var(--blue), var(--blue-dk));
  border: 2px solid var(--bg);
  border-radius: 50%;
  font-size: 9px; font-weight: 800;
  color: #fff;
  display: flex; align-items: center; justify-content: center;
  font-family: 'Rajdhani', sans-serif;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.6);
}

/* Glare de hover */
.stk-glare {
  position: absolute; inset: 0;
  background: linear-gradient(135deg, rgba(255,255,255,0.09) 0%, transparent 55%);
  border-radius: 12px;
  z-index: 4;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.3s;
}
.sticker:hover .stk-glare { opacity: 1; }

/* Cabeçalho */
.stk-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 7px 9px 6px;
  background: linear-gradient(90deg, var(--blue-dk) 0%, var(--blue) 55%, #8b0d1f 100%);
  flex-shrink: 0;
}
.stk-head-l { display: flex; align-items: center; gap: 5px; }
.stk-flag   { width: 22px; height: 15px; object-fit: cover; border-radius: 2px; box-shadow: 0 1px 4px rgba(0,0,0,0.5); }
.stk-country { font-size: 8px; font-weight: 800; letter-spacing: 0.5px; color: rgba(255,255,255,0.7); }
.stk-ball   { font-size: 12px; opacity: 0.75; }

/* Área da foto */
.stk-photo-wrap {
  position: relative;
  width: 100%; aspect-ratio: 1;
  overflow: hidden;
  background: linear-gradient(180deg, #0d1628 0%, #080c14 100%);
  flex-shrink: 0;
}
.stk-photo-bg {
  position: absolute; inset: 0;
  background: radial-gradient(ellipse 80% 60% at 50% 20%, rgba(0,61,165,0.22), transparent 70%);
}
.stk-photo-fallback {
  position: absolute; inset: 0;
  z-index: 1;
  display: flex; align-items: center; justify-content: center;
  font-family: 'Bebas Neue', sans-serif;
  font-size: 38px;
  color: rgba(255, 255, 255, 0.25);
  letter-spacing: 2px;
}
.stk-photo {
  width: 100%; height: 100%;
  object-fit: cover; display: block;
  position: relative; z-index: 2;
  transition: transform 0.35s cubic-bezier(0.34, 1.56, 0.64, 1);
}
.sticker:hover .stk-photo { transform: scale(1.1); }
.stk-photo-grad {
  position: absolute; bottom: 0; left: 0; right: 0;
  height: 45%;
  background: linear-gradient(transparent, var(--bg3));
  z-index: 3;
}

/* OVR badge */
.stk-overall {
  position: absolute; bottom: 8px; right: 8px; z-index: 4;
  display: flex; flex-direction: column; align-items: center;
  background: rgba(0, 0, 0, 0.75);
  backdrop-filter: blur(6px);
  border: 1px solid rgba(255,255,255,0.12);
  border-radius: 7px;
  padding: 4px 8px; min-width: 36px;
}
.so-val { font-family: 'Bebas Neue', sans-serif; font-size: 18px; color: #fff; line-height: 1; }
.so-lbl { font-size: 7px; color: var(--gold); font-weight: 800; letter-spacing: 0.5px; }

/* Nome e posição */
.stk-identity { padding: 9px 9px 4px; text-align: center; flex-shrink: 0; }
.stk-name {
  font-family: 'Rajdhani', sans-serif;
  font-size: 12px; font-weight: 700;
  color: var(--text);
  text-transform: uppercase; letter-spacing: 0.3px;
  white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
  line-height: 1.2;
}
.stk-pos {
  font-size: 8px; font-weight: 800; letter-spacing: 1px;
  color: #fff; text-transform: uppercase;
  background: linear-gradient(90deg, var(--blue), var(--blue-lt));
  padding: 2px 9px; border-radius: 4px;
  display: inline-block; margin-top: 4px;
}

/* Atributos */
.stk-attrs {
  display: flex; flex-direction: column; gap: 4px;
  padding: 6px 9px 4px; flex: 1;
}
.attr {
  display: grid;
  grid-template-columns: 22px 1fr 20px;
  align-items: center; gap: 5px;
}
.attr-s { font-size: 7px; font-weight: 800; letter-spacing: 0.5px; color: var(--text-mid); text-transform: uppercase; }
.attr-bar-bg  { height: 3px; background: rgba(255,255,255,0.07); border-radius: 3px; overflow: hidden; }
.attr-bar-fill { height: 100%; border-radius: 3px; transition: width 0.9s cubic-bezier(0.16, 1, 0.3, 1); }
.attr-v { font-family: 'Rajdhani', sans-serif; font-size: 10px; font-weight: 800; text-align: right; }

/* Rodapé */
.stk-foot {
  display: flex; align-items: center; justify-content: center; gap: 5px;
  padding: 5px 8px 8px;
  border-top: 1px solid var(--border);
  background: rgba(255, 255, 255, 0.02);
  flex-shrink: 0;
}
.stk-foot span { font-size: 7px; font-weight: 800; letter-spacing: 1px; color: var(--text-dim); }
.stk-foot-dot  { width: 2px; height: 2px; border-radius: 50%; background: var(--text-dim); }
</style>