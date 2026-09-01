## Lucas Santos

Engenheiro de dados. Passo a maior parte do tempo ligando dois mundos: os
sistemas que a empresa já roda no dia a dia e a plataforma analítica que ela
ainda quer ter. Na prática, isso é ingestão, orquestração, modelagem em camadas
e governança — do banco de origem até o mart que alguém abre no BI.

O que eu levo a sério é o porquê de cada escolha. Uma decisão de arquitetura só
vale se aguenta a segunda pergunta, e saber quando *não* usar uma ferramenta
costuma pesar mais do que conhecer todas.

<img src="stack.png" width="520"
  alt="Orquestração: Airflow · Transformação: dbt, SQL, Python · Processamento: pandas, PySpark · Dados: SQL Server, MySQL, Parquet, DuckDB · Qualidade e governança: dbt tests, Great Expectations, Data Masking · Infra: Docker Compose, GitHub Actions">

---

### Em destaque — [legacy-to-lakehouse](https://github.com/lsantos-data/legacy-to-lakehouse)

Uma migração de ponta a ponta: o AdventureWorks, um backoffice em SQL Server,
sendo modernizado e integrado com dados de e-commerce (Olist) e câmbio (Banco
Central). Roda inteiro na máquina local — sem nuvem, sem cartão, sem serviço
pago. Um `docker compose up` e o pipeline todo sobe.

```text
fontes             ingestão           Bronze            Silver        Gold
───────────────    ───────────────    ──────────────    ──────────    ────
SQL Server         Airflow            Parquet / MinIO   dbt-duckdb    marts por
MySQL · API BCB    watermark, 4 DAGs  particionado      18 staging    domínio
```

O que costuma render conversa numa entrevista:

- **A arquitetura medallion está montada das duas pontas** — Airflow puxando as
  fontes de forma incremental por watermark, Parquet particionado no MinIO como
  Bronze, dbt sobre DuckDB montando os 18 modelos de staging e os marts por
  domínio, e Great Expectations checando a Bronze ainda crua.
- **O SQL legado não é enfeite** — uma view que dependia de XQuery e XML nativos
  do SQL Server foi portada para `regexp_extract` no dbt e bate 1:1 com a
  original, nas 19.972 linhas. Um `PIVOT` antigo virou formato long/tidy em vez
  de reproduzir o anti-padrão.
- **A governança foi testada no motor** — Dynamic Data Masking de verdade no SQL
  Server: um login sem permissão vê `K*** S***` onde o `sa` vê `Ken Sánchez`. E
  a máscara é reaplicada na camada Gold, porque ela não viaja junto com a
  extração.
- **O benchmark é honesto** — Pandas contra PySpark no mesmo dado. O Pandas
  ganhou por ~3,3× em ~6 milhões de linhas, e o repositório explica por que esse
  era o resultado esperado nesse volume.
- **O tuning tem número dos dois lados** — reescrever um predicado não-sargável
  para range com índice de apoio derrubou as leituras lógicas de 686 para 49.

---

### Como eu penso sobre engenharia de dados

Toda decisão de arquitetura tem um porquê que foi testado contra o ambiente real
e escrito em algum lugar — inclusive as que falharam antes de dar certo.

A camada Bronze fica crua. Transformação e mascaramento são responsabilidade de
cada ponto de acesso ao dado, não do dado que está guardado.

E o pipeline não está pronto até um clone limpo rodar. É por isso que o CI roda
`dbt parse`, lint de SQL e Python, varredura de segredo e auditoria de
dependência a cada push.

---

### Contato

- LinkedIn — [linkedin.com/in/lucas-santos-696061186](https://www.linkedin.com/in/lucas-santos-696061186/)
- Email — lucasss.sillva@hotmail.com
