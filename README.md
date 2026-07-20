# MinerAÍ - Inteligência de Projetos Sotreq

Portal interno de inteligência comercial para consultores da Sotreq acompanharem oportunidades de mineração no Brasil.

O site organiza duas carteiras em uma mesma experiência:

- **Terras Raras**: projetos de minerais críticos e terras raras mapeados no Brasil.
- **Green Field**: novos projetos de mineração em implantação, licenciamento, estudo ou ramp-up, como bauxita, ouro, lítio, níquel, potássio, fosfato e minério de ferro.

A proposta é transformar informação de mercado em ação comercial: entender quais projetos existem, em que fase estão, qual CAPEX está previsto, qual frota CAT pode ser demandada e quando a Sotreq deve se aproximar do cliente.

## Principais Recursos

- Alternância entre as carteiras **Terras Raras** e **Green Field** por um switch no topo do portal.
- Resumo executivo responsivo para leitura rápida e apresentação gerencial.
- Mapa do Brasil com escala visual por quantidade de projetos e modal detalhado por estado.
- Dashboard com gráficos interativos em D3.js.
- Camada de inteligência comercial com matriz de priorização, funil Sotreq, simulador de cenários e ranking de próxima ação.
- Tabela de projetos com filtros por estado, estágio, prioridade, mineral/depósito e status Sotreq.
- Estimativa de oportunidade Sotreq, frota potencial e mix de equipamentos CAT.
- Notícias por carteira, com filtros por empresa/projeto.
- MinerAÍ, assistente de IA para perguntas sobre projetos, CAPEX, frota, timing e abordagem comercial.
- Atualização automática diária de notícias via GitHub Actions.

## Abas do Portal

### MinerAÍ

Assistente de inteligência de projetos. Ele usa o contexto do portfólio ativo para responder perguntas como:

- "Me fala sobre os novos projetos"
- "Quais contas devo priorizar?"
- "Qual CAPEX previsto para determinado projeto?"
- "Qual frota CAT pode fazer sentido?"
- "Quais projetos estão mais próximos de comprar equipamentos?"
- "Como abordar essa conta comercialmente?"

O MinerAÍ usa `gpt-4o-mini` via Cloudflare Worker configurado em `_worker.js`.

### Resumo Executivo

Visão para diretoria, liderança e consultores:

- mapa de oportunidades por estado;
- indicadores-chave de CAPEX, frota e oportunidade Sotreq;
- pipeline por estágio;
- conexão com oportunidades comerciais;
- leitura executiva por carteira.

### Dashboard

Painel analítico com gráficos interativos:

- CAPEX por empresa;
- oportunidade por estado;
- timing comercial por estágio;
- perfil da oportunidade ou tipo de mineral extraído;
- prioridade de abordagem;
- matriz de priorização comercial por maturidade, oportunidade e frota;
- funil comercial por status Sotreq;
- simulador de cenários para conversão Sotreq, câmbio e ajuste de caminhões;
- ranking de próxima ação para orientar follow-up dos consultores;
- demanda estimada de equipamentos;
- demanda por ano;
- quantidade de equipamentos por empresa.

Os gráficos permitem clique para filtrar a carteira.

### Notícias

Feed de notícias e comunicados públicos relacionados aos projetos mapeados. O botão **Buscar Notícias** atualiza manualmente, e a automação do GitHub Actions pode atualizar diariamente.

## Carteiras

### Terras Raras

Carteira focada em minerais críticos e terras raras, incluindo projetos como Meteoric, Viridis, Serra Verde, BRE, Aclara, St George, Power Minerals, ADL, Spark Energy Minerals e outros projetos mapeados.

### Green Field

Carteira focada em novos projetos de mineração fora do recorte exclusivo de terras raras. Exemplos atuais:

- Brazil Potash Corp. - Autazes
- Sul Americana de Metais (SAM) - Bloco 8
- Alurion Resources - Amargosa
- Belo Sun Mining - Volta Grande
- Centaurus Metals - Jaguar
- Consórcio Santa Quitéria (INB / Galvani) - Santa Quitéria
- Lithium Ionic - Bandeira
- Sigma Lithium - Grota do Cirilo Fase 2 / Barreiro
- Atlas Lithium - Neves
- Aura Minerals - Matupá

