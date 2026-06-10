<template>
  <div
    class="figurinha-card"
    :style="{ '--cor-primaria': corPrimaria, '--cor-secundaria': corSecundaria }"
    @mouseenter="holografico = true"
    @mouseleave="holografico = false"
    :class="{ 'holo-ativo': holografico }"
  >
    <!-- Brilho holográfico (camada decorativa) -->
    <div class="holo-brilho" aria-hidden="true"></div>

    <!-- Cabeçalho: número + bandeira -->
    <div class="card-header">
      <span class="numero-figurinha">{{ numero }}</span>
      <img
        class="bandeira"
        :src="bandeira"
        :alt="'Bandeira de ' + (jogador.pais || 'seleção')"
        loading="lazy"
      />
    </div>

    <!-- Foto do jogador -->
    <div class="foto-container">
      <img
        class="foto-jogador"
        :src="jogador.foto || jogador.photo || jogador.strCutout || ''"
        :alt="jogador.nome || jogador.strPlayer || 'Jogador'"
        loading="lazy"
        @error="onFotoErro"
      />
      <!-- Fallback silhueta SVG se foto quebrar -->
      <svg
        class="foto-fallback"
        v-if="fotoQuebrada"
        viewBox="0 0 80 100"
        xmlns="http://www.w3.org/2000/svg"
        aria-hidden="true"
      >
        <circle cx="40" cy="28" r="18" fill="rgba(255,255,255,0.35)" />
        <path d="M8 95 Q8 62 40 62 Q72 62 72 95Z" fill="rgba(255,255,255,0.35)" />
      </svg>
    </div>

    <!-- Rodapé: nome e posição -->
    <div class="card-footer">
      <div class="escudo-wrapper">
        <img
          class="escudo"
          :src="escudoClube"
          :alt="'Escudo do clube'"
          v-if="escudoClube"
          loading="lazy"
          @error="onEscudoErro"
        />
      </div>
      <div class="info-jogador">
        <span class="nome-jogador">{{ nomeAbreviado }}</span>
        <span class="posicao-jogador">{{ posicaoTraduzida }}</span>
      </div>
    </div>

    <!-- Borda brilhante temática -->
    <div class="borda-tema" aria-hidden="true"></div>
  </div>
</template>

<script>
/**
 * FigurinhaCard.vue
 * Componente de figurinha de álbum da Copa do Mundo.
 *
 * Props:
 *   jogador  {Object}  — objeto do jogador vindo da API (compatível com API-Football e TheSportsDB)
 *   bandeira {String}  — URL da imagem da bandeira da seleção
 *   numero   {Number}  — número da figurinha no álbum
 *
 * Exemplos de campos esperados em `jogador`:
 *   jogador.nome | jogador.strPlayer | jogador.name
 *   jogador.foto | jogador.photo | jogador.strCutout | jogador.strThumb
 *   jogador.posicao | jogador.strPosition | jogador.position
 *   jogador.pais | jogador.strNationality | jogador.nationality
 *   jogador.escudo | jogador.strClubBadge  (opcional)
 */

// Mapeamento de cores por seleção — adicione mais conforme necessário
const COR_SELECAO = {
  // Nome em português
  'brasil':     { primaria: '#009c3b', secundaria: '#FFDF00' },
  'argentina':  { primaria: '#74ACDF', secundaria: '#FFFFFF' },
  'franca':     { primaria: '#0055A4', secundaria: '#EF4135' },
  'frança':     { primaria: '#0055A4', secundaria: '#EF4135' },
  'alemanha':   { primaria: '#000000', secundaria: '#DD0000' },
  'portugal':   { primaria: '#006600', secundaria: '#FF0000' },
  'espanha':    { primaria: '#AA151B', secundaria: '#F1BF00' },
  'italia':     { primaria: '#003DA5', secundaria: '#009246' },
  'itália':     { primaria: '#003DA5', secundaria: '#009246' },
  'inglaterra': { primaria: '#003090', secundaria: '#FFFFFF' },
  'holanda':    { primaria: '#FF6600', secundaria: '#FFFFFF' },
  'belgica':    { primaria: '#000000', secundaria: '#FAE042' },
  'bélgica':    { primaria: '#000000', secundaria: '#FAE042' },
  'croatia':    { primaria: '#FF0000', secundaria: '#FFFFFF' },
  'croacia':    { primaria: '#FF0000', secundaria: '#FFFFFF' },
  'croácia':    { primaria: '#FF0000', secundaria: '#FFFFFF' },
  'mexico':     { primaria: '#006847', secundaria: '#CE1126' },
  'méxico':     { primaria: '#006847', secundaria: '#CE1126' },
  'uruguai':    { primaria: '#5EB6E4', secundaria: '#FFFFFF' },
  'senegal':    { primaria: '#00853F', secundaria: '#E31B23' },
  'marrocos':   { primaria: '#C1272D', secundaria: '#006233' },
  'japao':      { primaria: '#003DA5', secundaria: '#FFFFFF' },
  'japão':      { primaria: '#003DA5', secundaria: '#FFFFFF' },
  'coreia':     { primaria: '#C60C30', secundaria: '#003478' },
  'estados unidos': { primaria: '#002868', secundaria: '#BF0A30' },
  'eua':        { primaria: '#002868', secundaria: '#BF0A30' },
};

