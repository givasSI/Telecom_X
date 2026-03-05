# Projeto TelecomX - Documentação

## Estrutura do Projeto
O projeto está organizado para execução em ambiente Google Colab, seguindo o layout abaixo:
- `/content/TelecomX_Data.json`: Dataset principal contendo os dados brutos de clientes em formato JSON.
- `/content/`: Diretório raiz para armazenamento de arquivos temporários e exports.
- `Notebook principal`: Localizado na raiz do ambiente de execução.

## Dependências
As seguintes bibliotecas Python são obrigatórias para a execução deste projeto:
- **pandas**: Manipulação e análise de dados.
- **numpy**: Processamento numérico e operações de matriz.
- **seaborn**: Visualização de dados estatísticos.
- **matplotlib**: Criação de gráficos e plots.
- **scikit-learn**: Pré-processamento de dados e algoritmos de Machine Learning.

## Pré-requisitos / Instalação
Caso as bibliotecas não estejam presentes no ambiente, elas podem ser instaladas utilizando o comando abaixo:

```bash
!pip install pandas numpy seaborn matplotlib scikit-learn
```

## Extração Inicial
O processo de análise inicia-se com o carregamento do arquivo `TelecomX_Data.json`. O arquivo é lido através da função `pd.read_json()` do pandas, convertendo a estrutura aninhada de dados de telecomunicações em um DataFrame para as etapas subsequentes de transformação e limpeza.

# Processamento e Limpeza de Dados

Este documento detalha as etapas técnicas realizadas para transformar os dados brutos do TelecomX em um formato pronto para análise.

### 1. Normalização de Estruturas JSON
Os dados originais continham colunas aninhadas em formato de dicionário (`customer`, `phone`, `internet`, `account`). Utilizamos a função `pd.json_normalize` do Pandas para expandir essas chaves em colunas individuais. 
- **Processo:** Cada dicionário foi extraído separadamente e concatenado de volta ao DataFrame principal utilizando `pd.concat`.
- **Resultado:** Um DataFrame 'flat' (plano) onde cada atributo de serviço tornou-se uma coluna distinta.

### 2. Tratamento da Coluna 'Charges.Total'
Identificamos uma inconsistência na coluna `account_Charges.Total`, que estava como tipo `object` devido à presença de espaços vazios (`" "`).
- **Causa:** Os espaços correspondiam a clientes com `tenure = 0` (recém-contratados), que ainda não haviam gerado faturamento acumulado.
- **Correção:** Substituímos os espaços por `"0.0"` e convertemos a coluna para o tipo `float64` para permitir cálculos matemáticos.

### 3. Limpeza e Codificação do 'Churn'
A variável alvo `Churn` passou por um processo de higienização e transformação:
- **Remoção de Nulos:** Linhas com valores em branco na coluna Churn foram descartadas, pois são essenciais para a modelagem.
- **Mapeamento Binário:** Convertemos os valores categóricos `Yes` e `No` para os inteiros `1` e `0`, respectivamente, facilitando a análise de correlação e o uso em algoritmos de Machine Learning.

### 4. Simplificação de Atributos (Renomeação)
Para melhorar a legibilidade e facilitar a manipulação do código, removemos os prefixos redundantes gerados na normalização:
- Prefixos como `customer_`, `phone_`, `internet_` e `account_` foram removidos.
- Exemplo: `customer_gender` tornou-se apenas `gender`.

# Relatório de Análise Exploratória (EDA)

### 1. Variáveis Categóricas e Impacto no Churn
A análise das variáveis categóricas revelou padrões claros de comportamento dos clientes:
- **Tipo de Contrato:** Clientes com contratos **'Month-to-month' (mês a mês)** apresentam uma taxa de evasão significativamente superior (~42.7%) em comparação com contratos de um ou dois anos. Isso sugere que a falta de fidelidade contratual facilita a saída.
- **Serviço de Internet:** Usuários de **'Fiber optic' (Fibra ótica)** têm uma propensão ao churn muito maior (~41.9%) do que usuários de DSL ou sem internet, o que pode indicar insatisfação com o preço ou instabilidade técnica nesse serviço específico.
- **Métodos de Pagamento:** O 'Electronic check' destaca-se com a maior taxa de cancelamento entre as formas de pagamento.

