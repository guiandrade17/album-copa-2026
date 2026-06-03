<script setup>
import { ref, onMounted, computed } from 'vue'

/* ══════════════════════════════════════════
   CONFIG
══════════════════════════════════════════ */
const token = '5dc0faddd67d4bd655e074569c6111e8'

/* ══════════════════════════════════════════
   AUTH STATE
══════════════════════════════════════════ */
const authView      = ref('login')   // 'login' | 'register'
const isLoggedIn    = ref(false)
const authError     = ref('')
const authLoading   = ref(false)

const loginForm     = ref({ email: '', password: '' })
const registerForm  = ref({ name: '', email: '', password: '', confirm: '' })

// Simulated user store (localStorage-less, in-memory)
const users         = ref([{ name: 'Demo', email: 'demo@copa.com', password: '123456' }])
const currentUser   = ref(null)

const doLogin = async () => {
  authError.value   = ''
  authLoading.value = true
  await new Promise(r => setTimeout(r, 800))
  const found = users.value.find(
    u => u.email === loginForm.value.email && u.password === loginForm.value.password
  )
  if (found) {
    currentUser.value = found
    isLoggedIn.value  = true
  } else {
    authError.value = 'E-mail ou senha incorretos.'
  }
  authLoading.value = false
}

const doRegister = async () => {
  authError.value = ''
  if (!registerForm.value.name || !registerForm.value.email || !registerForm.value.password) {
    authError.value = 'Preencha todos os campos.'
    return
  }
  if (registerForm.value.password !== registerForm.value.confirm) {
    authError.value = 'As senhas não coincidem.'
    return
  }
  if (users.value.find(u => u.email === registerForm.value.email)) {
    authError.value = 'Este e-mail já está cadastrado.'
    return
  }
  authLoading.value = true
  await new Promise(r => setTimeout(r, 800))
  users.value.push({
    name: registerForm.value.name,
    email: registerForm.value.email,
    password: registerForm.value.password
  })
  currentUser.value = users.value[users.value.length - 1]
  isLoggedIn.value  = true
  authLoading.value = false
}

const doLogout = () => {
  isLoggedIn.value  = false
  currentUser.value = null
  loginForm.value   = { email: '', password: '' }
  registerForm.value = { name: '', email: '', password: '', confirm: '' }
}

/* ══════════════════════════════════════════
   NAVIGATION
══════════════════════════════════════════ */
const navAtivo = ref('album')

/* ══════════════════════════════════════════
   ALBUM — API DATA
══════════════════════════════════════════ */
const paises          = ref([])
const paisSelecionado = ref('')
const jogadores       = ref([])
const carregando      = ref(false)
const paginaAtual     = ref(1)
const porPagina       = 12

const bandeiraSelecionada = computed(() =>
  paises.value.find(p => p.name === paisSelecionado.value)?.flag ?? null
)
const totalPaginas    = computed(() => Math.ceil(jogadores.value.length / porPagina))
const jogadoresPagina = computed(() => {
  const inicio = (paginaAtual.value - 1) * porPagina
  return jogadores.value.slice(inicio, inicio + porPagina)
})

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
    { s: 'ELA', v: rng(60,92,1) }, { s: 'MAN', v: rng(62,93,2) },
    { s: 'CHU', v: rng(45,78,3) }, { s: 'REF', v: rng(65,95,4) },
    { s: 'VEL', v: rng(55,85,5) }, { s: 'POS', v: rng(68,95,6) },
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

const corStat    = (v) => v >= 85 ? '#00d48c' : v >= 72 ? '#f5c842' : '#ff6b6b'
const mediaGeral = (jogador) => {
  const s = gerarStats(jogador)
  return Math.round(s.reduce((a, x) => a + x.v, 0) / s.length)
}

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
    const rTime  = await fetch(`https://v3.football.api-sports.io/teams?name=${paisSelecionado.value}`, { headers: { 'x-apisports-key': token } })
    const dTime  = await rTime.json()
    const teamId = dTime.response[0].team.id
    const rElenco = await fetch(`https://v3.football.api-sports.io/players/squads?team=${teamId}`, { headers: { 'x-apisports-key': token } })
    const dElenco = await rElenco.json()
    jogadores.value = dElenco.response[0].players
  } catch (e) { console.error(e) }
  finally { carregando.value = false }
}

const irParaPagina   = (n) => { paginaAtual.value = n }
const proximaPagina  = () => { if (paginaAtual.value < totalPaginas.value) paginaAtual.value++ }
const paginaAnterior = () => { if (paginaAtual.value > 1) paginaAtual.value-- }

onMounted(carregarPaises)

/* ══════════════════════════════════════════
   MERCADO — dados fictícios
══════════════════════════════════════════ */
const mercadoFiltro = ref('todos')

// Iniciais para avatar fallback do mercado
const getInitials = (name) => name.split(' ').map(w=>w[0]).slice(0,2).join('').toUpperCase()

// Cores por posição para avatar fallback
const posColor = (pos) => {
  if (pos === 'Atacante') return ['#C8102E','#9a0c23']
  if (pos === 'Meia')     return ['#003DA5','#002070']
  if (pos === 'Defensor') return ['#00a650','#007a3d']
  return ['#554400','#332200']
}

const figurinhasRaras = [
  { id:1,  name:'Kylian Mbappé',    country:'France',    code:'fr',     ovr:96, pos:'Atacante', price:850, rare:true  },
  { id:2,  name:'Erling Haaland',   country:'Norway',    code:'no',     ovr:94, pos:'Atacante', price:780, rare:true  },
  { id:3,  name:'Vinicius Jr',      country:'Brazil',    code:'br',     ovr:93, pos:'Atacante', price:720, rare:true  },
  { id:4,  name:'Pedri',            country:'Spain',     code:'es',     ovr:91, pos:'Meia',     price:600, rare:true  },
  { id:5,  name:'Rodri',            country:'Spain',     code:'es',     ovr:92, pos:'Meia',     price:640, rare:false },
  { id:6,  name:'Jude Bellingham',  country:'England',   code:'gb-eng', ovr:92, pos:'Meia',     price:670, rare:true  },
  { id:7,  name:'Bukayo Saka',      country:'England',   code:'gb-eng', ovr:89, pos:'Atacante', price:520, rare:false },
  { id:8,  name:'Lautaro Martínez', country:'Argentina', code:'ar',     ovr:90, pos:'Atacante', price:550, rare:false },
  { id:9,  name:'Gavi',             country:'Spain',     code:'es',     ovr:89, pos:'Meia',     price:510, rare:false },
  { id:10, name:'Phil Foden',       country:'England',   code:'gb-eng', ovr:90, pos:'Meia',     price:560, rare:true  },
  { id:11, name:'Federico Valverde',country:'Uruguay',   code:'uy',     ovr:88, pos:'Meia',     price:480, rare:false },
  { id:12, name:'Raphinha',         country:'Brazil',    code:'br',     ovr:88, pos:'Atacante', price:490, rare:false },
]

const filteredMercado = computed(() => {
  if (mercadoFiltro.value === 'raras')  return figurinhasRaras.filter(f => f.rare)
  if (mercadoFiltro.value === 'oferta') return figurinhasRaras.filter(f => f.price < 600)
  return figurinhasRaras
})

const trocasRecentes = [
  { de:'João S.',   para:'Pedro L.',  jogador:'Neymar Jr',    pais:'Brasil',   code:'br'  },
  { de:'Mariana R.',para:'Carlos M.', jogador:'Messi',        pais:'Argentina',code:'ar'  },
  { de:'Rafael T.', para:'Ana C.',    jogador:'Ronaldo',      pais:'Portugal', code:'pt'  },
  { de:'Lucas B.',  para:'Fernanda K.',jogador:'Mbappé',      pais:'França',   code:'fr'  },
]

/* ══════════════════════════════════════════
   PERFIL — dados do usuário
══════════════════════════════════════════ */
const perfilSeleçaoFavorita = ref('Brazil')
const perfilFavoritas = ['Brazil','Argentina','France','Spain','Germany','Portugal','England','Italy']

const statsColecao = computed(() => ({
  coletadas: 134,
  total: 440,
  raras: 12,
  duplicadas: 47,
  nivel: 'Ouro',
  xp: 2340,
  xpProximo: 3000,
}))

const conquistas = [
  { icon:'⚽', nome:'Primeiro Álbum',   desc:'Complete 10 figurinhas',    ok:true  },
  { icon:'🌍', nome:'Caçador Global',   desc:'Colete 5 seleções',          ok:true  },
  { icon:'💎', nome:'Figurinha Rara',   desc:'Obtenha 1 figurinha rara',   ok:true  },
  { icon:'🔥', nome:'Em Chamas',        desc:'Login por 7 dias seguidos',  ok:false },
  { icon:'🏆', nome:'Campeão',          desc:'Complete um álbum inteiro',  ok:false },
  { icon:'⭐', nome:'Lendário',         desc:'Colete 50 figurinhas raras', ok:false },
]

const nivelCor   = (n) => ({ 'Bronze':'#cd7f32','Prata':'#c0c0c0','Ouro':'#f5c842','Diamante':'#b9f2ff' }[n] || '#fff')
const xpPct      = computed(() => Math.round((statsColecao.value.xp / statsColecao.value.xpProximo) * 100))
const colPct     = computed(() => Math.round((statsColecao.value.coletadas / statsColecao.value.total) * 100))
</script>