// Mapeamento de posições inglês → português
const POSICAO_PT = {
  'Goalkeeper':  'Goleiro',
  'Defender':    'Zagueiro / Lateral',
  'Midfielder':  'Meia-Campo',
  'Forward':     'Atacante',
  'Attacker':    'Atacante',
  'Winger':      'Ponta',
  'Striker':     'Centroavante',
  'Libero':      'Líbero',
  'Centre-Back': 'Zagueiro',
  'Left Back':   'Lateral Esq.',
  'Right Back':  'Lateral Dir.',
  'Centre Midfielder': 'Meia Central',
  'Attacking Midfielder': 'Meia Atacante',
  'Defensive Midfielder': 'Volante',
};

export default {
  name: 'FigurinhaCard',

  props: {
    jogador: {
      type: Object,
      required: true,
    },
    bandeira: {
      type: String,
      default: '',
    },
    numero: {
      type: Number,
      default: 1,
    },
  },

  data() {
    return {
      holografico: false,
      fotoQuebrada: false,
      escudoQuebrado: false,
    };
  },

  computed: {
    /** Nome normalizado do país em minúsculas para lookup de cor */
    paisNorm() {
      const raw =
        this.jogador?.pais ||
        this.jogador?.strNationality ||
        this.jogador?.nationality ||
        '';
      return raw.toLowerCase().trim();
    },

    corPrimaria() {
      return (COR_SELECAO[this.paisNorm] || { primaria: '#1a1a2e' }).primaria;
    },

    corSecundaria() {
      return (COR_SELECAO[this.paisNorm] || { secundaria: '#e94560' }).secundaria;
    },

    nomeAbreviado() {
      const nome =
        this.jogador?.nome ||
        this.jogador?.strPlayer ||
        this.jogador?.name ||
        'Jogador';
      // Se o nome tiver mais de 14 chars, exibe apenas o último sobrenome
      if (nome.length > 14) {
        const partes = nome.trim().split(' ');
        return partes[partes.length - 1].toUpperCase();
      }
      return nome.toUpperCase();
    },

    posicaoTraduzida() {
      const pos =
        this.jogador?.posicao ||
        this.jogador?.strPosition ||
        this.jogador?.position ||
        '';
      return POSICAO_PT[pos] || pos || '—';
    },

    escudoClube() {
      if (this.escudoQuebrado) return '';
      return (
        this.jogador?.escudo ||
        this.jogador?.strClubBadge ||
        ''
      );
    },
  },

  methods: {
    onFotoErro(e) {
      // Tenta campo alternativo antes de mostrar fallback SVG
      const alternativa =
        this.jogador?.strThumb ||
        this.jogador?.strRender ||
        '';
      if (alternativa && e.target.src !== alternativa) {
        e.target.src = alternativa;
      } else {
        this.fotoQuebrada = true;
        e.target.style.display = 'none';
      }
    },
    onEscudoErro() {
      this.escudoQuebrado = true;
    },
  },
};
</script>

<style scoped>
/* ─── Fonte esportiva via Google Fonts ──────────────────── */
@import url('https://fonts.googleapis.com/css2?family=Oswald:wght@500;700&family=Roboto+Condensed:wght@400;700&display=swap');

/* ─── Variáveis e reset ─────────────────────────────────── */
.figurinha-card *,
.figurinha-card *::before,
.figurinha-card *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* ─── Card principal ────────────────────────────────────── */
.figurinha-card {
  /* Proporção 2:3 — igual às figurinhas Panini */
  width: 140px;
  aspect-ratio: 2 / 3;

  border-radius: 8px;
  overflow: hidden;
  position: relative;
  display: flex;
  flex-direction: column;

  background: #ffffff;
  box-shadow:
    0 4px 12px rgba(0, 0, 0, 0.22),
    0 1px 3px rgba(0, 0, 0, 0.14);

  cursor: pointer;
  user-select: none;

  /* Transição suave */
  transition:
    transform 0.28s cubic-bezier(0.34, 1.56, 0.64, 1),
    box-shadow 0.28s ease;

  /* Isolamento para efeito holográfico */
  isolation: isolate;
}

/* Hover: eleva e inclina levemente */
.figurinha-card:hover {
  transform: translateY(-6px) scale(1.04) rotate(-0.5deg);
  box-shadow:
    0 16px 32px rgba(0, 0, 0, 0.28),
    0 4px 8px rgba(0, 0, 0, 0.18);
}

