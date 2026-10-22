# PontoCerto — Especificação do Produto

> Plataforma de análise de ponto comercial com dados públicos.
> Referência funcional: ondeabrir.com — usar SÓ como referência de funcionalidades. Não copiar textos, marca, layout, depoimentos nem exemplos.
> Dono do produto: Murilo · Versão do documento: 1.0 · Out/2026
> "PontoCerto" é codinome provisório (ver D1).

---

## 0. Como usar este documento (para o Claude Code)

1. Leia o documento inteiro antes de escrever qualquer código.
2. Trabalhe por fases (seção 14). Só avance de fase com os critérios de aceite da anterior passando.
3. Antes de cada fase, confira a seção 15 (Decisões em aberto) e pergunte ao Murilo o que estiver pendente para aquela fase.
4. **Nunca invente dado.** Todo número mostrado ao usuário precisa de fonte e data de referência rastreáveis. Se um dado não existir para a região, o relatório diz isso explicitamente.
5. Tudo marcado com **⚠️ VALIDAR** (códigos CNAE, tipos do Google, variáveis do IBGE, layout de arquivos, termos de uso) deve ser conferido na fonte oficial antes de usar.
6. **Não altere os serviços que já estão em produção** na VPS (n8n, Evolution API, Postgres atual, Redis, Traefik, Portainer) nem os workflows existentes do Murilo. Tudo novo entra como serviços novos no Swarm e workflows novos com prefixo `PC — `.

---

## 1. Visão do produto

O usuário informa **endereço + tipo de negócio** e recebe, em menos de 1 minuto, um relatório com:

- nota do ponto de 0 a 10 e os fatores que compõem a nota;
- concorrência no raio (distância, nota e nº de avaliações);
- perfil do público (população, domicílios, idade, renda);
- fluxo estimado e acesso (transporte, tipo de via, estacionamento, âncoras);
- aluguel estimado e potencial de faturamento (sempre como faixa);
- tendência do setor (aberturas e fechamentos de CNPJ);
- percepção da região a partir de avaliações públicas, interpretada por IA;
- conclusão com checklist de validação presencial.

O relatório é exportado em PDF, fica no histórico e pode ser comparado com outro ponto lado a lado.

**Público:** quem vai abrir negócio físico, quem quer expandir, franqueados, consultores e corretores de imóvel comercial.

**Promessa:** reduzir o risco de escolher o ponto errado. **Nunca** prometer sucesso do negócio.

---

## 2. Escopo funcional

### 2.1 Jornada do usuário

1. Landing page → botão "Analisar localização".
2. Cadastro/login (e-mail + senha; login com Google — D7).
3. Tela de nova análise:
   - endereço com autocomplete;
   - tipo de negócio (lista com busca);
   - raio (pré-preenchido pelo tipo, editável: 300, 500, 1000, 1500, 2000 m);
   - área do imóvel em m² (opcional, usada no cálculo de aluguel).
4. Confirmação do pino no mapa (arrastável para ajuste fino).
5. Consome 1 crédito → tela de progresso mostrando a etapa atual ("Buscando concorrentes…", "Calculando público…").
6. Relatório na tela + botão "Baixar PDF".
7. Histórico; selecionar 2 análises para comparar.
8. Sem crédito → página de planos.

Visitante pode ver um **relatório de exemplo** completo (gerado pela própria plataforma, endereço público) antes de comprar.

### 2.2 Tipos de negócio

Tabela configurável no banco (seção 6.3). MVP com ~14 tipos; meta de 40+. Se o tipo do usuário não existir, ele escolhe o mais próximo; o admin pode criar tipos novos sem deploy.

### 2.3 O relatório — seções

Cada número mostra a fonte e a data de referência (ex.: "IBGE — Censo 2022", "Receita Federal — base CNPJ de set/2026", "Google — consulta em 22/10/2026", "© OpenStreetMap contributors").

1. **Visão geral:** nota 0–10, classificação (Fraco < 4 · Regular 4–5,9 · Bom 6–7,9 · Ótimo ≥ 8), nível de confiança (alta/média/baixa), fatores da nota com a contribuição de cada um, resumo executivo (IA), mapa do raio.
2. **Concorrência:** mapa com o ponto e os concorrentes; lista com nome, distância, nota Google e nº de avaliações; concorrente direto mais próximo; concorrentes por 10 mil habitantes no raio vs. município; leitura (IA).
3. **Público:** população e domicílios no raio, densidade (hab/km²), faixas etárias, renda média e posição vs. município (percentil).
4. **Potencial financeiro:** mercado potencial mensal da categoria no raio (R$), faturamento potencial (faixa mín–máx), aluguel estimado (faixa R$/m² e total, se a área foi informada), relação aluguel/faturamento vs. referência do tipo.
5. **Fluxo e acesso:** índice de fluxo (Baixo/Médio/Alto + 0–10), paradas de ônibus e estações em 500 m, tipo de via do endereço, estacionamentos próximos, âncoras geradoras de tráfego (supermercado, escola, hospital, shopping, universidade…), potencial de conversão do ponto.
6. **Tendência do setor:** aberturas e baixas de CNPJ do setor nos últimos 24 meses (raio ampliado e município), saldo, taxa de mortalidade vs. município, gráfico mensal.
7. **Percepção da região (IA):** o que clientes elogiam e reclamam nos concorrentes e no entorno, e as oportunidades (lacunas que os concorrentes não atendem).
8. **Conclusão:** recomendação (IA), pontos fortes, pontos de atenção, checklist de validação presencial.
9. **Rodapé em todas as páginas:** fontes, datas e o aviso "Estimativa baseada em dados públicos. Não garante o resultado do negócio."

### 2.4 Comparação

- Selecionar 2 análises (3 na Fase 5) do **mesmo tipo de negócio** (avisar se forem de tipos diferentes ou de versões diferentes do motor).
- Tabela lado a lado destacando só os indicadores que diferem de verdade (> 5% ou mudança de faixa).
- Vencedor por indicador e placar ("Ponto A vence em 7 de 11 indicadores").

### 2.5 Histórico

Lista com data, endereço, tipo e nota. Ações: abrir, baixar PDF, renomear (apelido, ex.: "Loja Rua X"), excluir.

### 2.6 Planos e créditos

Preços ficam em `cfg.planos`, editáveis no admin. Valores abaixo são **referência de mercado observada em out/2026**, não decisão (D1):

| Plano | Créditos | Validade | Cobrança | Referência |
|---|---|---|---|---|
| Avulso | 1 | não expira | pagamento único | R$ 97 |
| Básico | 5 | não expira | pagamento único | R$ 197 |
| Pro mensal | 30/mês | acumulam enquanto a assinatura estiver ativa; zeram ao cancelar ou após a carência de inadimplência | assinatura no cartão | R$ 297/mês |
| Pro anual | 360 | 12 meses da compra | pagamento único (Pix, ou cartão à vista/parcelado) | 40% de desconto sobre 12× o mensal |