## Arquitetura

```text
GitHub Pages / Navegador
        |
        v
index.html
  - Interface do portal
  - Dados embutidos das carteiras
  - Gráficos D3.js
  - MinerAÍ
  - Painel administrativo
  - localStorage
        |
        +--> Supabase
        |      tabela: site_data
        |      usado principalmente para persistência da carteira Terras Raras
        |
        +--> Cloudflare Worker (_worker.js)
               proxy para o modelo de IA

GitHub Actions
        |
        v
.github/workflows/daily-news.yml
  - roda diariamente às 05:00 de Brasília
  - executa automacao/update-news.mjs
  - busca notícias para Terras Raras e Green Field
  - commita index.html se houver notícia nova
```

## Tecnologias

| Área | Tecnologia |
| --- | --- |
| Frontend | HTML, CSS e JavaScript vanilla |
| Gráficos | D3.js v7 |
| Ícones | Bootstrap Icons |
| Assistente | `gpt-4o-mini` via Cloudflare Worker |
| Persistência | Supabase + localStorage |
| Notícias | Google News RSS via `rss2json` |
| Automação | Node.js + GitHub Actions |
| Publicação | GitHub Pages |

## Publicação no GitHub Pages

Para publicar, suba estes arquivos e pastas principais:

- `index.html`
- `_worker.js`
- `minerai.png`
- imagens de equipamentos: `caminhao.png`, `escavadeira.png`, `carregadeira.png`, `perfuratriz.png`, etc.
- logos: `logo-sotreq.png`, `logo-cat.png`
- pasta `mapa/`
- pasta `automacao/`
- pasta `.github/workflows/`

Depois configure o GitHub Pages para servir a branch/pasta onde está o `index.html`.

## Automação de Notícias

O arquivo [.github/workflows/daily-news.yml](.github/workflows/daily-news.yml) agenda a atualização diária:

```yaml
cron: '0 8 * * *'
```

Isso equivale a **05:00 no horário de Brasília**.

O workflow executa:

```bash
node automacao/update-news.mjs
```

Se encontrar notícias novas, ele atualiza o `index.html`, cria um commit automático e publica no GitHub Pages após o deploy da branch.

Também é possível rodar manualmente pelo GitHub em **Actions > Atualizar noticias > Run workflow**.

## Uso Local

O site é estático. Para testar localmente:

```bash
python -m http.server 8080
```

Depois acesse:

```text
http://localhost:8080
```

Também é possível abrir o `index.html` diretamente no navegador, mas o servidor local evita limitações de carregamento de alguns recursos.

## Dados e Persistência

O portal usa dados embutidos no `index.html` e pode salvar alterações localmente no navegador.

A ordem prática é:

1. dados embutidos no `index.html`;
2. dados salvos no `localStorage`;
3. Supabase, quando configurado e disponível.

As carteiras Terras Raras e Green Field usam chaves separadas no `localStorage` para evitar mistura de dados.

## Arquivos Importantes

```text
index.html                         Portal principal
_worker.js                         Proxy para chamadas ao modelo de IA
minerai.png                        Ícone do MinerAÍ
automacao/update-news.mjs          Script de atualização diária de notícias
.github/workflows/daily-news.yml   Agendamento no GitHub Actions
mapa/uf.json                       GeoJSON do Brasil
sotreq-dados-2026-05-25.json       Exportação/base histórica
```

## Segurança e Uso

Este portal foi pensado para uso interno/comercial da Sotreq.

- A senha no front-end é apenas uma barreira simples de acesso visual, não autenticação forte.
- O Worker deve proteger a chave do modelo. Nunca coloque chave secreta diretamente no `index.html`.
- Revise informações sensíveis antes de compartilhar externamente.
- As premissas de dimensionamento são apresentadas em alto nível no portal.

## Objetivo Comercial

O objetivo central é apoiar consultores da Sotreq a:

- identificar oportunidades antes da fase formal de compra;
- priorizar contas com maior potencial;
- conversar com clientes usando CAPEX, estágio e timing como contexto;
- antecipar demanda de frota, peças, serviços e suporte;
- transformar sinais de mercado em ações comerciais concretas.

Material de inteligência de mercado para uso interno/comercial da Sotreq.