/* Acessibilidade: respeita preferência de movimento reduzido */
@media (prefers-reduced-motion: reduce) {
  .figurinha-card,
  .figurinha-card:hover {
    transition: none;
    transform: none;
  }
}

/* ─── Efeito holográfico (foil) ─────────────────────────── */
.holo-brilho {
  position: absolute;
  inset: 0;
  z-index: 3;
  pointer-events: none;
  opacity: 0;
  border-radius: inherit;
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 200, 80, 0.18) 25%,
    rgba(100, 220, 255, 0.22) 50%,
    rgba(200, 100, 255, 0.18) 75%,
    rgba(255, 255, 255, 0) 100%
  );
  transition: opacity 0.3s ease;
}

.holo-ativo .holo-brilho {
  opacity: 1;
}

/* ─── Cabeçalho (número + bandeira) ────────────────────── */
.card-header {
  flex-shrink: 0;
  height: 30px;
  background: var(--cor-primaria, #1a1a2e);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 8px;
  position: relative;
  z-index: 2;
}

.numero-figurinha {
  font-family: 'Oswald', 'Arial Narrow', sans-serif;
  font-weight: 700;
  font-size: 15px;
  color: var(--cor-secundaria, #ffffff);
  line-height: 1;
  letter-spacing: 0.02em;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.4);
  min-width: 18px;
}

.bandeira {
  width: 24px;
  height: 16px;
  object-fit: cover;
  border-radius: 2px;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.35);
  flex-shrink: 0;
}

/* ─── Container da foto ─────────────────────────────────── */
.foto-container {
  flex: 1;
  background: linear-gradient(
    180deg,
    var(--cor-primaria, #1a1a2e) 0%,
    color-mix(in srgb, var(--cor-primaria, #1a1a2e) 70%, #000 30%) 100%
  );
  position: relative;
  overflow: hidden;
  display: flex;
  align-items: flex-end;
  justify-content: center;
}

/* Gradiente sutil no rodapé da foto para transição suave */
.foto-container::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 28px;
  background: linear-gradient(transparent, rgba(255, 255, 255, 0.9));
  z-index: 1;
  pointer-events: none;
}

.foto-jogador {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: top center;
  display: block;
}

/* Fallback silhueta SVG */
.foto-fallback {
  position: absolute;
  inset: 0;
  width: 60%;
  height: 80%;
  margin: auto;
  opacity: 0.5;
}

/* ─── Rodapé (nome + posição) ───────────────────────────── */
.card-footer {
  flex-shrink: 0;
  height: 44px;
  background: #ffffff;
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 0 8px;
  position: relative;
  z-index: 2;

  /* Linha decorativa na cor primária no topo do rodapé */
  border-top: 3px solid var(--cor-primaria, #1a1a2e);
}

.escudo-wrapper {
  flex-shrink: 0;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.escudo {
  width: 20px;
  height: 20px;
  object-fit: contain;
}

.info-jogador {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  overflow: hidden;
  min-width: 0;
}

.nome-jogador {
  font-family: 'Oswald', 'Arial Narrow', sans-serif;
  font-weight: 700;
  font-size: 11px;
  letter-spacing: 0.06em;
  color: #111111;
  line-height: 1.1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  display: block;
}

.posicao-jogador {
  font-family: 'Roboto Condensed', 'Arial Narrow', sans-serif;
  font-weight: 400;
  font-size: 8px;
  letter-spacing: 0.04em;
  color: #666666;
  text-transform: uppercase;
  line-height: 1.2;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  display: block;
}

/* ─── Borda brilhante temática (detalhe foil interno) ───── */
.borda-tema {
  position: absolute;
  inset: 2px;
  border-radius: 6px;
  border: 1px solid rgba(255, 255, 255, 0.08);
  pointer-events: none;
  z-index: 4;
  /* Brilho sutil na borda ao hover */
  box-shadow: inset 0 0 0 1px transparent;
  transition: box-shadow 0.28s ease;
}

.holo-ativo .borda-tema {
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.28);
}

/* ─── Responsividade ────────────────────────────────────── */

/* Telas médias: card um pouco maior */
@media (min-width: 480px) {
  .figurinha-card {
    width: 158px;
  }

  .numero-figurinha { font-size: 16px; }
  .bandeira         { width: 26px; height: 17px; }
  .card-footer      { height: 48px; }
  .nome-jogador     { font-size: 12px; }
  .posicao-jogador  { font-size: 9px; }
  .escudo           { width: 22px; height: 22px; }
}

/* Telas grandes: card padrão */
@media (min-width: 768px) {
  .figurinha-card {
    width: 175px;
  }

  .card-header      { height: 34px; padding: 0 10px; }
  .numero-figurinha { font-size: 18px; }
  .bandeira         { width: 28px; height: 18px; }
  .card-footer      { height: 52px; padding: 0 10px; gap: 8px; }
  .nome-jogador     { font-size: 13px; }
  .posicao-jogador  { font-size: 10px; }
  .escudo           { width: 24px; height: 24px; }
}
</style>