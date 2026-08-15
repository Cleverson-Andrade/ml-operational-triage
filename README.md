# Case Study — Modelo de Machine Learning para Triagem Operacional 🤖

> Case study de um modelo de classificação supervisionada desenvolvido para estimar a probabilidade de um item exigir priorização em uma etapa de triagem operacional, a partir de medições numéricas e atributos derivados.

---

## Contexto

Processos de inspeção e classificação operacional podem envolver grande volume de itens e múltiplas medições. Quando a avaliação depende exclusivamente de regras manuais ou revisões individuais, o fluxo tende a consumir tempo, dificultar a priorização e tornar a identificação de casos de atenção menos consistente.

Este projeto apresenta, de forma conceitual e anonimizada, uma abordagem de Machine Learning criada para apoiar a triagem. O modelo identifica padrões em dados históricos e produz uma probabilidade de priorização, ajudando a direcionar a revisão humana para os itens mais relevantes.

A solução foi concebida como **suporte à decisão**, não como substituição de critérios técnicos, validação especializada ou responsabilidade humana.

---

## Objetivo da Solução

O objetivo foi construir e avaliar um classificador binário capaz de:

- Transformar medições brutas em atributos analíticos relevantes;
- Estimar a probabilidade de um item pertencer à classe que exige atenção;
- Avaliar o comportamento do modelo com validação cruzada;
- Comparar diferentes limiares de decisão conforme o trade-off operacional;
- Gerar artefatos de avaliação para análise técnica;
- Apoiar a priorização de itens para revisão humana.

---

## Minha Contribuição

Atuei no desenvolvimento do experimento de ponta a ponta, incluindo:

- Estruturação do problema como classificação binária supervisionada;
- Preparação de dados tabulares e separação entre identificadores e atributos preditivos;
- Desenvolvimento de *features* derivadas a partir de medições numéricas;
- Construção de um pipeline de treinamento com CatBoost;
- Implementação de validação cruzada estratificada com previsões out-of-fold;
- Cálculo de métricas por classe e matriz de confusão;
- Análise exploratória de limiares de probabilidade;
- Geração de arquivos de avaliação para rastreabilidade técnica;
- Documentação de limitações, riscos metodológicos e critérios de uso responsável.

---

## Formulação do Problema

O projeto trata uma tarefa de **classificação binária**. Para cada registro, o modelo estima a probabilidade de pertencer a uma classe de priorização.

A saída é utilizada para apoiar uma decisão de triagem:

- **Classe prioritária:** item direcionado para revisão ou tratamento adicional;
- **Classe não prioritária:** item sem indicação de priorização pelo modelo naquele momento.

A classificação final deve considerar contexto operacional, regras vigentes e revisão humana. A probabilidade produzida pelo modelo é um sinal analítico, não uma decisão autônoma.

---

## Pipeline de Machine Learning

A solução foi organizada em etapas sequenciais:

1. **Leitura e validação da base**  
   Carregamento de dados tabulares, separação de identificadores e verificação da estrutura necessária para o experimento.

2. **Preparação de atributos**  
   Limpeza de dados, tratamento de valores numéricos inválidos e preparação da matriz de variáveis preditoras.

3. **Feature engineering**  
   Criação de atributos derivados para representar relações geométricas, proporções, diferenças e medidas de forma.

4. **Definição do rótulo histórico**  
   Construção da variável-alvo a partir de uma classificação histórica anonimizada e previamente definida no contexto do experimento.

5. **Treinamento e validação**  
   Treinamento com validação cruzada estratificada e geração de previsões out-of-fold.

6. **Avaliação e limiar operacional**  
   Comparação entre probabilidades previstas e diferentes pontos de corte para apoiar a escolha de um limiar compatível com o objetivo de triagem.

7. **Saídas de avaliação**  
   Geração de métricas agregadas, métricas por fold, matriz de confusão, relatório por classe e predições de validação.

---

## Feature Engineering

Além das medições originais, foram construídos atributos derivados para representar melhor a estrutura dos dados. Entre as categorias de *features* utilizadas estão:

- Razões entre medidas dimensionais;
- Diferenças entre medidas relacionadas;
- Estimativas de diâmetro e largura equivalentes;
- Proporções entre área, perímetro e medidas de referência;
- Indicadores de compacidade;
- Relações entre área observada, área convexa e lacunas.

Esse processo busca fornecer ao modelo sinais mais expressivos do que as medições brutas isoladas. Como em qualquer exercício de *feature engineering*, as variáveis devem ser avaliadas quanto a estabilidade, qualidade de medição, correlação e potencial de leakage.

---

## Modelo e Validação

Foi utilizado um modelo baseado em **gradient boosting para dados tabulares**, escolhido por sua capacidade de capturar relações não lineares e interações entre variáveis numéricas, com baixo esforço de pré-processamento em comparação a algumas alternativas.

A avaliação foi realizada com:

- **Validação cruzada estratificada em 5 folds** para preservar a distribuição das classes em cada particionamento;
- **Previsões out-of-fold (OOF)**, em que cada registro é previsto por um modelo que não utilizou aquele registro no treinamento;
- Semente aleatória controlada para reprodutibilidade do experimento;
- Métricas calculadas tanto por fold quanto sobre o conjunto consolidado de previsões OOF.

