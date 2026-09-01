## Lucas Santos

Engenheiro de dados. Passo a maior parte do tempo ligando dois mundos: os
sistemas que a empresa já roda no dia a dia e a plataforma analítica que ela
ainda quer ter. Na prática, isso é ingestão, orquestração, modelagem em camadas
e governança — do banco de origem até o mart que alguém abre numa ferramenta
de BI.

O que eu levo a sério é o porquê de cada escolha. Uma decisão de arquitetura só
vale se aguenta a segunda pergunta, e saber quando *não* usar uma ferramenta
costuma pesar mais do que conhecer todas.

<img src="stack.svg" width="490"
  alt="Orquestração: Apache Airflow, Databricks Lakeflow · Transformação: dbt, SQL, PySpark, pandas · Plataforma e cloud: Databricks, AWS S3 · Armazenamento: Delta Lake, Parquet, DuckDB · Bancos: SQL Server, MySQL · Qualidade e governança: Great Expectations, Dynamic Data Masking · Entrega e infra: Power BI, Docker Compose, GitHub Actions">

---

## Em destaque

A mesma disciplina de arquitetura medallion em dois ambientes bem diferentes —
um provando que dá pra fazer engenharia de dados séria sem gastar nada, o outro
num ambiente gerenciado de nuvem.

### [legacy-to-lakehouse](https://github.com/lsantos-data/legacy-to-lakehouse) — migração local, custo zero

Um backoffice em SQL Server (AdventureWorks) modernizado e integrado com dados
de e-commerce (Olist) e câmbio (Banco Central). Roda inteiro na máquina —
`docker compose up` e o pipeline todo sobe.

```text
fontes             ingestão           Bronze             Silver        Gold
───────────────    ───────────────    ───────────────    ──────────    ────
SQL Server         Airflow            MinIO / Parquet    dbt-duckdb    marts por
MySQL · API BCB    watermark, 4 DAGs  particionado       18 staging    domínio
```

- **É de ponta a ponta, não um trecho** — da extração incremental no Airflow até
  os marts, com o dbt lendo o Parquet direto do MinIO via `httpfs`: não existe
  etapa de carga no meio, e ainda assim o comportamento é o de um warehouse
  colunar.
- **O SQL legado não é enfeite** — uma view que dependia de XQuery e XML nativos
  do SQL Server foi portada para `regexp_extract` no dbt e bate 1:1 com a
  original, nas 19.972 linhas. Um `PIVOT` antigo virou formato long/tidy em vez
  de reproduzir o anti-padrão.
- **A governança foi testada no motor** — Dynamic Data Masking de verdade no SQL
  Server: um login sem permissão vê `K*** S***` onde o `sa` vê `Ken Sánchez`. E
  a máscara é reaplicada na Gold, porque ela não viaja junto com a extração.
- **O benchmark é honesto** — Pandas contra PySpark no mesmo dado. O Pandas
  ganhou por ~3,3× em ~6 milhões de linhas, e o repositório explica por quê.
- **O tuning tem número de antes e depois** — reescrever um predicado
  não-sargável para range com índice de apoio derrubou as leituras lógicas de
  686 para 49.

### [data-engineering-credit-risk](https://github.com/lsantos-data/data-engineering-credit-risk) — o mesmo rigor, na nuvem

Pipeline de risco de crédito sobre o dataset do Lending Club (500 mil
empréstimos, 151 colunas), em **Databricks + AWS S3 + Delta Lake**, orquestrado
por **Lakeflow Jobs**.

- **Bronze / Silver / Gold terminando num star schema** — fato de empréstimo
  mais três dimensões (tomador, produto, data), com integridade referencial
  validada por teste automatizado.
- **Qualidade que bloqueia o pipeline** — 14 testes ao final da run (range,
  domínio, not-null, integridade). Qualquer falha para tudo e dispara e-mail;
  zero retry em teste de qualidade, porque retry mascara problema real.
- **Retry calibrado por tipo de task** — ingestão tenta de novo (falha de rede
  é transitória), transformação não (é determinística — retry ajuda pouco).
- **Governança de credencial via Databricks Secrets**, dashboard executivo em
  Power BI com as medidas DAX versionadas no repo.

<sub>Também no GitHub: <a href="https://github.com/lsantos-data/medallion-commerce-pipeline">medallion-commerce-pipeline</a> — contratos de qualidade e zona de quarentena em Python puro.</sub>

---

## Como eu penso sobre engenharia de dados

Toda decisão de arquitetura tem um porquê que foi testado contra o ambiente real
e escrito em algum lugar — inclusive as que falharam antes de dar certo.

A camada Bronze fica crua. Transformação e mascaramento são responsabilidade de
cada ponto de acesso ao dado, não do dado que está guardado.

E o pipeline não está pronto até rodar limpo do zero — um clone novo no projeto
local, a run inteira do início ao fim no da nuvem. Nos dois, um teste de
qualidade que falha interrompe a esteira, em vez de só registrar um aviso e
seguir em frente.

---

## Contato

Aberto a conversas sobre posições de Engenheiro de Dados (Pleno/Sênior).

- LinkedIn — [linkedin.com/in/lucas-santos-696061186](https://www.linkedin.com/in/lucas-santos-696061186/)
- Email — [lucasss.sillva@hotmail.com](mailto:lucasss.sillva@hotmail.com)