### 2. Variáveis Numéricas e Correlações
As métricas financeiras e de tempo de casa (tenure) são indicadores cruciais:
- **Tenure (Tempo de Contrato):** Existe uma relação inversa clara; clientes novos (baixo Tenure) saem com muito mais frequência. À medida que o cliente ultrapassa os primeiros meses, a densidade de churn diminui drasticamente.
- **Mensalidades (Monthly Charges):** Observou-se uma correlação positiva entre valores altos de fatura e o churn. O boxplot demonstrou que a mediana das mensalidades dos clientes que saíram é superior à dos que permaneceram.
- **Cobranças Totais:** Embora o valor mensal seja alto para quem sai, o valor total acumulado é baixo, reforçando que o churn ocorre precocemente no ciclo de vida do cliente.

### 3. Análise Visual e Metodologia
Para chegar a estas conclusões, foram utilizadas as seguintes ferramentas visuais:
- **Count Plots:** Utilizados para comparar a distribuição de Churn em variáveis como gênero, parceiros e dependentes.
- **KDE Plots (Densidade):** Essenciais para visualizar a concentração de churn nos primeiros meses de 'Tenure'.
- **Boxplots:** Aplicados para identificar a dispersão e outliers nas cobranças mensais e totais, permitindo comparar quartis entre os grupos de churn.
- **Matriz de Correlação (Spearman/Pearson):** Utilizada para quantificar a força da relação entre variáveis como 'Contas_Diarias', 'Qtd_Servicos' e o alvo 'Churn'.

### 4. Conclusão
As variáveis **Contract**, **InternetService**, **Tenure** e **MonthlyCharges** servem como os preditores mais fortes para o comportamento de evasão. Estratégias de retenção devem focar em clientes de fibra ótica com contratos mensais e incentivar a migração para planos de longo prazo nos primeiros meses de serviço.

## Extra: Engenharia de Variáveis

Nesta etapa, criamos novas métricas para enriquecer a análise do comportamento dos clientes e entender melhor os fatores que levam à evasão.

### 1. Contas Diárias (Daily Charges)
A variável `Contas_Diarias` foi calculada dividindo o faturamento mensal (`Charges.Monthly`) por 30. O objetivo é granularizar o custo do serviço para uma base diária, permitindo avaliar se a percepção de custo cotidiano influencia a decisão do cliente de permanecer na empresa.

### 2. Quantidade de Serviços (Qtd_Servicos)
A métrica `Qtd_Servicos` representa a soma total de serviços contratados por cada cliente. Foram consolidados os seguintes serviços binários (onde 1 indica presença e 0 ausência):
- PhoneService
- OnlineSecurity
- OnlineBackup
- DeviceProtection
- TechSupport
- StreamingTV
- StreamingMovies

Essa variável mede a 'aderência' (stickiness) do cliente: quanto mais serviços integrados, maior o custo de mudança e a utilidade percebida.

### 3. Análise de Correlação e Insights
Utilizamos o método de **Spearman** para analisar a relação entre essas novas variáveis e o `Churn`:
- **Qtd_Servicos vs Churn**: Observamos uma correlação negativa significativa. A análise visual via gráfico de barras demonstrou que clientes com 0 ou 1 serviço têm taxas de evasão drasticamente superiores àqueles que possuem 6 ou 7 serviços.
- **Insight Quantitativo**: Clientes com uma cesta de serviços mais completa tendem a ser mais fiéis. Isso sugere que estratégias de *cross-selling* (venda cruzada) são fundamentais para aumentar a retenção.
- **Contas Diárias**: Embora correlacionada ao gasto mensal, a visão diária auxilia na identificação de faixas de preço críticas onde a densidade de churn aumenta.
