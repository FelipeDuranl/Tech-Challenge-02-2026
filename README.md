# Tech-Challenge-02-2026

## Link do vídeo de demonstração

<a href="https://www.youtube.com/watch?v=TWPqJF_naAo">Vídeo</a> ou link: https://www.youtube.com/watch?v=rVzxaHRoK9E

# Relatório Técnico

## 1. Visão Geral da Solução

Nesta fase, o projeto foi dividido em três etapas principais:

1. **Preparação dos dados:** reutilização do pipeline desenvolvido na Fase 1, mantendo as mesmas etapas de tratamento e divisão dos dados.

2. **Otimização dos modelos:** aplicação de um Algoritmo Genético para encontrar combinações de hiperparâmetros que maximizem o desempenho dos modelos de classificação.

3. **Interpretação dos resultados:** integração com uma LLM para transformar as predições dos modelos e métricas obtidas em explicações em linguagem natural, facilitando a interpretação dos resultados.

O fluxo desenvolvido combina técnicas de Machine Learning tradicional, otimização evolutiva e modelos de linguagem para criar uma ferramenta de apoio à análise de risco de diabetes.

## 2. Reaproveitamento da Preparação de Dados (Fase 1)

Nesta fase, reaproveitamos toda a preparação dos dados realizada na Fase 1, incluindo a junção dos seis datasets do Kaggle, a remoção de registros duplicados, o tratamento dos valores ausentes com a mediana e a limitação dos valores extremos (outliers). Com isso, todos os testes desta fase foram realizadas sobre a mesma base de dados.

## 3. Algoritmo Genético para Otimização de Hiperparâmetros

### Representação Genética
Cada indivíduo representa um conjunto de hiperparâmetros de um modelo (um gene por hiperparâmetro), gerado a partir de um espaço de busca definido para cada algoritmo:

| Modelo | Hiperparâmetros otimizados |
|---|---|
| Regressão Logística | `C` (0.1–10.0), `solver` (lbfgs/liblinear), `max_iter` (100–500) |
| Random Forest | `n_estimators` (50–200), `max_depth` (3–15), `min_samples_split` (2–10) |
| KNN | `n_neighbors` (3–15), `weights` (uniform/distance) |

O SVM foi mantido fora da otimização genética nesta rodada (espaço de busca comentado no notebook).

### Operadores Implementados
- **Fitness:** F1-Score médio via validação cruzada no conjunto de treino, usado como critério de qualidade de cada indivíduo;
- **Seleção por Torneio:** grupos pequenos de indivíduos competem entre si, e o de maior fitness é selecionado para reprodução;
- **Crossover Uniforme:** cada gene do filho é herdado aleatoriamente de um dos dois pais, favorecendo a diversidade genética;
- **Mutação:** cada gene tem uma probabilidade de ser substituído por um novo valor dentro do espaço de busca;
- **Elitismo:** os melhores indivíduos de cada geração são preservados diretamente na próxima geração, evitando perda de boas soluções por crossover/mutação.

### Configuração dos Experimentos
Foram executados 3 experimentos, variando simultaneamente tamanho da população, número de gerações e taxa de mutação (elitismo e seleção por torneio mantidos fixos):

| Experimento | População | Gerações | Taxa de Mutação |
|---|---|---|---|
| 1 | 20 | 10 | 0.10 |
| 2 | 30 | 15 | 0.15 |
| 3 | 40 | 20 | 0.20 |

### Resultados por Experimento (F1-Score de validação cruzada)

| Modelo | Experimento 1 | Experimento 2 | Experimento 3 | Melhor Experimento |
|---|---|---|---|---|
| Regressão Logística | 0.6459 | 0.6459 | 0.6459 | Experimento 1 |
| Random Forest | 0.6833 | **0.6920** | 0.6871 | Experimento 2 |
| KNN | 0.6517 | 0.6517 | 0.6517 | Experimento 1 |

<img width="709" height="393" alt="fitness_evolution_randomforest" src="https://github.com/user-attachments/assets/e6e7e60c-6328-41d9-b107-264dc7bf9634" />
<img width="726" height="393" alt="fitness_evolution_logisticregression" src="https://github.com/user-attachments/assets/b6d92953-daee-4d24-86ec-2ece33e1a253" />
<img width="726" height="393" alt="fitness_evolution_knn" src="https://github.com/user-attachments/assets/031eb0ff-f287-4025-81f9-3d1e182cda31" />