Regras:
- Créditos comprados avulsos **nunca** somem, mesmo que a assinatura seja cancelada.
- Análise que falha → crédito estornado automaticamente.
- Recursos por plano ficam em `cfg.planos.recursos` (jsonb). Referência: comparação e histórico completo a partir do Básico.

### 2.7 Área administrativa (`/admin`)

- CRUD de tipos de negócio (CNAEs, tipos Google, raio, pesos, parâmetros).
- Planos e preços.
- Parâmetros do motor (`cfg.parametros`).
- Lista de análises com status, etapa, tempo, custo de APIs e erro.
- Ajuste manual de créditos (motivo obrigatório).
- Status das cargas de dados (data de referência e data da última carga de cada fonte).

### 2.8 Fora do MVP (Fase 5)

- Páginas SEO programáticas ("Onde abrir padaria em <cidade>") com dados agregados.
- Página "Panorama" por município.
- Pedido e entrega de análise por WhatsApp (Evolution API) — D8.
- Zoneamento no checklist (São Paulo: GeoSampa).
- Comparação de 3 pontos; mapa de calor de oportunidade no município.

---

## 3. Arquitetura

### 3.1 Infra existente (NÃO alterar)

VPS Hostinger com Docker Swarm, Traefik (TLS/roteamento), Portainer, n8n, Evolution API, PostgreSQL, Redis. Domínio `automaxhub.com.br`. Hostname `AutomaxhubAI`.

### 3.2 Serviços novos (stack `pontocerto` no Swarm)

| Serviço | Imagem | Função |
|---|---|---|
| `pc-db` | `postgis/postgis` (PG 16 + PostGIS 3.x) | dados de referência + dados da aplicação. **Separado** do Postgres do n8n. Volume próprio. |
| `pc-web` | build próprio (Next.js) | site, login, análises, créditos, pagamentos, admin, armazenamento dos PDFs (volume `pc-relatorios`) |
| `pc-gotenberg` | `gotenberg/gotenberg:8` | HTML → PDF |
| `pc-etl` | build próprio (Python 3.12 + GDAL) | jobs de carga; roda sob demanda/agendado (replicas 0 fora da execução) |

Rede:
- Colocar os serviços novos **também na rede overlay `AutomaxHubNet`**, em que o n8n já está. Assim o n8n alcança `pc-db`, `pc-gotenberg` e `pc-web` sem alterar o serviço do n8n.
- Só o `pc-web` é exposto pelo Traefik (copiar o padrão de labels e certresolver dos serviços existentes).
- `pc-db` e `pc-gotenberg` **sem** porta publicada.

### 3.3 Fluxo de uma análise

```
[pc-web] usuário confirma
   → transação: debita 1 crédito + cria app.analises (status='pendente')
   → POST http://n8n_n8n_webhook:5678/webhook/pc-analise  {analise_id}   header X-PC-Secret
[n8n]  PC — Analise
   → carrega análise + tipo → geocode (se faltar) → funções PostGIS
   → Google Places (concorrentes + avaliações) → Overpass (transporte, via, âncoras)
   → Code node score.js (indicadores + nota) → Claude API (textos)
   → HTML → Gotenberg → PDF → POST interno pc-web /api/internal/relatorios/{id}
   → UPDATE app.analises (status='concluida', resultado, nota, …)
   → em erro: workflow PC — Erro (status='erro', estorno, alerta)
[pc-web] polling a cada 2 s em GET /api/analises/{id} mostrando app.analises.etapa
```

### 3.4 Divisão de responsabilidades

| Camada | Responsável por |
|---|---|
| `pc-web` | regras de crédito e pagamento, autenticação, UI, armazenamento e entrega dos PDFs |
| `pc-db` | dados de referência e **toda a lógica espacial** em funções SQL versionadas (`db/functions/`) |
| n8n | orquestração, chamadas externas (Google, Overpass, Claude), montagem do HTML, PDF |
| `engine/score.js` | cálculo dos indicadores e da nota. Fonte da verdade no repositório, com testes (`node --test`). Script `scripts/sync-n8n` injeta o conteúdo no Code node do workflow pela API do n8n. |
| `pc-etl` | carga e atualização das bases públicas + pré-cálculo dos benchmarks |

---

## 4. Fontes de dados

### 4.1 IBGE — Malha de setores censitários 2022

- **O quê:** polígonos dos setores censitários (menor unidade do Censo).
- **Onde:** FTP de Geociências do IBGE (malhas de setores censitários), por UF, GPKG ou SHP. ⚠️ VALIDAR versão (preliminar × definitiva) e usar a mais recente.
- **Carga:** `ogr2ogr` → `ref.setores` (SRID 4674 — SIRGAS 2000), com `area_m2` calculada via `geography`.
- **Frequência:** única; recarregar se sair versão nova.

### 4.2 IBGE — Agregados por setores censitários (Censo 2022)

- **O quê:** população, domicílios, faixas etárias e, se publicado, rendimento.
- **Onde:** FTP do IBGE (Censo 2022 › Agregados por Setores Censitários), CSV nacional por tema + dicionário de variáveis.
- **Carga:** `ref.setores_agregados`. Mapear os códigos de variável pelo dicionário oficial. ⚠️ VALIDAR nomes e códigos.
- **Renda:** verificar se o Censo 2022 já tem rendimento por setor. Se não tiver, usar o Censo 2010 por setor, reponderado para os setores 2022 por interseção de área e corrigido pelo IPCA. Gravar `renda_fonte='censo2010_ajustado'` e mostrar isso no relatório.

### 4.3 IBGE — CNEFE 2022 (Cadastro Nacional de Endereços para Fins Estatísticos)

- **O quê:** endereços com coordenadas, código do setor e espécie (domicílio, estabelecimento etc.).
- **Uso:** (a) geocodificar os CNPJs da Receita sem pagar API; (b) centróide de CEP como fallback.
- **Carga:** `ref.cnefe`. ⚠️ VALIDAR colunas no dicionário.

### 4.4 Receita Federal — Dados abertos do CNPJ

