## Lucas Santos

**Engenheiro de Dados** — pipelines que levam sistemas transacionais legados
para uma plataforma analítica moderna: ingestão, orquestração, modelagem em
camadas e governança.

Trabalho com trade-off explícito. Saber *quando não* usar uma ferramenta conta
tanto quanto saber usá-la — e toda decisão de arquitetura merece um porquê que
sobrevive a uma pergunta de acompanhamento.

---

### Trabalho em destaque

#### [legacy-to-lakehouse](https://github.com/lsantos-data/legacy-to-lakehouse)

Migração de ponta a ponta de um backoffice em SQL Server (AdventureWorks) para
um lakehouse local — integrado com e-commerce (Olist) e câmbio (Banco Central).
100% local, custo zero, sobe com `docker compose up`.

- **Arquitetura medallion completa** — Airflow com extração incremental por
  watermark → MinIO/Parquet → dbt-duckdb (18 modelos de staging + marts por
  domínio) → Great Expectations sobre a Bronze crua
- **SQL legado de verdade** — porta uma view com XQuery/XML nativo do SQL Server
  para `regexp_extract` no dbt, validada 1:1 contra a original (19.972 linhas);
  reescreve um `PIVOT` para formato long/tidy em vez de reproduzir o anti-padrão
- **Governança no motor** — Dynamic Data Masking real no SQL Server, reaplicado
  na camada Gold porque a máscara não viaja junto com a extração
- **Benchmark honesto** — Pandas × PySpark no mesmo dado; Pandas ganhou ~3,3× em
  ~6M linhas, e o README explica por que isso era o resultado esperado
- **Tuning com antes/depois medido** — predicado não-sargável reescrito para
  range + índice de apoio: 686 → 49 leituras lógicas

---

### Stack

| Área | Ferramentas |
|---|---|
| Orquestração | Apache Airflow |
| Transformação | dbt (core) · SQL · Python |
| Processamento | pandas · PySpark |
| Armazenamento | S3 / Parquet · DuckDB · SQL Server · MySQL |
| Qualidade e governança | dbt tests · Great Expectations · Dynamic Data Masking |
| Infraestrutura | Docker Compose · GitHub Actions |

---

### Como eu penso sobre engenharia de dados

- **Trade-off explícito supera "boa prática" genérica.** Cada decisão de
  arquitetura tem um porquê testado contra o ambiente real e documentado —
  inclusive as que não deram certo de primeira.
- **Bronze fica crua, sempre.** Transformação e mascaramento são propriedade de
  cada ponto de acesso ao dado, não do dado armazenado.
- **O pipeline não está pronto até um clone limpo rodar.** CI a cada push:
  `dbt parse`, lint de SQL e Python, scan de segredo, audit de dependência.

---

### Contato

- LinkedIn: <adicione o link do seu perfil>
- Email: <adicione um email de contato>
