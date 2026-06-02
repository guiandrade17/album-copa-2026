<div align="center">

<img src="https://img.shields.io/badge/Vue.js-3.x-4FC08D?style=for-the-badge&logo=vuedotjs&logoColor=white"/>
<img src="https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white"/>
<img src="https://img.shields.io/badge/JavaScript-ES2022-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black"/>
<img src="https://img.shields.io/badge/API--Football-v3-00A8E8?style=for-the-badge&logoColor=white"/>
<img src="https://img.shields.io/badge/Status-Concluído-brightgreen?style=for-the-badge"/>

<br/><br/>

# ⚽ Álbum da Copa 2026

> Álbum virtual interativo de seleções da Copa do Mundo 2026, desenvolvido em Vue 3 com consumo de API REST.

🔗 **[Acesse o projeto ao vivo](https://guiandrade17.github.io/album-copa-2026/)**

</div>

---

## 📌 Sobre o Projeto

O **Álbum da Copa 2026** é uma aplicação web que simula um álbum de figurinhas digital, permitindo ao usuário selecionar qualquer seleção participante da Copa do Mundo e visualizar o elenco completo com fotos oficiais, nomes e posições dos jogadores — tudo consumido em tempo real pela [API-Football](https://www.api-football.com/).

O projeto foi desenvolvido como atividade prática da disciplina de **Desenvolvimento Web**, com foco em integração assíncrona de APIs REST utilizando Vue 3.

---

## ✅ Funcionalidades

| Funcionalidade | Descrição |
|---|---|
| 🌍 Dropdown dinâmico | Lista de seleções carregada via API ao iniciar |
| 🔍 Busca automática | Busca o ID da seleção selecionada automaticamente |
| 👥 Elenco completo | Carrega todos os jogadores da seleção escolhida |
| 🖼️ Fotos oficiais | Exibe fotos dos atletas fornecidas pela API |
| 🏳️ Bandeiras | Bandeira real de cada país no select, banner e cards |
| 🃏 Cards estilizados | Design inspirado em figurinhas Panini e FIFA Ultimate Team |
| ✨ Animações | Efeitos de hover, entrada dos cards e anel holográfico |
| 📱 Responsivo | Layout adaptável para desktop, tablet e mobile |
| ⚡ Async/Await | Requisições assíncronas com tratamento de erro |

---

## 🛠️ Tecnologias

- **[Vue 3](https://vuejs.org/)** — Framework JavaScript progressivo com Composition API
- **[Vite](https://vitejs.dev/)** — Build tool rápido para desenvolvimento moderno
- **[API-Football v3](https://www.api-football.com/)** — API REST de dados de futebol
- **Fetch API** — Requisições HTTP nativas do browser
- **CSS3** — Grid, variáveis CSS, animações e glassmorphism

---

## 📂 Estrutura do Projeto

```
album-copa-2026/
├── public/
├── src/
│   ├── assets/
│   │   ├── printBrasil.png
│   │   └── printArgentina.png
│   ├── App.vue
│   └── main.js
├── .env
├── .gitignore
├── index.html
├── package.json
└── vite.config.js
```

---

## ⚙️ Como Executar

### Pré-requisitos

- Node.js 18+
- Conta gratuita na [API-Football](https://dashboard.api-football.com/register)

### Passo a passo

```bash
# 1. Clone o repositório
git clone https://github.com/guiandrade17/album-copa-2026.git

# 2. Entre na pasta
cd album-copa-2026

# 3. Instale as dependências
npm install

# 4. Crie o arquivo .env na raiz e adicione sua chave:
# VITE_API_KEY=SUA_CHAVE_AQUI

# 5. Execute em modo de desenvolvimento
npm run dev
```

Acesse em: **http://localhost:5173**

---

## 🌐 API Utilizada

**API-Football — v3**  
`https://v3.football.api-sports.io`

| Endpoint | Uso |
|---|---|
| `/teams/countries` | Lista todos os países disponíveis |
| `/teams?name={name}` | Busca o ID da seleção pelo nome |
| `/players/squads?team={id}` | Retorna o elenco completo da seleção |

> 📘 Documentação: [api-football.com/documentation-v3](https://www.api-football.com/documentation-v3)

---

## 📱 Responsividade

| Dispositivo | Colunas dos cards |
|---|---|
| Desktop (> 768px) | `auto-fill` — mínimo 200px por card |
| Tablet (≤ 768px) | `auto-fill` — mínimo 155px por card |
| Mobile (≤ 480px) | 2 colunas fixas |

---

## 🎯 Objetivo Acadêmico

Projeto desenvolvido para a disciplina de **Desenvolvimento Web**, com foco em:

- Consumo de APIs REST externas
- Integração assíncrona com `async/await`
- Reatividade e Composition API no Vue 3
- Responsividade com CSS Grid
- Boas práticas de desenvolvimento Front-End

---

## 👨‍💻 Autor

**Guilherme Brito Andrade**

[![GitHub](https://img.shields.io/badge/GitHub-guiandrade17-181717?style=flat-square&logo=github)](https://github.com/guiandrade17)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Guilherme%20Andrade-0077B5?style=flat-square&logo=linkedin)](https://linkedin.com/in/guilherme-andrade)

---

## 📄 Licença

Este projeto possui fins **acadêmicos e educacionais**.  
Sinta-se livre para estudar, adaptar e usar como referência.

---

<div align="center">
  <sub>Desenvolvido com ⚽ e muito ☕ por Guilherme Brito Andrade</sub>
</div>