Durante os testes, a Regressão Logística e o KNN estabilizaram em poucas gerações, enquanto o Random Forest levou um pouco mais de tempo para convergir.

### Melhores hiperparâmetros encontrados

Ao final da otimização, o Algoritmo Genético encontrou as seguintes configurações para cada modelo:

| Modelo | Melhor experimento | Melhores hiperparâmetros |
|---|---|---|
| Regressão Logística | Experimento 1 | `C = 1.0`, `solver = liblinear`, `max_iter = 470` |
| Random Forest | Experimento 2 | `n_estimators = 62`, `max_depth = 15`, `min_samples_split = 4` |
| KNN | Experimento 1 | `n_neighbors = 9`, `weights = distance` |

## 4. Comparação entre o modelo original e o modelo otimizado

Para garantir uma comparação justa, o modelo original e o modelo otimizado usam exatamente o mesmo X_train/X_test e o mesmo pipeline da Fase 1, a única diferença é o conjunto de hiperparâmetros.

| Modelo | F1 Original | F1 Otimizado | Ganho (%) |
|---|---|---|---|
| Regressão Logística | 0.5918 | 0.5918 | 0.00% |
| Random Forest | 0.6471 | 0.6214 | **-3.97%** |
| KNN | 0.5607 | 0.5825 | **+3.88%** |

<img width="989" height="590" alt="f1_comparacao_modelos" src="https://github.com/user-attachments/assets/0db9c767-ca2a-43a8-9e19-67fc6782f581" />


### Interpretação
O resultado mais interessante foi o do **Random Forest**. Os hiperparâmetros escolhidos pelo Algoritmo Genético obtiveram o melhor F1 em validação cruzada (0.6920) durante o próprio processo evolutivo, mas ao treinar o modelo final e avaliar no conjunto de teste isolado, o desempenho caiu (0.6214 vs. 0.6471 do modelo original). Esse comportamento pode indicar um certo overfitting durante o processo de otimização, já que os hiperparâmetros apresentaram ótimo desempenho na validação cruzada, mas não conseguiram manter o mesmo resultado no conjunto de teste.

Já o **KNN** teve ganho real e consistente (+3.88%), beneficiado principalmente pela troca do peso de vizinhança de **uniform** para **distance** e pelo ajuste fino de **n_neighbors**. A **Regressão Logística** não mostrou variação, já que os hiperparâmetros ótimos encontrados ficaram muito próximos dos valores padrão do scikit-learn.

Esse resultado mostra que utilizar apenas o F1-Score da validação cruzada como função de fitness nem sempre garante que o modelo terá o melhor desempenho em dados novos.

## 5. Integração com LLM para Interpretação de Resultados

Foi integrada uma LLM (GPT-5 Mini via Azure AI Foundry) para interpretar os resultados dos modelos. Também foi implementado um modo simulado, para que o notebook possa ser executado mesmo sem configurar uma chave de API.

### Técnicas de Prompt Engineering aplicadas

| Técnica | Como foi aplicada |
|---|---|
| Role | O `system prompt` define a LLM como assistente de apoio à decisão clínica, nunca como substituto do médico |
| Contexto | Números reais do experimento (F1, accuracy, ganho %, hiperparâmetros) são inseridos diretamente no prompt, evitando que a LLM "invente" valores |
| Instruções estruturadas de saída | Limite de palavras, formato em tópicos e recomendação final obrigatória |
| Guardrails explícitos | Proibição de diagnóstico definitivo; a LLM sempre remete à validação médica |
| LLM como validação | Uma segunda chamada avalia a qualidade da resposta gerada, com base nos mesmos critérios de avaliação. |

### Explicações por Paciente
Foi criada a função **gerar_explicacao_diagnostico**, que utiliza o modelo **Random Forest** otimizado para realizar as predições. A função recebe os dados clínicos de um paciente (Glicose, IMC, Idade, Insulina, DPF etc.) e a predição do modelo, gerando uma explicação em linguagem natural sobre o risco de diabetes. Em todas as respostas, a LLM reforça que se trata de um indicador de risco, e não de um diagnóstico definitivo.

Foi implementada também uma **Área de Teste Interativa**, que permite digitar manualmente os dados de um paciente e receber, na hora, a predição do modelo otimizado e a explicação gerada pela LLM.

