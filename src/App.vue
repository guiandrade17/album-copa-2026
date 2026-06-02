<script setup>
import { ref, onMounted, computed } from 'vue'

const token = '5dc0faddd67d4bd655e074569c6111e8'

const paises      = ref([])
const paisSelecionado = ref('')
const jogadores   = ref([])
const carregando  = ref(false)
const paginaAtual = ref(1)
const porPagina   = 12
const navAtivo    = ref('album')

/* ── Computados ── */
const bandeiraSelecionada = computed(() =>
  paises.value.find(p => p.name === paisSelecionado.value)?.flag ?? null
)
const totalPaginas = computed(() => Math.ceil(jogadores.value.length / porPagina))
const jogadoresPagina = computed(() => {
  const inicio = (paginaAtual.value - 1) * porPagina
  return jogadores.value.slice(inicio, inicio + porPagina)
})

/* ── Flags de destaque para empty state ── */
const flagsDestaque = [
  { name: 'Brazil',    code: 'br' },
  { name: 'Germany',   code: 'de' },
  { name: 'France',    code: 'fr' },
  { name: 'Argentina', code: 'ar' },
  { name: 'Spain',     code: 'es' },
  { name: 'England',   code: 'gb-eng' },
  { name: 'Portugal',  code: 'pt' },
  { name: 'Italy',     code: 'it' },
]

/* ── Stats simuladas ── */
const gerarStats = (jogador) => {
  const seed   = jogador.name.split('').reduce((a, c) => a + c.charCodeAt(0), 0)
  const rng    = (min, max, off = 0) => {
    const v = ((seed + off) * 2654435761) >>> 0
    return min + (v % (max - min + 1))
  }
  const pos     = (jogador.position || '').toLowerCase()
  const isGk    = pos.includes('goalkeeper')
  const isDef   = pos.includes('defender') || pos.includes('back')
  const isMid   = pos.includes('midfielder')

  if (isGk) return [
    { s: 'ELA', v: rng(60, 92, 1) }, { s: 'MAN', v: rng(62, 93, 2) },
    { s: 'CHU', v: rng(45, 78, 3) }, { s: 'REF', v: rng(65, 95, 4) },
    { s: 'VEL', v: rng(55, 85, 5) }, { s: 'POS', v: rng(68, 95, 6) },
  ]
  return [
    { s: 'RIT', v: isDef ? rng(55,80,1) : rng(65,95,1) },
    { s: 'FIN', v: isDef ? rng(40,70,2) : isMid ? rng(55,85,2) : rng(72,96,2) },
    { s: 'PAS', v: isMid ? rng(75,95,3) : rng(58,88,3) },
    { s: 'CON', v: rng(60,92,4) },
    { s: 'DEF', v: isDef ? rng(72,95,5) : rng(38,72,5) },
    { s: 'FÍS', v: rng(62,93,6) },
  ]
}

const corStat = (v) => v >= 85 ? '#00d48c' : v >= 72 ? '#f5c842' : '#ff6b6b'

const mediaGeral = (jogador) => {
  const stats = gerarStats(jogador)
  return Math.round(stats.reduce((a, s) => a + s.v, 0) / stats.length)
}

/* ── API ── */
const carregarPaises = async () => {
  try {
    const r = await fetch('https://v3.football.api-sports.io/teams/countries', {
      headers: { 'x-apisports-key': token }
    })
    const d = await r.json()
    paises.value = d.response
  } catch (e) { console.error(e) }
}

const carregarFigurinhas = async () => {
  if (!paisSelecionado.value) return
  carregando.value  = true
  jogadores.value   = []
  paginaAtual.value = 1
  try {
    const rTime = await fetch(
      `https://v3.football.api-sports.io/teams?name=${paisSelecionado.value}`,
      { headers: { 'x-apisports-key': token } }
    )
    const dTime  = await rTime.json()
    const teamId = dTime.response[0].team.id

    const rElenco = await fetch(
      `https://v3.football.api-sports.io/players/squads?team=${teamId}`,
      { headers: { 'x-apisports-key': token } }
    )
    const dElenco  = await rElenco.json()
    jogadores.value = dElenco.response[0].players
  } catch (e) { console.error(e) }
  finally { carregando.value = false }
}

const irParaPagina   = (n) => { paginaAtual.value = n }
const proximaPagina  = () => { if (paginaAtual.value < totalPaginas.value) paginaAtual.value++ }
const paginaAnterior = () => { if (paginaAtual.value > 1) paginaAtual.value-- }

onMounted(carregarPaises)
</script>

