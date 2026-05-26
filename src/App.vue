<script setup>
import { ref } from 'vue'

const paises = ref([
  { name: 'Brazil' },
  { name: 'Argentina' },
  { name: 'France' },
  { name: 'Germany' },
  { name: 'Portugal' },
  { name: 'Spain' }
])

const paisSelecionado = ref('')
const jogadores = ref([])

const bancoJogadores = {
  Brazil: [
    {
      id: 1,
      name: 'Alisson Becker',
      position: 'Goalkeeper',
      photo: 'https://media.api-sports.io/football/players/1257.png',
      status: {
        ELA: 90,
        MAN: 88,
        CHU: 75,
        REF: 92,
        VEL: 70,
        POS: 91
      }
    },
    {
      id: 2,
      name: 'Vinicius Junior',
      position: 'Attacker',
      photo: 'https://media.api-sports.io/football/players/18788.png',
      status: {
        RIT: 97,
        FIN: 86,
        PAS: 80,
        CON: 94,
        DEF: 35,
        FIS: 78
      }
    },
    {
      id: 3,
      name: 'Rodrygo',
      position: 'Attacker',
      photo: 'https://media.api-sports.io/football/players/18776.png',
      status: {
        RIT: 90,
        FIN: 84,
        PAS: 81,
        CON: 89,
        DEF: 42,
        FIS: 74
      }
    }
  ],

  Argentina: [
    {
      id: 4,
      name: 'Emiliano Martinez',
      position: 'Goalkeeper',
      photo: 'https://media.api-sports.io/football/players/3495.png',
      status: {
        ELA: 91,
        MAN: 87,
        CHU: 73,
        REF: 93,
        VEL: 65,
        POS: 90
      }
    },
    {
      id: 5,
      name: 'Lionel Messi',
      position: 'Attacker',
      photo: 'https://media.api-sports.io/football/players/154.png',
      status: {
        RIT: 85,
        FIN: 95,
        PAS: 97,
        CON: 98,
        DEF: 40,
        FIS: 70
      }
    }
  ],

  France: [
    {
      id: 6,
      name: 'Kylian Mbappé',
      position: 'Attacker',
      photo: 'https://media.api-sports.io/football/players/278.png',
      status: {
        RIT: 99,
        FIN: 93,
        PAS: 82,
        CON: 95,
        DEF: 38,
        FIS: 82
      }
    }
  ]
}

const carregarFigurinhas = () => {
  jogadores.value = bancoJogadores[paisSelecionado.value] || []
}
</script>

