# MinerAI - Inteligencia de Projetos Sotreq

Portal interno para consultores da Sotreq acompanharem oportunidades em terras raras, minerais criticos, green field e projetos de mineracao no Brasil.

O objetivo do site e transformar informacao de mercado em acao: entender quais projetos existem, em que fase estao, qual CAPEX esta previsto, que frota CAT pode ser demandada e quando a Sotreq deve se aproximar do cliente.

## Para que serve

- Mapear projetos de terras raras e minerais estrategicos no Brasil.
- Acompanhar CAPEX, estagio, localizacao, capacidade, licenciamento e previsao de cada projeto.
- Estimar oportunidade Sotreq com base no CAPEX configurado no portal.
- Dimensionar demanda potencial de equipamentos CAT por projeto e por carteira.
- Apoiar consultores em prospeccao, preparacao de abordagem e priorizacao de oportunidades.
- Centralizar noticias e sinais de mercado que podem indicar novos movimentos comerciais.
- Permitir consulta ao MinerAI, um assistente treinado com os dados do portal.

## Principais areas do site

### MinerAI

Assistente de inteligencia de projetos. Ele responde perguntas sobre:

- CAPEX por projeto e por periodo.
- Estagio dos projetos e timing de abordagem.
- Dimensionamento estimado de frota CAT.
- Oportunidade Sotreq por cliente, estado ou carteira.
- Riscos, gargalos e proximos passos.
- Resumos executivos para reunioes e preparacao de visitas.

O MinerAI usa `gpt-4o-mini` via Worker/proxy configurado em `_worker.js`.

### Resumo Executivo

Visao consolidada para leitura rapida:

- mapa de oportunidades por estado;
- indicadores-chave da carteira;
- pipeline dos projetos;
- contexto de mercado;
- conexao com oportunidades Sotreq.

### Painel Comercial

Dashboard interativo com graficos D3:

- carteira de oportunidades por cliente;
- oportunidade por estado;
- timing comercial por estagio;
- perfil da oportunidade;
- prioridade de abordagem;
- demanda estimada de equipamentos CAT;
- demanda por ano;
- contas prioritarias e mix de equipamentos.

Os graficos permitem filtro por clique e ajudam a navegar na tabela de projetos.

### Noticias

Feed de noticias relacionadas aos projetos e ao setor. Pode ser usado para acompanhar fatos novos e sinais de atualizacao comercial.

### Painel Administrativo

Interface dentro do proprio site para editar:

- projetos;
- noticias;
- indicadores e premissas;
- configuracoes do portal;
- importacao/exportacao de dados.

## Arquitetura

```text
GitHub Pages / Navegador
        |
        v
index.html
  - UI do portal
  - D3.js
  - MinerAI
  - Admin panel
  - localStorage fallback
        |
        +--> Supabase
        |      tabela: site_data
        |      campos: id, projects, news, updated_at
        |
        +--> Cloudflare Worker / _worker.js
               proxy para Chat Completions

automacao/
  - agente Node.js opcional
  - busca RSS/Google News
  - processa noticias com LLM
  - cruza com projetos existentes
  - atualiza Supabase/cache local
```

## Tecnologias

| Area | Tecnologia |
| --- | --- |
| Frontend | HTML, CSS e JavaScript vanilla |
| Graficos | D3.js v7 |
| Icones | Bootstrap Icons + imagens PNG |
| Persistencia | Supabase + localStorage |
| Assistente | `gpt-4o-mini` via Worker |
| Automacao | Node.js |
| Noticias | Google News RSS / rss-parser |
| Publicacao | GitHub Pages |

## Como publicar no GitHub Pages

Para disponibilizar a versao estatica, suba pelo menos:

- `index.html`
- `minerai.png`
- `perfuratriz.png`
- imagens de equipamentos (`caminhao.png`, `escavadeira.png`, `carregadeira.png`, etc.)
- logos (`logo-sotreq.png`, `logo-cat.png`)
- pasta `mapa/`
- `_worker.js` apenas se for usar/consultar como referencia do proxy

Depois configure o GitHub Pages para servir a branch/pasta onde esta o `index.html`.

## Como usar localmente

O site e estatico. Para abrir rapidamente:

```bash
python -m http.server 8080
```

Depois acesse:

```text
http://localhost:8080
```

Tambem e possivel abrir o `index.html` diretamente no navegador, mas o servidor local evita algumas limitacoes de carregamento de assets.

## Dados e persistencia

O portal tenta carregar dados nesta ordem:

1. Supabase (`site_data`)
2. `localStorage`
3. dados embutidos no `index.html`

Isso permite que o site continue funcionando mesmo se o Supabase nao estiver disponivel.

## Agente automatizado

A pasta `automacao/` contem um agente Node.js opcional para atualizar noticias e dados.

### Configuracao

```bash
cd automacao
npm install
```

Crie um arquivo `.env` em `automacao/` com as chaves necessarias:

```text
SUPABASE_URL=...
SUPABASE_ANON_KEY=...
LLM_PROVIDER=groq|openai|gemini
LLM_API_KEY=...
LLM_MODEL=...
SEARCH_DAYS=7
```

### Comandos

```bash
npm start       # busca padrao
npm run dev     # logs detalhados
npm run daily   # ultimas 24h
npm run weekly  # ultimos 7 dias
```

## Arquivos importantes

```text
index.html                     # Portal principal
minerai.png                    # Icone do assistente MinerAI
perfuratriz.png                # Imagem usada para perfuratrizes
_worker.js                     # Proxy para chamadas ao modelo
sotreq-dados-2026-05-25.json   # Exportacao/base de dados
mapa/uf.json                   # GeoJSON do Brasil
automacao/                     # Agente Node.js opcional
```

## Observacoes de seguranca

Este projeto foi pensado como portal interno/compartilhado para consultores.

- A senha no front-end e uma barreira simples de acesso visual, nao autenticacao forte.
- A chave anon do Supabase e publica por natureza, mas as politicas do banco precisam estar configuradas com cuidado.
- O Worker deve proteger a chave do modelo; nunca coloque chave secreta diretamente no `index.html`.
- Antes de compartilhar externamente, revise dados sensiveis, politicas de escrita no Supabase e permissao de acesso ao Worker.

## Escopo atual

O portal monitora projetos como:

- Meteoric Resources - Caldeira
- Viridis Mining & Minerals - Colossus
- Serra Verde / USA Rare Earth - Pela Ema
- Brazilian Rare Earths - Monte Alto
- St George Mining - Araxa
- Aclara Resources - Carina
- Rainbow REE + Mosaic - Uberaba
- Terra Brasil Minerals - Alto Paranaiba
- Resouro Strategic Metals - Tiros
- Power Minerals - Morro do Ferro
- ADL Mineracao - Buena
- Brazilian Critical Minerals - EMA/Apui
- Belo Sun Mining - Volta Grande

A lista pode ser alterada pelo painel administrativo ou pela automacao.

## Licenca e uso

Material de inteligencia de mercado para uso interno/comercial da Sotreq.
