# Tech-Challenge-01-2026

## Link do video de demonstração

<a href="https://youtu.be/pyBCIGB0OYs">Video</a> ou link: https://youtu.be/pyBCIGB0OYs


# Relatório Técnico

## 1. Estratégias de Pré-processamento

O pré-processamento foi a etapa principal para garantir que o modelo não fosse treinado com dados não normalizado e padronizados, levando a uma possivel falha no diagnostico. As seguintes estratégias foram aplicadas:

### Consolidação e Limpeza de Base de Dados
Foram baixados 6 datasets diferentes do Kaggle. Através de uma análise de similaridade, observamos que alguns eram duplicados. Construimos uma  fontes únicas, resultando em um dataset inicial de 4.310 registros, que após a remoção de duplicatas, foi reduzido para 778 registros únicos.

<img width="940" height="519" alt="image" src="https://github.com/user-attachments/assets/ce1d650e-0d87-4c76-9bdb-bf8d6efeb9fc" />

### Tratamento de Valores Faltantes
Identificamos que colunas como Glucose, BloodPressure, IMC, Insulin e SkinThickness continham valores "0", o que é clinicamente impossível.

**Insulin (48.46% de valores igual a zero)** e **SkinThickness (29.43% de valores igual a zero)** apresentaram as maiores taxas de dados ausentes.

### Imputação por Mediana
Para não perder nosso dataset, substituímos os zeros por NaN e utilizamos o SimpleImputer com a estratégia de mediana, que substitui esses valores de zero para a mediana, ajudando a normalização do nosso dataset, resultando em um melhor treinamento do modelo e por fim seu diagnostico.

<img width="940" height="520" alt="image" src="https://github.com/user-attachments/assets/deadf9d7-d75c-4d03-9c5d-c0bb9a7a6a19" />

### Controle de Outliers
Analisamos que variáveis como Insulin e SkinThickness tinham valores extremos (máximos de 846 e 110, respectivamente). Aplicamos o **Capping**, limitando os valores para reduzir o ruído, estabilizando o desvio padrão da Insulina de 85.97 para 78.89, trazendo o valores mais para o centro.

<img width="940" height="578" alt="image" src="https://github.com/user-attachments/assets/8ecbcabc-ed54-49c8-bc02-d4e5d583b5b1" />


| Antes do capping | Depois do capping |
|------------------|------------------|
| <img width="459" height="484" alt="image" src="https://github.com/user-attachments/assets/a9fe1e7c-d539-4aed-9157-ee99525a9062" /> | <img width="434" height="484" alt="image" src="https://github.com/user-attachments/assets/21a40c34-8b2d-4b8a-8a5d-d84fddef484d" />
 |

---

## 2. Modelos Usados e Por Quê

Para este desafio de classificação binária de diabetes, Sim ou Não, os seguintes modelos são propostos para comparação:

**Random Forest:** Foi a escolhida pela sua estabiliade nos testes. Sua capacidade de lidar com dados não-lineares e trabalhar contra o overfitting foi um dos motivos para ser usado. Dado que o dataset possui muitas variáveis que interagem entre si e depende uma da outra, este modelo baseado em árvores tende a performar melhor.

**Regressão Logística:** Utilizada como modelo de baseline. É ideal por ser muito  interpretável, Permite que os médicos entendam a probabilidade por trás de cada diagnóstico.

**KNN (K-Nearest Neighbors):** Útil para identificar padrões de "pacientes similares". Se um novo paciente possui indicadores clínicos próximos a outros pacientes já diagnosticados, o KNN faz a classificação por proximidade.

**SVM (Support Vector Machines):** Utilizado para encontrar a melhor separação possível entre os grupos (diabético vs. saudável). É ideal por ser muito conciso contra erros e eficiente em lidar com exames médicos que não possuem uma divisão clara e direta.

---

## 3. Resultados e Interpretação dos Dados

A análise exploratória mostrou pontos para a interpretação clínica:

### Perfil de Distribuição
Outcome apresenta um desbalanceamento de aproximadamente **1.86:1** ou seja, 65% sem diabetes e 35% com diabetes. Isso indica que o modelo deve ser avaliado com atenção ao **F1-Score**, para evitar falsos negativos e falsos positivos  em pacientes doentes.

<img width="940" height="319" alt="image" src="https://github.com/user-attachments/assets/c3d249fd-e5f4-4160-ae29-4046d542bef5" />


### Correlações Clínicas
Vimos que a **Glicose** e o **IMC** possuem distribuições que se deslocam para a direita no grupo positivo. A análise de correlação mostra como variáveis como idade e número de gestações impactam a probabilidade da doença.

<img width="940" height="576" alt="image" src="https://github.com/user-attachments/assets/d26faaf2-a291-4550-8352-0361cd547999" />

<img width="940" height="533" alt="image" src="https://github.com/user-attachments/assets/dc0d997e-1482-40e4-9a21-3fcf8b0efbcb" />

---

### Análise de Importância
Utilizando _Feature Importance_, espera-se que a Glicose seja o preditor mais forte. Através do **SHAP**, poderemos demonstrar ao médico não apenas "se" o paciente tem diabetes, mas "quais" indicadores (ex: IMC alto combinado com Glicose elevada) foram determinantes para aquela decisão da IA.

---

# Resultados finais

### Comparação dos resultados com e sem validação cruzada

<img width="846" height="643" alt="image" src="https://github.com/user-attachments/assets/d308c029-caeb-4ae8-8ef0-8c626ff02cf3" />

### Comparação de modelos

<img width="826" height="640" alt="image" src="https://github.com/user-attachments/assets/15f9c1e6-fe5c-42b5-a655-4e95ddd6bf87" />

## Conclusão

Concluimos que o modelo Random Forest foi o mais estável dentre todos os modelos testados, por mais que o SVM seja o que obteve melhores métricas de Recall e F1-Score, ele não mostrou resultados estáveis dentre os nossos testes, após executarmos algumas vezes, ele não teve a mesma performace.

Portanto, decidimos seguir com o Random Forest, seus testes se mantiveram estaveis com as métricas importantes para disgnosticos de saúde. É um modelo que trabalha de forma mais acertiva e eficiencite em cima de dados não-lineares, e consegue interelacionar as variaveis. O Random Forest treina subconjuntos diferentes de dados  em cada árvore, e usa apenas um conjunto aleatorio  de variaveis em cada divisão, que posteriormente, para um novo dado, cada árvore da floresta da seu voto, a resposta final é a maioria dos votos (Classificação), ou a media da previsão (Regressão).

O modelo serve como uma **ferramenta de triagem**. Os resultados mostram que, embora a IA possa acelerar a análise, ainda precisamos garantir que o médico tenha a palavra final na validação dos exames laboratoriais.