<template>
<div class="root">

  <!-- ════════════ SIDEBAR ════════════ -->
  <aside class="sidebar">

    <div class="sb-logo">
      <div class="sb-emblem">⚽</div>
      <div class="sb-brand">
        <span class="sb-name">FutÁlbum</span>
        <span class="sb-sub">COPA 2026</span>
      </div>
    </div>

    <nav class="sb-nav">
      <button class="sb-link" :class="{active: navAtivo==='album'}"   @click="navAtivo='album'">
        <svg class="sb-ico" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 19.5A2.5 2.5 0 016.5 17H20"/><path d="M6.5 2H20v20H6.5A2.5 2.5 0 014 19.5v-15A2.5 2.5 0 016.5 2z"/></svg>
        <span>Álbum</span>
        <span class="sb-badge">NOVO</span>
      </button>
      <button class="sb-link" :class="{active: navAtivo==='mercado'}" @click="navAtivo='mercado'">
        <svg class="sb-ico" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="9" cy="21" r="1"/><circle cx="20" cy="21" r="1"/><path d="M1 1h4l2.68 13.39A2 2 0 009.68 16h9.72a2 2 0 001.98-1.61L23 6H6"/></svg>
        <span>Mercado</span>
      </button>
      <button class="sb-link" :class="{active: navAtivo==='perfil'}"  @click="navAtivo='perfil'">
        <svg class="sb-ico" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
        <span>Perfil</span>
      </button>
    </nav>

    <div class="sb-divider"></div>

    <div class="sb-select-block">
      <div class="sb-select-label">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" width="11" height="11"><circle cx="12" cy="12" r="10"/><path d="m8 12 2.5 2.5L16 9"/></svg>
        SELECIONE A SELEÇÃO
      </div>
      <div class="sb-select-wrap">
        <select v-model="paisSelecionado" @change="carregarFigurinhas" class="sb-select">
          <option value="">Escolha um país...</option>
          <option v-for="p in paises" :key="p.name" :value="p.name">{{ p.name }}</option>
        </select>
        <svg class="sb-chevron" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="m6 9 6 6 6-6"/></svg>
      </div>
    </div>

    <!-- Team card sidebar -->
    <transition name="slide-up">
      <div v-if="paisSelecionado && !carregando" class="sb-team-card">
        <div class="stc-header">
          <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="stc-flag" alt="flag"/>
          <div class="stc-info">
            <span class="stc-name">{{ paisSelecionado }}</span>
            <span class="stc-label">Seleção Nacional</span>
          </div>
        </div>
        <div class="stc-stats">
          <div class="stcs">
            <span class="stcs-v">{{ jogadores.length }}</span>
            <span class="stcs-l">Jogadores</span>
          </div>
          <div class="stcs-div"></div>
          <div class="stcs">
            <span class="stcs-v">{{ totalPaginas }}</span>
            <span class="stcs-l">Páginas</span>
          </div>
          <div class="stcs-div"></div>
          <div class="stcs">
            <span class="stcs-v">26</span>
            <span class="stcs-l">Edição</span>
          </div>
        </div>
      </div>
    </transition>

    <!-- Copa badge -->
    <div class="sb-copa">
      <div class="sb-copa-stripe-top">
        <span style="background:var(--red)"></span>
        <span style="background:var(--blue)"></span>
        <span style="background:var(--green)"></span>
      </div>
      <div class="sb-copa-body">
        <span class="sb-copa-trophy">🏆</span>
        <div>
          <div class="sb-copa-title">FIFA WORLD CUP</div>
          <div class="sb-copa-year">2026</div>
          <div class="sb-copa-hosts">USA · CAN · MEX</div>
        </div>
      </div>
    </div>

  </aside>

  <!-- ════════════ MAIN ════════════ -->
  <main class="main">

    <!-- Hero topo -->
    <header class="hero">
      <div class="hero-mesh"></div>
      <div class="hero-grid"></div>
      <div class="hero-inner">
        <div class="hero-left">
          <div class="hero-eyebrow">⚽ COPA DO MUNDO FIFA 2026</div>
          <h1 class="hero-title">ÁLBUM DE<br><span class="hero-accent">FIGURINHAS</span></h1>
          <p class="hero-sub">Colecione os melhores jogadores do mundo</p>
        </div>
        <div class="hero-right">
          <div class="hero-year-bg">2026</div>
          <div class="hero-hosts-row">
            <span>🇺🇸 USA</span>
            <span>🇨🇦 CAN</span>
            <span>🇲🇽 MEX</span>
          </div>
        </div>
      </div>
      <div class="hero-bar"><span></span><span></span><span></span></div>
    </header>

    <!-- ── EMPTY STATE ── -->
    <section v-if="!paisSelecionado && !carregando" class="empty-wrap">

      <!-- Banner principal -->
      <div class="empty-banner">
        <div class="eb-mesh"></div>
        <div class="eb-content">
          <div class="eb-left">
            <div class="eb-eyebrow">🏆 FIFA WORLD CUP 2026</div>
            <h2 class="eb-title">Bem-vindo ao<br><span>FutÁlbum</span></h2>
            <p class="eb-desc">Explore os elencos de todas as seleções do mundo. Selecione um país no menu lateral para começar a colecionar.</p>
            <div class="eb-arrow">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="m19 12-7-7-7 7"/><path d="M12 5v14"/></svg>
              Use o menu lateral
            </div>
          </div>
          <div class="eb-right">
            <div class="eb-trophy-wrap">
              <div class="eb-trophy-ring r1"></div>
              <div class="eb-trophy-ring r2"></div>
              <span class="eb-trophy">🏆</span>
            </div>
          </div>
        </div>
        <div class="eb-bar"><span></span><span></span><span></span></div>
      </div>

      <!-- Stats do álbum -->
      <div class="empty-stats">
        <div class="es-card" v-for="(item, i) in [
          { icon:'🌍', val:'211',   lbl:'Seleções',    col:'var(--blue-lt)' },
          { icon:'👕', val:'26+',   lbl:'Jogadores/time', col:'var(--green)' },
          { icon:'🏆', val:'2026',  lbl:'Edição Copa', col:'var(--gold)' },
          { icon:'⚽', val:'48',   lbl:'Times na Copa', col:'var(--red)' },
        ]" :key="i">
          <div class="es-icon">{{ item.icon }}</div>
          <div class="es-val" :style="{color: item.col}">{{ item.val }}</div>
          <div class="es-lbl">{{ item.lbl }}</div>
        </div>
      </div>

      <!-- Seleções em destaque -->
      <div class="empty-featured">
        <div class="ef-header">
          <h3 class="ef-title">Seleções em destaque</h3>
          <span class="ef-sub">Clique no menu lateral para selecionar</span>
        </div>
        <div class="ef-flags">
          <div class="ef-flag-card" v-for="f in flagsDestaque" :key="f.code">
            <div class="efc-img-wrap">
              <img :src="`https://flagcdn.com/w80/${f.code}.png`" :alt="f.name" class="efc-img"/>
              <div class="efc-shine"></div>
            </div>
            <span class="efc-name">{{ f.name }}</span>
          </div>
        </div>
      </div>

      <!-- Como funciona -->
      <div class="empty-howto">
        <h3 class="ht-title">Como funciona</h3>
        <div class="ht-steps">
          <div class="ht-step" v-for="(s,i) in [
            { n:'01', t:'Escolha a Seleção', d:'Use o menu lateral para selecionar o país que deseja explorar.' },
            { n:'02', t:'Veja os Jogadores', d:'Explore as figurinhas dos jogadores com atributos detalhados.' },
            { n:'03', t:'Navegue pelo Álbum', d:'Use a paginação para ver todos os jogadores do elenco.' },
          ]" :key="i">
            <div class="hts-num">{{ s.n }}</div>
            <div class="hts-body">
              <div class="hts-title">{{ s.t }}</div>
              <div class="hts-desc">{{ s.d }}</div>
            </div>
          </div>
        </div>
      </div>

    </section>

    <!-- ── LOADING ── -->
    <div v-else-if="carregando" class="loading">
      <div class="ld-card">
        <div class="ld-spinner">
          <div></div><div></div><div></div><div></div>
        </div>
        <p class="ld-text">Carregando figurinhas...</p>
        <span class="ld-sub">Buscando elenco de <strong>{{ paisSelecionado }}</strong></span>
      </div>
    </div>

    <!-- ── ALBUM ── -->
    <div v-else class="album">

      <!-- Topo da seleção -->
      <div class="album-hero">
        <div class="ah-bg"></div>
        <div class="ah-pattern"></div>

        <div class="ah-content">
          <!-- Bandeira em destaque -->
          <div class="ah-flag-block">
            <div class="ah-flag-frame">
              <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="ah-flag" alt="Bandeira"/>
              <div class="ah-flag-glow"></div>
            </div>
          </div>

          <!-- Info da seleção -->
          <div class="ah-info">
            <span class="ah-label">SELEÇÃO NACIONAL · COPA 2026</span>
            <h2 class="ah-name">{{ paisSelecionado.toUpperCase() }}</h2>
            <div class="ah-tags">
              <span class="ah-tag">
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87"/><path d="M16 3.13a4 4 0 010 7.75"/></svg>
                {{ jogadores.length }} jogadores
              </span>
              <span class="ah-tag gold">🏆 Copa do Mundo 2026</span>
              <span class="ah-tag green">
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M22 11.08V12a10 10 0 11-5.93-9.14"/><path d="m22 4-10 10.01-3-3"/></svg>
                Elenco Oficial
              </span>
            </div>
          </div>

          <!-- Anel de progresso -->
          <div class="ah-ring-wrap">
            <svg class="ah-ring-svg" viewBox="0 0 100 100">
              <circle cx="50" cy="50" r="42" fill="none" stroke="rgba(255,255,255,0.08)" stroke-width="7"/>
              <circle cx="50" cy="50" r="42" fill="none" stroke="url(#rg)" stroke-width="7"
                stroke-dasharray="263.9" stroke-dashoffset="0" stroke-linecap="round"/>
              <defs>
                <linearGradient id="rg" x1="0" y1="0" x2="1" y2="1">
                  <stop offset="0%" stop-color="#00d48c"/>
                  <stop offset="100%" stop-color="#0061ff"/>
                </linearGradient>
              </defs>
            </svg>
            <div class="ah-ring-center">
              <span class="ah-ring-n">{{ jogadores.length }}</span>
              <span class="ah-ring-l">jogadores</span>
            </div>
          </div>

          <!-- Mini-stats da seleção -->
          <div class="ah-mini-stats">
            <div class="ams-item" v-for="(s,i) in [
              { v: totalPaginas, l:'Páginas' },
              { v: Math.min(porPagina, jogadores.length), l:'Por página' },
              { v: paginaAtual, l:'Página atual' },
            ]" :key="i">
              <span class="ams-v">{{ s.v }}</span>
              <span class="ams-l">{{ s.l }}</span>
            </div>
          </div>
        </div>

        <div class="ah-bar"><span></span><span></span><span></span></div>
      </div>

      <!-- Cabeçalho da seção de cards -->
      <div class="album-section-bar">
        <div class="asb-left">
          <span class="asb-dot"></span>
          <span class="asb-title">ELENCO COMPLETO</span>
          <span class="asb-count">{{ jogadores.length }} figurinhas</span>
        </div>
        <span class="asb-page">Página {{ paginaAtual }} de {{ totalPaginas }}</span>
      </div>

      <!-- ── GRID DE FIGURINHAS ── -->
      <div class="stickers-grid">
        <div
          class="sticker"
          v-for="(jogador, index) in jogadoresPagina"
          :key="jogador.id"
          :style="{ '--i': index, '--media': mediaGeral(jogador) }"
        >
          <!-- Número flutuante -->
          <div class="stk-num">{{ (paginaAtual - 1) * porPagina + index + 1 }}</div>

          <!-- Camada de brilho no hover -->
          <div class="stk-glare"></div>

          <!-- Cabeçalho da figurinha -->
          <div class="stk-head">
            <div class="stk-head-l">
              <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="stk-flag" alt="País"/>
              <span class="stk-country">{{ paisSelecionado.slice(0,3).toUpperCase() }}</span>
            </div>
            <span class="stk-ball">⚽</span>
          </div>

          <!-- Foto do jogador -->
          <div class="stk-photo-wrap">
            <div class="stk-photo-bg"></div>
            <img
              :src="jogador.photo"
              :alt="jogador.name"
              class="stk-photo"
              @error="e => e.target.style.opacity='0'"
            />
            <div class="stk-photo-grad"></div>

            <!-- Média geral sobreposta na foto -->
            <div class="stk-overall">
              <span class="so-val">{{ mediaGeral(jogador) }}</span>
              <span class="so-lbl">OVR</span>
            </div>
          </div>

          <!-- Nome + posição -->
          <div class="stk-identity">
            <div class="stk-name">{{ jogador.name }}</div>
            <div class="stk-pos">{{ jogador.position || 'Jogador' }}</div>
          </div>

          <!-- Atributos -->
          <div class="stk-attrs">
            <div class="attr" v-for="st in gerarStats(jogador)" :key="st.s">
              <span class="attr-s">{{ st.s }}</span>
              <div class="attr-bar-bg">
                <div class="attr-bar-fill" :style="{ width: st.v + '%', background: corStat(st.v) }"></div>
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
      </div>

      <!-- ── PAGINAÇÃO ── -->
      <div class="pagination">
        <button class="pg-btn pg-prev" @click="paginaAnterior" :disabled="paginaAtual===1">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="m15 18-6-6 6-6"/></svg>
          Anterior
        </button>

        <div class="pg-dots">
          <template v-for="n in totalPaginas" :key="n">
            <button
              v-if="n===1 || n===totalPaginas || Math.abs(n-paginaAtual)<=1"
              class="pg-dot"
              :class="{active: n===paginaAtual, near: Math.abs(n-paginaAtual)===1}"
              @click="irParaPagina(n)"
            >{{ n }}</button>
            <span v-else-if="n===2 && paginaAtual>3" class="pg-ellipsis">…</span>
            <span v-else-if="n===totalPaginas-1 && paginaAtual<totalPaginas-2" class="pg-ellipsis">…</span>
          </template>
        </div>

        <button class="pg-btn pg-next" @click="proximaPagina" :disabled="paginaAtual===totalPaginas">
          Próxima
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="m9 18 6-6-6-6"/></svg>
        </button>
      </div>

    </div>
  </main>