- **Onde:** portal de dados abertos da Receita; arquivos mensais `.zip` (Empresas, Estabelecimentos, Cnaes, Municipios, Motivos…). CSV sem cabeçalho, separador `;`, encoding latin-1, campos entre aspas. ⚠️ VALIDAR layout no documento de metadados que acompanha os arquivos.
- **Carga** (filtrar UF na leitura; MVP = estado de SP): `ref.cnpj_estab`, `ref.cnpj_empresa`, `ref.cnaes`, `ref.municipios_rf`.
- **Atenção:** o código de município da Receita **não** é o código IBGE → criar de-para em `ref.municipios_rf`.
- **Situação cadastral:** 02 = ativa, 08 = baixada (as demais: nula, suspensa, inapta). ⚠️ VALIDAR.
- **LGPD:** NÃO carregar e-mail, telefone nem dados de sócios. Não exibir razão social de MEI (é nome de pessoa física): exibir só nome fantasia, ou "Estabelecimento sem nome fantasia".
- **Geocodificação** (job `geocode_cnpj`): normalizar endereço → casar CEP + número com o CNEFE (`geo_precisao='exata'`) → senão centróide do CEP no CNEFE (`'cep'`) → senão sem geometria (`'sem'`). **Nunca** geocodificar a base em massa com o Google (custo e termos de uso).
- **Frequência:** mensal. Carregar em tabela `_novo` e trocar por `RENAME` dentro de transação (zero downtime). Depois recalcular `ref.benchmarks_municipio`.

### 4.5 Google Maps Platform

- **Places Autocomplete / Geocoding:** endereço digitado pelo usuário.
- **Places API (New) — Nearby Search:** concorrentes por tipo no raio (id, nome, location, rating, userRatingCount, priceLevel, businessStatus). Usar **field mask** só com os campos necessários (o custo depende dos campos).
- **Place Details (reviews):** até 5 avaliações por lugar para a "Percepção da região".
- **Text Search:** para tipos sem categoria específica no Google (ex.: estúdio de tatuagem).
- **Static Maps:** imagem do mapa no PDF. **Maps JavaScript:** mapa na tela.
- **Termos de uso (obrigatório, ⚠️ VALIDAR a versão atual antes da Fase 2):**
  - não montar base própria com conteúdo do Google; `place_id` pode ser guardado;
  - coordenadas em cache por no máximo 30 dias;
  - dados do Places exibidos em mapa devem estar em mapa Google;
  - exibir a atribuição do Google;
  - avaliações não são armazenadas: vão só em memória para a IA, que gera um resumo próprio;
  - para o histórico, guardar `place_id` + métricas agregadas; ao reabrir uma análise antiga, rebuscar detalhes pelo `place_id` ou servir o PDF gerado, com aviso da data.

### 4.6 OpenStreetMap

- **MVP:** Overpass API (endpoint configurável em `cfg.parametros`, timeout 25 s, retry com backoff). Consultas:
  - transporte em 500 m: `highway=bus_stop`, `public_transport=platform`, `railway=station`, `station=subway`;
  - via mais próxima do ponto: `highway=*`;
  - estacionamento em 300 m: `amenity=parking`;
  - âncoras no raio: `shop=supermarket`, `shop=mall`, `amenity=school|university|hospital|clinic|marketplace` (lista por tipo em `cfg.tipos_negocio.ancoras`).
- **Fase 5:** carregar o extrato da Geofabrik (Sudeste) no `pc-db` com `osm2pgsql` e trocar o Overpass por SQL.
- Atribuição obrigatória: "© OpenStreetMap contributors".

### 4.7 IBGE — POF (Pesquisa de Orçamentos Familiares)

- **O quê:** despesa monetária média mensal por categoria e por classe de rendimento. Usar POF 2017-2018. ⚠️ VALIDAR se já saiu edição mais nova.
- **Carga:** curadoria manual em `data/seeds/pof_despesa.csv` (categoria_pof, faixa_renda_min, faixa_renda_max, despesa_mensal_familia, ano_base, fonte_tabela).
- **Correção:** IPCA via API SIDRA do IBGE até o mês da análise (guardar o fator em `cfg.parametros`, atualizado mensalmente pelo ETL).

### 4.8 Aluguel comercial

- **MVP:** `data/seeds/aluguel_referencia.csv` (uf, cd_mun, bairro, r_m2_min, r_m2_max, fonte, data_ref), começando pelo município de São Paulo. Fonte preferida: índice FipeZap comercial onde existir; completar por bairro manualmente.
- Sem dado para a região → "Aluguel: sem referência para esta região".
- **Não** fazer scraping de portais imobiliários sem a decisão D5.

---

## 5. Modelo de dados (`pc-db`)

Schemas: `ref` (dados públicos), `cfg` (configuração), `app` (aplicação). Migrations numeradas e idempotentes em `db/migrations/`. Extensões: `postgis`, `pg_trgm`, `unaccent`, `citext`.

### 5.1 `ref` — dados públicos

```sql
create table ref.setores (
  cd_setor   text primary key,
  cd_mun     text not null,
  geom       geometry(MultiPolygon, 4674) not null,
  area_m2    double precision not null
);
create index on ref.setores using gist (geom);
create index on ref.setores (cd_mun);

create table ref.setores_agregados (
  cd_setor        text primary key references ref.setores,
  pop_total       int,
  domicilios      int,
  pop_0_14        int,
  pop_15_29       int,
  pop_30_59       int,
  pop_60_mais     int,
  renda_media_dom numeric,       -- R$ correntes do ano de referência
  renda_fonte     text,          -- 'censo2022' | 'censo2010_ajustado' | null
  renda_ano       int
);

create table ref.cnefe (
  id              bigserial primary key,
  cd_setor        text,
  cep             char(8),
  logradouro_norm text,          -- sem acento, minúsculo, sem tipo
  numero          text,
  especie         smallint,
  geom            geometry(Point, 4674)
);
create index on ref.cnefe using gist (geom);
create index on ref.cnefe (cep, numero);

create table ref.cnpj_estab (
  cnpj              char(14) primary key,
  cnpj_basico       char(8) not null,
  matriz_filial     smallint,
  nome_fantasia     text,
  situacao          smallint,      -- 2 ativa, 8 baixada…
  data_situacao     date,
  data_inicio       date,
  cnae_principal    int,           -- subclasse, 7 dígitos
  cnaes_secundarios int[],
  cep               char(8),
  tipo_logradouro   text,
  logradouro        text,
  numero            text,
  bairro            text,
  municipio_rf      text,
  cd_mun            text,
  geom              geometry(Point, 4674),
  geo_precisao      text check (geo_precisao in ('exata','cep','sem'))
);
create index on ref.cnpj_estab using gist (geom);
create index on ref.cnpj_estab (cnae_principal, situacao);
create index on ref.cnpj_estab (cd_mun, cnae_principal);

create table ref.cnpj_empresa (
  cnpj_basico       char(8) primary key,
  capital_social    numeric,
  porte             smallint,
  natureza_juridica int
);

create table ref.cnaes (codigo int primary key, descricao text not null);

create table ref.municipios_rf (
  codigo_rf text primary key,
  cd_mun    text,                  -- código IBGE (7 dígitos)
  nome      text,
  uf        char(2)
);

create table ref.pof_despesa (
  categoria_pof          text,
  faixa_renda_min        numeric,
  faixa_renda_max        numeric,
  despesa_mensal_familia numeric,
  ano_base               int,
  fonte_tabela           text,
  primary key (categoria_pof, faixa_renda_min)
);

create table ref.aluguel_referencia (
  uf       char(2),
  cd_mun   text,
  bairro   text,                   -- null = média do município
  r_m2_min numeric,
  r_m2_max numeric,
  fonte    text,
  data_ref date
);

-- Pré-calculado pelo ETL após cada carga
create table ref.benchmarks_municipio (
  cd_mun                  text,
  tipo_id                 text,
  pop                     int,
  ativos                  int,
  hab_por_estab           numeric,
  aberturas_24m           int,
  baixas_24m              int,
  mortalidade_24m         numeric,
  crescimento_liquido_24m numeric,
  dist_renda              numeric[],   -- percentis 0..100 da renda por setor (ponderado por domicílios)
  dist_densidade          numeric[],   -- percentis 0..100 de hab/km² por setor
  dist_estab_km2          numeric[],   -- percentis de estabelecimentos ativos/km²
  dist_transporte         numeric[],   -- percentis do índice de transporte (amostra)
  atualizado_em           timestamptz,
  primary key (cd_mun, tipo_id)
);

create table ref.cargas (
  fonte         text primary key,      -- 'ibge_setores', 'cnpj', …
  referencia    text,                  -- ex.: '2026-09'
  carregado_em  timestamptz,
  linhas        bigint,
  observacao    text
);
```