<template>
<div class="root">

  <!-- ══════════════════════ AUTH SCREEN ══════════════════════ -->
  <transition name="auth-fade">
  <div v-if="!isLoggedIn" class="auth-screen">
    <div class="auth-bg">
      <div class="auth-mesh"></div>
      <div class="auth-particles">
        <span v-for="n in 20" :key="n" :style="{ '--d': n+'s', '--x': (n*17%100)+'%', '--s': (.5+n*.03)+'rem', '--del': (n*.15)+'s' }"></span>
      </div>
    </div>

    <div class="auth-card">
      <!-- Brand -->
      <div class="auth-brand">
        <div class="auth-emblem">⚽</div>
        <div>
          <div class="auth-brand-name">FutÁlbum</div>
          <div class="auth-brand-sub">FIFA WORLD CUP 2026</div>
        </div>
      </div>

      <!-- Tabs -->
      <div class="auth-tabs">
        <button :class="['auth-tab', {active: authView==='login'}]"    @click="authView='login'; authError=''">Entrar</button>
        <button :class="['auth-tab', {active: authView==='register'}]" @click="authView='register'; authError=''">Cadastrar</button>
      </div>

      <!-- Error -->
      <transition name="fade-down">
        <div v-if="authError" class="auth-error">{{ authError }}</div>
      </transition>

      <!-- LOGIN -->
      <transition name="fade-slide" mode="out-in">
      <div v-if="authView==='login'" key="login" class="auth-form">
        <div class="auth-field">
          <label>E-mail</label>
          <div class="auth-input-wrap">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/></svg>
            <input v-model="loginForm.email" type="email" placeholder="seu@email.com" @keyup.enter="doLogin"/>
          </div>
        </div>
        <div class="auth-field">
          <label>Senha</label>
          <div class="auth-input-wrap">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/></svg>
            <input v-model="loginForm.password" type="password" placeholder="••••••••" @keyup.enter="doLogin"/>
          </div>
        </div>
        <div class="auth-hint">Teste: demo@copa.com / 123456</div>
        <button class="auth-btn" @click="doLogin" :disabled="authLoading">
          <span v-if="!authLoading">Entrar no Álbum</span>
          <span v-else class="auth-spinner"></span>
        </button>
      </div>

      <!-- REGISTER -->
      <div v-else key="register" class="auth-form">
        <div class="auth-field">
          <label>Nome</label>
          <div class="auth-input-wrap">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
            <input v-model="registerForm.name" type="text" placeholder="Seu nome"/>
          </div>
        </div>
        <div class="auth-field">
          <label>E-mail</label>
          <div class="auth-input-wrap">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/></svg>
            <input v-model="registerForm.email" type="email" placeholder="seu@email.com"/>
          </div>
        </div>
        <div class="auth-field">
          <label>Senha</label>
          <div class="auth-input-wrap">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/></svg>
            <input v-model="registerForm.password" type="password" placeholder="••••••••"/>
          </div>
        </div>
        <div class="auth-field">
          <label>Confirmar Senha</label>
          <div class="auth-input-wrap">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m22 4-10 10.01-3-3"/><path d="M22 4 12 14.01l-3-3"/><path d="M3 12v7a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/></svg>
            <input v-model="registerForm.confirm" type="password" placeholder="••••••••" @keyup.enter="doRegister"/>
          </div>
        </div>
        <button class="auth-btn" @click="doRegister" :disabled="authLoading">
          <span v-if="!authLoading">Criar Conta</span>
          <span v-else class="auth-spinner"></span>
        </button>
      </div>
      </transition>
    </div>

    <!-- Decorative side -->
    <div class="auth-deco">
      <div class="auth-deco-year">2026</div>
      <div class="auth-deco-flags">
        <img src="https://flagcdn.com/w80/us.png" alt="USA"/><span>USA</span>
        <img src="https://flagcdn.com/w80/ca.png" alt="CAN"/><span>CAN</span>
        <img src="https://flagcdn.com/w80/mx.png" alt="MEX"/><span>MEX</span>
      </div>
      <div class="auth-deco-trophy">🏆</div>
      <div class="auth-deco-tagline">O maior álbum<br><strong>do mundo</strong></div>
    </div>
  </div>
  </transition>

  <!-- ══════════════════════ APP SHELL ══════════════════════ -->
  <transition name="auth-fade">
  <div v-if="isLoggedIn" class="app-shell">

    <!-- ════════ SIDEBAR ════════ -->
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
          <span class="sb-badge">PRINCIPAL</span>
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

      <!-- Seleção (only on album tab) -->
      <transition name="slide-up">
      <div v-if="navAtivo==='album'" class="sb-select-block">
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
      </transition>

      <!-- Team card sidebar -->
      <transition name="slide-up">
      <div v-if="navAtivo==='album' && paisSelecionado && !carregando" class="sb-team-card">
        <div class="stc-header">
          <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="stc-flag" alt="flag"/>
          <div class="stc-info">
            <span class="stc-name">{{ paisSelecionado }}</span>
            <span class="stc-label">Seleção Nacional</span>
          </div>
        </div>
        <div class="stc-stats">
          <div class="stcs"><span class="stcs-v">{{ jogadores.length }}</span><span class="stcs-l">Jogadores</span></div>
          <div class="stcs-div"></div>
          <div class="stcs"><span class="stcs-v">{{ totalPaginas }}</span><span class="stcs-l">Páginas</span></div>
          <div class="stcs-div"></div>
          <div class="stcs"><span class="stcs-v">26</span><span class="stcs-l">Edição</span></div>
        </div>
      </div>
      </transition>

      <!-- User quick card -->
      <div class="sb-user" @click="navAtivo='perfil'">
        <div class="sb-user-avatar">{{ currentUser?.name?.[0]?.toUpperCase() }}</div>
        <div class="sb-user-info">
          <span class="sb-user-name">{{ currentUser?.name }}</span>
          <span class="sb-user-level">Nível Ouro ⭐</span>
        </div>
      </div>

      <button class="sb-logout" @click="doLogout">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="16" height="16"><path d="M9 21H5a2 2 0 01-2-2V5a2 2 0 012-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></svg>
        <span>Sair</span>
      </button>

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

    <!-- ════════ MAIN ════════ -->
    <main class="main">

      <!-- Hero -->
      <header class="hero">
        <div class="hero-mesh"></div>
        <div class="hero-grid"></div>
        <div class="hero-inner">
          <div class="hero-left">
            <div class="hero-eyebrow">⚽ COPA DO MUNDO FIFA 2026</div>
            <h1 class="hero-title">
              <span v-if="navAtivo==='album'">ÁLBUM DE<br><span class="hero-accent">FIGURINHAS</span></span>
              <span v-else-if="navAtivo==='mercado'">MERCADO DE<br><span class="hero-accent">FIGURINHAS</span></span>
              <span v-else>MEU<br><span class="hero-accent">PERFIL</span></span>
            </h1>
            <p class="hero-sub">
              <span v-if="navAtivo==='album'">Colecione os melhores jogadores do mundo</span>
              <span v-else-if="navAtivo==='mercado'">Troque e adquira figurinhas raras</span>
              <span v-else>Gerencie sua coleção, {{ currentUser?.name }}</span>
            </p>
          </div>
          <div class="hero-right">
            <div class="hero-year-bg">2026</div>
            <div class="hero-hosts-row">
              <div class="hero-host-flag"><img src="https://flagcdn.com/w80/us.png" alt="USA" class="host-flag-img"/><span>USA</span></div>
              <div class="hero-host-flag"><img src="https://flagcdn.com/w80/ca.png" alt="CAN" class="host-flag-img"/><span>CAN</span></div>
              <div class="hero-host-flag"><img src="https://flagcdn.com/w80/mx.png" alt="MEX" class="host-flag-img"/><span>MEX</span></div>
            </div>
          </div>
        </div>
        <div class="hero-bar"><span></span><span></span><span></span></div>
      </header>

      <!-- ══════ TAB: ÁLBUM ══════ -->
      <transition name="tab-fade" mode="out-in">
      <div v-if="navAtivo==='album'" key="album">

        <!-- Empty state -->
        <section v-if="!paisSelecionado && !carregando" class="empty-wrap">
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
          <div class="empty-stats">
            <div class="es-card" v-for="(item,i) in [{icon:'🌍',val:'211',lbl:'Seleções',col:'var(--blue-lt)'},{icon:'👕',val:'26+',lbl:'Jogadores/time',col:'var(--green)'},{icon:'🏆',val:'2026',lbl:'Edição Copa',col:'var(--gold)'},{icon:'⚽',val:'48',lbl:'Times na Copa',col:'var(--red)'}]" :key="i">
              <div class="es-icon">{{ item.icon }}</div>
              <div class="es-val" :style="{color:item.col}">{{ item.val }}</div>
              <div class="es-lbl">{{ item.lbl }}</div>
            </div>
          </div>
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
          <div class="empty-howto">
            <h3 class="ht-title">Como funciona</h3>
            <div class="ht-steps">
              <div class="ht-step" v-for="(s,i) in [{n:'01',t:'Escolha a Seleção',d:'Use o menu lateral para selecionar o país que deseja explorar.'},{n:'02',t:'Veja os Jogadores',d:'Explore as figurinhas dos jogadores com atributos detalhados.'},{n:'03',t:'Navegue pelo Álbum',d:'Use a paginação para ver todos os jogadores do elenco.'}]" :key="i">
                <div class="hts-num">{{ s.n }}</div>
                <div class="hts-body"><div class="hts-title">{{ s.t }}</div><div class="hts-desc">{{ s.d }}</div></div>
              </div>
            </div>
          </div>
        </section>

        <!-- Loading -->
        <div v-else-if="carregando" class="loading">
          <div class="ld-card">
            <div class="ld-spinner"><div></div><div></div><div></div><div></div></div>
            <p class="ld-text">Carregando figurinhas...</p>
            <span class="ld-sub">Buscando elenco de <strong>{{ paisSelecionado }}</strong></span>
          </div>
        </div>

        <!-- Album grid -->
        <div v-else class="album">
          <div class="album-hero">
            <div class="ah-bg"></div>
            <div class="ah-pattern"></div>
            <div class="ah-content">
              <div class="ah-flag-block">
                <div class="ah-flag-frame">
                  <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="ah-flag" alt="Bandeira"/>
                  <div class="ah-flag-glow"></div>
                </div>
              </div>
              <div class="ah-info">
                <span class="ah-label">SELEÇÃO NACIONAL · COPA 2026</span>
                <h2 class="ah-name">{{ paisSelecionado.toUpperCase() }}</h2>
                <div class="ah-tags">
                  <span class="ah-tag"><svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87"/><path d="M16 3.13a4 4 0 010 7.75"/></svg>{{ jogadores.length }} jogadores</span>
                  <span class="ah-tag gold">🏆 Copa do Mundo 2026</span>
                  <span class="ah-tag green"><svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M22 11.08V12a10 10 0 11-5.93-9.14"/><path d="m22 4-10 10.01-3-3"/></svg>Elenco Oficial</span>
                </div>
              </div>
              <div class="ah-ring-wrap">
                <svg class="ah-ring-svg" viewBox="0 0 100 100">
                  <circle cx="50" cy="50" r="42" fill="none" stroke="rgba(255,255,255,0.08)" stroke-width="7"/>
                  <circle cx="50" cy="50" r="42" fill="none" stroke="url(#rg)" stroke-width="7" stroke-dasharray="263.9" stroke-dashoffset="0" stroke-linecap="round"/>
                  <defs><linearGradient id="rg" x1="0" y1="0" x2="1" y2="1"><stop offset="0%" stop-color="#00d48c"/><stop offset="100%" stop-color="#0061ff"/></linearGradient></defs>
                </svg>
                <div class="ah-ring-center">
                  <span class="ah-ring-n">{{ jogadores.length }}</span>
                  <span class="ah-ring-l">jogadores</span>
                </div>
              </div>
              <div class="ah-mini-stats">
                <div class="ams-item" v-for="(s,i) in [{v:totalPaginas,l:'Páginas'},{v:Math.min(porPagina,jogadores.length),l:'Por página'},{v:paginaAtual,l:'Página atual'}]" :key="i">
                  <span class="ams-v">{{ s.v }}</span><span class="ams-l">{{ s.l }}</span>
                </div>
              </div>
            </div>
            <div class="ah-bar"><span></span><span></span><span></span></div>
          </div>

          <div class="album-section-bar">
            <div class="asb-left">
              <span class="asb-dot"></span>
              <span class="asb-title">ELENCO COMPLETO</span>
              <span class="asb-count">{{ jogadores.length }} figurinhas</span>
            </div>
            <span class="asb-page">Página {{ paginaAtual }} de {{ totalPaginas }}</span>
          </div>

          <!-- STICKERS GRID -->
          <div class="stickers-grid">
            <div class="sticker" v-for="(jogador,index) in jogadoresPagina" :key="jogador.id" :style="{'--i':index,'--media':mediaGeral(jogador)}">
              <div class="stk-num">{{ (paginaAtual-1)*porPagina+index+1 }}</div>
              <div class="stk-glare"></div>
              <div class="stk-head">
                <div class="stk-head-l">
                  <img v-if="bandeiraSelecionada" :src="bandeiraSelecionada" class="stk-flag" alt="País"/>
                  <span class="stk-country">{{ paisSelecionado.slice(0,3).toUpperCase() }}</span>
                </div>
                <span class="stk-ball">⚽</span>
              </div>
              <div class="stk-photo-wrap">
                <div class="stk-photo-bg"></div>
                <div class="stk-photo-fallback">{{ jogador.name.split(' ').map(w=>w[0]).slice(0,2).join('').toUpperCase() }}</div>
                <img :src="jogador.photo" :alt="jogador.name" class="stk-photo" @error="e=>{e.target.style.display='none'}"/>
                <div class="stk-photo-grad"></div>
                <div class="stk-overall"><span class="so-val">{{ mediaGeral(jogador) }}</span><span class="so-lbl">OVR</span></div>
              </div>
              <div class="stk-identity">
                <div class="stk-name">{{ jogador.name }}</div>
                <div class="stk-pos">{{ jogador.position || 'Jogador' }}</div>
              </div>
              <div class="stk-attrs">
                <div class="attr" v-for="st in gerarStats(jogador)" :key="st.s">
                  <span class="attr-s">{{ st.s }}</span>
                  <div class="attr-bar-bg"><div class="attr-bar-fill" :style="{width:st.v+'%',background:corStat(st.v)}"></div></div>
                  <span class="attr-v" :style="{color:corStat(st.v)}">{{ st.v }}</span>
                </div>
              </div>
              <div class="stk-foot">
                <span>FIFA</span><span class="stk-foot-dot"></span>
                <span>COPA 2026</span><span class="stk-foot-dot"></span>
                <span>OFFICIAL</span>
              </div>
            </div>
          </div>

          <!-- PAGINAÇÃO -->
          <div class="pagination">
            <button class="pg-btn pg-prev" @click="paginaAnterior" :disabled="paginaAtual===1">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="m15 18-6-6 6-6"/></svg>Anterior
            </button>
            <div class="pg-dots">
              <template v-for="n in totalPaginas" :key="n">
                <button v-if="n===1||n===totalPaginas||Math.abs(n-paginaAtual)<=1" class="pg-dot" :class="{active:n===paginaAtual,near:Math.abs(n-paginaAtual)===1}" @click="irParaPagina(n)">{{ n }}</button>
                <span v-else-if="n===2&&paginaAtual>3" class="pg-ellipsis">…</span>
                <span v-else-if="n===totalPaginas-1&&paginaAtual<totalPaginas-2" class="pg-ellipsis">…</span>
              </template>
            </div>
            <button class="pg-btn pg-next" @click="proximaPagina" :disabled="paginaAtual===totalPaginas">
              Próxima<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="m9 18 6-6-6-6"/></svg>
            </button>
          </div>
        </div>

      </div>

      <!-- ══════ TAB: MERCADO ══════ -->
      <div v-else-if="navAtivo==='mercado'" key="mercado" class="mercado-wrap">

        <!-- Filtros -->
        <div class="mkt-filters">
          <button v-for="f in [{k:'todos',l:'Todos'},{k:'raras',l:'⭐ Raras'},{k:'oferta',l:'🔥 Oferta'}]" :key="f.k"
            class="mkt-filter-btn" :class="{active:mercadoFiltro===f.k}" @click="mercadoFiltro=f.k">{{ f.l }}</button>
        </div>

        <!-- Cards de figurinhas do mercado -->
        <div class="mkt-grid">
          <div class="mkt-card" v-for="(f,i) in filteredMercado" :key="f.id" :style="{'--i':i,'--rare':f.rare?1:0}">
            <div v-if="f.rare" class="mkt-rare-badge">⭐ RARA</div>
            <div class="mkt-card-glow" v-if="f.rare"></div>
            <div class="mkt-photo-wrap">
              <div class="mkt-avatar" :style="{'background':'linear-gradient(135deg,'+posColor(f.pos)[0]+','+posColor(f.pos)[1]+')'}">
                <span class="mkt-avatar-initials">{{ getInitials(f.name) }}</span>
                <img :src="`https://flagcdn.com/w40/${f.code}.png`" class="mkt-avatar-flag" :alt="f.country"/>
              </div>
              <div class="mkt-photo-overlay"></div>
              <div class="mkt-ovr">
                <span>{{ f.ovr }}</span>
                <small>OVR</small>
              </div>
            </div>
            <div class="mkt-info">
              <div class="mkt-flag-row">
                <img :src="`https://flagcdn.com/w40/${f.code}.png`" :alt="f.country" class="mkt-flag"/>
                <span class="mkt-country">{{ f.country }}</span>
              </div>
              <div class="mkt-name">{{ f.name }}</div>
              <div class="mkt-pos-tag">{{ f.pos }}</div>
            </div>
            <div class="mkt-footer">
              <div class="mkt-price">
                <span class="mkt-coin">🪙</span>
                <span class="mkt-price-val">{{ f.price }}</span>
              </div>
              <button class="mkt-buy-btn">Adquirir</button>
            </div>
          </div>
        </div>

        <!-- Trocas recentes -->
        <div class="mkt-section">
          <div class="mkt-section-header">
            <h3>🔄 Trocas Recentes</h3>
            <span>Últimas movimentações da comunidade</span>
          </div>
          <div class="mkt-trocas">
            <div class="mkt-troca" v-for="(t,i) in trocasRecentes" :key="i" :style="{'--i':i}">
              <div class="mt-user">{{ t.de[0] }}</div>
              <div class="mt-info">
                <span class="mt-names">{{ t.de }} → {{ t.para }}</span>
                <span class="mt-card">
                  <img :src="`https://flagcdn.com/w20/${t.code}.png`" :alt="t.pais"/>
                  {{ t.jogador }}
                </span>
              </div>
              <div class="mt-badge">✓ Concluída</div>
            </div>
          </div>
        </div>

        <!-- Banner de destaque -->
        <div class="mkt-banner">
          <div class="mkt-banner-bg"></div>
          <div class="mkt-banner-content">
            <div class="mkt-banner-left">
              <div class="mkt-banner-eyebrow">🌟 EVENTO ESPECIAL</div>
              <h3 class="mkt-banner-title">Pacote Copa 2026</h3>
              <p class="mkt-banner-desc">Ganhe figurinhas exclusivas das seleções anfitriãs com chance aumentada de raras.</p>
              <button class="mkt-banner-btn">Abrir Pacote 🎴</button>
            </div>
            <div class="mkt-banner-flags">
              <img src="https://flagcdn.com/w80/us.png" alt="USA"/>
              <img src="https://flagcdn.com/w80/ca.png" alt="CAN"/>
              <img src="https://flagcdn.com/w80/mx.png" alt="MEX"/>
            </div>
          </div>
        </div>

      </div>

      <!-- ══════ TAB: PERFIL ══════ -->
      <div v-else-if="navAtivo==='perfil'" key="perfil" class="perfil-wrap">

        <!-- Hero do perfil -->
        <div class="prf-hero">
          <div class="prf-hero-bg"></div>
          <div class="prf-hero-content">
            <div class="prf-avatar-area">
              <div class="prf-avatar-ring" :style="{borderColor: nivelCor(statsColecao.nivel)}">
                <div class="prf-avatar">{{ currentUser?.name?.[0]?.toUpperCase() }}</div>
              </div>
              <div class="prf-nivel-badge" :style="{background: nivelCor(statsColecao.nivel)+'22', borderColor: nivelCor(statsColecao.nivel)+'55', color: nivelCor(statsColecao.nivel)}">
                {{ statsColecao.nivel }}
              </div>
            </div>
            <div class="prf-identity">
              <h2 class="prf-name">{{ currentUser?.name }}</h2>
              <p class="prf-email">{{ currentUser?.email }}</p>
              <div class="prf-tags">
                <span class="prf-tag">⚽ Colecionador</span>
                <span class="prf-tag">🏆 Copa 2026</span>
                <span class="prf-tag">📅 2026</span>
              </div>
            </div>
            <div class="prf-xp-block">
              <div class="prf-xp-label">
                <span>XP: {{ statsColecao.xp }}</span>
                <span>Próximo: {{ statsColecao.xpProximo }}</span>
              </div>
              <div class="prf-xp-bar">
                <div class="prf-xp-fill" :style="{width: xpPct+'%'}"></div>
              </div>
              <div class="prf-xp-pct">{{ xpPct }}% para próximo nível</div>
            </div>
          </div>
          <div class="ah-bar"><span></span><span></span><span></span></div>
        </div>

        <!-- Stats grid -->
        <div class="prf-stats-grid">
          <div class="prf-stat-card" v-for="(s,i) in [
            {icon:'🃏',val:statsColecao.coletadas,lbl:'Figurinhas',sub:'de '+statsColecao.total,col:'var(--blue-lt)'},
            {icon:'💎',val:statsColecao.raras,lbl:'Raras',sub:'figurinhas especiais',col:'var(--gold)'},
            {icon:'📋',val:statsColecao.duplicadas,lbl:'Duplicadas',sub:'disponíveis p/ troca',col:'var(--green)'},
            {icon:'📊',val:colPct+'%',lbl:'Progresso',sub:'do álbum completo',col:'var(--red)'},
          ]" :key="i" :style="{'--i':i}">
            <div class="psc-icon">{{ s.icon }}</div>
            <div class="psc-val" :style="{color:s.col}">{{ s.val }}</div>
            <div class="psc-lbl">{{ s.lbl }}</div>
            <div class="psc-sub">{{ s.sub }}</div>
          </div>
        </div>

        <!-- Progresso do álbum -->
        <div class="prf-section">
          <h3 class="prf-section-title">📊 Progresso do Álbum</h3>
          <div class="prf-progress-card">
            <div class="ppc-header">
              <span>Figurinhas Coletadas</span>
              <span class="ppc-pct">{{ statsColecao.coletadas }}/{{ statsColecao.total }}</span>
            </div>
            <div class="ppc-bar">
              <div class="ppc-fill" :style="{width: colPct+'%'}">
                <div class="ppc-glow"></div>
              </div>
            </div>
            <div class="ppc-footer">
              <span>🔒 {{ statsColecao.total - statsColecao.coletadas }} restantes</span>
              <span>{{ colPct }}% completo</span>
            </div>
          </div>
        </div>

        <!-- Seleção favorita -->
        <div class="prf-section">
          <h3 class="prf-section-title">🏳️ Seleção Favorita</h3>
          <div class="prf-fav-grid">
            <div class="prf-fav-card" v-for="s in perfilFavoritas" :key="s"
              :class="{selected: perfilSeleçaoFavorita===s}"
              @click="perfilSeleçaoFavorita=s">
              <img :src="`https://flagcdn.com/w40/${flagsDestaque.find(f=>f.name===s)?.code||'br'}.png`" :alt="s"/>
              <span>{{ s }}</span>
              <div v-if="perfilSeleçaoFavorita===s" class="prf-fav-check">✓</div>
            </div>
          </div>
        </div>

        <!-- Conquistas -->
        <div class="prf-section">
          <h3 class="prf-section-title">🏅 Conquistas</h3>
          <div class="prf-conquistas">
            <div class="prf-conquista" v-for="(c,i) in conquistas" :key="i" :class="{unlocked:c.ok}" :style="{'--i':i}">
              <div class="pc-icon" :class="{locked:!c.ok}">{{ c.ok ? c.icon : '🔒' }}</div>
              <div class="pc-body">
                <div class="pc-nome">{{ c.nome }}</div>
                <div class="pc-desc">{{ c.desc }}</div>
              </div>
              <div v-if="c.ok" class="pc-badge">Desbloqueado</div>
            </div>
          </div>
        </div>

      </div>
      </transition>

    </main>
  </div>
  </transition>