### Avaliação de Qualidade
A qualidade das explicações foi avaliada por dois meios complementares:
- Um **checklist qualitativo manual**, cobrindo clareza para não-especialistas, segurança clínica, fidelidade aos dados numéricos e presença de uma recomendação acionável;
- Uma abordagem de **LLM como validação**, em que uma segunda chamada à LLM avalia a resposta gerada com base nos mesmos critérios de avaliação.

## 6. Desafios Enfrentados

Durante o desenvolvimento desta etapa, encontramos alguns desafios que influenciaram os resultados obtidos.

- **Limitações da otimização de hiperparâmetros:** apesar de o Algoritmo Genético encontrar diferentes combinações de hiperparâmetros, os ganhos foram pequenos e, em alguns casos, o desempenho no conjunto de teste foi inferior ao modelo original. Isso mostrou que otimizar hiperparâmetros nem sempre é suficiente para melhorar o desempenho do modelo.

- **Qualidade da base de dados:** o dataset utilizado possui uma quantidade considerável de valores ausentes (representados por zeros) e uma quantidade limitada de amostras. Esses fatores dificultam a obtenção de melhorias significativas apenas com a otimização dos hiperparâmetros, indicando que uma base de dados mais completa poderia contribuir para resultados melhores.

- **Tempo de execução do SVM:** durante os testes, a otimização do SVM apresentou um tempo de execução muito elevado, tornando os experimentos pouco viáveis. Por esse motivo, optamos por não incluí-lo na etapa de otimização genética.

- **Desafios na construção dos prompts da LLM:** um dos principais desafios foi definir como a LLM deveria interpretar os resultados dos modelos. Foi necessário criar um **system prompt** bem estruturado para orientar o comportamento da IA, deixando claro que ela deveria atuar apenas como uma ferramenta de apoio à decisão clínica, sem realizar diagnósticos ou substituir a avaliação de um profissional de saúde.

- **Organização do contexto enviado para a LLM:** outro ponto importante foi definir quais informações deveriam ser enviadas para a IA. Foi necessário estruturar o contexto com os dados do paciente, a predição realizada pelo modelo, as métricas obtidas nos experimentos e outras informações relevantes, para que a resposta fosse baseada nos resultados reais e evitasse interpretações incorretas ou informações não presentes nos dados.

- **Integração com a API da LLM:** a comunicação com a API também trouxe alguns desafios, principalmente relacionados à configuração da autenticação, gerenciamento da chave de acesso e tratamento de possíveis erros durante as chamadas. Além disso, foi criado um modo simulado para que o notebook continuasse funcionando mesmo sem uma chave de API configurada, facilitando a execução e avaliação do projeto.

- **Avaliação da qualidade das respostas geradas:** também foi necessário pensar em como verificar se as explicações produzidas pela LLM estavam adequadas. Para isso, foram definidos critérios de avaliação relacionados à clareza das respostas, fidelidade aos dados fornecidos e segurança das informações, garantindo que a IA gerasse explicações úteis sem ultrapassar o papel de apoio à análise clínica.

## Conclusão

O Algoritmo Genético conseguiu encontrar boas combinações de hiperparâmetros de forma automática. Ao mesmo tempo, os resultados mostraram que um bom desempenho na validação cruzada nem sempre significa que o modelo terá o mesmo resultado em dados novos. O Random Forest foi o exemplo mais claro desse comportamento, enquanto o KNN apresentou um ganho consistente de F1-Score em relação ao modelo original.

A integração com a LLM (Azure AI Foundry, gpt-5-mini) ajudou a transformar métricas e predições em explicações mais fáceis de interpretar pelos profissionais da saúde. Além disso, todas as respostas deixam claro que o sistema serve apenas como apoio à decisão e que o diagnóstico final continua sendo responsabilidade do médico. Essa estrutura também pode ser reaproveitada em futuras etapas do projeto, com a possibilidade de incorporar novas informações ao processo de interpretação dos resultados.

Assim como na Fase 1, reforçamos que o sistema — incluindo a etapa de otimização por Algoritmo Genético e a interpretação por LLM — funciona como uma ferramenta de apoio à triagem, e não como um substituto da avaliação clínica. Dessa forma, o projeto atingiu os objetivos propostos para esta etapa, combinando a otimização automática de hiperparâmetros com a geração de explicações em linguagem natural para apoiar a interpretação dos resultados.
