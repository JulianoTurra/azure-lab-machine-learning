# 🚲 Aprendizado de Máquina Automatizado no Azure ML

Este guia prático demonstra como utilizar o recurso de **Automated ML (AutoML)** no Azure Machine Learning para treinar, avaliar, implantar e testar um modelo de regressão preditiva (previsão de aluguel de bicicletas).

---

## 1. Configurando o Workspace do Azure ML

1. Acesse o [Portal do Azure](https://portal.azure.com).
2. Clique em **+ Criar um recurso**, pesquise por *Machine Learning* e configure com os seguintes dados:
   * **Assinatura / Grupo de Recursos:** *Sua escolha*.
   * **Nome:** *Insira um nome exclusivo*.
   * **Região:** *Mais próxima de você*.
   * **Demais campos:** Mantenha os valores padrões automáticos.
3. Selecione **Examinar + criar** e clique em **Criar**. 
4. Após a implantação, acesse o recurso e clique em **Iniciar o estúdio** (ou acesse diretamente [ml.azure.com](https://ml.azure.com)).

---

## 2. Treinando o Modelo com AutoML

O AutoML testa múltiplos algoritmos e hiperparâmetros para encontrar o melhor modelo de regressão para prever a quantidade diária de aluguéis de bicicletas.

### Configuração do Trabalho (Job):
1. No menu esquerdo do Estúdio, acesse **ML Automatizado** e clique em **Novo trabalho de ML automatizado**.
2. Configure as etapas seguindo os parâmetros abaixo:

| Seção | Campo | Valor / Configuração |
| :--- | :--- | :--- |
| **Configurações Básicas** | Nome do trabalho / Experimento | `mslearn-bike-automl` / `mslearn-bike-rental` |
| **Tipo de Tarefa** | Tipo de tarefa | Regressão |
| **Dados** | Criar Novo Ativo de Dados | **Nome:** `bike-rentals`<br>**Tipo:** Arquivo (`uri_file`) ou Tabular<br>**Fonte:** De arquivos locais *(Baixe o arquivo de `https://aka.ms/bike-rentals` antes)*<br>**Destino:** `workspaceblobstore` |
| **Configurações da Tarefa** | Coluna de destino | `rentals` (Inteiro) |
| | Métrica primária | `NormalizedRootMeanSquaredError` |
| | Modelos permitidos | Selecione apenas **RandomForest** e **LightGBM** |
| **Limites** | Avaliações / Nós Máximos | `3` avaliações / `3` nós / `15` minutos de tempo limite |
| **Validação** | Tipo de validação | Divisão de validação de treinamento (10% para validação) |
| **Computação** | Tipo de cálculo | Sem servidor (Serverless) / CPU Dedicada / `Standard_DS3_V2` |

3. Envie o trabalho de treinamento e aguarde a conclusão (Status: **Completed**).

---

## 3. Avaliação do Melhor Modelo

1. Na aba **Visão geral (Overview)** do trabalho concluído, identifique o algoritmo vencedor na seção **Resumo do melhor modelo** (geralmente *VotingEnsemble*).
2. Clique no nome do algoritmo para abrir os detalhes do modelo.
3. Explore a aba **Métricas** e analise os gráficos:
   * **Resíduos (Residuals):** Mostra a diferença entre os valores previstos e reais.
   * **Previsto vs. Verdadeiro (Predicted_true):** Avalia a precisão linear do modelo.

---

## 4. Implantação e Teste (Deployment)

### Implantando o Endpoint:
1. Na página do melhor modelo treinado, clique no botão **Implantar (Deploy)** no menu superior.
2. Selecione **Ponto de extremidade em tempo real (Real-time endpoint)** ou *Serviço Web*:
   * **Nome do ponto de extremidade:** `predict-rentals`
   * **Tipo de computação:** Instância de Contêiner do Azure (ACI) ou gerenciado.
   * **Ativar autenticação:** Selecionado (True).
3. Aguarde o status do endpoint mudar para **Saudável (Healthy)** na aba *Pontos de Extremidade* (Leva de 5 a 10 minutos).

### Testando o Serviço:
1. Acesse o menu lateral **Pontos de Extremidade (Endpoints)** e selecione `predict-rentals`.
2. Vá até a aba **Testar (Test)**.
3. Substitua o conteúdo do painel de entrada por este JSON estruturado:

```
{
  "input_data": {
    "columns": [
      "day", "mnth", "year", "season", "holiday", "weekday", 
      "workingday", "weathersit", "temp", "atemp", "hum", "windspeed"
    ],
    "index": [0],
    "data": [[1, 1, 2022, 2, 0, 1, 1, 2, 0.3, 0.3, 0.3, 0.3]]
  }
}
```

4. Clique no botão **Testar**. O resultado retornará a previsão exata de locações no formato:

```
[
  375.8124937043105
]
```