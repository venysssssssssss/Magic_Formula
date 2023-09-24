## Documentação do Projeto de Backtest da Fórmula Mágica de Investimento

Este projeto tem como objetivo testar a eficácia da regra de investimento da Fórmula Mágica de Joel Greenblatt no mercado de ações brasileiro nos últimos anos. A seguir, detalhamos cada passo do código.

### Passo 1: Importando os Módulos Necessários
```python
import pandas as pd
import quantstats as qs
```
Neste passo, importamos as bibliotecas `pandas` e `quantstats`, que serão usadas para manipulação de dados e análise quantitativa, respectivamente.

### Passo 2: Baixando os Dados Disponibilizados
```python
dados_empresas = pd.read_csv("dados_empresas.csv")
```
Neste passo, importamos os dados das empresas brasileiras de um arquivo CSV chamado "dados_empresas.csv" e armazenamos esses dados em um DataFrame chamado `dados_empresas`.

### Passo 3: Filtrando a Liquidez
```python
dados_empresas = dados_empresas[dados_empresas['volume_negociado'] > 1000000]
```
Aqui, filtramos as empresas com base no volume de negociação diário, mantendo apenas aquelas com um volume superior a 1.000.000 de ações negociadas.

### Passo 4: Calculando os Retornos Mensais das Empresas
```python
dados_empresas['retorno'] = dados_empresas.groupby('ticker')['preco_fechamento_ajustado'].pct_change()
dados_empresas['retorno'] = dados_empresas.groupby('ticker')['retorno'].shift(-1)
```
Neste passo, calculamos os retornos mensais das empresas, considerando a variação percentual dos preços de fechamento ajustados. Os retornos são calculados em relação ao mês anterior.

### Passo 5: Criando o Ranking dos Indicadores
```python
dados_empresas['ranking_ev_ebit'] = dados_empresas.groupby('data')['ebit_ev'].rank(ascending=False)
dados_empresas['ranking_roic'] = dados_empresas.groupby('data')['roic'].rank(ascending=False)

dados_empresas['ranking_final'] = dados_empresas['ranking_ev_ebit'] + dados_empresas['ranking_roic']
dados_empresas['ranking_final'] = dados_empresas.groupby('data')['ranking_final'].rank()
```
Neste passo, criamos rankings para as empresas com base nas métricas EV/EBIT e ROIC. Um ranking final é obtido somando os rankings das duas métricas. 

### Passo 6: Criando as Carteiras
```python
dados_empresas = dados_empresas[dados_empresas['ranking_final'] <= 10]
```
Aqui, formamos carteiras selecionando apenas as empresas que estão classificadas entre as 10 melhores com base no ranking final.

### Passo 7: Calculando a Rentabilidade por Carteira
```python
rentabilidade_por_carteiras = dados_empresas.groupby('data')['retorno'].mean()
rentabilidade_por_carteiras = rentabilidade_por_carteiras.to_frame()
```
Neste passo, calculamos a rentabilidade média das carteiras ao longo do tempo, considerando a média dos retornos das empresas que compõem cada carteira.

### Passo 8: Calculando a Rentabilidade do Modelo
```python
rentabilidade_por_carteiras['modelo'] = (rentabilidade_por_carteiras['retorno'] + 1).cumprod() - 1
rentabilidade_por_carteiras = rentabilidade_por_carteiras.shift(1)
rentabilidade_por_carteiras = rentabilidade_por_carteiras.dropna()
```
Aqui, calculamos a rentabilidade do modelo de investimento como o retorno acumulado das carteiras ao longo do tempo, considerando o reinvestimento dos retornos.

### Passo 9: Calculando a Rentabilidade do Ibovespa no Mesmo Período
```python
ibov = pd.read_csv('ibov.csv')

retornos_ibov = ibov['fechamento'].pct_change().dropna()
retorno_acum_ibov = (1 + retornos_ibov).cumprod() - 1
rentabilidade_por_carteiras['ibovespa'] = retorno_acum_ibov.values
rentabilidade_por_carteiras = rentabilidade_por_carteiras.drop('retorno', axis=1)
```
Aqui, lemos os dados do índice Ibovespa a partir de um arquivo CSV e calculamos sua rentabilidade acumulada no mesmo período em que as carteiras foram avaliadas.

### Passo 10: Analisando os Resultados
```python
qs.extend_pandas()
rentabilidade_por_carteiras.index = pd.to_datetime(rentabilidade_por_carteiras.index)

rentabilidade_por_carteiras['modelo'].plot_monthly_heatmap()
rentabilidade_por_carteiras['ibovespa'].plot_monthly_heatmap()
rentabilidade_por_carteiras.plot()
```
Neste passo, usamos a biblioteca `quantstats` para realizar análises e visualizações dos resultados. Foram gerados gráficos de heatmap mensal das rentabilidades do modelo e do Ibovespa, bem como um gráfico geral de rentabilidades.

Isso conclui a documentação deste projeto de backtest da Fórmula Mágica de Investimento no mercado de ações brasileiro. O objetivo é testar a estratégia e comparar seu desempenho com o índice Ibovespa.
