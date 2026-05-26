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

const gerarStatus = (posicao) => {
  if (posicao === 'Goalkeeper') {
    return {
      ELA: Math.floor(Math.random() * 30) + 70,
      MAN: Math.floor(Math.random() * 30) + 70,
      CHU: Math.floor(Math.random() * 30) + 60,
      REF: Math.floor(Math.random() * 30) + 75,
      VEL: Math.floor(Math.random() * 30) + 60,
      POS: Math.floor(Math.random() * 30) + 70
    }
  }

  return {
    RIT: Math.floor(Math.random() * 30) + 70,
    FIN: Math.floor(Math.random() * 30) + 65,
    PAS: Math.floor(Math.random() * 30) + 68,
    CON: Math.floor(Math.random() * 30) + 70,
    DEF: Math.floor(Math.random() * 30) + 60,
    FIS: Math.floor(Math.random() * 30) + 65
  }
}

const carregarFigurinhas = async () => {
  if (!paisSelecionado.value) return

  carregando.value = true
  jogadores.value = []

  try {
    // Buscar time
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

    // Buscar jogadores
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

    jogadores.value = dadosElenco.response[0].players.map((jogador) => ({
      ...jogador,
      status: gerarStatus(jogador.position)
    }))
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

    <section class="selecoes-info">
      <h2 class="selecionada">
        {{ paisSelecionado }}
      </h2>

      <p class="contador">
        Figurinhas coletadas: {{ jogadores.length }}
      </p>
    </section>

    <p v-if="carregando" class="loading">
      Carregando jogadores...
    </p>

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
  background: white;
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

.loading {
  text-align: center;
  margin-bottom: 30px;
  font-size: 22px;
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
  box-shadow: 0 5px 15px rgba(0,0,0,0.2);
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

@media (max-width: 768px) {

  .topo h1 {
    font-size: 36px;
  }

  .selecoes-info {
    flex-direction: column;
    gap: 15px;
    text-align: center;
  }

  .selecionada {
    font-size: 28px;
  }

}
</style>