As previsões OOF são especialmente úteis porque oferecem uma estimativa mais honesta do desempenho do modelo durante o desenvolvimento, evitando avaliar cada observação com um modelo que já a viu no treinamento.

---

## Métricas Avaliadas

O experimento avalia métricas complementares para evitar conclusões baseadas apenas em accuracy:

| Métrica | Por que é relevante |
|---|---|
| **Accuracy** | Mede a proporção total de classificações corretas, mas pode ser insuficiente quando as classes são desbalanceadas. |
| **F1 Macro** | Equilibra precision e recall, atribuindo importância equivalente a cada classe. |
| **Precision da classe prioritária** | Indica a proporção de itens sinalizados pelo modelo que realmente pertenciam à classe prioritária. |
| **Recall da classe prioritária** | Indica a capacidade de encontrar os itens que deveriam ser priorizados. |
| **Recall da classe não prioritária** | Ajuda a monitorar classificações indevidas de itens não prioritários. |
| **Matriz de confusão** | Mostra a distribuição de acertos, falsos positivos e falsos negativos. |

O equilíbrio entre essas métricas depende do custo operacional dos erros. Em uma triagem, errar ao deixar de sinalizar um item relevante pode ter custo diferente de encaminhar um item adicional para revisão.

---

## Análise de Limiar de Decisão

O modelo produz uma probabilidade para a classe prioritária. Essa probabilidade é convertida em classificação por meio de um limiar de decisão.

- **Limiar menor:** aumenta a sensibilidade para identificar itens prioritários, mas pode elevar falsos positivos;
- **Limiar maior:** torna a classificação mais conservadora, mas pode deixar de sinalizar alguns itens relevantes.

Em vez de assumir que `0,50` é sempre o ponto ideal, o experimento compara diferentes limiares e observa o impacto em precision, recall, F1 e accuracy. A escolha deve ser orientada pelo risco e pelo custo operacional de cada tipo de erro, com validação das partes responsáveis pelo processo.

---

## Saídas Técnicas do Experimento

A execução original gerou artefatos de avaliação, mantidos apenas no ambiente autorizado:

- Métricas por fold da validação cruzada;
- Métricas consolidadas para o limiar avaliado;
- Grade comparativa de limiares;
- Matriz de confusão;
- Relatório de classificação por classe;
- Predições OOF com probabilidade, fold e sinalização de acerto;
- Metadados do experimento para rastreabilidade.

Nenhum dado, métrica real, predição individual ou arquivo de resultado é publicado neste repositório.

---

## Limitações e Próximos Passos

Este case representa um experimento de apoio à triagem. Antes de qualquer uso operacional ampliado, recomenda-se:

- Validar o modelo em dados de períodos futuros ou ambientes independentes;
- Usar validação por grupos quando houver lotes, origens ou entidades correlacionadas;
- Avaliar a calibração das probabilidades previstas;
- Comparar o modelo com baselines simples e regras já existentes;
- Investigar importância de variáveis e explicabilidade local/global;
- Monitorar *data drift*, qualidade das medições e estabilidade das métricas;
- Definir governança para revisão humana, rastreabilidade de decisões e reavaliação periódica;
- Revisar o limiar conforme mudanças no custo operacional de falsos positivos e falsos negativos.

Machine Learning não elimina incerteza; ele apenas a organiza em probabilidades. O polvo estatístico ainda precisa de supervisão.

---

## Tecnologias e Competências Aplicadas

- **Python** para automação do experimento e processamento de dados;
- **pandas** e **NumPy** para manipulação de dados tabulares e *feature engineering*;
- **CatBoost** para classificação supervisionada baseada em gradient boosting;
- **scikit-learn** para validação cruzada e métricas de avaliação;
- **Validação out-of-fold** para estimativa de desempenho durante o desenvolvimento;
- **Threshold analysis** para análise de trade-offs entre tipos de erro;
- **Exportação estruturada** de métricas e resultados técnicos;
- **Reprodutibilidade** com particionamento estratificado e controle de aleatoriedade.

---

## Confidencialidade e Uso Responsável

Este repositório apresenta exclusivamente uma visão conceitual, generalizada e não identificável de um experimento desenvolvido em ambiente corporativo.

Por motivos de confidencialidade, proteção de propriedade intelectual e privacidade, este material não inclui:

- Código-fonte, notebooks, scripts executáveis ou configurações do ambiente;
- Dados reais, medições, identificadores ou rótulos originais;
- Regras operacionais específicas de classificação ou descarte;
- Métricas reais, matrizes de confusão, predições ou relatórios produtivos;
- Credenciais, caminhos de arquivos, fontes de dados ou infraestrutura;
- Informações que permitam reproduzir ou inferir o processo operacional original.

Os termos, descrições e exemplos foram adaptados exclusivamente para fins de portfólio profissional e comunicação técnica responsável.

---

## Autor

**Cleverson Moura**  
Data Science / Analytics

[LinkedIn](https://www.linkedin.com/in/cleversonmandrade/)
