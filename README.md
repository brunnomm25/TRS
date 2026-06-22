# Sotreq — Inteligência de Mercado: Terras Raras e Green Field no Brasil

Dashboard interativo + agente automatizado de inteligência de mercado para **Sotreq** (dealer Caterpillar no Brasil), focado em oportunidades de negócio no setor de **Terras Raras (REE)** e projetos **Green Field** de mineração no Brasil.

## Funcionalidades

- **Mapa de oportunidades** — Projetos mapeados em MG, BA, GO, AM, PA, RJ e PI
- **Funil de maturidade** — Classificação em 6 estágios (S1 Prospecção → S6 Operação Comercial)
- **Dashboard analítico** — Gráficos interativos (D3.js) de CAPEX por empresa, estado, estágio e prioridade
- **Estimativa de frota CAT** — Dimensionamento automático de máquinas por tipo de depósito
- **Resumo executivo** — Visão consolidada com mapa do Brasil interativo e indicadores-chave
- **Notícias do setor** — Feed automático com filtros e carrossel
- **Agente automatizado** — Busca e processa notícias com IA, extrai CAPEX e atualiza projetos automaticamente
- **Persistência** — Dados salvos no Supabase + localStorage
- **Painel administrativo** — Interface no próprio dashboard para editar projetos, notícias e configurações
- **Tema claro/escuro** — Alternância com preferência salva

## Arquitetura

```
┌─────────────────────────────────────────────────┐
│                 FRONTEND (index.html)            │
│  Dashboard · Gráficos D3.js · Admin Panel        │
└──────────────────────┬──────────────────────────┘
                       │ Supabase API
┌──────────────────────▼──────────────────────────┐
│              SUPABASE (site_data)                │
│  Tabela única: id, projects (jsonb),            │
│  news (jsonb), updated_at                       │
└──────────────┬─────────────────────▲────────────┘
               │                     │
┌──────────────▼─────────────────────┴────────────┐
│           AGENTE AUTOMATIZADO (Node.js)          │
│                                                  │
│  ┌──────────┐  ┌────────────┐  ┌────────────┐  │
│  │ RSS Feed │─▶│   LLM      │─▶│ Matcher    │  │
│  │ Fetcher  │  │ Processor  │  │ (projetos) │  │
│  └──────────┘  │(OpenAI/GPT)│  └──────┬─────┘  │
│                └────────────┘         │         │
│  ┌──────────┐  ┌────────────┐         │         │
│  │ Cache    │  │ Supabase   │◀────────┘         │
│  │ Local    │  │ Client     │                    │
│  └──────────┘  └────────────┘                    │
└──────────────────────────────────────────────────┘
```

### Fluxo de Dados

1. **Frontend** carrega dados: Supabase → localStorage → hardcoded (fallback)
2. **Agente** (Node.js) roda diariamente/semanalmente:
   - Busca notícias em 9 queries do Google News RSS
   - Processa cada notícia com GPT-4o-mini para extrair: empresa, projeto, CAPEX, estágio, localização
   - Cruza com projetos existentes (matcher fuzzy)
   - Atualiza CAPEX e estágio quando detecta mudanças
   - Adiciona novos projetos automaticamente
   - Salva no Supabase

## Tecnologias