</div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Rajdhani:wght@400;500;600;700&family=Syne:wght@400;500;600;700;800&display=swap');

*, *::before, *::after { margin:0; padding:0; box-sizing:border-box; }

:root {
  --red:     #C8102E;
  --red-dk:  #9a0c23;
  --blue:    #003DA5;
  --blue-dk: #002070;
  --blue-lt: #2563eb;
  --green:   #00a650;
  --green-dk:#007a3d;
  --gold:    #f5c842;

  --bg:      #0b0f18;
  --bg2:     #101520;
  --bg3:     #161d2a;
  --bg4:     #1c2537;
  --border:  rgba(255,255,255,0.07);
  --bdr2:    rgba(255,255,255,0.12);
  --text:    #e8edf5;
  --text-mid:#6b7a99;
  --text-dim:#344055;

  --sh-sm: 0 2px 8px rgba(0,0,0,.5);
  --sh-md: 0 8px 28px rgba(0,0,0,.55);
  --sh-lg: 0 16px 56px rgba(0,0,0,.65);
  --r: 12px;

  --sidebar-w: 260px;
}

html, body {
  min-height: 100vh;
  background: var(--bg);
  font-family: 'Syne', sans-serif;
  color: var(--text);
  -webkit-font-smoothing: antialiased;
}

