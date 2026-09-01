## Lucas Santos

**Engenheiro de Dados** — levo sistemas transacionais legados para uma
plataforma analítica moderna: ingestão, orquestração, modelagem em camadas
e governança.

Trabalho com trade-off explícito. Saber *quando não* usar uma ferramenta conta
tanto quanto saber usá-la — e toda decisão de arquitetura merece um porquê que
sobrevive a uma pergunta de acompanhamento.

```text
fontes             ingestão           Bronze            Silver        Gold
───────────────    ───────────────    ──────────────    ──────────    ────
SQL Server         Airflow            Parquet / MinIO   dbt-duckdb    marts por
MySQL · API BCB    watermark, 4 DAGs  particionado      18 staging    domínio
```

---

### Em destaque · [legacy-to-lakehouse](https://github.com/lsantos-data/legacy-to-lakehouse)

Migração de ponta a ponta de um backoffice em SQL Server (AdventureWorks) para
um lakehouse local — integrado com e-commerce (Olist) e câmbio (Banco Central).
100% local, custo zero, sobe com `docker compose up`.

|  |  |
|---|---|
| **Arquitetura medallion completa** | Airflow com extração incremental por watermark → MinIO/Parquet → dbt-duckdb (18 staging + marts por domínio) → Great Expectations sobre a Bronze crua |
| **SQL legado de verdade** | view com XQuery/XML nativo do SQL Server portada para `regexp_extract` no dbt, validada 1:1 (19.972 linhas); `PIVOT` reescrito para long/tidy em vez de reproduzir o anti-padrão |
| **Governança no motor** | Dynamic Data Masking real no SQL Server, reaplicado na Gold porque a máscara não viaja com a extração |
| **Benchmark honesto** | Pandas × PySpark no mesmo dado — Pandas ~3,3× mais rápido em ~6M linhas, com a explicação de por que era o esperado |
| **Tuning medido** | predicado não-sargável → range + índice de apoio: 686 → 49 leituras lógicas |

---

### Stack

**Orquestração** Apache Airflow &nbsp;·&nbsp; **Transformação** dbt · SQL · Python
&nbsp;·&nbsp; **Processamento** pandas · PySpark &nbsp;·&nbsp; **Armazenamento**
S3/Parquet · DuckDB · SQL Server · MySQL &nbsp;·&nbsp; **Qualidade e governança**
dbt tests · Great Expectations · Dynamic Data Masking &nbsp;·&nbsp; **Infra**
Docker Compose · GitHub Actions

---

### Como eu penso sobre engenharia de dados

- **Trade-off explícito supera "boa prática" genérica** — cada decisão de
  arquitetura tem um porquê testado contra o ambiente real e documentado,
  inclusive as que não deram certo de primeira.
- **Bronze fica crua, sempre** — transformação e mascaramento são propriedade
  de cada ponto de acesso ao dado, não do dado armazenado.
- **O pipeline não está pronto até um clone limpo rodar** — CI a cada push:
  `dbt parse`, lint de SQL e Python, scan de segredo, audit de dependência.

---

### Contato

- LinkedIn: [linkedin.com/in/lucas-santos-696061186](https://www.linkedin.com/in/lucas-santos-696061186/)
- Email: lucasss.sillva@hotmail.com