### 5.2 `cfg` — configuração

```sql
create table cfg.tipos_negocio (
  id              text primary key,          -- 'padaria'
  nome            text not null,
  cnaes           int[] not null,
  usar_cnae_sec   boolean default false,     -- contar CNAE secundário como concorrente
  google_types    text[] not null default '{}',
  google_text     text,                      -- Text Search quando não há type adequado
  categoria_pof   text,
  raio_m          int not null default 500,
  area_padrao_m2  int,
  renda_alvo      text check (renda_alvo in ('baixa','media','alta','todas')),
  cluster         boolean default false,     -- concorrente vizinho ajuda (polo gastronômico etc.)
  fator_local     numeric not null,          -- fração do gasto da categoria feita perto de casa/trabalho
  aluguel_max_pct numeric,                   -- aluguel/faturamento saudável (ex.: 0.10)
  pesos           jsonb not null,            -- sobrescreve os pesos padrão (seção 7.2)
  ancoras         text[],                    -- chaves de âncoras relevantes para o tipo
  checklist_extra text[],
  ativo           boolean default true
);

create table cfg.planos (
  id             text primary key,           -- 'avulso','basico','pro_mensal','pro_anual'
  nome           text not null,
  creditos       int not null,
  preco_centavos int not null,
  cobranca       text check (cobranca in ('unico','assinatura')),
  validade_meses int,                        -- null = não expira
  recursos       jsonb not null default '{}',
  ativo          boolean default true
);

create table cfg.parametros (chave text primary key, valor jsonb not null);
-- Exemplos: coeficientes da nota, faixas de classificação, endpoint Overpass,
-- fator IPCA, preços unitários das APIs (para cálculo de custo), versão do motor.
```

### 5.3 `app` — aplicação

```sql
create table app.usuarios (
  id               uuid primary key default gen_random_uuid(),
  email            citext unique not null,
  nome             text,
  senha_hash       text,
  google_sub       text unique,
  papel            text not null default 'user' check (papel in ('user','admin')),
  aceite_termos_em timestamptz,
  criado_em        timestamptz default now()
);

create table app.creditos_lotes (
  id             uuid primary key default gen_random_uuid(),
  usuario_id     uuid not null references app.usuarios,
  origem         text not null,      -- 'avulso','basico','pro_mensal','pro_anual','admin'
  quantidade     int not null,
  saldo          int not null check (saldo >= 0),
  expira_em      timestamptz,        -- null = não expira
  assinatura_id  uuid,               -- lotes do Pro mensal
  pagamento_id   uuid,
  criado_em      timestamptz default now()
);

create table app.creditos_mov (      -- livro-razão imutável
  id          bigserial primary key,
  usuario_id  uuid not null,
  lote_id     uuid not null references app.creditos_lotes,
  delta       int not null,
  motivo      text not null check (motivo in ('compra','assinatura','consumo','estorno','ajuste','expiracao')),
  analise_id  uuid,
  observacao  text,
  criado_em   timestamptz default now()
);

create table app.analises (
  id                 uuid primary key default gen_random_uuid(),
  usuario_id         uuid not null references app.usuarios,
  tipo_id            text not null references cfg.tipos_negocio,
  endereco_digitado  text not null,
  endereco_formatado text,
  bairro             text,
  cd_mun             text,
  geom               geometry(Point, 4674),
  raio_m             int not null,
  area_m2            int,
  apelido            text,
  status             text not null check (status in ('pendente','processando','concluida','erro')),
  etapa              text,
  erro               text,
  resultado          jsonb,           -- contrato na seção 6.5
  nota               numeric(3,1),
  confianca          text,
  pdf_path           text,
  versao_motor       text,
  custo_apis         jsonb,           -- contagem de chamadas por SKU + custo estimado
  duracao_ms         int,
  criado_em          timestamptz default now(),
  concluido_em       timestamptz
);
create index on app.analises (usuario_id, criado_em desc);

create table app.pagamentos (
  id             uuid primary key default gen_random_uuid(),
  usuario_id     uuid not null,
  plano_id       text not null,
  provedor       text not null,
  provedor_ref   text unique not null,  -- idempotência dos webhooks
  metodo         text check (metodo in ('pix','cartao')),
  status         text not null,
  valor_centavos int not null,
  criado_em      timestamptz default now(),
  pago_em        timestamptz
);

create table app.assinaturas (
  id            uuid primary key default gen_random_uuid(),
  usuario_id    uuid not null,
  plano_id      text not null,
  provedor_ref  text unique not null,
  status        text not null check (status in ('ativa','inadimplente','cancelada')),
  periodo_fim   timestamptz,
  criado_em     timestamptz default now()
);
```

**Consumo de crédito:** em transação, `SELECT … FOR UPDATE` do lote com saldo > 0 e não expirado, ordenando por `expira_em` (nulos por último) e depois `criado_em` → `saldo - 1` + `creditos_mov` (`consumo`). Saldo nunca negativo. Estorno devolve ao mesmo lote.

**Expiração:** job diário zera lotes vencidos (`expiracao`). Assinatura cancelada (fim do período pago) ou inadimplente além da carência (D3) → zera os lotes com aquele `assinatura_id`.

---

## 6. Motor da análise (n8n)

### 6.1 Entrada

