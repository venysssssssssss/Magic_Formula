# Magic_Formula
## Documentação do Código do Projeto 1 - Backtest modelo de investimento Magic Formula

Este é um projeto de backtest que visa testar a eficácia da regra de investimento da fórmula mágica de Joel Greenblatt no mercado de ações brasileiro nos últimos anos. A fórmula mágica é uma estratégia de investimento que classifica ações com base em métricas de valuation (EV/EBIT) e retorno sobre o capital investido (ROIC).

### Passo 1 - Importar os módulos e bibliotecas
```python
import pandas as pd
import quantstats as qs
```
Neste passo, os módulos `pandas` e `quantstats` são importados. O `pandas` é uma biblioteca amplamente usada para manipulação de dados em Python, enquanto o `quantstats` é usado para análise quantitativa de séries temporais financeiras.

### Passo 2 - Baixar os dados disponibilizados
```python
dados_empresas = pd.read_csv("dados_empresas.csv")
```
Neste passo, o código lê os dados das empresas a partir de um arquivo CSV chamado "dados_empresas.csv" e armazena-os em um DataFrame do pandas chamado `dados_empresas`.

### Passo 3 - Filtrar a liquidez
```python
dados_empresas = dados_empresas[dados_empresas['volume_negociado'] > 1000000]
```
Neste passo, o código filtra as empresas com base no volume negociado diário, mantendo apenas aquelas com um volume superior a 1.000.000 de ações negociadas.

### Passo 4 - Calcula os retornos mensais das empresas
```python
dados_empresas['retorno'] = dados_empresas.groupby('ticker')['preco_fechamento_ajustado'].pct_change()
dados_empresas['retorno'] = dados_empresas.groupby('ticker')['retorno'].shift(-1)
```
Neste passo, o código calcula os retornos mensais das empresas, considerando a diferença percentual entre os preços de fechamento ajustados. Os retornos são calculados em relação ao mês anterior.

### Passo 5 - Cria o ranking dos indicadores
```python
dados_empresas['ranking_ev_ebit'] = dados_empresas.groupby('data')['ebit_ev'].rank(ascending=False)
dados_empresas['ranking_roic'] = dados_empresas.groupby('data')['roic'].rank(ascending=False)

dados_empresas['ranking_final'] = dados_empresas['ranking_ev_ebit'] + dados_empresas['ranking_roic']
dados_empresas['ranking_final'] = dados_empresas.groupby('data')['ranking_final'].rank()
```
Neste passo, o código calcula os rankings das empresas com base nas métricas EV/EBIT e ROIC. O ranking é calculado em relação aos valores dessas métricas em cada data. Um ranking final é obtido somando os rankings das duas métricas.

### Passo 6 - Cria as carteiras
```python
dados_empresas = dados_empresas[dados_empresas['ranking_final'] <= 10]
```
Neste passo, o código cria carteiras selecionando apenas as empresas que estão classificadas entre as 10 melhores com base no ranking final.

### Passo 7 - Calcula a rentabilidade por carteira
```python
rentabilidade_por_carteiras = dados_empresas.groupby('data')['retorno'].mean()
rentabilidade_por_carteiras = rentabilidade_por_carteiras.to_frame()
```
Neste passo, o código calcula a rentabilidade média das carteiras ao longo do tempo. A rentabilidade é calculada como a média dos retornos das empresas que compõem cada carteira.

### Passo 8 - Calcula a rentabilidade do modelo
```python
rentabilidade_por_carteiras['modelo'] = (rentabilidade_por_carteiras['retorno'] + 1).cumprod() - 1
rentabilidade_por_carteiras = rentabilidade_por_carteiras.shift(1)
rentabilidade_por_carteiras = rentabilidade_por_carteiras.dropna()
```
Neste passo, o código calcula a rentabilidade do modelo de investimento. A rentabilidade é calculada como o retorno acumulado das carteiras ao longo do tempo, considerando a reinvestimento dos retornos.

### Passo 9 - Calcula a rentabilidade do Ibovespa no mesmo período
```python
ibov = pd.read_csv('ibov.csv')

retornos_ibov = ibov['fechamento'].pct_change().dropna()
retorno_acum_ibov = (1 + retornos_ibov).cumprod() - 1
rentabilidade_por_carteiras['ibovespa'] = retorno_acum_ibov.values
rentabilidade_por_carteiras = rentabilidade_por_carteiras.drop('retorno', axis=1)
```
Neste passo, o código lê os dados do índice Ibovespa a partir de um arquivo CSV chamado 'ibov.csv' e calcula a rentabilidade acumulada do Ibovespa no mesmo período em que as carteiras foram avaliadas.

### Passo 10 - Analisa os resultados
```python
qs.extend_pandas()
rentabilidade_por_carteiras.index = pd.to_datetime(rentabilidade_por_carteiras.index)

rentabilidade_por_carteiras['modelo'].plot_monthly_heatmap()
rentabilidade_por_carteiras['ibovespa'].plot_monthly_heatmap()
```
Neste passo, o código utiliza a biblioteca `quantstats` para analisar e visualizar os resultados. Ele gera um heatmap mensal das rentabilidades do modelo e do Ibovespa, permitindo uma análise visual da performance ao longo do tempo.

Além disso, o código calcula a rentabilidade anualizada do modelo.

Isso conclui a documentação do código do projeto de backtest da fórmula mágica de investimento no mercado de ações brasileiro. O projeto visa testar a eficácia dessa estratégia nos últimos anos e analisar sua performance em relação ao índice Ibovespa.