/* ══════════ LAYOUT ROOT ══════════ */
.root {
  display: flex;
  min-height: 100vh;
  width: 100%;
}

/* ══════════ SIDEBAR ══════════ */
.sidebar {
  width: var(--sidebar-w);
  min-width: var(--sidebar-w);
  background: #080c13;
  border-right: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  gap: 0;
  position: sticky;
  top: 0;
  height: 100vh;
  overflow-y: auto;
  scrollbar-width: none;
  flex-shrink: 0;
}
.sidebar::-webkit-scrollbar { display:none; }

/* Logo */
.sb-logo {
  display: flex;
  align-items: center;
  gap: 11px;
  padding: 20px 18px 18px;
  border-bottom: 1px solid var(--border);
  flex-shrink: 0;
}
.sb-emblem {
  width: 40px; height: 40px;
  background: linear-gradient(135deg, var(--blue), var(--red));
  border-radius: 10px;
  display: flex; align-items: center; justify-content: center;
  font-size: 20px;
  box-shadow: 0 4px 14px rgba(0,61,165,.4);
  flex-shrink: 0;
}
.sb-brand { display:flex; flex-direction:column; gap:1px; }
.sb-name {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 21px; letter-spacing: 1.5px;
  color: #fff; line-height: 1;
}
.sb-sub {
  font-size: 9px; font-weight: 700;
  letter-spacing: 2px; color: var(--gold);
  text-transform: uppercase;
}

/* Nav */
.sb-nav {
  display: flex; flex-direction: column; gap: 2px;
  padding: 14px 10px;
}
.sb-link {
  display: flex; align-items: center; gap: 9px;
  width: 100%; padding: 10px 12px;
  border: none; border-radius: 9px;
  background: transparent;
  color: var(--text-mid);
  font-family: 'Syne', sans-serif;
  font-size: 14px; font-weight: 600;
  cursor: pointer;
  transition: all .2s;
  text-align: left;
}
.sb-link:hover { background: rgba(255,255,255,.05); color: var(--text); }
.sb-link.active {
  background: linear-gradient(135deg, rgba(0,61,165,.3), rgba(200,16,46,.2));
  color: #fff;
  border: 1px solid rgba(37,99,235,.35);
}
.sb-link.active .sb-ico { color: var(--blue-lt); }
.sb-ico { width:18px; height:18px; flex-shrink:0; opacity:.75; }
.sb-link.active .sb-ico { opacity:1; }
.sb-badge {
  margin-left: auto; font-size: 8px; font-weight: 700;
  letter-spacing: .5px; background: var(--green);
  color: #fff; padding: 2px 7px; border-radius: 20px;
}

/* Divider */
.sb-divider { height:1px; background:var(--border); margin:0 10px; }