`POST http://n8n_n8n_webhook:5678/webhook/pc-analise` · header `X-PC-Secret` · body `{ "analise_id": "<uuid>" }`. Responder `202` imediatamente e processar em seguida.

### 6.2 Workflow `PC — Analise` (etapas)

Atualizar `app.analises.etapa` entre as etapas (alimenta a barra de progresso).

1. **Webhook** → valida segredo → responde 202.
2. **Postgres:** carrega a análise + `cfg.tipos_negocio` + `cfg.parametros`; `status='processando'`.
3. **Geocode** (se `geom` nula): Google Geocoding → grava `geom`, `endereco_formatado`, `bairro`; `cd_mun` via `ST_Intersects` com `ref.setores`. Endereço fora da área com dados carregados → erro amigável + estorno.
4. **Postgres em paralelo** (funções da 6.4): `pc_demografia`, `pc_concorrentes_cnpj`, `pc_tendencia`, `pc_estab_total`, `pc_benchmarks`, `pc_aluguel`, `pc_mercado`.
5. **Google Places Nearby** por `google_types` no raio (ou Text Search com `google_text`). Se vier o máximo de resultados, subdividir o raio em quadrantes.
6. **Deduplicação** CNPJ × Google: nome normalizado com similaridade trigram ≥ 0,4 **e** distância ≤ 60 m → mesmo estabelecimento. Saída: lista unificada com `origem` (`cnpj`, `google`, `ambos`).
7. **Avaliações:** Place Details (reviews) dos até 8 concorrentes mais próximos + até 3 âncoras. Ficam só em memória para a etapa 10.
8. **Overpass:** transporte em 500 m, via mais próxima, estacionamentos em 300 m, âncoras do tipo no raio.
9. **Code node `score.js`:** indicadores, fatores, nota, confiança (seção 7).
10. **Claude API:** textos (seção 8). Validar o JSON; 1 nova tentativa se inválido.
11. **Static Maps:** imagem do raio + concorrentes, embutida em base64 no HTML.
12. **HTML** (template `templates/relatorio.html`) → **Gotenberg** (`/forms/chromium/convert/html`) → PDF.
13. **POST interno** `http://pc-web:3000/api/internal/relatorios/{id}` com o PDF (header `X-PC-Secret`).
14. **Postgres:** `resultado`, `nota`, `confianca`, `pdf_path`, `versao_motor`, `custo_apis`, `duracao_ms`, `status='concluida'`, `concluido_em`.

**Workflow `PC — Erro`** (Error Trigger): `status='erro'`, `erro=<mensagem>`, estorno do crédito, alerta para o Murilo via Evolution API.

**Meta de tempo:** p95 ≤ 45 s. Toda chamada externa com timeout e retry. Falha de uma fonte **não** derruba a análise: o fator sai como indisponível e a nota é recalculada sem ele (seção 7.3).

### 6.3 Tipos de negócio iniciais (MVP)

⚠️ VALIDAR cada CNAE contra `ref.cnaes` e cada tipo contra a tabela oficial de tipos do Places API (New) antes do seed.

| id | Nome | CNAEs (subclasse) | Google types | Raio (m) | Renda alvo | Cluster |
|---|---|---|---|---|---|---|
| restaurante | Restaurante | 5611201 | restaurant | 800 | todas | sim |
| lanchonete | Lanchonete / cafeteria | 5611203 | cafe, coffee_shop, fast_food_restaurant | 500 | todas | sim |
| bar | Bar | 5611204, 5611205 | bar | 800 | todas | sim |
| padaria | Padaria | 4721102, 1091102 | bakery | 500 | todas | não |
| hortifruti | Hortifrúti | 4724500 | grocery_store, market | 800 | media | não |
| minimercado | Minimercado / mercearia | 4712100 | grocery_store, convenience_store | 500 | todas | não |
| farmacia | Farmácia | 4771701, 4771702 | pharmacy, drugstore | 800 | todas | não |
| academia | Academia | 9313100 | gym, fitness_center | 1000 | media | não |
| salao | Salão de beleza / estética | 9602501, 9602502 | beauty_salon, hair_salon | 800 | media | não |
| barbearia | Barbearia | 9602501 | barber_shop | 800 | todas | não |
| petshop | Pet shop | 4789004, 9609208 | pet_store | 1000 | media | não |
| lavanderia | Lavanderia | 9601701 | laundry | 1000 | media | não |
| odonto | Clínica odontológica | 8630504 | dentist, dental_clinic | 1000 | media | não |
| tatuagem | Estúdio de tatuagem | 9609206 | — (Text Search "estúdio de tatuagem") | 2000 | todas | não |

Observação: salão e barbearia compartilham a subclasse 9602501. Na concorrência por CNAE os dois se misturam; o Google separa. O relatório deve mencionar isso nesses tipos.

`fator_local`, `aluguel_max_pct`, `categoria_pof`, `area_padrao_m2`, `ancoras` e `checklist_extra` de cada tipo: propor valores iniciais, listar numa tabela para o Murilo aprovar e só então fazer o seed.

### 6.4 Funções SQL (`db/functions/`)

Todas recebem `lng, lat` em WGS84 e convertem para 4674. Distâncias e áreas sempre via `geography`.

- `pc_demografia(lng, lat, raio_m)` → população, domicílios, faixas etárias, renda média (ponderada por domicílios), densidade (hab/km²), `cobertura` (fração do buffer coberta por setores com dado) e `renda_fonte`. Ponderação por área: cada setor contribui com `valor × área_interseção / área_setor`.
- `pc_concorrentes_cnpj(lng, lat, raio_m, cnaes int[], usar_sec bool)` → ativos (`situacao=2`) com CNAE principal (e secundário, se `usar_sec`) no conjunto; nome fantasia, distância, `data_inicio`, `geo_precisao`.
- `pc_tendencia(lng, lat, raio_m, cnaes int[], meses int)` → série mensal de aberturas (`data_inicio`) e baixas (`situacao=8` por `data_situacao`) no raio ampliado (2× o raio) e no município; totais; `mortalidade_24m = baixas / (ativos + baixas)`; `crescimento_liquido_24m = (aberturas − baixas) / ativos`.
- `pc_estab_total(lng, lat, raio_m)` → estabelecimentos ativos de qualquer CNAE no raio e por km² (proxy de movimento comercial e empregos).
- `pc_benchmarks(cd_mun, tipo_id)` → linha de `ref.benchmarks_municipio`.
- `pc_aluguel(cd_mun, bairro)` → faixa R$/m² do bairro; senão a do município; senão nulo.
- `pc_mercado(lng, lat, raio_m, categoria_pof)` → Σ por setor de `domicílios_ponderados × despesa_mensal_familia(faixa da renda média do setor) × fator_ipca`.

### 6.5 Contrato do `resultado` (jsonb)