| Componente | Tecnologia |
|------------|------------|
| Frontend | HTML5 + CSS3 (vanilla) |
| Gráficos | [D3.js v7](https://d3js.org/) |
| Persistência | [Supabase](https://supabase.com/) + localStorage |
| Iconografia | [Bootstrap Icons](https://icons.getbootstrap.com/) |
| Agente (back-end) | Node.js + OpenAI API (GPT-4o-mini) |
| RSS | rss-parser + Google News RSS |

## Como usar

### Dashboard

1. Abra o `index.html` em qualquer navegador moderno
2. Digite a senha de acesso
3. Navegue pelas abas: **Dashboard**, **Resumo Executivo**, **Notícias**
4. Use os filtros para refinar a visualização dos projetos

### Agente Automatizado

#### Pré-requisitos

- [Node.js 18+](https://nodejs.org/) instalado
- Chave da [OpenAI API](https://platform.openai.com/api-keys)
- Projeto no [Supabase](https://supabase.com/) (opcional — funciona com cache local)

#### Configuração

```bash
cd automacao
cp .env.example .env
# Edite .env com suas chaves:
#   OPENAI_API_KEY=sk-...
#   SUPABASE_URL=https://...
#   SUPABASE_ANON_KEY=...
npm install
```

#### Executar manualmente

```bash
# Busca completa (últimos 7 dias)
npm start

# Busca detalhada com logs
npm run dev

# Busca diária (último 1 dia)
npm run daily

# Busca semanal (últimos 7 dias)
npm run weekly
```

#### Agendar no Windows (execução diária)

```powershell
# Abra o PowerShell como Administrador e execute:
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\automacao\scripts\install-scheduler.ps1 -Time "08:00"

# Ou manualmente com argumentos:
.\automacao\scripts\install-scheduler.ps1 `
  -TaskName "Sotreq-Agente" `
  -Time "09:00" `
  -Schedule "daily"
```

Também é possível usar o **Agendador de Tarefas do Windows** manualmente:
- Ação: iniciar programa `C:\caminho\automacao\scripts\run.bat`
- Trigger: diário às 08:00
- Pasta: `C:\caminho\automacao`

## Estrutura do Projeto

```
Project 01/
├── index.html                    # Dashboard completo
├── sotreq-dados-2026-05-25.json  # Dados exportados
├── prompt-agente-copilot.md      # Prompt para manutenção via Copilot
├── automacao/                    # Agente automatizado (Node.js)
│   ├── package.json
│   ├── .env.example              # Configuração de chaves
│   ├── src/
│   │   ├── index.js              # Orquestrador principal
│   │   ├── config.js             # Configurações
│   │   ├── news-fetcher.js       # Busca RSS (9 fontes)
│   │   ├── llm-processor.js      # Extração com OpenAI GPT
│   │   ├── project-matcher.js    # Match fuzzy com projetos
│   │   └── supabase-client.js    # Persistência Supabase
│   ├── scripts/
│   │   ├── run.bat               # .bat para Windows
│   │   └── install-scheduler.ps1 # Instalar tarefa agendada
│   └── dados/                    # Cache local
├── mapa/
│   └── uf.json                   # GeoJSON do Brasil (D3.js)
├── *.png                         # Ícones dos equipamentos CAT
└── README.md
```

## Projetos Monitorados

- Meteoric Resources — Caldeira (MG)
- Viridis Mining — Colossus (MG)
- Serra Verde / USA Rare Earth — Pela Ema (GO)
- Brazilian Rare Earths — Monte Alto (BA)
- St George Mining — Araxá (MG)
- Aclara Resources — Carina (GO)
- Rainbow REE + Mosaic — Uberaba (MG)
- Terra Brasil Minerals — Alto Paranaíba (MG)
- Resouro Strategic Metals — Tiros (MG)
- Power Minerals — Morro do Ferro (MG)
- Cabo Verde Mineração — Botelhos (MG)
- ADL Mineração — Buena (RJ)
- Brazilian Critical Minerals — EMA/Apuí (AM)
- Origen Resources — Piauí/Bahia (PI/BA)
- Terras Raras Brasil — Atlas/Argus (MG)
- Taboca / CNMC — Pitinga (AM)
- Belo Sun Mining — Volta Grande (PA)

*Novos projetos são adicionados automaticamente pelo agente conforme notícias são detectadas.*

## Metodologia

A frota CAT é estimada aplicando percentuais do CAPEX de cada projeto sobre o preço médio unitário de cada família de equipamento, variando por tipo de depósito:

- **Argila Iônica (IAC):** Frota média, lavra seletiva, cava rasa
- **Rocha Dura:** Frota pesada, perfuração, desmonte, britagem
- **Reprocessamento:** Foco em pátio, manuseio e alimentação de planta
- **Exploração:** Apenas equipamentos leves, sem frota operacional completa

**Oportunidade Sotreq:** 30% do CAPEX de cada projeto (percentual configurável no painel administrativo).

## Licença

Documento de inteligência de mercado para análise interna da Sotreq.
