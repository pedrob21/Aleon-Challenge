# Como o Senado vota? Análise das votações nominais de 2025

Desafio técnico Aleon: extração, tratamento e análise de dados da
[API de Dados Abertos do Senado Federal](https://legis.senado.leg.br/dadosabertos/api-docs/swagger-ui/index.html).

> Em construção. A documentação completa será escrita ao final do projeto.

## Como rodar

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux/macOS
pip install -r requirements.txt
jupyter notebook challenge.ipynb
```

## Estrutura

```
├── challenge.ipynb   # notebook com a análise completa
├── src/              # código de extração e tratamento
├── dados/
│   ├── brutos/       # respostas da API (não versionado)
│   └── tratados/     # CSVs limpos usados na análise
├── graficos/         # visualizações exportadas
└── docs/             # documentação do processo
```