</div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Rajdhani:wght@400;500;600;700&family=Syne:wght@400;500;600;700;800&display=swap');

*, *::before, *::after { margin:0; padding:0; box-sizing:border-box; }

:root {
  --red:      #C8102E;
  --red-dk:   #9a0c23;
  --blue:     #003DA5;
  --blue-dk:  #002070;
  --blue-lt:  #2563eb;
  --green:    #00a650;
  --green-dk: #007a3d;
  --gold:     #f5c842;

  --bg:       #0b0f18;
  --bg2:      #101520;
  --bg3:      #161d2a;
  --bg4:      #1c2537;
  --border:   rgba(255,255,255,0.07);
  --bdr2:     rgba(255,255,255,0.12);
  --text:     #e8edf5;
  --text-mid: #6b7a99;
  --text-dim: #344055;

  --sh-sm: 0 2px 8px rgba(0,0,0,.5);
  --sh-md: 0 8px 28px rgba(0,0,0,.55);
  --sh-lg: 0 16px 56px rgba(0,0,0,.65);
  --r: 12px;
  --sidebar-w: 272px;
}

html, body { min-height:100vh; background:var(--bg); font-family:'Syne',sans-serif; color:var(--text); -webkit-font-smoothing:antialiased; }

/* ══════════ ROOT / LAYOUT ══════════ */
.root { min-height:100vh; }
.app-shell { display:flex; min-height:100vh; }

