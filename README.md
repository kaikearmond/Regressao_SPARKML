# Regressão em Educação com Spark ML — OULAD (200 mil linhas)

Projeto da disciplina **Processamento de Dados Massivos**, professor **Alexandre Roriz**.

## Objetivo
Predizer a quantidade de cliques por observação dos estudantes em recursos de um
ambiente virtual de aprendizagem com **regressão linear do Spark ML**.

## Dataset público
- Kaggle: https://www.kaggle.com/datasets/rocki37/open-university-learning-analytics-dataset
- Fonte primária: https://research.stem.open.ac.uk/ouanalyse/dataset/
- Artigo: https://doi.org/10.1038/sdata.2017.171
- Licença da fonte primária: CC BY 4.0.
- `studentVle.csv`: **10.655.280 linhas** na versão publicada (6 colunas).
- `vle.csv`: 6.364 linhas de metadados na versão publicada.

## Amostragem de 200 mil registros

O notebook baixa o CSV original, conta as linhas por blocos de bytes e sorteia
**exatamente 200.000 índices distintos**, uniformemente e sem reposição, com
semente **42**. Os registros selecionados são copiados para
`dados_locais/studentVle_amostra_200000.csv`. Depois disso, **o Spark só lê a
amostra de 200 mil linhas**, além do arquivo pequeno de metadados.

O processo de seleção **não carrega os 10,6 milhões de registros no Spark** e
não utiliza Pandas para ler o arquivo grande. O CSV amostrado é reutilizado
em novas execuções na mesma sessão graças ao manifesto de cache.

**Nota:** como a amostragem é por registro, um aluno pode ter parte do histórico
fora da amostra. A modelagem não usa histórico individual. O split treino/teste
é por estudante, prevenindo presença do mesmo aluno nos dois conjuntos.
A limpeza/consolidação pode reduzir ligeiramente o número de linhas modeladas.

## Como executar
1. Abra `Regressao_Educacao_OULAD_200mil_SparkML.ipynb` no Google Colab ou Jupyter.
2. Execute as células na ordem (internet necessária para o download inicial).
3. Consulte **RMSE, MAE e R²** na seção 7 e compare o baseline (média do treino).
4. O modelo treinado é salvo em `modelos/regressao_oulad_200mil_sparkml/` e as
   métricas em `resultados/metricas.json`.
5. Publique somente arquivos de código e documentação no GitHub. O dataset é
   baixado durante a execução; **não envie CSV para o GitHub**.

## Otimizações de tempo
- A ingestão no Spark já é limitada a 200.000 linhas.
- `spark.sql.shuffle.partitions` foi reduzido para **8**.
- O ajuste compara **dois** valores de `regParam` (0.01, 0.1), sem grade extensa.
- Regressão L2 com `solver='normal'` para poucos atributos.
- Apenas uma etapa final de ajuste do Pipeline nos dados de treino completos.
- Gráficos com agregações em Spark ou pequeno subconjunto de predições.

## Pipeline
`Kaggle → amostragem AAS exata de 200 mil linhas → ingestão Spark (StructType) →
EDA e limpeza → consolidação de duplicatas → join de metadados → engenharia de
atributos → split por estudante → StringIndexer → OneHotEncoder → VectorAssembler
→ StandardScaler → LinearRegression (Ridge) → RMSE, MAE, R² e baseline → save()`.

## Observações
- O dataset original sustenta a **justificativa de Big Data**; a amostra é um
  recorte operacional para viabilizar treinamento em equipamentos modestos.
- `sum_click` mede interações digitais; não é nota nem desempenho acadêmico.
- Métricas reais dependem da execução do notebook, não foram antecipadas.

## Entrega
Enviar o **link do repositório GitHub**, conforme o enunciado. Os arquivos de
dados e os modelos treinados permanecem locais e são ignorados pelo Git.