```json
{
  "versao_motor": "1.0.0",
  "entrada": { "endereco": "", "tipo_id": "", "raio_m": 500, "area_m2": null, "lat": 0, "lng": 0 },
  "nota": 7.2,
  "classificacao": "Bom",
  "confianca": "media",
  "fatores": [
    { "id": "demanda", "score": 8.1, "peso": 0.20, "contribuicao": 1.62, "disponivel": true, "explicacao": "" }
  ],
  "publico": {
    "pop": 0, "domicilios": 0, "densidade_km2": 0,
    "faixas_etarias": { "0_14": 0, "15_29": 0, "30_59": 0, "60_mais": 0 },
    "renda_media_dom": 0, "renda_percentil_mun": 0, "renda_fonte": "censo2022", "cobertura": 1.0
  },
  "concorrencia": {
    "total": 0, "por_10k_hab": 0, "por_10k_hab_mun": 0,
    "mais_proximo_m": 0, "nota_media_ponderada": 0,
    "lista": [ { "place_id": null, "cnpj": null, "nome": "", "distancia_m": 0, "rating": null, "avaliacoes": null, "origem": "ambos" } ]
  },
  "fluxo": {
    "indice": 0, "faixa": "Medio", "paradas_onibus_500m": 0, "estacoes_500m": 0,
    "via": "secondary", "estacionamentos_300m": 0,
    "ancoras": [ { "tipo": "supermercado", "nome": "", "distancia_m": 0 } ],
    "potencial_conversao": 0
  },
  "financeiro": {
    "mercado_mensal": 0, "faturamento_min": 0, "faturamento_max": 0,
    "aluguel_m2_min": null, "aluguel_m2_max": null, "aluguel_total_min": null, "aluguel_total_max": null,
    "aluguel_sobre_faturamento": null, "aluguel_max_pct_tipo": 0.10
  },
  "tendencia": {
    "aberturas_24m": 0, "baixas_24m": 0, "mortalidade_24m": 0, "mortalidade_24m_mun": 0,
    "crescimento_liquido_24m": 0, "crescimento_liquido_24m_mun": 0,
    "serie": [ { "mes": "2024-11", "aberturas": 0, "baixas": 0 } ]
  },
  "ia": { },
  "avisos": [ "Renda do Censo 2010 ajustada pelo IPCA" ],
  "fontes": [ { "nome": "IBGE — Censo 2022", "referencia": "2022" } ]
}
```

O pc-web renderiza a tela a partir desse JSON; o PDF usa o mesmo JSON. Qualquer mudança no contrato → nova `versao_motor`.

---

## 7. Indicadores e nota (`engine/score.js`)

Todos os coeficientes ficam em `cfg.parametros` (calibráveis sem deploy). Cada fator gera um score 0–10, comparado com a distribuição do **próprio município** (percentil), para a nota ser relativa à cidade.

### 7.1 Fatores

| Fator | Cálculo | Score alto quando |
|---|---|---|
| `demanda` | percentil da densidade populacional do raio em `dist_densidade` → ×10 | muita gente morando perto |
| `renda` | p = percentil da renda média em `dist_renda` (0–1). `alta`: 10p · `media`: 10(1 − 2·\|p − 0,5\|) · `baixa`: 10(1 − p) · `todas`: 5 + 5p | renda combina com o público do tipo |
| `concorrencia` | r = (hab. por concorrente no raio) ÷ (`hab_por_estab` do município); score = clamp(5 + 5·log2 r, 0, 10). Zero concorrentes → 9 e aviso para a IA comentar que pode indicar falta de demanda | menos saturado que a média da cidade |
| `proximidade` | concorrente direto mais próximo: < 50 m → 2 · 50–150 m → 5 · 150–300 m → 7 · > 300 m → 9. Se `cluster=true` → 6 fixo | concorrente direto longe |
| `qualidade_conc` | nota média dos concorrentes ponderada por nº de avaliações: ≥ 4,6 → 3 · 4,3–4,6 → 5 · 4,0–4,3 → 7 · < 4,0 → 9 | concorrência mal avaliada (oportunidade) |
| `fluxo` | média dos percentis de: densidade pop., estabelecimentos ativos/km², transporte (ônibus × 1 + estação × 5 em 500 m), âncoras do tipo no raio → ×10. Faixa: < 4 Baixo · 4–7 Médio · > 7 Alto | muito movimento estimado |
| `acesso` | via mais próxima: trunk/primary 9 · secondary 8 · tertiary 7 · residential 4 · outras 3; +1 se há estacionamento em ≤ 300 m (máx. 10) | via de passagem e com estacionamento |
| `tendencia` | clamp(5 + k·(crescimento_raio − crescimento_mun), 0, 10), k em `cfg.parametros` (inicial 25) | setor crescendo mais que na cidade |
| `viabilidade` | só com área e aluguel: x = aluguel_médio ÷ faturamento_médio. x ≤ 0,5·`aluguel_max_pct` → 10; x = `aluguel_max_pct` → 5; x ≥ 2·`aluguel_max_pct` → 0 (linear entre os pontos) | aluguel cabe no faturamento |

### 7.2 Nota final

- Pesos padrão (somam 1): demanda 0,20 · renda 0,15 · concorrencia 0,20 · proximidade 0,05 · qualidade_conc 0,05 · fluxo 0,15 · acesso 0,05 · tendencia 0,05 · viabilidade 0,10. Cada tipo pode sobrescrever em `cfg.tipos_negocio.pesos`.
- `nota = Σ(peso·score) ÷ Σ(peso dos fatores disponíveis)`, 1 casa decimal.
- `contribuicao` de cada fator = peso normalizado × score (é o que aparece em "fatores que compõem a nota").

### 7.3 Confiança

Começa em "alta" e cai um nível para cada condição (mínimo "baixa"):
- `cobertura` demográfica < 0,8;
- renda vinda do Censo 2010 ajustado;
- < 70% dos concorrentes CNPJ com `geo_precisao='exata'`;
- 2 ou mais fatores indisponíveis.

### 7.4 Faturamento e conversão

- `share = 1 ÷ (concorrentes_unificados + 1) × ajuste`, com ajuste 1,2 se `nota_media_ponderada` < 4,0 e 0,8 se > 4,6 (senão 1).
- `faturamento_medio = mercado_mensal × fator_local × share`.
- Exibir sempre como faixa: `[0,7 × médio ; 1,3 × médio]`, com o rótulo "estimativa".
- `potencial_conversao = 0,5·fluxo + 0,3·acesso + 0,2·proximidade` (rótulo "experimental").

### 7.5 Testes

`engine/score.test.js` com fixtures (ponto denso e saturado; ponto sem concorrente; ponto sem dado de renda; fatores ausentes; tipo cluster). Testes "golden": mesma entrada → mesmo resultado. Mudou fórmula ou peso → sobe `versao_motor`.