/* ══════════════════════════════════════════
   AUTH SCREEN
══════════════════════════════════════════ */
.auth-screen {
  min-height:100vh; width:100%;
  display:flex; align-items:stretch;
  position:relative; overflow:hidden;
}
.auth-bg {
  position:absolute; inset:0;
  background:linear-gradient(135deg,#02060f 0%,#080f20 40%,#130610 80%,#1a0305 100%);
}
.auth-particles {
  position:absolute; inset:0; overflow:hidden; pointer-events:none;
}
.auth-particles span {
  position:absolute; bottom:-20px; left:var(--x);
  width:var(--s); height:var(--s); border-radius:50%;
  background:rgba(255,255,255,.06);
  animation:float-up var(--d) ease-in-out infinite;
  animation-delay: var(--del);
}
@keyframes float-up {
  0%   { transform:translateY(0) scale(1);   opacity:.6; }
  50%  { transform:translateY(-40vh) scale(1.2); opacity:.2; }
  100% { transform:translateY(-100vh) scale(.8); opacity:0; }
}

/* Card */
.auth-card {
  position:relative; z-index:2;
  width:460px; min-width:360px;
  margin:auto 0 auto 8vw;
  background:rgba(16,21,32,.92);
  border:1px solid rgba(255,255,255,.1);
  border-radius:20px;
  padding:40px 44px;
  backdrop-filter:blur(20px);
  box-shadow:0 32px 80px rgba(0,0,0,.7), 0 0 0 1px rgba(255,255,255,.05);
}

.auth-brand {
  display:flex; align-items:center; gap:14px;
  margin-bottom:32px;
}
.auth-emblem {
  width:52px; height:52px;
  background:linear-gradient(135deg,var(--blue),var(--red));
  border-radius:14px;
  display:flex; align-items:center; justify-content:center;
  font-size:26px; box-shadow:0 8px 24px rgba(0,61,165,.4);
  flex-shrink:0;
}
.auth-brand-name {
  font-family:'Bebas Neue',sans-serif;
  font-size:28px; letter-spacing:2px; color:#fff; line-height:1;
}
.auth-brand-sub {
  font-size:9px; font-weight:800; letter-spacing:2.5px;
  color:var(--gold); text-transform:uppercase;
}

/* Tabs */
.auth-tabs {
  display:flex; background:var(--bg3);
  border-radius:10px; padding:4px; gap:4px; margin-bottom:28px;
}
.auth-tab {
  flex:1; padding:10px; border:none; border-radius:8px;
  font-family:'Syne',sans-serif; font-size:14px; font-weight:700;
  cursor:pointer; transition:all .25s;
  background:transparent; color:var(--text-mid);
}
.auth-tab.active {
  background:linear-gradient(135deg,var(--blue),var(--blue-lt));
  color:#fff; box-shadow:0 4px 14px rgba(0,61,165,.45);
}

/* Error */
.auth-error {
  background:rgba(200,16,46,.15); border:1px solid rgba(200,16,46,.4);
  border-radius:8px; padding:10px 14px; font-size:13px; color:#ff8a9a;
  margin-bottom:16px;
}

/* Form */
.auth-form { display:flex; flex-direction:column; gap:16px; }
.auth-field { display:flex; flex-direction:column; gap:7px; }
.auth-field label { font-size:12px; font-weight:700; color:var(--text-mid); letter-spacing:.5px; }
.auth-input-wrap {
  position:relative; display:flex; align-items:center;
}
.auth-input-wrap svg {
  position:absolute; left:13px; width:16px; height:16px;
  color:var(--text-mid); flex-shrink:0; pointer-events:none;
}
.auth-input-wrap input {
  width:100%; padding:12px 14px 12px 40px;
  border-radius:9px; border:1px solid var(--border);
  background:var(--bg4); color:var(--text);
  font-family:'Syne',sans-serif; font-size:14px;
  outline:none; transition:border-color .2s, box-shadow .2s;
}
.auth-input-wrap input:focus {
  border-color:var(--blue-lt);
  box-shadow:0 0 0 3px rgba(37,99,235,.2);
}
.auth-input-wrap input::placeholder { color:var(--text-dim); }

.auth-hint {
  font-size:11px; color:var(--text-mid);
  background:rgba(245,200,66,.08); border:1px solid rgba(245,200,66,.2);
  border-radius:7px; padding:8px 12px;
}

.auth-btn {
  width:100%; padding:14px;
  background:linear-gradient(135deg,var(--blue),var(--blue-lt));
  border:none; border-radius:10px; color:#fff;
  font-family:'Syne',sans-serif; font-size:15px; font-weight:800;
  cursor:pointer; letter-spacing:.5px; margin-top:4px;
  box-shadow:0 6px 22px rgba(0,61,165,.45);
  transition:transform .2s, box-shadow .2s, opacity .2s;
  display:flex; align-items:center; justify-content:center;
  min-height:50px;
}
.auth-btn:hover:not(:disabled) {
  transform:translateY(-2px);
  box-shadow:0 10px 30px rgba(0,61,165,.55);
}
.auth-btn:disabled { opacity:.6; cursor:not-allowed; }
.auth-spinner {
  width:20px; height:20px; border-radius:50%;
  border:2px solid rgba(255,255,255,.3);
  border-top-color:#fff;
  animation:ring-spin .8s linear infinite;
}

/* Deco side */
.auth-deco {
  position:relative; z-index:1;
  flex:1; display:flex; flex-direction:column;
  align-items:center; justify-content:center; gap:28px;
}
.auth-deco-year {
  font-family:'Bebas Neue',sans-serif;
  font-size:clamp(80px,12vw,160px);
  color:rgba(255,255,255,.06);
  letter-spacing:10px; line-height:1;
  user-select:none;
  text-shadow:0 0 120px rgba(0,61,165,.3);
}
.auth-deco-flags {
  display:flex; align-items:center; gap:16px;
}
.auth-deco-flags img {
  width:60px; height:40px; object-fit:cover;
  border-radius:6px; border:1px solid rgba(255,255,255,.15);
  box-shadow:0 4px 16px rgba(0,0,0,.5);
}
.auth-deco-flags span {
  font-size:10px; font-weight:800; letter-spacing:1.5px;
  color:rgba(255,255,255,.35); text-transform:uppercase;
}
.auth-deco-trophy {
  font-size:80px;
  filter:drop-shadow(0 8px 30px rgba(245,200,66,.5));
  animation:trophy-float 4s ease-in-out infinite;
}
@keyframes trophy-float {
  0%,100%{ transform:translateY(0) rotate(-2deg); }
  50%    { transform:translateY(-14px) rotate(2deg); }
}
.auth-deco-tagline {
  font-size:clamp(16px,2vw,26px); text-align:center;
  color:rgba(255,255,255,.35); line-height:1.4;
  font-weight:500;
}
.auth-deco-tagline strong { color:rgba(255,255,255,.7); font-weight:800; }

/* ══════════ SIDEBAR ══════════ */
.sidebar {
  width:var(--sidebar-w); min-width:var(--sidebar-w);
  background:#080c13; border-right:1px solid var(--border);
  display:flex; flex-direction:column; gap:0;
  position:sticky; top:0; height:100vh;
  overflow-y:auto; scrollbar-width:none; flex-shrink:0;
}
.sidebar::-webkit-scrollbar { display:none; }

.sb-logo {
  display:flex; align-items:center; gap:11px;
  padding:20px 18px 18px;
  border-bottom:1px solid var(--border); flex-shrink:0;
}
.sb-emblem {
  width:40px; height:40px;
  background:linear-gradient(135deg,var(--blue),var(--red));
  border-radius:10px; display:flex; align-items:center; justify-content:center;
  font-size:20px; box-shadow:0 4px 14px rgba(0,61,165,.4); flex-shrink:0;
}
.sb-brand { display:flex; flex-direction:column; gap:1px; }
.sb-name { font-family:'Bebas Neue',sans-serif; font-size:21px; letter-spacing:1.5px; color:#fff; line-height:1; }
.sb-sub  { font-size:9px; font-weight:700; letter-spacing:2px; color:var(--gold); text-transform:uppercase; }

.sb-nav { display:flex; flex-direction:column; gap:2px; padding:14px 10px; }
.sb-link {
  display:flex; align-items:center; gap:9px;
  width:100%; padding:10px 12px;
  border:none; border-radius:9px;
  background:transparent; color:var(--text-mid);
  font-family:'Syne',sans-serif; font-size:14px; font-weight:600;
  cursor:pointer; transition:all .2s; text-align:left;
  border:1px solid transparent;
}
.sb-link:hover { background:rgba(255,255,255,.05); color:var(--text); }
.sb-link.active {
  background:linear-gradient(135deg,rgba(0,61,165,.3),rgba(200,16,46,.2));
  color:#fff; border-color:rgba(37,99,235,.35);
}
.sb-link.active .sb-ico { color:var(--blue-lt); opacity:1; }
.sb-ico { width:18px; height:18px; flex-shrink:0; opacity:.75; }
.sb-badge {
  margin-left:auto; font-size:8px; font-weight:700; letter-spacing:.5px;
  background:var(--green); color:#fff; padding:2px 7px; border-radius:20px;
}

.sb-divider { height:1px; background:var(--border); margin:0 10px; }

.sb-select-block { padding:14px 10px 10px; }
.sb-select-label {
  display:flex; align-items:center; gap:6px;
  font-size:9px; font-weight:700; letter-spacing:1.5px;
  color:var(--text-dim); text-transform:uppercase; margin-bottom:8px;
}
.sb-select-wrap { position:relative; }
.sb-select {
  width:100%; padding:10px 34px 10px 12px;
  border-radius:8px; border:1px solid var(--border);
  background:var(--bg3); color:var(--text);
  font-family:'Syne',sans-serif; font-size:13px;
  cursor:pointer; appearance:none; -webkit-appearance:none;
  outline:none; transition:border-color .2s, box-shadow .2s;
}
.sb-select:focus { border-color:var(--blue-lt); box-shadow:0 0 0 3px rgba(37,99,235,.2); }
.sb-select option { background:#1a2230; }
.sb-chevron { position:absolute; right:10px; top:50%; transform:translateY(-50%); color:var(--text-mid); pointer-events:none; width:14px; height:14px; }

.sb-team-card { margin:0 10px; background:var(--bg3); border:1px solid var(--border); border-radius:10px; overflow:hidden; }
.stc-header { display:flex; align-items:center; gap:11px; padding:12px 13px; background:linear-gradient(135deg,rgba(0,61,165,.2),rgba(200,16,46,.15)); border-bottom:1px solid var(--border); }
.stc-flag { width:36px; height:24px; object-fit:cover; border-radius:4px; box-shadow:var(--sh-sm); }
.stc-info { display:flex; flex-direction:column; gap:2px; }
.stc-name { font-family:'Rajdhani',sans-serif; font-size:15px; font-weight:700; color:#fff; letter-spacing:.5px; }
.stc-label { font-size:10px; color:var(--text-mid); }
.stc-stats { display:flex; align-items:center; padding:10px 13px; }
.stcs { flex:1; display:flex; flex-direction:column; align-items:center; gap:2px; }
.stcs-v { font-family:'Bebas Neue',sans-serif; font-size:22px; color:#fff; line-height:1; }
.stcs-l { font-size:9px; color:var(--text-mid); text-transform:uppercase; letter-spacing:.5px; }
.stcs-div { width:1px; height:30px; background:var(--border); }

/* User card */
.sb-user {
  display:flex; align-items:center; gap:10px;
  margin:8px 10px 0; padding:11px 13px;
  background:var(--bg3); border:1px solid var(--border);
  border-radius:10px; cursor:pointer;
  transition:border-color .2s, background .2s;
}
.sb-user:hover { border-color:var(--bdr2); background:var(--bg4); }
.sb-user-avatar {
  width:36px; height:36px; border-radius:50%;
  background:linear-gradient(135deg,var(--blue),var(--blue-lt));
  display:flex; align-items:center; justify-content:center;
  font-size:16px; font-weight:800; color:#fff; flex-shrink:0;
}
.sb-user-info { display:flex; flex-direction:column; gap:2px; min-width:0; }
.sb-user-name { font-size:13px; font-weight:700; color:#fff; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
.sb-user-level { font-size:10px; color:var(--gold); }

.sb-logout {
  display:flex; align-items:center; gap:8px;
  margin:6px 10px 0; padding:9px 13px;
  border:1px solid rgba(200,16,46,.2);
  border-radius:9px; background:transparent;
  color:var(--text-mid); font-family:'Syne',sans-serif;
  font-size:13px; font-weight:600; cursor:pointer;
  transition:all .2s; width:calc(100% - 20px);
}
.sb-logout:hover { background:rgba(200,16,46,.1); color:#ff8a9a; border-color:rgba(200,16,46,.4); }

/* Copa badge */
.sb-copa { margin:auto 10px 14px; background:var(--bg3); border:1px solid var(--border); border-radius:10px; overflow:hidden; }
.sb-copa-stripe-top { display:flex; height:3px; }
.sb-copa-stripe-top span { flex:1; }
.sb-copa-body { display:flex; align-items:center; gap:12px; padding:14px 13px; }
.sb-copa-trophy { font-size:30px; filter:drop-shadow(0 2px 6px rgba(0,0,0,.5)); }
.sb-copa-title { font-size:9px; font-weight:700; letter-spacing:1.5px; color:var(--text-mid); }
.sb-copa-year { font-family:'Bebas Neue',sans-serif; font-size:26px; color:#fff; letter-spacing:2px; line-height:1; }
.sb-copa-hosts { font-size:10px; color:var(--gold); font-weight:700; }

/* ══════════ MAIN ══════════ */
.main { flex:1; min-width:0; overflow-y:auto; background:var(--bg); }

/* ══════════ HERO HEADER ══════════ */
.hero { position:relative; overflow:hidden; }
.hero-mesh { position:absolute; inset:0; background:linear-gradient(135deg,#03091e 0%,#0c1830 40%,#10203e 70%,#1a080f 100%); }
.hero-grid {
  position:absolute; inset:0;
  background-image:
    repeating-linear-gradient(0deg,transparent,transparent 39px,rgba(255,255,255,.02) 39px,rgba(255,255,255,.02) 40px),
    repeating-linear-gradient(90deg,transparent,transparent 39px,rgba(255,255,255,.02) 39px,rgba(255,255,255,.02) 40px);
}
.hero-inner { position:relative; z-index:1; display:flex; align-items:center; justify-content:space-between; padding:36px 44px 28px; }
.hero-left  { display:flex; flex-direction:column; gap:10px; }
.hero-eyebrow { display:inline-flex; align-items:center; gap:8px; background:rgba(255,255,255,.07); border:1px solid rgba(255,255,255,.12); border-radius:20px; padding:5px 16px; font-size:11px; font-weight:700; letter-spacing:1.5px; color:rgba(255,255,255,.65); text-transform:uppercase; width:fit-content; }
.hero-title { font-family:'Bebas Neue',sans-serif; font-size:56px; line-height:.9; letter-spacing:2px; color:#fff; }
.hero-accent { background:linear-gradient(90deg,var(--gold),#ffe98a); -webkit-background-clip:text; -webkit-text-fill-color:transparent; background-clip:text; }
.hero-sub { font-size:14px; color:rgba(255,255,255,.4); font-weight:500; }
.hero-right { display:flex; flex-direction:column; align-items:flex-end; gap:12px; }
.hero-year-bg { font-family:'Bebas Neue',sans-serif; font-size:96px; line-height:.85; letter-spacing:5px; color:rgba(255,255,255,.9); text-shadow:0 0 80px rgba(255,255,255,.07); user-select:none; }
.hero-hosts-row { display:flex; gap:10px; }
.hero-host-flag { display:flex; flex-direction:column; align-items:center; gap:6px; background:rgba(255,255,255,.06); border:1px solid rgba(255,255,255,.12); border-radius:10px; padding:8px 12px; transition:transform .2s, border-color .2s, background .2s; }
.hero-host-flag:hover { transform:translateY(-3px); border-color:rgba(255,255,255,.25); background:rgba(255,255,255,.1); }
.host-flag-img { width:48px; height:32px; object-fit:cover; border-radius:5px; box-shadow:0 2px 10px rgba(0,0,0,.5); border:1px solid rgba(255,255,255,.15); }
.hero-host-flag span { font-size:10px; font-weight:800; letter-spacing:1.5px; color:rgba(255,255,255,.6); text-transform:uppercase; }
.hero-bar { display:flex; height:4px; position:relative; z-index:1; }
.hero-bar span:nth-child(1){ flex:1; background:var(--red); }
.hero-bar span:nth-child(2){ flex:1; background:var(--blue); }
.hero-bar span:nth-child(3){ flex:1; background:var(--green); }

/* ══════════ TAB TRANSITION ══════════ */
.tab-fade-enter-active, .tab-fade-leave-active { transition:opacity .25s, transform .25s; }
.tab-fade-enter-from { opacity:0; transform:translateY(10px); }
.tab-fade-leave-to   { opacity:0; transform:translateY(-6px); }

/* ══════════ EMPTY STATE ══════════ */
.empty-wrap { display:flex; flex-direction:column; gap:20px; padding:24px 40px 40px; }
.empty-banner { position:relative; border-radius:16px; overflow:hidden; border:1px solid var(--border); }
.eb-mesh { position:absolute; inset:0; background:linear-gradient(135deg,rgba(0,61,165,.35) 0%,rgba(0,10,40,.9) 45%,rgba(40,0,10,.85) 75%,rgba(200,16,46,.25) 100%); }
.eb-content { position:relative; z-index:1; display:flex; align-items:center; justify-content:space-between; padding:48px 52px; }
.eb-left { display:flex; flex-direction:column; gap:14px; max-width:600px; }
.eb-eyebrow { font-size:11px; font-weight:700; letter-spacing:2px; color:var(--gold); text-transform:uppercase; }
.eb-title { font-family:'Bebas Neue',sans-serif; font-size:52px; line-height:.9; letter-spacing:2px; color:#fff; }
.eb-title span { background:linear-gradient(90deg,var(--gold),#ffe98a); -webkit-background-clip:text; -webkit-text-fill-color:transparent; background-clip:text; }
.eb-desc { font-size:15px; color:rgba(255,255,255,.55); line-height:1.6; max-width:460px; }
.eb-arrow { display:inline-flex; align-items:center; gap:8px; font-size:13px; font-weight:700; color:var(--blue-lt); background:rgba(37,99,235,.15); border:1px solid rgba(37,99,235,.3); padding:8px 18px; border-radius:20px; width:fit-content; }
.eb-right { display:flex; align-items:center; justify-content:center; }
.eb-trophy-wrap { position:relative; width:130px; height:130px; display:flex; align-items:center; justify-content:center; }
.eb-trophy-ring { position:absolute; border-radius:50%; animation:pulse-ring 3s ease-in-out infinite; }
.eb-trophy-ring.r1 { width:130px; height:130px; border:1px solid rgba(240,180,41,.25); background:rgba(240,180,41,.05); }
.eb-trophy-ring.r2 { width:90px; height:90px; border:1px solid rgba(240,180,41,.3); background:rgba(240,180,41,.08); animation-delay:.7s; }
.eb-trophy { font-size:52px; position:relative; z-index:1; filter:drop-shadow(0 4px 16px rgba(240,180,41,.4)); }
@keyframes pulse-ring { 0%,100%{transform:scale(1);opacity:1} 50%{transform:scale(1.06);opacity:.7} }
.eb-bar { display:flex; height:4px; }
.eb-bar span:nth-child(1){ flex:1; background:var(--red); }
.eb-bar span:nth-child(2){ flex:1; background:var(--blue); }
.eb-bar span:nth-child(3){ flex:1; background:var(--green); }
.empty-stats { display:grid; grid-template-columns:repeat(4,1fr); gap:16px; }
.es-card { background:var(--bg3); border:1px solid var(--border); border-radius:12px; padding:24px 20px; display:flex; flex-direction:column; align-items:center; gap:8px; transition:transform .2s, border-color .2s; }
.es-card:hover { transform:translateY(-4px); border-color:var(--bdr2); }
.es-icon { font-size:28px; }
.es-val  { font-family:'Bebas Neue',sans-serif; font-size:36px; line-height:1; }
.es-lbl  { font-size:12px; color:var(--text-mid); font-weight:600; text-align:center; }
.empty-featured { background:var(--bg3); border:1px solid var(--border); border-radius:16px; padding:28px 32px; }
.ef-header { display:flex; align-items:baseline; gap:14px; margin-bottom:20px; }
.ef-title  { font-family:'Bebas Neue',sans-serif; font-size:22px; letter-spacing:1px; color:#fff; }
.ef-sub    { font-size:12px; color:var(--text-mid); }
.ef-flags  { display:grid; grid-template-columns:repeat(8,1fr); gap:12px; }
.ef-flag-card { display:flex; flex-direction:column; align-items:center; gap:8px; }
.efc-img-wrap { position:relative; width:100%; aspect-ratio:16/10; overflow:hidden; border-radius:8px; border:1px solid var(--bdr2); box-shadow:var(--sh-md); transition:transform .25s cubic-bezier(.34,1.56,.64,1), box-shadow .25s; }
.efc-img-wrap:hover { transform:translateY(-4px) scale(1.06); box-shadow:var(--sh-lg); }
.efc-img  { width:100%; height:100%; object-fit:cover; display:block; }
.efc-shine { position:absolute; inset:0; background:linear-gradient(135deg,rgba(255,255,255,.12) 0%,transparent 60%); pointer-events:none; }
.efc-name { font-size:11px; font-weight:700; color:var(--text-mid); text-align:center; }
.empty-howto { background:var(--bg3); border:1px solid var(--border); border-radius:16px; padding:28px 32px; }
.ht-title  { font-family:'Bebas Neue',sans-serif; font-size:22px; letter-spacing:1px; color:#fff; margin-bottom:20px; }
.ht-steps  { display:grid; grid-template-columns:repeat(3,1fr); gap:24px; }
.ht-step   { display:flex; align-items:flex-start; gap:16px; padding:20px; border-radius:10px; background:var(--bg4); border:1px solid var(--border); }
.hts-num   { font-family:'Bebas Neue',sans-serif; font-size:32px; line-height:1; color:rgba(255,255,255,.1); flex-shrink:0; }
.hts-body  { display:flex; flex-direction:column; gap:6px; }
.hts-title { font-size:14px; font-weight:700; color:#fff; }
.hts-desc  { font-size:13px; color:var(--text-mid); line-height:1.5; }

/* ══════════ LOADING ══════════ */
.loading { display:flex; align-items:center; justify-content:center; min-height:400px; padding:40px; }
.ld-card  { display:flex; flex-direction:column; align-items:center; gap:18px; background:var(--bg3); border:1px solid var(--border); border-radius:16px; padding:56px 64px; }
.ld-spinner { display:inline-block; position:relative; width:60px; height:60px; }
.ld-spinner div { box-sizing:border-box; display:block; position:absolute; width:48px; height:48px; margin:6px; border:4px solid transparent; border-radius:50%; animation:ring-spin 1.2s cubic-bezier(.5,0,.5,1) infinite; }
.ld-spinner div:nth-child(1){ border-top-color:var(--blue-lt); animation-delay:-.45s; }
.ld-spinner div:nth-child(2){ border-top-color:var(--red);     animation-delay:-.3s; }
.ld-spinner div:nth-child(3){ border-top-color:var(--green);   animation-delay:-.15s; }
.ld-spinner div:nth-child(4){ border-top-color:var(--gold); }
@keyframes ring-spin { 0%{transform:rotate(0)} 100%{transform:rotate(360deg)} }
.ld-text { font-family:'Rajdhani',sans-serif; font-size:20px; font-weight:700; color:var(--text); }
.ld-sub  { font-size:14px; color:var(--text-mid); }

/* ══════════ ALBUM ══════════ */
.album { display:flex; flex-direction:column; gap:20px; padding:24px 40px 40px; }
.album-hero { position:relative; border-radius:16px; overflow:hidden; border:1px solid var(--border); flex-shrink:0; }
.ah-bg { position:absolute; inset:0; background:linear-gradient(135deg,rgba(0,61,165,.3) 0%,rgba(0,0,0,.95) 50%,rgba(200,16,46,.2) 100%); }
.ah-pattern { position:absolute; inset:0; background-image:radial-gradient(rgba(255,255,255,.025) 1px,transparent 1px); background-size:22px 22px; }
.ah-content { position:relative; z-index:1; display:flex; align-items:center; gap:36px; padding:32px 40px; flex-wrap:wrap; }
.ah-flag-block { flex-shrink:0; }
.ah-flag-frame { position:relative; width:120px; height:80px; }
.ah-flag { width:120px; height:80px; object-fit:cover; border-radius:8px; border:2px solid rgba(255,255,255,.2); box-shadow:0 6px 28px rgba(0,0,0,.6); }
.ah-flag-glow { position:absolute; inset:-8px; background:radial-gradient(ellipse at center,rgba(0,61,165,.4),transparent 70%); border-radius:14px; z-index:-1; }
.ah-info { flex:1; min-width:200px; display:flex; flex-direction:column; gap:10px; }
.ah-label { font-size:10px; font-weight:700; letter-spacing:2px; color:var(--text-mid); text-transform:uppercase; }
.ah-name  { font-family:'Bebas Neue',sans-serif; font-size:48px; color:#fff; letter-spacing:3px; line-height:1; }
.ah-tags  { display:flex; flex-wrap:wrap; gap:8px; }
.ah-tag   { display:inline-flex; align-items:center; gap:5px; font-size:12px; font-weight:700; color:var(--text-mid); background:rgba(255,255,255,.07); border:1px solid var(--border); padding:5px 12px; border-radius:7px; }
.ah-tag.gold  { color:var(--gold); border-color:rgba(245,200,66,.3); background:rgba(245,200,66,.08); }
.ah-tag.green { color:var(--green); border-color:rgba(0,166,80,.3); background:rgba(0,166,80,.08); }
.ah-ring-wrap { position:relative; width:100px; height:100px; flex-shrink:0; }
.ah-ring-svg  { width:100px; height:100px; transform:rotate(-90deg); }
.ah-ring-center { position:absolute; inset:0; display:flex; flex-direction:column; align-items:center; justify-content:center; }
.ah-ring-n { font-family:'Bebas Neue',sans-serif; font-size:28px; color:#fff; line-height:1; }
.ah-ring-l { font-size:9px; color:var(--text-mid); letter-spacing:.5px; }
.ah-mini-stats { display:flex; gap:0; background:rgba(255,255,255,.04); border:1px solid var(--border); border-radius:10px; overflow:hidden; }
.ams-item { flex:1; display:flex; flex-direction:column; align-items:center; padding:14px 10px; gap:4px; border-right:1px solid var(--border); }
.ams-item:last-child { border-right:none; }
.ams-v { font-family:'Bebas Neue',sans-serif; font-size:26px; color:#fff; line-height:1; }
.ams-l { font-size:9px; color:var(--text-mid); text-transform:uppercase; letter-spacing:.5px; }
.ah-bar { display:flex; height:3px; }
.ah-bar span:nth-child(1){ flex:1; background:var(--red); }
.ah-bar span:nth-child(2){ flex:1; background:var(--blue); }
.ah-bar span:nth-child(3){ flex:1; background:var(--green); }
.album-section-bar { display:flex; align-items:center; justify-content:space-between; padding:0 4px; }
.asb-left  { display:flex; align-items:center; gap:10px; }
.asb-dot   { width:8px; height:8px; border-radius:50%; background:var(--green); box-shadow:0 0 8px var(--green); }
.asb-title { font-size:11px; font-weight:800; letter-spacing:2px; color:var(--text-mid); }
.asb-count { font-size:12px; color:var(--text-dim); background:var(--bg3); border:1px solid var(--border); padding:2px 10px; border-radius:20px; }
.asb-page  { font-size:13px; color:var(--text-mid); font-weight:600; }

/* ══════════ STICKERS ══════════ */
.stickers-grid { display:grid; grid-template-columns:repeat(6,1fr); gap:16px; }
.sticker {
  position:relative; background:var(--bg3); border-radius:12px; overflow:hidden;
  border:1px solid var(--border); box-shadow:var(--sh-md); cursor:pointer;
  display:flex; flex-direction:column;
  animation:stk-in .45s cubic-bezier(.34,1.56,.64,1) both;
  animation-delay:calc(var(--i)*.05s);
  transition:transform .28s cubic-bezier(.34,1.56,.64,1), box-shadow .28s, border-color .28s;
}
@keyframes stk-in { from{opacity:0;transform:translateY(24px) scale(.88)} to{opacity:1;transform:translateY(0) scale(1)} }
.sticker:hover {
  transform:translateY(-8px) scale(1.04) rotate(.4deg);
  box-shadow:0 24px 48px rgba(0,0,0,.7), 0 0 0 1px rgba(37,99,235,.45), 0 0 30px rgba(37,99,235,.15);
  border-color:rgba(37,99,235,.5); z-index:5;
}
.stk-num { position:absolute; top:-7px; left:-7px; z-index:10; width:24px; height:24px; background:linear-gradient(135deg,var(--blue),var(--blue-dk)); border:2px solid var(--bg); border-radius:50%; font-size:9px; font-weight:800; color:#fff; display:flex; align-items:center; justify-content:center; font-family:'Rajdhani',sans-serif; box-shadow:0 2px 10px rgba(0,0,0,.6); }
.stk-glare { position:absolute; inset:0; background:linear-gradient(135deg,rgba(255,255,255,.09) 0%,transparent 55%); border-radius:12px; z-index:4; pointer-events:none; opacity:0; transition:opacity .3s; }
.sticker:hover .stk-glare { opacity:1; }
.stk-head { display:flex; align-items:center; justify-content:space-between; padding:7px 9px 6px; background:linear-gradient(90deg,var(--blue-dk) 0%,var(--blue) 55%,#8b0d1f 100%); flex-shrink:0; }
.stk-head-l { display:flex; align-items:center; gap:5px; }
.stk-flag   { width:22px; height:15px; object-fit:cover; border-radius:2px; box-shadow:0 1px 4px rgba(0,0,0,.5); }
.stk-country{ font-size:8px; font-weight:800; letter-spacing:.5px; color:rgba(255,255,255,.7); }
.stk-ball   { font-size:12px; opacity:.75; }
.stk-photo-wrap { position:relative; width:100%; aspect-ratio:1; overflow:hidden; background:linear-gradient(180deg,#0d1628 0%,#080c14 100%); flex-shrink:0; }
.stk-photo-bg { position:absolute; inset:0; background:radial-gradient(ellipse 80% 60% at 50% 20%,rgba(0,61,165,.22),transparent 70%); }
.stk-photo { width:100%; height:100%; object-fit:cover; display:block; position:relative; z-index:1; transition:transform .35s cubic-bezier(.34,1.56,.64,1); }
.sticker:hover .stk-photo { transform:scale(1.1); }
.stk-photo-grad { position:absolute; bottom:0; left:0; right:0; height:45%; background:linear-gradient(transparent,var(--bg3)); z-index:2; }
.stk-overall { position:absolute; bottom:8px; right:8px; z-index:3; display:flex; flex-direction:column; align-items:center; background:rgba(0,0,0,.75); backdrop-filter:blur(6px); border:1px solid rgba(255,255,255,.12); border-radius:7px; padding:4px 8px; min-width:36px; }
.so-val { font-family:'Bebas Neue',sans-serif; font-size:18px; color:#fff; line-height:1; }
.so-lbl { font-size:7px; color:var(--gold); font-weight:800; letter-spacing:.5px; }
.stk-identity { padding:9px 9px 4px; text-align:center; flex-shrink:0; }
.stk-name { font-family:'Rajdhani',sans-serif; font-size:12px; font-weight:700; color:var(--text); text-transform:uppercase; letter-spacing:.3px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; line-height:1.2; }
.stk-pos  { font-size:8px; font-weight:800; letter-spacing:1px; color:#fff; text-transform:uppercase; background:linear-gradient(90deg,var(--blue),var(--blue-lt)); padding:2px 9px; border-radius:4px; display:inline-block; margin-top:4px; }
.stk-attrs { display:flex; flex-direction:column; gap:4px; padding:6px 9px 4px; flex:1; }
.attr { display:grid; grid-template-columns:22px 1fr 20px; align-items:center; gap:5px; }
.attr-s { font-size:7px; font-weight:800; letter-spacing:.5px; color:var(--text-mid); text-transform:uppercase; }
.attr-bar-bg   { height:3px; background:rgba(255,255,255,.07); border-radius:3px; overflow:hidden; }
.attr-bar-fill { height:100%; border-radius:3px; transition:width .9s cubic-bezier(.16,1,.3,1); }
.attr-v { font-family:'Rajdhani',sans-serif; font-size:10px; font-weight:800; text-align:right; }
.stk-foot { display:flex; align-items:center; justify-content:center; gap:5px; padding:5px 8px 8px; border-top:1px solid var(--border); background:rgba(255,255,255,.02); flex-shrink:0; }
.stk-foot span { font-size:7px; font-weight:800; letter-spacing:1px; color:var(--text-dim); }
.stk-foot-dot  { width:2px; height:2px; border-radius:50%; background:var(--text-dim); }

/* ══════════ PAGINAÇÃO ══════════ */
.pagination { display:flex; align-items:center; justify-content:space-between; background:var(--bg3); border:1px solid var(--border); border-radius:14px; padding:16px 24px; gap:12px; }
.pg-btn { display:flex; align-items:center; gap:6px; padding:10px 20px; border-radius:9px; border:1px solid var(--border); background:var(--bg4); color:var(--text-mid); font-family:'Syne',sans-serif; font-size:13px; font-weight:700; cursor:pointer; transition:all .2s; }
.pg-btn:hover:not(:disabled) { border-color:var(--blue-lt); color:#fff; background:rgba(37,99,235,.15); box-shadow:0 0 0 3px rgba(37,99,235,.15); }
.pg-next:hover:not(:disabled) { border-color:var(--red); background:rgba(200,16,46,.15); box-shadow:0 0 0 3px rgba(200,16,46,.15); }
.pg-btn:disabled { opacity:.3; cursor:not-allowed; }
.pg-dots { display:flex; align-items:center; gap:4px; }
.pg-dot { width:36px; height:36px; border-radius:9px; border:1px solid var(--border); background:transparent; color:var(--text-mid); font-family:'Rajdhani',sans-serif; font-size:14px; font-weight:700; cursor:pointer; transition:all .2s; display:flex; align-items:center; justify-content:center; }
.pg-dot:hover { background:rgba(255,255,255,.06); color:var(--text); }
.pg-dot.near  { color:var(--text); border-color:rgba(255,255,255,.15); }
.pg-dot.active { background:linear-gradient(135deg,var(--blue),var(--blue-lt)); border-color:var(--blue-lt); color:#fff; box-shadow:0 4px 14px rgba(0,61,165,.45); transform:scale(1.12); }
.pg-ellipsis { width:24px; text-align:center; color:var(--text-dim); font-size:14px; display:flex; align-items:center; justify-content:center; }

/* ══════════ MERCADO ══════════ */
.mercado-wrap { display:flex; flex-direction:column; gap:24px; padding:24px 40px 40px; }

.mkt-filters { display:flex; gap:10px; }
.mkt-filter-btn { padding:10px 22px; border-radius:10px; border:1px solid var(--border); background:var(--bg3); color:var(--text-mid); font-family:'Syne',sans-serif; font-size:13px; font-weight:700; cursor:pointer; transition:all .2s; }
.mkt-filter-btn:hover  { border-color:var(--bdr2); color:var(--text); }
.mkt-filter-btn.active { background:linear-gradient(135deg,var(--blue),var(--blue-lt)); border-color:var(--blue-lt); color:#fff; box-shadow:0 4px 16px rgba(0,61,165,.35); }

.mkt-grid { display:grid; grid-template-columns:repeat(auto-fill,minmax(200px,1fr)); gap:18px; }
.mkt-card {
  position:relative; background:var(--bg3); border-radius:14px; overflow:hidden;
  border:1px solid var(--border); display:flex; flex-direction:column;
  animation:stk-in .4s cubic-bezier(.34,1.56,.64,1) both;
  animation-delay:calc(var(--i)*.06s);
  transition:transform .28s cubic-bezier(.34,1.56,.64,1), box-shadow .28s, border-color .28s;
}
.mkt-card:hover {
  transform:translateY(-7px) scale(1.03);
  box-shadow:0 20px 50px rgba(0,0,0,.7);
  border-color:rgba(37,99,235,.4);
  z-index:3;
}
.mkt-card[style*="--rare: 1"] { border-color:rgba(245,200,66,.25); }
.mkt-card[style*="--rare: 1"]:hover { border-color:rgba(245,200,66,.6); box-shadow:0 20px 50px rgba(0,0,0,.7), 0 0 40px rgba(245,200,66,.15); }
.mkt-rare-badge { position:absolute; top:10px; right:10px; z-index:10; background:linear-gradient(135deg,#b8760a,var(--gold)); color:#000; font-size:9px; font-weight:900; padding:4px 10px; border-radius:20px; letter-spacing:.5px; box-shadow:0 2px 10px rgba(245,200,66,.4); }
.mkt-card-glow { position:absolute; inset:0; background:radial-gradient(ellipse at top,rgba(245,200,66,.08),transparent 60%); pointer-events:none; }
.mkt-photo-wrap { position:relative; aspect-ratio:1; overflow:hidden; background:linear-gradient(180deg,var(--bg4),var(--bg2)); }
.mkt-photo { width:100%; height:100%; object-fit:cover; transition:transform .35s cubic-bezier(.34,1.56,.64,1); }
.mkt-card:hover .mkt-photo { transform:scale(1.08); }
.mkt-photo-overlay { position:absolute; bottom:0; left:0; right:0; height:50%; background:linear-gradient(transparent,var(--bg3)); }
/* Avatar fallback para o mercado */
.mkt-avatar {
  width:100%; height:100%; display:flex; flex-direction:column;
  align-items:center; justify-content:center; gap:8px;
  position:relative;
}
.mkt-avatar-initials {
  font-family:'Bebas Neue',sans-serif; font-size:52px; color:rgba(255,255,255,.9);
  line-height:1; text-shadow:0 2px 16px rgba(0,0,0,.5);
}
.mkt-avatar-flag {
  width:40px; height:27px; object-fit:cover; border-radius:5px;
  border:2px solid rgba(255,255,255,.3); box-shadow:0 2px 10px rgba(0,0,0,.4);
}
/* Photo fallback for album stickers */
.stk-photo-fallback {
  position:absolute; inset:0; z-index:1;
  display:flex; align-items:center; justify-content:center;
  font-family:'Bebas Neue',sans-serif; font-size:38px;
  color:rgba(255,255,255,.25); letter-spacing:2px;
}
.mkt-ovr { position:absolute; bottom:10px; left:10px; display:flex; flex-direction:column; align-items:center; background:rgba(0,0,0,.8); backdrop-filter:blur(8px); border:1px solid rgba(255,255,255,.12); border-radius:8px; padding:4px 10px; }
.mkt-ovr span { font-family:'Bebas Neue',sans-serif; font-size:20px; color:#fff; line-height:1; }
.mkt-ovr small { font-size:7px; color:var(--gold); font-weight:800; letter-spacing:.5px; }
.mkt-info { padding:12px 14px 8px; display:flex; flex-direction:column; gap:6px; }
.mkt-flag-row { display:flex; align-items:center; gap:6px; }
.mkt-flag    { width:24px; height:16px; object-fit:cover; border-radius:3px; box-shadow:var(--sh-sm); }
.mkt-country { font-size:10px; color:var(--text-mid); font-weight:700; }
.mkt-name    { font-family:'Rajdhani',sans-serif; font-size:15px; font-weight:700; color:#fff; text-transform:uppercase; letter-spacing:.3px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
.mkt-pos-tag { display:inline-block; font-size:9px; font-weight:800; letter-spacing:.5px; color:#fff; background:linear-gradient(90deg,var(--blue),var(--blue-lt)); padding:3px 10px; border-radius:5px; width:fit-content; }
.mkt-footer { display:flex; align-items:center; justify-content:space-between; padding:10px 14px 14px; border-top:1px solid var(--border); margin-top:auto; }
.mkt-price  { display:flex; align-items:center; gap:5px; }
.mkt-coin   { font-size:16px; }
.mkt-price-val { font-family:'Bebas Neue',sans-serif; font-size:22px; color:var(--gold); letter-spacing:1px; }
.mkt-buy-btn { padding:7px 16px; border-radius:8px; border:none; background:linear-gradient(135deg,var(--blue),var(--blue-lt)); color:#fff; font-family:'Syne',sans-serif; font-size:12px; font-weight:800; cursor:pointer; transition:transform .2s, box-shadow .2s; }
.mkt-buy-btn:hover { transform:scale(1.05); box-shadow:0 4px 16px rgba(0,61,165,.45); }

.mkt-section { background:var(--bg3); border:1px solid var(--border); border-radius:16px; padding:26px 32px; }
.mkt-section-header { display:flex; align-items:baseline; gap:14px; margin-bottom:18px; }
.mkt-section-header h3 { font-family:'Bebas Neue',sans-serif; font-size:20px; color:#fff; letter-spacing:1px; }
.mkt-section-header span { font-size:12px; color:var(--text-mid); }
.mkt-trocas { display:flex; flex-direction:column; gap:10px; }
.mkt-troca {
  display:flex; align-items:center; gap:14px;
  background:var(--bg4); border:1px solid var(--border);
  border-radius:10px; padding:14px 18px;
  animation:stk-in .35s both; animation-delay:calc(var(--i)*.08s);
  transition:border-color .2s;
}
.mkt-troca:hover { border-color:var(--bdr2); }
.mt-user { width:36px; height:36px; border-radius:50%; background:linear-gradient(135deg,var(--blue),var(--blue-lt)); display:flex; align-items:center; justify-content:center; font-weight:800; font-size:15px; color:#fff; flex-shrink:0; }
.mt-info { display:flex; flex-direction:column; gap:3px; flex:1; min-width:0; }
.mt-names { font-size:13px; font-weight:700; color:#fff; }
.mt-card  { display:flex; align-items:center; gap:6px; font-size:12px; color:var(--text-mid); }
.mt-card img { width:20px; height:14px; object-fit:cover; border-radius:2px; }
.mt-badge { font-size:11px; font-weight:700; color:var(--green); background:rgba(0,166,80,.12); border:1px solid rgba(0,166,80,.25); padding:4px 12px; border-radius:20px; }

.mkt-banner { position:relative; border-radius:16px; overflow:hidden; border:1px solid rgba(245,200,66,.2); }
.mkt-banner-bg { position:absolute; inset:0; background:linear-gradient(135deg,rgba(120,80,0,.6) 0%,rgba(0,0,0,.9) 50%,rgba(0,30,80,.5) 100%); }
.mkt-banner-content { position:relative; z-index:1; display:flex; align-items:center; justify-content:space-between; padding:40px 48px; }
.mkt-banner-left { display:flex; flex-direction:column; gap:12px; max-width:500px; }
.mkt-banner-eyebrow { font-size:11px; font-weight:700; letter-spacing:2px; color:var(--gold); text-transform:uppercase; }
.mkt-banner-title { font-family:'Bebas Neue',sans-serif; font-size:38px; color:#fff; letter-spacing:2px; line-height:1; }
.mkt-banner-desc { font-size:14px; color:rgba(255,255,255,.55); line-height:1.6; }
.mkt-banner-btn { display:inline-block; padding:12px 26px; background:linear-gradient(135deg,#b8760a,var(--gold)); color:#000; font-family:'Syne',sans-serif; font-size:14px; font-weight:800; border:none; border-radius:10px; cursor:pointer; box-shadow:0 6px 22px rgba(245,200,66,.3); transition:transform .2s, box-shadow .2s; width:fit-content; }
.mkt-banner-btn:hover { transform:translateY(-2px); box-shadow:0 10px 30px rgba(245,200,66,.4); }
.mkt-banner-flags { display:flex; gap:12px; align-items:center; }
.mkt-banner-flags img { width:70px; height:46px; object-fit:cover; border-radius:8px; border:2px solid rgba(255,255,255,.2); box-shadow:0 4px 18px rgba(0,0,0,.5); }

/* ══════════ PERFIL ══════════ */
.perfil-wrap { display:flex; flex-direction:column; gap:24px; padding:24px 40px 40px; }

.prf-hero { position:relative; border-radius:16px; overflow:hidden; border:1px solid var(--border); }
.prf-hero-bg { position:absolute; inset:0; background:linear-gradient(135deg,rgba(0,61,165,.35),rgba(0,0,0,.9) 50%,rgba(200,16,46,.2)); }
.prf-hero-bg::after { content:''; position:absolute; inset:0; background-image:radial-gradient(rgba(255,255,255,.025) 1px,transparent 1px); background-size:22px 22px; }
.prf-hero-content { position:relative; z-index:1; display:flex; align-items:center; gap:36px; padding:36px 44px; flex-wrap:wrap; }

.prf-avatar-area { display:flex; flex-direction:column; align-items:center; gap:10px; flex-shrink:0; }
.prf-avatar-ring { width:100px; height:100px; border-radius:50%; border:3px solid; padding:4px; display:flex; align-items:center; justify-content:center; box-shadow:0 0 30px rgba(0,0,0,.5); }
.prf-avatar { width:86px; height:86px; border-radius:50%; background:linear-gradient(135deg,var(--blue),var(--blue-lt)); display:flex; align-items:center; justify-content:center; font-family:'Bebas Neue',sans-serif; font-size:42px; color:#fff; }
.prf-nivel-badge { padding:4px 16px; border-radius:20px; font-size:12px; font-weight:800; letter-spacing:.5px; border:1px solid; }

.prf-identity { flex:1; min-width:200px; display:flex; flex-direction:column; gap:10px; }
.prf-name     { font-family:'Bebas Neue',sans-serif; font-size:44px; color:#fff; letter-spacing:2px; line-height:1; }
.prf-email    { font-size:14px; color:var(--text-mid); }
.prf-tags     { display:flex; flex-wrap:wrap; gap:8px; }
.prf-tag      { font-size:12px; font-weight:700; color:var(--text-mid); background:rgba(255,255,255,.07); border:1px solid var(--border); padding:5px 12px; border-radius:7px; }

.prf-xp-block { min-width:260px; display:flex; flex-direction:column; gap:8px; }
.prf-xp-label { display:flex; justify-content:space-between; font-size:12px; color:var(--text-mid); font-weight:600; }
.prf-xp-bar   { height:8px; background:rgba(255,255,255,.08); border-radius:8px; overflow:hidden; }
.prf-xp-fill  { height:100%; background:linear-gradient(90deg,var(--blue),var(--blue-lt)); border-radius:8px; transition:width 1.2s cubic-bezier(.16,1,.3,1); box-shadow:0 0 12px rgba(37,99,235,.5); }
.prf-xp-pct   { font-size:11px; color:var(--blue-lt); font-weight:700; }

.prf-stats-grid { display:grid; grid-template-columns:repeat(4,1fr); gap:16px; }
.prf-stat-card {
  background:var(--bg3); border:1px solid var(--border);
  border-radius:14px; padding:24px 20px;
  display:flex; flex-direction:column; align-items:center; gap:6px;
  animation:stk-in .4s both; animation-delay:calc(var(--i)*.08s);
  transition:transform .2s, border-color .2s;
}
.prf-stat-card:hover { transform:translateY(-4px); border-color:var(--bdr2); }
.psc-icon { font-size:30px; }
.psc-val  { font-family:'Bebas Neue',sans-serif; font-size:38px; line-height:1; }
.psc-lbl  { font-size:13px; font-weight:700; color:#fff; }
.psc-sub  { font-size:11px; color:var(--text-mid); text-align:center; }

.prf-section { background:var(--bg3); border:1px solid var(--border); border-radius:16px; padding:28px 32px; }
.prf-section-title { font-family:'Bebas Neue',sans-serif; font-size:20px; color:#fff; letter-spacing:1px; margin-bottom:18px; }

.prf-progress-card { background:var(--bg4); border:1px solid var(--border); border-radius:12px; padding:20px 24px; display:flex; flex-direction:column; gap:12px; }
.ppc-header { display:flex; justify-content:space-between; font-size:14px; color:var(--text-mid); font-weight:600; }
.ppc-pct    { color:#fff; font-weight:800; }
.ppc-bar    { height:12px; background:rgba(255,255,255,.06); border-radius:8px; overflow:hidden; }
.ppc-fill   { height:100%; background:linear-gradient(90deg,var(--blue),var(--blue-lt)); border-radius:8px; position:relative; transition:width 1.5s cubic-bezier(.16,1,.3,1); }
.ppc-glow   { position:absolute; right:0; top:0; bottom:0; width:30px; background:linear-gradient(90deg,transparent,rgba(255,255,255,.3)); }
.ppc-footer { display:flex; justify-content:space-between; font-size:12px; color:var(--text-mid); }

.prf-fav-grid { display:grid; grid-template-columns:repeat(8,1fr); gap:10px; }
.prf-fav-card { display:flex; flex-direction:column; align-items:center; gap:6px; padding:10px 8px; border-radius:10px; border:1px solid var(--border); background:var(--bg4); cursor:pointer; transition:all .2s; position:relative; }
.prf-fav-card:hover   { border-color:var(--bdr2); transform:translateY(-2px); }
.prf-fav-card.selected { border-color:var(--blue-lt); background:rgba(37,99,235,.12); }
.prf-fav-card img { width:42px; height:28px; object-fit:cover; border-radius:4px; border:1px solid rgba(255,255,255,.1); }
.prf-fav-card span { font-size:10px; font-weight:700; color:var(--text-mid); text-align:center; }
.prf-fav-check { position:absolute; top:-5px; right:-5px; width:18px; height:18px; background:var(--green); border-radius:50%; display:flex; align-items:center; justify-content:center; font-size:9px; color:#fff; border:2px solid var(--bg3); }

.prf-conquistas { display:grid; grid-template-columns:repeat(2,1fr); gap:12px; }
@media (min-width:1200px) { .prf-conquistas { grid-template-columns:repeat(3,1fr); } }
.prf-conquista {
  display:flex; align-items:center; gap:12px;
  background:var(--bg4); border:1px solid var(--border);
  border-radius:12px; padding:14px 16px;
  animation:stk-in .4s both; animation-delay:calc(var(--i)*.06s);
  transition:border-color .2s;
  opacity:.5; min-width:0;
}
.prf-conquista.unlocked { opacity:1; border-color:rgba(0,166,80,.25); background:rgba(0,166,80,.04); }
.prf-conquista.unlocked:hover { border-color:rgba(0,166,80,.5); }
.pc-icon  { font-size:26px; flex-shrink:0; width:32px; text-align:center; }
.pc-icon.locked { filter:grayscale(1); opacity:.4; }
.pc-body  { display:flex; flex-direction:column; gap:3px; flex:1; min-width:0; overflow:hidden; }
.pc-nome  { font-size:13px; font-weight:700; color:#fff; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
.pc-desc  { font-size:11px; color:var(--text-mid); white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
.pc-badge { font-size:10px; font-weight:700; color:var(--green); white-space:nowrap; flex-shrink:0; background:rgba(0,166,80,.1); border:1px solid rgba(0,166,80,.25); padding:3px 10px; border-radius:20px; }

/* ══════════ TRANSITIONS ══════════ */
.auth-fade-enter-active, .auth-fade-leave-active { transition:opacity .4s ease; }
.auth-fade-enter-from, .auth-fade-leave-to       { opacity:0; }
.fade-down-enter-active { transition:all .3s ease; }
.fade-down-enter-from   { opacity:0; transform:translateY(-8px); }
.fade-down-leave-active { transition:all .2s ease; }
.fade-down-leave-to     { opacity:0; }
.fade-slide-enter-active, .fade-slide-leave-active { transition:all .25s ease; }
.fade-slide-enter-from { opacity:0; transform:translateX(10px); }
.fade-slide-leave-to   { opacity:0; transform:translateX(-10px); }
.slide-up-enter-active { transition:all .3s cubic-bezier(.34,1.56,.64,1); }
.slide-up-enter-from   { opacity:0; transform:translateY(12px); }
.slide-up-leave-active { transition:all .2s ease; }
.slide-up-leave-to     { opacity:0; transform:translateY(8px); }

/* ══════════ RESPONSIVIDADE ══════════ */
@media (max-width:1279px) {
  .stickers-grid    { grid-template-columns:repeat(5,1fr); }
  .ef-flags         { grid-template-columns:repeat(4,1fr); }
  .empty-stats      { grid-template-columns:repeat(2,1fr); }
  .ht-steps         { grid-template-columns:repeat(2,1fr); }
  .prf-stats-grid   { grid-template-columns:repeat(2,1fr); }
  .prf-fav-grid     { grid-template-columns:repeat(4,1fr); }
  .hero-year-bg     { font-size:72px; }
}
@media (max-width:1023px) {
  :root{ --sidebar-w:72px; }
  .sb-brand,.sb-link>span:not(.sb-ico),.sb-badge,
  .sb-select-block,.sb-team-card,.sb-copa-body>div,
  .sb-copa-title,.sb-copa-year,.sb-copa-hosts,
  .sb-user-info,.sb-logout span { display:none; }
  .sb-logo   { justify-content:center; padding:16px 0; }
  .sb-link   { justify-content:center; padding:12px 0; }
  .sb-ico    { opacity:1 !important; width:22px; height:22px; }
  .sb-copa   { padding:0; }
  .sb-copa-body { justify-content:center; }
  .sb-copa-trophy { font-size:22px; }
  .sb-user   { justify-content:center; padding:10px 0; }
  .sb-user-avatar { width:36px; height:36px; }
  .sb-logout { justify-content:center; padding:10px 0; }
  .sb-divider { display:none; }
  .album,.empty-wrap,.mercado-wrap,.perfil-wrap { padding:20px 24px 36px; }
  .hero-inner { padding:28px 24px 22px; }
  .stickers-grid { gap:12px; grid-template-columns:repeat(4,1fr); }
  .mkt-grid  { grid-template-columns:repeat(auto-fill,minmax(160px,1fr)); }
}
@media (max-width:767px) {
  .root, .app-shell { flex-direction:column; }
  .auth-screen      { flex-direction:column; }
  .auth-card        { margin:auto; width:calc(100% - 32px); }
  .auth-deco        { display:none; }
  .sidebar { width:100%; min-width:unset; height:auto; position:static; flex-direction:row; align-items:center; padding:0 12px; height:58px; overflow:visible; }
  .sb-logo   { border:none; padding:0; margin-right:auto; }
  .sb-name   { font-size:18px; }
  .sb-sub    { display:none; }
  .sb-nav    { flex-direction:row; gap:4px; padding:0; margin:0 0 0 8px; }
  .sb-link   { padding:8px 10px; border-radius:8px; }
  .sb-brand,.sb-badge,.sb-link>span:not(.sb-ico) { display:none; }
  .sb-divider,.sb-select-block,.sb-team-card,.sb-copa,.sb-user,.sb-logout { display:none; }
  .hero-inner    { padding:22px 18px 16px; flex-direction:column; gap:12px; align-items:flex-start; }
  .hero-year-bg,.hero-hosts-row { display:none; }
  .hero-title    { font-size:42px; }
  .album,.empty-wrap,.mercado-wrap,.perfil-wrap { padding:16px 16px 32px; }
  .eb-content    { padding:28px 24px; flex-direction:column; gap:24px; }
  .eb-title      { font-size:38px; }
  .ef-flags      { grid-template-columns:repeat(4,1fr); }
  .ht-steps      { grid-template-columns:1fr; }
  .ah-content    { flex-direction:column; gap:20px; padding:24px 20px; }
  .ah-ring-wrap,.ah-mini-stats { display:none; }
  .ah-name       { font-size:34px; }
  .stickers-grid { grid-template-columns:repeat(3,1fr); gap:10px; }
  .mkt-grid  { grid-template-columns:repeat(3,1fr); gap:12px; }
  .pagination    { padding:12px 14px; }
  .pg-dots       { display:none; }
  .mkt-grid      { grid-template-columns:repeat(2,1fr); gap:12px; }
  .prf-hero-content { flex-direction:column; gap:24px; padding:28px 24px; }
  .prf-stats-grid   { grid-template-columns:repeat(2,1fr); }
  .prf-fav-grid     { grid-template-columns:repeat(4,1fr); }
  .prf-conquistas   { grid-template-columns:1fr; }
  .mkt-banner-content { flex-direction:column; gap:24px; padding:28px 24px; }
}
@media (max-width:479px) {
  .stickers-grid { grid-template-columns:repeat(2,1fr); gap:10px; }
  .empty-stats   { grid-template-columns:repeat(2,1fr); }
  .ef-flags      { grid-template-columns:repeat(2,1fr); }
  .mkt-grid      { grid-template-columns:repeat(2,1fr); }
  .prf-fav-grid  { grid-template-columns:repeat(2,1fr); }
  .eb-title      { font-size:30px; }
  .hero-title    { font-size:34px; }
}
</style>