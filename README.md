# Como o Senado vota? Análise das votações nominais de 2025

Extração, tratamento e análise das votações nominais do Plenário do Senado Federal em 2025, a partir da
[API de Dados Abertos do Senado](https://legis.senado.leg.br/dadosabertos/api-docs/swagger-ui/index.html).
Todo o processo está no notebook [`challenge.ipynb`](challenge.ipynb), que pode ser lido no GitHub sem
ser executado: as saídas e os gráficos ficam salvos nele.

## Principais descobertas

1. **A maior parte das votações nominais é secreta.** Das 125 votações de 2025, 72 (58%) foram secretas,
   todas de indicações de autoridades (embaixadores, ministros de tribunais, diretores de agências). Nelas
   só o placar é público, o que limita a análise do voto individual às 53 votações abertas.
2. **Nenhuma indicação de autoridade foi rejeitada.** As 72 foram aprovadas, com média de 94% de votos
   favoráveis. Agosto concentrou 40 delas em apenas três dias, num esforço concentrado após o recesso.
3. **O consenso predomina.** Em 34 das 53 votações abertas, ao menos 90% dos votos foram Sim, e 25 não
   tiveram nenhum voto contrário. As poucas derrotas foram em emendas destacadas e pedidos de urgência,
   não no texto principal das propostas.
4. **Propostas da Câmara encontram mais resistência que as do próprio Senado.** Os votos Não somam 20%
   nas propostas vindas da Câmara, 16% nas do Governo federal e 10% nas do Senado.
5. **Nas votações apertadas, a polarização segue o eixo governo × oposição.** Nas quatro votações abertas
   mais equilibradas, a Vanguarda (PL) e o Pelo Brasil (PT) ficaram sempre em lados opostos, ambos
   coesos, enquanto o bloco Democracia (MDB, União, PSDB, Podemos) se dividiu e decidiu o resultado.
6. **A Vanguarda (PL) é o bloco que mais registra presença sem votar**: 21% dos registros, contra 12% a
   16% nos demais. A diferença se concentra nas votações secretas de autoridades (29%).

Os gráficos e a discussão de cada ponto estão na seção 5 do notebook.

## Como rodar

Requer Python 3.11+ e acesso à internet.

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux/macOS
pip install -r requirements.txt
jupyter notebook challenge.ipynb
```

Execute todas as células em ordem (*Run All*). A extração leva cerca de 1 minuto, porque faz uma
chamada à API por proposta votada. Para analisar outro ano, altere `ANO` na seção 1.

## Estrutura

```
├── challenge.ipynb    # extração, tratamento, análise e gráficos
├── README.md          # esta documentação
└── requirements.txt   # dependências com versões fixadas
```

O notebook segue o fluxo dos dados:

| Seção | Conteúdo |
|---|---|
| 1. Configuração | Bibliotecas e parâmetros (ano, tempo limite, pausa entre requisições) |
| 2. Cliente da API | Sessão com novas tentativas automáticas, exceção única (`ErroAPI`) e testes de casos de borda |
| 3. Extração | Votações com o voto de cada senador, senadores em exercício e autoria das propostas |
| 4. Tratamento | Pipeline genérico de limpeza, regras do domínio e checagens de qualidade |
| 5. Visualizações | Cinco gráficos, cada um com a interpretação logo abaixo |

## Dados utilizados

| Conjunto | Endpoint | Volume |
|---|---|---|
| Votações nominais com o voto de cada senador | `/votacao?dataInicio&dataFim` | 125 votações, 10.125 votos |
| Senadores em exercício (partido e bloco) | `/senador/lista/atual` | 81 senadores |
| Autoria de cada proposta votada | `/processo/{id}` | 111 propostas |

Licenças, filiações partidárias, projetos de lei e discursos foram avaliados e descartados. As licenças e
as filiações são redundantes: cada voto já informa o motivo da ausência e o partido do senador naquela data.

## Decisões de análise

- **Recorte em 2025:** um ano completo, com volume suficiente para a análise e extração rápida.
- **Blocos parlamentares em vez de partidos:** são 13 partidos, vários com 1 a 3 senadores; os 5 blocos,
  definidos pelo próprio Senado, dão amostras maiores e evitam classificar partidos como "governo" ou
  "oposição" por critério próprio. O bloco de cada voto é atribuído pelo partido do senador na data do voto.
- **Grupos de autoria:** cada proposta é classificada como do Governo federal, da Câmara, do Senado ou de
  outros órgãos (STF, Procuradoria-Geral etc.), conforme o autor principal.
- **Partido do autor em 2025:** a API guarda o partido da época em que a proposta foi apresentada (às vezes
  um partido extinto, como o DEM). Para comparar com os votos de 2025, usamos o partido do autor no ano.
- **Votações procedimentais:** requerimentos e questões de ordem ficam fora da análise das votações mais
  apertadas, porque nelas o Sim e o Não se referem ao procedimento, não ao conteúdo da proposta.
- **Pipeline de limpeza genérico:** a função `limpar()` serve para qualquer `DataFrame` (padroniza nomes,
  remove colunas irrelevantes, converte tipos, remove duplicatas e gera um relatório). As regras
  específicas do Senado ficam separadas dela.

## Problemas encontrados e soluções

| Problema | Impacto | Solução |
|---|---|---|
| Sem o cabeçalho `Accept: application/json`, a API responde em XML | Falha ao ler a resposta | A sessão envia o cabeçalho em toda requisição |
| Código inexistente (ex.: `/senador/99999999`) retorna HTTP 200 só com metadados | Erro silencioso | `extrair()` devolve lista vazia quando o caminho não existe |
| Cada endpoint aninha os dados de um jeito, e um item único pode vir como objeto em vez de lista | Código frágil | `extrair()` percorre o caminho informado e sempre devolve uma lista |
| Endpoint documentado responde 404 (`/plenario/votacao/orientacaoBancada`) | Sem orientação de bancada | Erro tratado como `ErroAPI`; a análise usa os blocos no lugar |
| `/composicao/lista/blocos` responde com corpo vazio | Sem composição dos blocos | Blocos derivados da lista de senadores em exercício |
| **Votações duplicadas:** 3 requerimentos aparecem duas vezes, sob o próprio requerimento e sob o projeto a que se referem, com votos idênticos | Infla as contagens e atribui um pedido de urgência ao projeto (o PL 2234/2022 parecia ter sido rejeitado, mas foi a urgência) | As cópias são as únicas sem `sequencialVotacao` e são removidas no tratamento |
| Os totais de votos só vêm preenchidos nas votações secretas | Placar ausente nas abertas | Totais calculados a partir dos votos individuais |
| O código de voto `NA` (*dispositivo não citado*) coincide com um marcador comum de vazio | 15 votos virariam nulos | Parâmetro `manter_texto` protege colunas de código |
| A autoria guarda o partido da época da apresentação | Partidos extintos na análise | Partido do autor substituído pelo de 2025 |
| `/senador/{codigo}/profissao` devolve os dados sob a chave `HistoricoAcademicoParlamentar` | Documentação inconsistente | Endpoint não utilizado |
| Datas invertidas (início depois do fim) retornam lista vazia, sem erro | Erro silencioso | Datas montadas a partir do ano, sempre na ordem certa |

## Limitações e sugestões

- **Votações simbólicas não aparecem.** Boa parte das matérias é aprovada sem registro individual de voto, e
  a API de votações nominais não as inclui. *Sugestão:* complementar com os resultados de sessão
  (`/plenario/resultado/{data}`) para medir quanto do que é aprovado passa por votação nominal.
- **Votos secretos não são individualizáveis.** Para as 72 indicações de autoridades, só o placar é
  conhecido; não há solução nos dados públicos.
- **Composição dos blocos:** o mapa partido → bloco é o atual e supõe que ele não mudou ao longo de 2025.
- **Amostras pequenas em alguns cortes:** a coluna do Governo federal no mapa de calor reúne apenas 6
  votações abertas, e a maioria das propostas do Senado tem autores de vários blocos, o que impede
  comparar os blocos como autores.
- **Interpretações políticas:** associar o Pelo Brasil ao governo e a Vanguarda à oposição usa contexto
  externo aos dados; os textos do notebook sinalizam esses trechos.

## Processo de desenvolvimento

O projeto foi desenvolvido com o **Claude Code** como assistente, em etapas curtas, cada uma revisada antes
de seguir para a próxima:

1. **Exploração da API:** teste dos endpoints para entender o formato, o volume e os problemas dos dados
   antes de escrever o código.
2. **Planejamento:** escolha da pergunta ("como o Senado vota?"), do recorte (2025) e dos dados; a análise
   por grupo de autoria nasceu da pergunta "como votam os senadores nas propostas de outros grupos?".
3. **Cliente da API e extração**, com tratamento de erros e testes dos casos de borda.
4. **Tratamento:** o pipeline genérico de limpeza foi adaptado de um código de limpeza próprio, generalizado
   para funcionar em qualquer tabela.
5. **Simplificação:** revisão do código para eliminar repetições, com comparação das tabelas antes e depois
   para garantir o mesmo resultado.
6. **Visualizações:** paleta validada para daltonismo e conferência visual de cada gráfico.
7. **Interpretação:** os textos de cada gráfico foram escritos a partir de números conferidos nos dados. Foi
   nessa conferência que surgiram as votações duplicadas e os casos em que o rejeitado era uma emenda ou
   um pedido de urgência, e não a proposta.

O assistente AI foi usado para explorar a API,
escrever código (antes da revisão) e conferir números.

## Tecnologias

Python 3.13, requests, pandas, NumPy, Matplotlib e Jupyter (versões em [`requirements.txt`](requirements.txt)).