---

## 8. IA — textos do relatório

- Claude API via n8n. Modelo em variável de ambiente `PC_LLM_MODEL` (não fixar no código).
- **Entrada:** o `resultado` (sem a chave `ia`) + até 40 trechos de avaliações (sem nome de autor) + nome do tipo de negócio.
- **Saída:** JSON estrito, gravado em `resultado.ia`:

```json
{
  "resumo": "3 a 4 frases",
  "leitura": { "concorrencia": "", "publico": "", "potencial": "", "fluxo": "", "tendencia": "" },
  "percepcao_regiao": { "elogios": [], "reclamacoes": [], "oportunidades": [] },
  "pontos_fortes": [],
  "pontos_atencao": [],
  "recomendacao": "",
  "checklist": []
}
```

- **Regras do prompt de sistema:**
  - usar somente números presentes no JSON de entrada; nunca criar número novo;
  - fator indisponível → dizer que o dado não está disponível para a região;
  - não garantir sucesso; falar em risco e probabilidade;
  - não citar autores nem copiar avaliações literalmente: resumir com palavras próprias;
  - português do Brasil, frases curtas, sem jargão;
  - checklist com 6 a 10 itens práticos de validação presencial (ex.: contar o movimento nos horários de pico do tipo, conferir carga e descarga, checar alvará e zoneamento na prefeitura, conversar com comerciantes vizinhos), combinados com `checklist_extra` do tipo.
- **Validação:** JSON Schema. Depois, checar se todo número no texto existe no JSON de entrada (tolerância de arredondamento); se não existir, regenerar 1 vez. Falhou de novo → relatório sai sem os textos de IA, com aviso, e as seções numéricas continuam.

---

## 9. Relatório em PDF

- Template único `templates/relatorio.html` (HTML + CSS de impressão, sem JavaScript), preenchido no n8n.
- A4 retrato. Capa com endereço, tipo, data e a nota em destaque.
- Gráficos (faixas etárias, série de tendência, barras dos fatores) em **SVG inline** gerado no n8n.
- Mapa: imagem do Static Maps embutida em base64.
- Rodapé em toda página: fontes, datas, atribuições (Google, © OpenStreetMap contributors, IBGE, Receita Federal) e o aviso de estimativa.
- Nome do arquivo: `analise-<tipo>-<bairro>-<aaaammdd>.pdf`.

---

## 10. Web app (`pc-web`)

- **Stack:** Next.js (App Router) + TypeScript + Tailwind; Drizzle ORM; Auth.js (credenciais + Google); zod para validação.
- **Páginas:** `/` (landing) · `/exemplo` · `/entrar` · `/cadastro` · `/analise/nova` · `/analise/[id]` · `/historico` · `/comparar?a=&b=` · `/planos` · `/conta` · `/admin/*` · `/termos` · `/privacidade`.
- **APIs públicas (autenticadas):**
  - `POST /api/analises` → valida entrada, debita crédito e cria a análise na mesma transação, chama o webhook do n8n. Se o webhook falhar, estorna na hora.
  - `GET /api/analises/[id]` → status, etapa e resultado (só o dono).
  - `GET /api/analises/[id]/pdf` → stream do PDF (só o dono).
  - `PATCH /api/analises/[id]` (apelido) · `DELETE /api/analises/[id]`.
  - `POST /api/pagamentos/checkout`.
- **APIs internas** (rede interna + `X-PC-Secret`, bloqueadas no Traefik): `POST /api/internal/relatorios/[id]` (recebe o PDF e grava em `/data/relatorios/<id>.pdf`).
- **Webhooks:** `POST /api/webhooks/pagamento` — idempotente por `provedor_ref`.
- **Rate limit:** 5 análises por minuto por usuário (Redis próprio do stack ou memória).
- **Design:** limpo, mobile-first, claro/escuro, mapa Google. Nenhum texto, imagem ou layout copiado do site de referência.
- **Relatório de exemplo** (`/exemplo`): análise real gerada pela plataforma num endereço público, servida estática.

---

## 11. Pagamentos

- Provedor: decisão D2. Requisitos: Pix e cartão (à vista e parcelado) para compra única; assinatura mensal no cartão; webhooks; sandbox.
- Interface `PaymentProvider` no código (checkout, consultar, cancelar assinatura, validar webhook), para trocar de provedor sem reescrever.
- **Compra única paga** → cria `creditos_lotes` (validade conforme `cfg.planos`) + `creditos_mov` (`compra`).
- **Assinatura:** cada cobrança mensal paga → lote de 30 com `assinatura_id`. Cancelamento: acesso até `periodo_fim`, depois zera os lotes da assinatura. Inadimplência: `status='inadimplente'`; após a carência (D3) zera.
- **Pro anual:** lote de 360 com `expira_em = compra + 12 meses`.
- Nota fiscal: fora do MVP (D9).

---

## 12. Segurança, LGPD e termos

- Webhook do n8n só na rede interna + segredo; endpoints internos do pc-web bloqueados externamente.
- Senhas com argon2id; cookies `httpOnly`, `secure`, `sameSite=lax`.
- Segredos em Docker secrets / `.env` fora do git: `ANTHROPIC_API_KEY`, `GOOGLE_MAPS_KEY_SERVER`, `GOOGLE_MAPS_KEY_BROWSER` (restrita por domínio), `PC_N8N_SECRET`, credenciais do `pc-db`, credenciais do provedor de pagamento, `PC_LLM_MODEL`.
- **LGPD:** política de privacidade; coletar só nome e e-mail; exclusão de conta apaga análises e PDFs; base CNPJ sem contatos e sem sócios; não exibir razão social de MEI.
- Atribuições visíveis na tela e no PDF: Google, © OpenStreetMap contributors, IBGE, Receita Federal.
- Termos de uso: o relatório é estimativa e não garante resultado.
- Não usar marca, textos, layout, depoimentos ou exemplos do site de referência.

---

## 13. Custos variáveis por análise

- Componentes: 1 sessão de autocomplete/geocode; 1 a 4 Nearby/Text Search (campos de nota e avaliações caem em SKU mais caro); até 11 Place Details com reviews; 1 Static Map; 1 a 2 chamadas ao Claude; Overpass (gratuito).
- Gravar em `app.analises.custo_apis` a contagem de chamadas por SKU e o custo estimado usando os preços unitários em `cfg.parametros` (atualizar conforme as tabelas oficiais de preço).
- O admin mostra custo médio e p95 por análise.
- Meta: custo variável < 10% da receita por crédito do plano mais barato por análise. Se passar, reduzir Place Details primeiro.

---

## 14. Fases e critérios de aceite

### 14.0 MVP só no n8n (decidido com o Murilo em 22/10/2026)