<template>
  <div class="pagina">

    <header class="topo">
      <h1>⚽ Álbum da Copa 2026</h1>

      <select
        v-model="paisSelecionado"
        @change="carregarFigurinhas"
      >
        <option value="">
          Selecione uma seleção
        </option>

        <option
          v-for="pais in paises"
          :key="pais.name"
          :value="pais.name"
        >
          {{ pais.name }}
        </option>
      </select>
    </header>

    <section
      v-if="paisSelecionado"
      class="selecoes-info"
    >
      <h2 class="selecionada">
        {{ paisSelecionado }}
      </h2>

      <p class="contador">
        Figurinhas coletadas: {{ jogadores.length }}
      </p>
    </section>

    <div class="grid">

      <div
        class="figurinha"
        v-for="jogador in jogadores"
        :key="jogador.id"
      >

        <div class="overall">
          {{ Math.floor(Math.random() * 15) + 85 }}
        </div>

        <div class="foto-area">
          <img
            :src="jogador.photo"
            :alt="jogador.name"
          />
        </div>

        <div class="info">
          <h3>{{ jogador.name }}</h3>

          <span class="posicao">
            {{ jogador.position }}
          </span>
        </div>

        <div class="stats">

          <template v-if="jogador.position === 'Goalkeeper'">

            <div class="stat">
              <strong>{{ jogador.status.ELA }}</strong>
              <span>ELA</span>
            </div>

            <div class="stat">
              <strong>{{ jogador.status.MAN }}</strong>
              <span>MAN</span>
            </div>

            <div class="stat">
              <strong>{{ jogador.status.CHU }}</strong>
              <span>CHU</span>
            </div>

            <div class="stat">
              <strong>{{ jogador.status.REF }}</strong>
              <span>REF</span>
            </div>

            <div class="stat">
              <strong>{{ jogador.status.VEL }}</strong>
              <span>VEL</span>
            </div>

            <div class="stat">
              <strong>{{ jogador.status.POS }}</strong>
              <span>POS</span>
            </div>

          </template>

          <template v-else>

            <div class="stat">
              <strong>{{ jogador.status.RIT }}</strong>
              <span>RIT</span>
            </div>

            <div class="stat">
              <strong>{{ jogador.status.FIN }}</strong>
              <span>FIN</span>
            </div>

            <div class="stat">
              <strong>{{ jogador.status.PAS }}</strong>
              <span>PAS</span>
            </div>

            <div class="stat">
              <strong>{{ jogador.status.CON }}</strong>
              <span>CON</span>
            </div>

            <div class="stat">
              <strong>{{ jogador.status.DEF }}</strong>
              <span>DEF</span>
            </div>

            <div class="stat">
              <strong>{{ jogador.status.FIS }}</strong>
              <span>FÍS</span>
            </div>

          </template>

        </div>

      </div>

    </div>

  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: #edf2f7;
  font-family: Arial, Helvetica, sans-serif;
}

.pagina {
  max-width: 1500px;
  margin: auto;
  padding: 30px;
}

.topo {
  background: white;
  padding: 25px;
  border-radius: 20px;
  margin-bottom: 30px;
  box-shadow: 0 5px 15px rgba(0,0,0,0.08);
}

.topo h1 {
  text-align: center;
  margin-bottom: 25px;
  color: #1e293b;
  font-size: 52px;
}

select {
  width: 100%;
  padding: 15px;
  border-radius: 12px;
  border: 2px solid #d1d5db;
  font-size: 18px;
}

.selecoes-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  background: white;
  padding: 20px;
  border-radius: 20px;
  box-shadow: 0 5px 15px rgba(0,0,0,0.08);
}

.selecionada {
  font-size: 36px;
  color: #111827;
}

.contador {
  font-size: 18px;
  font-weight: bold;
  color: #2563eb;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 25px;
}

.figurinha {
  background: linear-gradient(180deg, #ffffff, #e5e7eb);
  border-radius: 20px;
  padding: 20px;
  position: relative;
  border: 3px solid #d1d5db;
  box-shadow: 0 8px 20px rgba(0,0,0,0.12);
  transition: 0.3s;
}

.figurinha:hover {
  transform: translateY(-8px) scale(1.02);
}

.overall {
  position: absolute;
  top: 15px;
  left: 15px;
  background: #facc15;
  color: #111827;
  font-weight: bold;
  padding: 8px 12px;
  border-radius: 12px;
  font-size: 20px;
}

.foto-area {
  display: flex;
  justify-content: center;
  margin-top: 30px;
  margin-bottom: 20px;
}

.foto-area img {
  width: 120px;
  height: 120px;
  object-fit: cover;
  border-radius: 50%;
  border: 4px solid white;
}

.info {
  text-align: center;
  margin-bottom: 20px;
}

.info h3 {
  font-size: 22px;
  color: #111827;
  margin-bottom: 10px;
}

.posicao {
  background: #2563eb;
  color: white;
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: bold;
}

.stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

.stat {
  background: white;
  border-radius: 12px;
  padding: 10px;
  text-align: center;
  border: 1px solid #d1d5db;
}

.stat strong {
  display: block;
  font-size: 20px;
  color: #111827;
}

.stat span {
  font-size: 12px;
  color: #6b7280;
  font-weight: bold;
}
</style>