/* Select */
.sb-select-block { padding: 14px 10px 10px; }
.sb-select-label {
  display: flex; align-items: center; gap:6px;
  font-size: 9px; font-weight:700; letter-spacing:1.5px;
  color: var(--text-dim); text-transform:uppercase; margin-bottom:8px;
}
.sb-select-wrap { position:relative; }
.sb-select {
  width: 100%; padding: 10px 34px 10px 12px;
  border-radius: 8px; border: 1px solid var(--border);
  background: var(--bg3); color: var(--text);
  font-family: 'Syne', sans-serif; font-size:13px;
  cursor: pointer; appearance:none; -webkit-appearance:none;
  outline: none; transition: border-color .2s, box-shadow .2s;
}
.sb-select:focus {
  border-color: var(--blue-lt);
  box-shadow: 0 0 0 3px rgba(37,99,235,.2);
}
.sb-select option { background: #1a2230; }
.sb-chevron {
  position:absolute; right:10px; top:50%;
  transform:translateY(-50%);
  color:var(--text-mid); pointer-events:none; width:14px; height:14px;
}

/* Team card sidebar */
.sb-team-card {
  margin: 0 10px;
  background: var(--bg3);
  border: 1px solid var(--border);
  border-radius: 10px; overflow:hidden;
}
.stc-header {
  display:flex; align-items:center; gap:11px;
  padding: 12px 13px;
  background: linear-gradient(135deg, rgba(0,61,165,.2), rgba(200,16,46,.15));
  border-bottom: 1px solid var(--border);
}
.stc-flag { width:36px; height:24px; object-fit:cover; border-radius:4px; box-shadow:var(--sh-sm); }
.stc-info { display:flex; flex-direction:column; gap:2px; }
.stc-name {
  font-family:'Rajdhani',sans-serif; font-size:15px; font-weight:700;
  color:#fff; letter-spacing:.5px;
}
.stc-label { font-size:10px; color:var(--text-mid); }
.stc-stats { display:flex; align-items:center; padding:10px 13px; }
.stcs { flex:1; display:flex; flex-direction:column; align-items:center; gap:2px; }
.stcs-v {
  font-family:'Bebas Neue',sans-serif; font-size:22px;
  color:#fff; line-height:1;
}
.stcs-l { font-size:9px; color:var(--text-mid); text-transform:uppercase; letter-spacing:.5px; }
.stcs-div { width:1px; height:30px; background:var(--border); }

/* Copa badge */
.sb-copa {
  margin: auto 10px 14px;
  background: var(--bg3);
  border: 1px solid var(--border);
  border-radius: 10px; overflow:hidden;
}
.sb-copa-stripe-top { display:flex; height:3px; }
.sb-copa-stripe-top span { flex:1; }
.sb-copa-body {
  display:flex; align-items:center; gap:12px;
  padding:14px 13px;
}
.sb-copa-trophy { font-size:30px; filter:drop-shadow(0 2px 6px rgba(0,0,0,.5)); }
.sb-copa-title { font-size:9px; font-weight:700; letter-spacing:1.5px; color:var(--text-mid); }
.sb-copa-year {
  font-family:'Bebas Neue',sans-serif; font-size:26px;
  color:#fff; letter-spacing:2px; line-height:1;
}
.sb-copa-hosts { font-size:10px; color:var(--gold); font-weight:700; }

/* Slide-up transition */
.slide-up-enter-active { transition: all .3s cubic-bezier(.34,1.56,.64,1); }
.slide-up-enter-from   { opacity:0; transform:translateY(12px); }
.slide-up-leave-active { transition: all .2s ease; }
.slide-up-leave-to     { opacity:0; transform:translateY(8px); }

/* ══════════ MAIN ══════════ */
.main {
  flex: 1;
  min-width: 0;
  overflow-y: auto;
  background: var(--bg);
}

/* ══════════ HERO HEADER ══════════ */
.hero {
  position:relative; overflow:hidden;
}
.hero-mesh {
  position:absolute; inset:0;
  background: linear-gradient(135deg, #03091e 0%, #0c1830 40%, #10203e 70%, #1a080f 100%);
}
.hero-grid {
  position:absolute; inset:0;
  background-image:
    repeating-linear-gradient(0deg,transparent,transparent 39px,rgba(255,255,255,.025) 39px,rgba(255,255,255,.025) 40px),
    repeating-linear-gradient(90deg,transparent,transparent 39px,rgba(255,255,255,.025) 39px,rgba(255,255,255,.025) 40px);
}
.hero-inner {
  position:relative; z-index:1;
  display:flex; align-items:center; justify-content:space-between;
  padding: 38px 48px 30px;
  max-width: 100%;
}
.hero-left { display:flex; flex-direction:column; gap:10px; }
.hero-eyebrow {
  display:inline-flex; align-items:center; gap:8px;
  background: rgba(255,255,255,.07);
  border: 1px solid rgba(255,255,255,.12);
  border-radius:20px; padding:5px 16px;
  font-size:11px; font-weight:700; letter-spacing:1.5px;
  color:rgba(255,255,255,.65); text-transform:uppercase; width:fit-content;
}
.hero-title {
  font-family:'Bebas Neue',sans-serif;
  font-size:58px; line-height:.9; letter-spacing:2px; color:#fff;
}
.hero-accent {
  background: linear-gradient(90deg, var(--gold), #ffe98a);
  -webkit-background-clip:text; -webkit-text-fill-color:transparent;
  background-clip:text;
}
.hero-sub { font-size:14px; color:rgba(255,255,255,.4); font-weight:500; }
.hero-right { display:flex; flex-direction:column; align-items:flex-end; gap:12px; }
.hero-year-bg {
  font-family:'Bebas Neue',sans-serif;
  font-size:100px; line-height:.85; letter-spacing:5px;
  color:rgba(255,255,255,.05); user-select:none;
}
.hero-hosts-row { display:flex; gap:8px; }
.hero-hosts-row span {
  font-size:12px; font-weight:700; color:rgba(255,255,255,.45);
  background:rgba(255,255,255,.07); border:1px solid rgba(255,255,255,.08);
  padding:4px 12px; border-radius:6px;
}
.hero-bar { display:flex; height:4px; position:relative; z-index:1; }
.hero-bar span:nth-child(1) { flex:1; background:var(--red); }
.hero-bar span:nth-child(2) { flex:1; background:var(--blue); }
.hero-bar span:nth-child(3) { flex:1; background:var(--green); }

/* ══════════ EMPTY STATE ══════════ */
.empty-wrap {
  display:flex; flex-direction:column; gap:20px;
  padding: 28px 40px 40px;
}

/* Banner */
.empty-banner {
  position:relative; border-radius:16px; overflow:hidden;
  border:1px solid var(--border);
}
.eb-mesh {
  position:absolute; inset:0;
  background: linear-gradient(135deg,
    rgba(0,61,165,.35) 0%, rgba(0,10,40,.9) 45%,
    rgba(40,0,10,.85) 75%, rgba(200,16,46,.25) 100%);
}
.eb-content {
  position:relative; z-index:1;
  display:flex; align-items:center; justify-content:space-between;
  padding: 48px 52px;
}
.eb-left { display:flex; flex-direction:column; gap:14px; max-width:600px; }
.eb-eyebrow {
  font-size:11px; font-weight:700; letter-spacing:2px;
  color:var(--gold); text-transform:uppercase;
}
.eb-title {
  font-family:'Bebas Neue',sans-serif;
  font-size:56px; line-height:.9; letter-spacing:2px; color:#fff;
}
.eb-title span {
  background:linear-gradient(90deg,var(--gold),#ffe98a);
  -webkit-background-clip:text; -webkit-text-fill-color:transparent;
  background-clip:text;
}
.eb-desc { font-size:15px; color:rgba(255,255,255,.55); line-height:1.6; max-width:460px; }
.eb-arrow {
  display:inline-flex; align-items:center; gap:8px;
  font-size:13px; font-weight:700; color:var(--blue-lt);
  background:rgba(37,99,235,.15); border:1px solid rgba(37,99,235,.3);
  padding:8px 18px; border-radius:20px; width:fit-content;
}
.eb-right { display:flex; align-items:center; justify-content:center; }
.eb-trophy-wrap {
  position:relative; width:130px; height:130px;
  display:flex; align-items:center; justify-content:center;
}
.eb-trophy-ring {
  position:absolute; border-radius:50%;
  animation:pulse-ring 3s ease-in-out infinite;
}
.eb-trophy-ring.r1 {
  width:130px; height:130px;
  border:1px solid rgba(240,180,41,.25);
  background:rgba(240,180,41,.05);
}
.eb-trophy-ring.r2 {
  width:90px; height:90px;
  border:1px solid rgba(240,180,41,.3);
  background:rgba(240,180,41,.08);
  animation-delay:.7s;
}
.eb-trophy { font-size:52px; position:relative; z-index:1; filter:drop-shadow(0 4px 16px rgba(240,180,41,.4)); }
@keyframes pulse-ring {
  0%,100%{transform:scale(1);opacity:1}
  50%{transform:scale(1.06);opacity:.7}
}
.eb-bar { display:flex; height:4px; }
.eb-bar span:nth-child(1) { flex:1; background:var(--red); }
.eb-bar span:nth-child(2) { flex:1; background:var(--blue); }
.eb-bar span:nth-child(3) { flex:1; background:var(--green); }

/* Stats row */
.empty-stats {
  display:grid; grid-template-columns:repeat(4,1fr); gap:16px;
}
.es-card {
  background:var(--bg3); border:1px solid var(--border);
  border-radius:12px; padding:24px 20px;
  display:flex; flex-direction:column; align-items:center; gap:8px;
  transition:transform .2s, border-color .2s;
}
.es-card:hover { transform:translateY(-4px); border-color:var(--bdr2); }
.es-icon { font-size:28px; }
.es-val { font-family:'Bebas Neue',sans-serif; font-size:36px; line-height:1; }
.es-lbl { font-size:12px; color:var(--text-mid); font-weight:600; text-align:center; }

/* Featured flags */
.empty-featured {
  background:var(--bg3); border:1px solid var(--border);
  border-radius:16px; padding:28px 32px;
}
.ef-header {
  display:flex; align-items:baseline; gap:14px; margin-bottom:20px;
}
.ef-title {
  font-family:'Bebas Neue',sans-serif; font-size:22px;
  letter-spacing:1px; color:#fff;
}
.ef-sub { font-size:12px; color:var(--text-mid); }
.ef-flags {
  display:grid; grid-template-columns:repeat(8,1fr); gap:12px;
}
.ef-flag-card {
  display:flex; flex-direction:column; align-items:center; gap:8px;
  cursor:default;
}
.efc-img-wrap {
  position:relative; width:100%; aspect-ratio:16/10; overflow:hidden;
  border-radius:8px; border:1px solid var(--bdr2);
  box-shadow:var(--sh-md);
  transition:transform .25s cubic-bezier(.34,1.56,.64,1), box-shadow .25s;
}
.efc-img-wrap:hover { transform:translateY(-4px) scale(1.06); box-shadow:var(--sh-lg); }
.efc-img { width:100%; height:100%; object-fit:cover; display:block; }
.efc-shine {
  position:absolute; inset:0;
  background:linear-gradient(135deg,rgba(255,255,255,.12) 0%,transparent 60%);
  pointer-events:none;
}
.efc-name { font-size:11px; font-weight:700; color:var(--text-mid); text-align:center; }

/* How-to */
.empty-howto {
  background:var(--bg3); border:1px solid var(--border);
  border-radius:16px; padding:28px 32px;
}
.ht-title {
  font-family:'Bebas Neue',sans-serif; font-size:22px;
  letter-spacing:1px; color:#fff; margin-bottom:20px;
}
.ht-steps { display:grid; grid-template-columns:repeat(3,1fr); gap:24px; }
.ht-step {
  display:flex; align-items:flex-start; gap:16px;
  padding:20px; border-radius:10px;
  background:var(--bg4); border:1px solid var(--border);
}
.hts-num {
  font-family:'Bebas Neue',sans-serif; font-size:32px; line-height:1;
  color:rgba(255,255,255,.1); flex-shrink:0;
}
.hts-body { display:flex; flex-direction:column; gap:6px; }
.hts-title { font-size:14px; font-weight:700; color:#fff; }
.hts-desc { font-size:13px; color:var(--text-mid); line-height:1.5; }

/* ══════════ LOADING ══════════ */
.loading {
  display:flex; align-items:center; justify-content:center;
  min-height:400px; padding:40px;
}
.ld-card {
  display:flex; flex-direction:column; align-items:center; gap:18px;
  background:var(--bg3); border:1px solid var(--border);
  border-radius:16px; padding:56px 64px;
}
.ld-spinner {
  display:inline-block; position:relative;
  width:60px; height:60px;
}
.ld-spinner div {
  box-sizing:border-box; display:block; position:absolute;
  width:48px; height:48px; margin:6px;
  border:4px solid transparent; border-radius:50%;
  animation:ring-spin 1.2s cubic-bezier(.5,0,.5,1) infinite;
}
.ld-spinner div:nth-child(1){ border-top-color:var(--blue-lt); animation-delay:-.45s; }
.ld-spinner div:nth-child(2){ border-top-color:var(--red);     animation-delay:-.3s; }
.ld-spinner div:nth-child(3){ border-top-color:var(--green);   animation-delay:-.15s; }
.ld-spinner div:nth-child(4){ border-top-color:var(--gold); }
@keyframes ring-spin { 0%{transform:rotate(0)} 100%{transform:rotate(360deg)} }
.ld-text {
  font-family:'Rajdhani',sans-serif; font-size:20px;
  font-weight:700; color:var(--text);
}
.ld-sub { font-size:14px; color:var(--text-mid); }

/* ══════════ ALBUM ══════════ */
.album {
  display:flex; flex-direction:column; gap:20px;
  padding: 24px 40px 40px;
}

/* Album hero seleção */
.album-hero {
  position:relative; border-radius:16px; overflow:hidden;
  border:1px solid var(--border); flex-shrink:0;
}
.ah-bg {
  position:absolute; inset:0;
  background:linear-gradient(135deg, rgba(0,61,165,.3) 0%, rgba(0,0,0,.95) 50%, rgba(200,16,46,.2) 100%);
}
.ah-pattern {
  position:absolute; inset:0;
  background-image:radial-gradient(rgba(255,255,255,.025) 1px, transparent 1px);
  background-size:22px 22px;
}
.ah-content {
  position:relative; z-index:1;
  display:flex; align-items:center; gap:36px;
  padding: 32px 40px;
  flex-wrap:wrap;
}

/* Flag block */
.ah-flag-block { flex-shrink:0; }
.ah-flag-frame { position:relative; width:120px; height:80px; }
.ah-flag {
  width:120px; height:80px; object-fit:cover;
  border-radius:8px; border:2px solid rgba(255,255,255,.2);
  box-shadow:0 6px 28px rgba(0,0,0,.6);
}
.ah-flag-glow {
  position:absolute; inset:-8px;
  background:radial-gradient(ellipse at center, rgba(0,61,165,.4), transparent 70%);
  border-radius:14px; z-index:-1;
}

/* Info */
.ah-info { flex:1; min-width:200px; display:flex; flex-direction:column; gap:10px; }
.ah-label { font-size:10px; font-weight:700; letter-spacing:2px; color:var(--text-mid); text-transform:uppercase; }
.ah-name {
  font-family:'Bebas Neue',sans-serif;
  font-size:48px; color:#fff; letter-spacing:3px; line-height:1;
}
.ah-tags { display:flex; flex-wrap:wrap; gap:8px; }
.ah-tag {
  display:inline-flex; align-items:center; gap:5px;
  font-size:12px; font-weight:700; color:var(--text-mid);
  background:rgba(255,255,255,.07); border:1px solid var(--border);
  padding:5px 12px; border-radius:7px;
}
.ah-tag.gold   { color:var(--gold); border-color:rgba(245,200,66,.3); background:rgba(245,200,66,.08); }
.ah-tag.green  { color:var(--green); border-color:rgba(0,166,80,.3); background:rgba(0,166,80,.08); }

/* Ring */
.ah-ring-wrap { position:relative; width:100px; height:100px; flex-shrink:0; }
.ah-ring-svg { width:100px; height:100px; transform:rotate(-90deg); }
.ah-ring-center {
  position:absolute; inset:0;
  display:flex; flex-direction:column; align-items:center; justify-content:center;
}
.ah-ring-n { font-family:'Bebas Neue',sans-serif; font-size:28px; color:#fff; line-height:1; }
.ah-ring-l { font-size:9px; color:var(--text-mid); letter-spacing:.5px; }

/* Mini stats */
.ah-mini-stats {
  display:flex; gap:0; background:rgba(255,255,255,.04);
  border:1px solid var(--border); border-radius:10px; overflow:hidden;
}
.ams-item {
  flex:1; display:flex; flex-direction:column;
  align-items:center; padding:14px 10px; gap:4px;
  border-right:1px solid var(--border);
}
.ams-item:last-child { border-right:none; }
.ams-v { font-family:'Bebas Neue',sans-serif; font-size:26px; color:#fff; line-height:1; }
.ams-l { font-size:9px; color:var(--text-mid); text-transform:uppercase; letter-spacing:.5px; }

.ah-bar { display:flex; height:3px; }
.ah-bar span:nth-child(1){ flex:1; background:var(--red); }
.ah-bar span:nth-child(2){ flex:1; background:var(--blue); }
.ah-bar span:nth-child(3){ flex:1; background:var(--green); }

/* Section bar */
.album-section-bar {
  display:flex; align-items:center; justify-content:space-between;
  padding:0 4px;
}
.asb-left { display:flex; align-items:center; gap:10px; }
.asb-dot {
  width:8px; height:8px; border-radius:50%;
  background:var(--green); box-shadow:0 0 8px var(--green);
}
.asb-title {
  font-size:11px; font-weight:800; letter-spacing:2px; color:var(--text-mid);
}
.asb-count {
  font-size:12px; color:var(--text-dim);
  background:var(--bg3); border:1px solid var(--border);
  padding:2px 10px; border-radius:20px;
}
.asb-page { font-size:13px; color:var(--text-mid); font-weight:600; }

/* ══════════ STICKERS GRID ══════════ */
.stickers-grid {
  display:grid;
  grid-template-columns:repeat(6,1fr);
  gap:16px;
}

/* ── Card da figurinha ── */
.sticker {
  position:relative;
  background:var(--bg3);
  border-radius:12px;
  overflow:hidden;
  border:1px solid var(--border);
  box-shadow:var(--sh-md);
  cursor:pointer;
  display:flex; flex-direction:column;

  animation: stk-in .45s cubic-bezier(.34,1.56,.64,1) both;
  animation-delay: calc(var(--i) * 0.05s);

  transition:
    transform .28s cubic-bezier(.34,1.56,.64,1),
    box-shadow .28s ease,
    border-color .28s ease;
}
@keyframes stk-in {
  from{ opacity:0; transform:translateY(24px) scale(.88); }
  to  { opacity:1; transform:translateY(0)    scale(1);   }
}
.sticker:hover {
  transform: translateY(-8px) scale(1.04) rotate(.4deg);
  box-shadow:
    0 24px 48px rgba(0,0,0,.7),
    0 0 0 1px rgba(37,99,235,.45),
    0 0 30px rgba(37,99,235,.15);
  border-color: rgba(37,99,235,.5);
  z-index:5;
}

/* Número flutuante */
.stk-num {
  position:absolute; top:-7px; left:-7px; z-index:10;
  width:24px; height:24px;
  background:linear-gradient(135deg,var(--blue),var(--blue-dk));
  border:2px solid var(--bg);
  border-radius:50%;
  font-size:9px; font-weight:800;
  color:#fff; display:flex; align-items:center; justify-content:center;
  font-family:'Rajdhani',sans-serif;
  box-shadow:0 2px 10px rgba(0,0,0,.6);
}

/* Glare de hover */
.stk-glare {
  position:absolute; inset:0;
  background:linear-gradient(135deg,rgba(255,255,255,.09) 0%,transparent 55%);
  border-radius:12px; z-index:4;
  pointer-events:none; opacity:0;
  transition:opacity .3s;
}
.sticker:hover .stk-glare { opacity:1; }

/* Cabeçalho */
.stk-head {
  display:flex; align-items:center; justify-content:space-between;
  padding:7px 9px 6px;
  background:linear-gradient(90deg, var(--blue-dk) 0%, var(--blue) 55%, #8b0d1f 100%);
  flex-shrink:0;
}
.stk-head-l { display:flex; align-items:center; gap:5px; }
.stk-flag { width:22px; height:15px; object-fit:cover; border-radius:2px; box-shadow:0 1px 4px rgba(0,0,0,.5); }
.stk-country { font-size:8px; font-weight:800; letter-spacing:.5px; color:rgba(255,255,255,.7); }
.stk-ball { font-size:12px; opacity:.75; }

/* Foto */
.stk-photo-wrap {
  position:relative; width:100%; aspect-ratio:1; overflow:hidden;
  background:linear-gradient(180deg, var(--bg4) 0%, var(--bg2) 100%);
  flex-shrink:0;
}
.stk-photo-bg {
  position:absolute; inset:0;
  background:radial-gradient(ellipse 80% 60% at 50% 20%, rgba(0,61,165,.22), transparent 70%);
}
.stk-photo {
  width:100%; height:100%; object-fit:cover; display:block;
  position:relative; z-index:1;
  transition:transform .35s cubic-bezier(.34,1.56,.64,1);
}
.sticker:hover .stk-photo { transform:scale(1.1); }
.stk-photo-grad {
  position:absolute; bottom:0; left:0; right:0; height:45%;
  background:linear-gradient(transparent, var(--bg3));
  z-index:2;
}
/* OVR badge */
.stk-overall {
  position:absolute; bottom:8px; right:8px; z-index:3;
  display:flex; flex-direction:column; align-items:center;
  background:rgba(0,0,0,.75); backdrop-filter:blur(6px);
  border:1px solid rgba(255,255,255,.12);
  border-radius:7px; padding:4px 8px;
  min-width:36px;
}
.so-val {
  font-family:'Bebas Neue',sans-serif;
  font-size:18px; color:#fff; line-height:1;
}
.so-lbl { font-size:7px; color:var(--gold); font-weight:800; letter-spacing:.5px; }

/* Identidade */
.stk-identity {
  padding:9px 9px 4px; text-align:center; flex-shrink:0;
}
.stk-name {
  font-family:'Rajdhani',sans-serif;
  font-size:12px; font-weight:700; color:var(--text);
  text-transform:uppercase; letter-spacing:.3px;
  white-space:nowrap; overflow:hidden; text-overflow:ellipsis; line-height:1.2;
}
.stk-pos {
  font-size:8px; font-weight:800; letter-spacing:1px;
  color:#fff; text-transform:uppercase;
  background:linear-gradient(90deg, var(--blue), var(--blue-lt));
  padding:2px 9px; border-radius:4px;
  display:inline-block; margin-top:4px;
}

/* Atributos */
.stk-attrs {
  display:flex; flex-direction:column; gap:4px;
  padding:6px 9px 4px; flex:1;
}
.attr {
  display:grid; grid-template-columns:22px 1fr 20px;
  align-items:center; gap:5px;
}
.attr-s {
  font-size:7px; font-weight:800; letter-spacing:.5px;
  color:var(--text-mid); text-transform:uppercase;
}
.attr-bar-bg {
  height:3px; background:rgba(255,255,255,.07);
  border-radius:3px; overflow:hidden;
}
.attr-bar-fill {
  height:100%; border-radius:3px;
  transition:width .9s cubic-bezier(.16,1,.3,1);
}
.attr-v {
  font-family:'Rajdhani',sans-serif;
  font-size:10px; font-weight:800; text-align:right;
}

/* Rodapé figurinha */
.stk-foot {
  display:flex; align-items:center; justify-content:center; gap:5px;
  padding:5px 8px 8px;
  border-top:1px solid var(--border);
  background:rgba(255,255,255,.02);
  flex-shrink:0;
}
.stk-foot span { font-size:7px; font-weight:800; letter-spacing:1px; color:var(--text-dim); }
.stk-foot-dot { width:2px; height:2px; border-radius:50%; background:var(--text-dim); }

/* ══════════ PAGINAÇÃO ══════════ */
.pagination {
  display:flex; align-items:center; justify-content:space-between;
  background:var(--bg3); border:1px solid var(--border);
  border-radius:14px; padding:16px 24px; gap:12px;
}
.pg-btn {
  display:flex; align-items:center; gap:6px;
  padding:10px 20px; border-radius:9px;
  border:1px solid var(--border); background:var(--bg4);
  color:var(--text-mid);
  font-family:'Syne',sans-serif; font-size:13px; font-weight:700;
  cursor:pointer; transition:all .2s;
}
.pg-btn:hover:not(:disabled) {
  border-color:var(--blue-lt); color:#fff;
  background:rgba(37,99,235,.15);
  box-shadow:0 0 0 3px rgba(37,99,235,.15);
}
.pg-next:hover:not(:disabled) {
  border-color:var(--red); background:rgba(200,16,46,.15);
  box-shadow:0 0 0 3px rgba(200,16,46,.15);
}
.pg-btn:disabled { opacity:.3; cursor:not-allowed; }
.pg-dots { display:flex; align-items:center; gap:4px; }
.pg-dot {
  width:36px; height:36px; border-radius:9px;
  border:1px solid var(--border); background:transparent;
  color:var(--text-mid);
  font-family:'Rajdhani',sans-serif; font-size:14px; font-weight:700;
  cursor:pointer; transition:all .2s;
  display:flex; align-items:center; justify-content:center;
}
.pg-dot:hover { background:rgba(255,255,255,.06); color:var(--text); }
.pg-dot.near  { color:var(--text); border-color:rgba(255,255,255,.15); }
.pg-dot.active {
  background:linear-gradient(135deg,var(--blue),var(--blue-lt));
  border-color:var(--blue-lt); color:#fff;
  box-shadow:0 4px 14px rgba(0,61,165,.45);
  transform:scale(1.12);
}
.pg-ellipsis {
  width:24px; text-align:center;
  color:var(--text-dim); font-size:14px;
  display:flex; align-items:center; justify-content:center;
}

/* ══════════ RESPONSIVIDADE ══════════ */
/* 5 colunas em telas < 1600px */
@media (max-width:1599px) {
  .stickers-grid { grid-template-columns:repeat(5,1fr); }
}
/* 4 colunas em telas < 1280px */
@media (max-width:1279px) {
  .stickers-grid { grid-template-columns:repeat(4,1fr); }
  .hero-year-bg  { font-size:72px; }
  .ef-flags      { grid-template-columns:repeat(4,1fr); }
  .empty-stats   { grid-template-columns:repeat(2,1fr); }
  .ht-steps      { grid-template-columns:repeat(2,1fr); }
}
/* Sidebar recolhida em < 1024px */
@media (max-width:1023px) {
  :root{ --sidebar-w:72px; }
  .sb-brand, .sb-link > span:not(.sb-ico), .sb-badge,
  .sb-select-block, .sb-team-card, .sb-copa-body > div,
  .sb-copa-title, .sb-copa-year, .sb-copa-hosts { display:none; }
  .sb-logo    { justify-content:center; padding:16px 0; }
  .sb-link    { justify-content:center; padding:12px 0; }
  .sb-ico     { opacity:1 !important; width:22px; height:22px; }
  .sb-copa    { padding:0; }
  .sb-copa-body { justify-content:center; }
  .sb-copa-trophy { font-size:22px; }
  .sb-divider { display:none; }
  .album      { padding:20px 24px 36px; }
  .empty-wrap { padding:20px 24px 36px; }
  .hero-inner { padding:28px 24px 22px; }
  .stickers-grid { grid-template-columns:repeat(3,1fr); gap:12px; }
  .ah-content { padding:24px 28px; gap:24px; }
  .ef-flags   { grid-template-columns:repeat(4,1fr); }
}
/* Tablet */
@media (max-width:767px) {
  .root { flex-direction:column; }
  .sidebar {
    width:100%; min-width:unset; height:auto;
    position:static; flex-direction:row;
    align-items:center; padding:0 12px; height:58px; overflow:visible;
  }
  .sb-logo   { border:none; padding:0; margin-right:auto; }
  .sb-name   { font-size:18px; }
  .sb-sub    { display:none; }
  .sb-nav    { flex-direction:row; gap:4px; padding:0; margin:0 0 0 8px; }
  .sb-link   { padding:8px 10px; border-radius:8px; }
  .sb-brand, .sb-badge, .sb-link > span:not(.sb-ico) { display:none; }
  .sb-ico    { opacity:1 !important; }
  .sb-divider,.sb-select-block,.sb-team-card,.sb-copa { display:none; }

  .hero-inner { padding:22px 18px 16px; flex-direction:column; gap:12px; align-items:flex-start; }
  .hero-year-bg,.hero-hosts-row { display:none; }
  .hero-title { font-size:42px; }

  .empty-wrap { padding:16px 16px 32px; }
  .eb-content { padding:28px 24px; flex-direction:column; gap:24px; }
  .eb-title   { font-size:38px; }
  .ef-flags   { grid-template-columns:repeat(4,1fr); }
  .ht-steps   { grid-template-columns:1fr; }

  .album      { padding:16px 16px 32px; }
  .ah-content { flex-direction:column; gap:20px; padding:24px 20px; }
  .ah-ring-wrap,.ah-mini-stats { display:none; }
  .ah-name    { font-size:34px; }
  .stickers-grid { grid-template-columns:repeat(3,1fr); gap:10px; }
  .pagination { padding:12px 14px; }
  .pg-dots    { display:none; }
}
/* Mobile */
@media (max-width:479px) {
  .stickers-grid { grid-template-columns:repeat(2,1fr); gap:10px; }
  .empty-stats   { grid-template-columns:repeat(2,1fr); }
  .ef-flags      { grid-template-columns:repeat(2,1fr); }
  .ht-steps      { grid-template-columns:1fr; }
  .eb-title      { font-size:30px; }
  .hero-title    { font-size:34px; }
}
</style>