Antes das Fases 3 e 4, o produto roda inteiro no n8n, sem site: entrada pelo **Form Trigger** do n8n (endereço, tipo, raio) e saída do relatório em HTML no navegador (PDF opcional). Sem usuários, créditos nem pagamentos. Objetivo: validar o relatório e a nota com casos reais antes de construir o web app. Avança em degraus; cada um gera um relatório testável:

| Degrau | Entra | Seções habilitadas | Banco |
|---|---|---|---|
| v0.1 | Google (geocode, Nearby, reviews, Static Maps), Overpass, Claude | concorrência, fluxo e acesso, percepção da região, conclusão | nenhum |
| v0.2 | IBGE setores + agregados Censo 2022 (capital SP) + Gotenberg | público, demanda, renda | `pc-db` |
| v0.3 | CNPJ + geocodificação CNEFE, benchmarks | tendência, concorrência CNPJ, comparação com o município | `pc-db` |
| v0.4 | POF + aluguel de referência | potencial financeiro, viabilidade | `pc-db` |

Fatores sem dado saem como indisponíveis (seção 7.3); nas primeiras versões a confiança será baixa, e o relatório diz isso.

### Fase 0 — Infra
- Stack `pontocerto` no Swarm: `pc-db` (PostGIS), `pc-gotenberg`, rede compartilhada com o n8n, volume do `pc-db`.
- **Aceite:** `select postgis_full_version();` funciona; Gotenberg converte um HTML de teste; um workflow de teste no n8n consulta o `pc-db` e gera PDF pelo Gotenberg; nenhum serviço existente foi alterado.

### Fase 1 — Dados (estado de SP)
- ETL: setores + agregados, CNEFE, CNPJ (+ geocodificação), CNAEs, municípios, seeds POF e aluguel (capital), fator IPCA, benchmarks.
- **Aceite:**
  - população somada dos setores do município de São Paulo bate com o total oficial do Censo 2022 (± 0,5%);
  - percentual de CNPJs ativos de SP com `geo_precisao` `exata` ou `cep` reportado (meta ≥ 85%; se ficar abaixo, propor melhoria antes de seguir);
  - `ref.cargas` preenchida com referência e data de cada fonte;
  - tabela de tipos (seção 6.3) com CNAEs validados e parâmetros aprovados pelo Murilo.

### Fase 2 — Motor (sem front)
- Workflows `PC — Analise` e `PC — Erro`; funções SQL; `score.js` com testes; template do PDF.
- Disparo manual com `analise_id` criado via SQL.
- **Aceite:**
  - 5 endereços de teste em SP, de tipos diferentes, geram `resultado` válido + PDF em ≤ 60 s;
  - população do raio conferida manualmente contra o Panorama do Censo em 1 caso;
  - casos-limite: endereço sem concorrentes; endereço com fonte de renda ausente; Overpass fora do ar (análise sai com o fator indisponível);
  - erro forçado estorna o crédito e manda alerta.

### Fase 3 — Web app + créditos (sem pagamento)
- Créditos dados pelo admin.
- **Aceite:** criar conta, receber créditos, rodar análise, ver progresso, relatório na tela, PDF, histórico, comparação de 2 análises; usuário não acessa análise de outro.

### Fase 4 — Pagamentos
- **Aceite (sandbox):** Pix e cartão geram lote correto; assinatura renova, cancela e trata inadimplência; reenviar o mesmo webhook não duplica crédito.

### Fase 5 — Expansão
- Outras UFs (carga por UF), 40+ tipos, OSM local, páginas SEO programáticas, panorama por município, WhatsApp, zoneamento SP, comparação de 3 pontos.

---

## 15. Decisões em aberto (perguntar ao Murilo)

| # | Decisão | Padrão sugerido | Decidir até |
|---|---|---|---|
| D1 | Nome, domínio e preços | codinome PontoCerto; subdomínio de automaxhub.com.br; preços da tabela 2.6 como ponto de partida | Fase 3 |
| D2 | Provedor de pagamento (Pix + cartão + assinatura) | Mercado Pago ou Asaas | Fase 4 |
| D3 | Carência de inadimplência da assinatura | 5 dias | Fase 4 |
| D4 | Onde guardar os PDFs | volume do `pc-web`; alternativa S3 compatível externo | Fase 0 |
| D5 | Aluguel: só seeds/FipeZap ou também portais | só seeds/FipeZap | Fase 1 |
| D6 | Cobertura do MVP | dados do estado de SP; testes na capital | Fase 1 |
| D7 | Login com Google além de e-mail/senha | sim | Fase 3 |
| D8 | Pedido e entrega por WhatsApp (Evolution) | Fase 5 | Fase 5 |
| D9 | Nota fiscal na compra | fora do MVP | Fase 4 |

---

## 16. Estrutura do repositório

```
pontocerto/
├── CLAUDE.md
├── SPEC.md                      # este documento
├── infra/
│   ├── stack.yml                # Swarm: pc-db, pc-web, pc-gotenberg, pc-etl
│   └── README.md
├── db/
│   ├── migrations/              # 001_extensions.sql, 002_ref.sql, …
│   └── functions/               # pc_demografia.sql, …
├── etl/                         # Python
│   ├── ibge_setores.py
│   ├── ibge_agregados.py
│   ├── cnefe.py
│   ├── cnpj.py
│   ├── geocode_cnpj.py
│   ├── seeds.py                 # POF, aluguel, tipos, planos
│   ├── ipca.py
│   └── benchmarks.py
├── engine/
│   ├── score.js
│   └── score.test.js
├── n8n/
│   └── workflows/               # JSON exportado de cada workflow PC —
├── templates/
│   └── relatorio.html
├── scripts/
│   └── sync-n8n.*               # injeta engine/score.js no Code node via API do n8n
├── data/
│   └── seeds/                   # pof_despesa.csv, aluguel_referencia.csv, tipos_negocio.csv, planos.csv
└── web/                         # Next.js
```

---

## 17. Regras de trabalho para o Claude Code

- Trabalhar fase a fase; ao fim de cada fase entregar um resumo curto: o que foi feito, como testar, pendências e decisões tomadas.
- Commits pequenos e descritivos; README em cada pasta de serviço.
- Workflows do n8n: criar e editar pela conexão com o n8n, sempre com prefixo `PC — `, e exportar o JSON para `n8n/workflows/` após cada mudança. Não tocar em workflows existentes.
- Migrations SQL numeradas e idempotentes; nada de alteração manual no banco sem migration.
- Nenhum segredo no git.
- Ao encontrar divergência entre este documento e a fonte oficial (layout de arquivo, código, termo de uso), seguir a fonte oficial, registrar a divergência no resumo da fase e atualizar este documento.
- Na dúvida sobre regra de negócio, perguntar antes de implementar.
