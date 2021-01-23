```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

# Introdução

Quando comecei meu doutorado eu realmente fiquei perdida em muita coisa. Nunca tinha estudado nada da área de Inteligência Artificial ou Aprendizado de Máquina com muita profundidade antes, apesar de ter feito um sistema de recomendação no meu mestrado (2008-2010). Levei um tempo até entender como tudo funcionava, principalmente na área de Classificação Multirrótulo. Mas próximo à minha banca de exame de qualificação de doutorado, eu senti uma enorme necessidade de entender com mais profundidade como as medidas de desempenho em classificação multirrótulo funcionam. Então, eu peguei um conjunto de dados pequeno, que estava usando em alguns experimentos, e os resultados gerados pra ele (usei o [CLUS](http://clus.sourceforge.net/doku.php)) e comecei a calcular em um caderno - na mão mesmo - cada uma das medidas.

A partir daí, eu criei uma planilha no excel pra ajudar a entender melhor as equações e também conferir os resultados. Em seguida, fiz um script em R, extremamente simples e, depois de entender tudo, criei um script em R mais elaborado, com funções pra cada medida e que retorna o resultado de todas elas. Achei o resultado desse meu estudo muito interessante e notei que dava para escrever um artigo técnico a respeito. Portanto, meu objetivo neste artigo não é fornecer um código enxuto e elegante, mas sim, mostrar passo a passo como o cálculo de cada uma das medidas de avaliação de desempenho multirrótulo são computadas. Vocês podem consultar o script R no meu [github](https://github.com/cissagatto/multilabel_performance_evaluation_measures).

Talvez muitas pessoas entendam mais rápido como essas medidas funcionam, mas acredito que existem pessoas por ai, que assim como eu, num primeiro momento ficam literalmente perdidas. Quero ajudar as pessoas com o conhecimento que venho adquirindo ao longo da minha vida e por isto estou compartilhando esta experiência.

Espero que você, que esteja lendo este artigo neste exato momento, seja muito beneficiado. Se gostar compartilhe com outras pessoas também. Então vamos começar o nosso passo a passo!!!   

# Matriz de Confusão

Antes de qualquer coisa, precisamos construir a matriz de confusão multirrótulo. Ela funciona da mesma forma que a matriz de confusão simples rótulo. A única diferença é que ela comportará todos os rótulos do conjunto. A partir dela, faremos os cálculos das medidas.

## Passo 1

Configurar a pasta onde está salvo o seu arquivo com os rótulos verdadeiros e os rótulos preditos.

```{r }
setwd("C:/Users/elain/multilabel_performance_evaluation_measures")
```

## Passo 2

Carregar dois vetores, cada um correspondendo aos rótulos verdadeiros (t = true ou $Y$) e rótulos preditos (p = pred ou $\hat{Y}$) do conjunto de dados que foi testado pelo classificador multirrótulo, ou então um dataframe com ambos desde que a nomenclatura das colunas indique quem são os rótulos verdadeiros e os preditos. Aqui usarei um conjunto de teste como exemplo, composto por $4$ rótulos, denominados de $L1$, $L2$, $L3$ e $L4$ e $51$ instâncias. 
 
```{r }
# abra o arquivo com o resultado do classificador
true_labels = read.csv("true_labels.csv")
pred_labels = read.csv("pred_labels.csv")
bipartition = data.frame(true_labels, pred_labels)
print(bipartition)

# número de instâncias do conjunto
m = nrow(bipartition)
print(m)

# número de rótulos do conjunto
l = ncol(true_labels)
print(l)

# sumario
summary(bipartition)
```

Nós também podemos obter aqui o número total de instâncias positivas e negativas para cada rótulo. Para isto, somamos as linhas de cada coluna Formalmente:

$$
\text{positive_instances_Yj} = \sum^{m}_{i=1} Y_{i}\\
\text{negative_instances_Yj} = m - \text{positive_instances}
$$

onde $Y_{j}$ indica um rótulo específico do espaço de rótulos e $Y_{i}$ indica o rótulo de uma instância específica. Assim, $i$ itera o número de linhas do dataframe (número de instâncias) e $j$ itera o número de colunas (número de rótulos). 

```{r }
positive_instances = apply(bipartition, 2, sum)
print(positive_instances)

negative_instances = m - positive_instances
print(negative_instances)
```


## Passo 3

Agora que já temos os rótulos preditos e os rótulos verdadeiros carregados, devemos confirmar algumas informações e salvá-las para usar posteriormente. O que devemos fazer agora é confirmar quando, em cada uma das instâncias, os rótulos verdadeiros são verdadeiros ou não, e os rótulos preditos são verdadeiros ou não, isto é:

\begin{align*}
se
\left\{\begin{matrix}
 & Y = 1        & \text{então} &  Y' = 1        & \text{caso contrário} & Y' = 0 & \\ 
 & Y = 0        & \text{então} &  Y' = 1        & \text{caso contrário} & Y' =  0 & \\ 
 & \hat{Y} = 1  & \text{então} &  \hat{Y'} = 1  & \text{caso contrário} & \hat{Y'} = 0 \\
 & \hat{Y} = 0  & \text{então} &  \hat{Y'} = 1  & \text{caso contrário} & \hat{Y'} = 0 
\end{matrix}\right.
\end{align*}

onde $Y'$ e $\hat{Y'}$ são os valores resultantes correspondentes à $Y$ e $\hat{Y}$. No R podemos então fazer:

```{r }
# calcular rótulo verdadeiro igual a 1
true_1 = ifelse(true_labels==1,1,0)
print(true_1)

# calcular rótulo verdadeiro igual a 0
true_0 = ifelse(true_labels==0,1,0)
print(true_0)

# calcular rótulo predito igual a 1
pred_1 = ifelse(pred_labels==1,1,0)
print(pred_1)

# calcular rótulo verdadeiro igual a 0
pred_0 = ifelse(pred_labels==0,1,0)
print(pred_0)
```

Também podemos calcucar o total para cada rótulo:

$$
\text{total_Yj} = \sum^{m}_{i=1} Y_{i}\\
$$

```{r }
total_true_1 = apply(true_1, 2, sum)
print(total_true_1)

total_true_0 = apply(true_0, 2, sum)
print(total_true_0)

total_pred_1 = apply(pred_1, 2, sum)
print(total_pred_1)

total_pred_0 = apply(pred_0, 2, sum)
print(total_pred_0)

matriz_totais = cbind(total_true_0,total_true_1, total_pred_0, total_pred_1)
print(matriz_totais)
```

## Passo 4

Agora que já temos os valores de $Y'$ e $\hat{Y'}$, podemos calcular o valor de verdadeiros positivos, verdadeiros negativos, falsos positivos e falsos negativos, que compoem a matriz de confusão. Vamos considerar que a classe positiva é 1 e a classe negativa é 0. Terminologia:

1. **Condição positiva (P)**: é o número de casos positivos reais nos dados, isto é, a instância é originalmente positiva para a classe;

2. **Condição negativa (N)**: é o número de casos negativos reais nos dados, isto é, a instância é originalmente negativa para a classe;

3. **Verdadeiro Positivo (TP = true positive)**: número de instâncias da classe positiva classificadas corretamete, isto é, a classe é 1 e o modelo previu 1;

4. **Verdadeiro Negativo (TN = true negative)**: número de instâncias da classe negativa classificadas corretamente, isto é, a classe é 0 e o modelo previu 0;

5. **Falso Positivo (FP = false positive)**: número de instâncias em que a classe verdadeira é negativa mas foram classificadas incorretamente, isto é, como se pertencessem à classe positiva (a classe é 0 mas o modelo previu 1);

6. **Falso Negativo (FN = false negative)**: número de instâncias originalmente pertencente à classe positiva que foram incorretamente preditas, isto é, como se pertencessem à classe negativa (a classe é 1 mas o modelo previu 0).

Podemos resumir da seguinte forma:

1. Se TPi = (Y' = 1 AND ^Y' = 1) então 1, caso contário 0 

2. Se TNi =  (Y' = 0 AND ^Y' = 0) então 1, caso contário 0 

3. Se FPi = (Y' = 0 AND Y' = 1) então 1, caso contário 0 

4. Se FNi = (Y' = 1  AND Y' = 0) então 1, caso contário 0 

A Figura a seguir apresenta várias formas de representar a matriz de confusão:

```{r pressure, echo=FALSE, fig.cap="Interpretações da matriz de confusão", out.width = '100%'}
knitr::include_graphics("C:/Users/elain/multilabel_performance_evaluation_measures/matriz_con.png")
```


No R fazemos:

```{r }
# Verdadeiro Positivo: O modelo previu 1 e a resposta correta é 1
TPi  = ifelse((true_1 & true_1),1,0)
colnames(TPi) = c("L1", "L2", "L3", "L4")
print(TPi)

# Verdadeiro Negativo: O modelo previu 0 e a resposta correta é 0
TNi  = ifelse((true_0 & pred_0),1,0)
colnames(TNi) = c("L1", "L2", "L3", "L4")
print(TNi)

# Falso Positivo: O modelo previu 1 e a resposta correta é 0
FPi  = ifelse((true_0 & pred_1),1,0)
colnames(FPi) = c("L1", "L2", "L3", "L4")
print(FPi)

# Falso Negativo: O modelo previu 0 e a resposta correta é 1
FNi  = ifelse((true_1 & pred_0),1,0)
colnames(FNi) = c("L1", "L2", "L3", "L4")
print(FNi)
```

## Passo 5

Agora vamos calcular os totais para cada rótulo:

$$
TP_{l} = \sum^{m}_{i=1} TP_{i} \\
TN_{l} = \sum^{m}_{i=1} TN_{i} \\
FP_{l} = \sum^{m}_{i=1} FP_{i} \\
TN_{l} = \sum^{m}_{i=1} TN_{i} \\
$$

No R:

```{r }
# total de verdadeiros positivos
TPl = apply(TPi, 2, sum)
print(TPl)

# total de verdadeiros negativos
TNl = apply(TNi, 2, sum)
print(TNl)

# total de falsos negativos
FNl = apply(FNi, 2, sum)
print(FNl)

# total de falsos positivos
FPl = apply(FPi, 2, sum)
print(FPl)
```

E assim podemos montar uma matriz com todos os rótulos:

```{r }
# montando a matriz de confusão em um dataframe
matriz_confusao_por_rotulos = data.frame(TPl, FPl, TNl, FNl)
colnames(matriz_confusao_por_rotulos) = c("TP","FP", "TN", "FN")
print(matriz_confusao_por_rotulos)
```

Podemos também calcular a porcentagem para cada rótulo dividindo os valores pelo número total de instâncias:

```{r }
# calculando a porcentagem 
matriz_confusao_por_rotulo_porcentagem = data.frame(matriz_confusao_por_rotulos/m)
print(matriz_confusao_por_rotulo_porcentagem)
```

Agora conseguimos calcular a quantidade de acertos e erros (assim como a porcentagem) e juntar todas as informações em uma única tabela.

$$
\text{certos} = FP + FN \\
\text{errados} = TP + FN \\
\text{percent_certos} = \frac{\text{certos}}{m} \\
\text{percent_errados} = \frac{\text{errados}}{m} \\
$$


```{r }
# calculando o total de rótulos classificados errados
errados = matriz_confusao_por_rotulos$FP + matriz_confusao_por_rotulos$FN
print(errados)

# calculando a porcentagem de rótulos classificados errados
percent_errados = errados/m
print(percent_errados)

# calculando o total de rótulos classificados corretamente
certos = matriz_confusao_por_rotulos$TP + matriz_confusao_por_rotulos$TN
print(certos)

# calculando a porcentagem de rótulos classificados corretamente
percent_certos = certos/m
print(percent_certos)

# juntando tudo
result = data.frame(matriz_confusao_por_rotulos, certos, percent_certos, errados, percent_errados)
print(result)
```

Podemos ainda somar os valores da matriz de confusão por rótulos, tanto por linhas, quanto por colunas. 

```{r }
total_matriz_1 = apply(matriz_confusao_por_rotulos, 2, sum)
print(total_matriz_1)

total_matriz_2 = apply(matriz_confusao_por_rotulos, 1, sum)
print(total_matriz_2)
```

Agora que já temos várias informações a respeito dos rótulos preditos e rótulos verdadeiros, podemos começar a computar as medidas de avaliação. 

# Bipartições 

## Baseadas em Instâncias

### Acurácia

A eficácia geral de um classificador multirrótulo é avaliada pela acurácia. Esta métrica calcula a proporção do número de rótulos corretamente preditos em comparação ao número total de rótulos (verdadeiros e preditos) para uma instância, fazendo em seguida a média sobre todas as instâncias.

$$
\begin{equation} \label{eq:A}
    \uparrow A = \frac{1}{m} \sum_{i=1}^{m} \frac{ \mid Y_{i} \cap \hat Y_{i}\mid}{\mid\hat Y_{i} \cup Y_{i}\mid}
\end{equation}
$$
Passo a Passo:

~~~~~~~
1. Se (Yi OR ^Yi) então 1 caso contrário 0
2. Somar o resultado do Passo 1
3. Calcular o verdadeiro positivo para cada rótulo
4. Somar Passo 3
5. Dividir Passo 4 por Passo 2 (e não esquecer de verificar a divisão por zero!)
6. Calcular a média
~~~~~~~

Ou seja:

$$
Passo1_{i} = \Big(( L1 | \hat{L1} ) + ( L2 | \hat{L2} ) + ( L3 | \hat{L3} ) + ( L4 | \hat{L4}) \Big) \\
Passo2_{i} = (TP_{L1} + TP_{L2} + TP_{L3} + TP_{L4}) \\
Passo3 = \Big( ( \frac{Passo1_{1}}{Passo2_{1}}) + (\frac{Passo1_{2}}{Passo2_{2}} ) + .... + (\frac{Passo1_{51}}{Passo2_{51}} )\Big) \text{[se divisão por zero atribuir zero]} \\
Passo4 = \frac{Passo3}{51}
$$

No R:

```{r}
# Calcular o OR
true_yi_ou_pred_yi = ifelse((true_labels|pred_labels),1,0)
colnames(true_yi_ou_pred_yi) = c("L1","L2","L3","L4")
print(true_yi_ou_pred_yi)

# 2. Somar
total_1 = apply(true_yi_ou_pred_yi, 1, sum)
print(total_1)

# 3. Calcular verdadeiro positivo
TPi  = ifelse((true_labels&pred_labels),1,0)
print(TPi)

# 4. Somar
total_2 = apply(TPi, 1, sum)
print(total_2)

# 5. Dividir total_2 por total_1 e calcular a média
acuracia = mean(total_2/total_1)

# Verificar divisão por zero 
print(acuracia)
```

Agora que você já sabe como funciona a equação, você pode elaborar uma função com poucas linhas, mais elegante e eficiente do que o código que mostrei aqui. Isto serve também para todas as próximas seções.

### Subset Accuraccy

Exact Match Ratio também conhecida como Classification Accuracy ou Subset Accuracy ignora os rótulos parcialmente corretos, atuando como a acurácia da classificação simples-rótulo. Como Subset Accuracy avalia apenas as instâncias classificadas corretamente, ela é bastante restritiva.

$$
\begin{equation}
    \uparrow EMR = \frac{1}{m} \sum_{i=1}^{m} I \Big(Y_{i} = \hat Y_{i}\Big)
\end{equation}    
$$
$$
I =
\begin{cases}
        1  & \quad \text{Se ^Yi =  Yi}\\
        0  & \quad \text{caso contrario}
\end{cases}
$$

Passo a Passo:

~~~~~~~
1. Verificar se Yi = ^Yi, se sim, então marcamos 1, caso contrário 0
2. Em seguida somar os rótulos: (Y1 + Y2 + .... Yi)
3. Encontrar a média: Passo2/l (l é o número total de rótulos do conjunto)
4. Somar Passo 3 (somar todas as linhas resultantes do passo 3)
5. Encontrar a média dividindo o passo 4 pelo número total de instâncias: Passo4/m
~~~~~~~

No R:

```{r}
# Passo1: Verificar se Yi = ^Yi, se sim, então marcamos 1, caso contrário 0
equal = ifelse(true_labels==pred_labels,1,0)
print(equal)

# Passo2: Somar os rótulos: (Y1 + Y2 + .... Yi)
soma1 = apply(equal,1,sum)
print(soma1)

# Passo3: Encontrar a média: Passo2/l (l é o número total de rótulos do conjunto)
media1 = soma1/l
print(media1)

# Passo4: Somar Passo 3 (somar todas as linhas resultantes do passo 3)
soma2 = sum(media1)
print(soma2)

# Passo5: Encontrar a média dividindo o passo 4 pelo número total de instâncias: Passo4/m
subset = soma2/m
print(subset)

# OU vc pode simplesmente executar a linha a seguir:
subsetAccuracy = mean(ifelse(true_labels==pred_labels,1,0))
print(subsetAccuracy)
```

### 0/1 Loss

De forma similar à Subset Accuracy, a medida de avaliação 0/1 Loss mede a diferença entre os rótulos verdadeiros e os rótulos preditos, ao invés da igualdade.

$$
\begin{equation}
    \uparrow 01L = \frac{1}{m} \sum_{i=1}^{m} I \Big(Y_{i} != \hat Y_{i}\Big)
\end{equation}    
$$
$$
I =
\begin{cases}
        1  & \quad \text{Se ^Yi != Yi}\\
        0  & \quad \text{caso contrário}
\end{cases}
$$


Passo a Passo:

~~~~~~~
1. Verificar se Yi != ^Yi, se sim, então marcamos 1, caso contrário 0
2. Em seguida somar os rótulos: (Y1 + Y2 + .... Yi)
3. Encontrar a média: Passo2/l (l é o número total de rótulos do conjunto)
4. Somar Passo 3 (somar todas as linhas resultantes do passo 3)
5. Encontrar a média dividindo o passo 4 pelo número total de instâncias: Passo4/m
~~~~~~~

No R:

```{r}
# Passo1: Verificar se Yi != ^Yi, se sim, então marcamos 1, caso contrário 0
diff = ifelse(true_labels!=pred_labels,1,0)
print(diff)

# Passo2: Somar os rótulos/colunas = (Y1 + Y2 + .... Yi)
soma1 = apply(diff,1,sum)
print(soma1)

# Passo3: Encontrar a média = Passo2/l (l é o número total de rótulos do conjunto)
media1 = soma1/l
print(media1)

# Passo4: Somar Passo 3 (somar todas as linhas resultantes do passo 3)
soma2 = sum(media1)
print(soma2)

# Passo5: Encontrar a média dividindo o passo 4 pelo número total de instâncias: Passo4/m
loss1 = soma2/m
print(loss1)

# Ou vc pode simplesmente executar a linha a seguir:
loss2 = mean(ifelse(true_labels!=pred_labels,1,0))
print(loss2)
```


### Hamming Loos

Avalia a fração de rótulos classificados erroneamente, isto é, quando um rótulo incorreto é predito e quando um rótulo relevante não é predito. Esta métrica calcula a diferença simétrica entre o conjunto de rótulos predito e o conjunto de rótulos verdadeiro.

$$
\begin{equation} \label{eq:HL}
    \downarrow HL = \frac{1}{m} \frac{1}{l} \sum_{i=1}^{m} \mid Y_{i} \Delta \hat Y_{i} \mid
\end{equation}
$$

onde:

$$Y_{i} \Delta \hat Y_{i} = ( Y_{i} - \hat Y_{i}) \cup (\hat Y_{i} - Y_{i})$$.

Passo a Passo:

~~~~~~~
1. Yi - ^Yi 
2. ^Yi - Yi
3. (Yi - ^Yi) OR (^Yi - Yi)
4. Somar os rótulos: ((Y1 - ^Y1) OR (^Y1 - Y1)) + ((Y2 - ^Y2) OR (^Y2 - Y2)) + ....
5. Somar as instâncias do Passo 4
6. Fazer (1/l) * ( 1/m) * Passo 5
~~~~~~~

No R:

```{r}
# Passo 1: Yi - ^Yi
sub1 = true_labels - pred_labels
print(sub1)

# Passo 2: ^Yi - Yi
sub2 = pred_labels - true_labels
print(sub2)

# Passo 3: (Yi - ^Yi) OR (^Yi - Yi)
ou = ifelse(sub1|sub2,1,0)
print(ou)

# Passo 4: sum(Yi - ^Yi) - (^Yi - Yi)
soma1 = apply(ou,1,sum)
print(soma1)

# Passo 5: somar as linhas
soma2 = sum(soma1)

# Passo 6: calcular o restante
hammingLoss1 = ((1/l)*(1/m))*soma2
print(hammingLoss1)

# Ou você pode simplesmente fazer:
hammingLoss2 = mean(ifelse((true_labels - pred_labels)|(pred_labels - true_labels),1,0))
print(hammingLoss2)
```

### Precisão

Calcula a fração de rótulos preditos que realmente são relevantes.

$$
\begin{equation}
    \uparrow P = \frac{1}{m} \sum_{i=1}^{m} \frac{\mid Y_{i} \cap \hat Y_{i} \mid }{\hat Y_{i}}  
\end{equation}
$$

Passo a Passo:

~~~~~~~
1. (Yi AND ^Yi) então atribuimos 1 caso contrario 0
2. ^Yi / Passo1
3. Se houver divisão por zero, substituir por zero
4. Somar os resultados do Passo 2
5. Somar Passo 3
6. Dividir pelo número de instâncias
~~~~~~~

No R:
```{r}
# passo 1
result1 = ifelse(true_labels&pred_labels,1,0)
print(result1)

# passo 2
result2 = (pred_labels/result1)
print(result2)

# passo 3
library(dplyr)
result2 <- result2 %>% mutate_if(is.numeric, function(x) ifelse(is.nan(x), 0, x))
result2 <- result2 %>% mutate_if(is.numeric, function(x) ifelse(is.infinite(x), 0, x))
print(result2)

# passo 4
result3 = apply(result2,1,sum)
print(result3)

# passo 5
result4 = sum(result3)
print(result4)

# passo 6
precision = result4/m
print(precision)

# ou
res = pred_labels / (ifelse(true_labels&pred_labels,1,0))
res = res %>% mutate_if(is.numeric, function(x) ifelse(is.nan(x), 0, x))
res = res %>% mutate_if(is.numeric, function(x) ifelse(is.infinite(x), 0, x))
precision2 = sum(apply(res,1,sum))/m
print(precision2)
```


### Revocação

Calcula a fração de rótulos relevantes verdadeiros que também são preditos

$$
\begin{equation}
    \uparrow R = \frac{1}{m} \sum_{i=1}^{m} \frac{ \mid Y_{i} \cap \hat Y_{i} \mid }{Y_{i}}
\end{equation}
$$

Passo a Passo:

~~~~~~~
1. (Yi AND ^Yi) então atribuimos 1 caso contrario 0
2. Yi / Passo1
3. Se houver divisão por zero, substituir por zero
4. Somar os resultados do Passo 2
5. Somar Passo 3
6. Dividir pelo número de instâncias
~~~~~~~

No R:
```{r}
# passo 1
result1 = ifelse(true_labels&pred_labels,1,0)
print(result1)

# passo 2
result2 = (true_labels/result1)
print(result2)

# passo 3
library(dplyr)
result2 <- result2 %>% mutate_if(is.numeric, function(x) ifelse(is.nan(x), 0, x))
result2 <- result2 %>% mutate_if(is.numeric, function(x) ifelse(is.infinite(x), 0, x))
print(result2)

# passo 4
result3 = apply(result2,1,sum)
print(result3)

# passo 5
result4 = sum(result3)
print(result4)

# passo 6
recall = result4/m
print(recall)

# ou
res = true_labels / (ifelse(true_labels&pred_labels,1,0))
res = res %>% mutate_if(is.numeric, function(x) ifelse(is.nan(x), 0, x))
res = res %>% mutate_if(is.numeric, function(x) ifelse(is.infinite(x), 0, x))
recall2 = sum(apply(res,1,sum))/m
print(recall2)
```

### F1

É a média harmônica da precisão e revocação

$$
\begin{equation}
    \uparrow F1 = \frac{1}{m} \sum_{i=1}^{m} \frac{2 \mid Y_{i} \cap \hat Y_{i} \mid }{ \mid \hat Y_{i} \mid + \mid Y_{i} \mid }
\end{equation}
$$

Passo a Passo:

~~~~~~~
1. (Yi AND ^Yi) então atribuimos 1 caso contrario 0
2. 2 * Passo 1
3. ^Yi + Yi
4. Dividir Passo 2 por Passo 3
5. Somar os rótulos/colunas (verificar divisão por zero)
6. Somar a coluna do Passo 5
7. Dividir o Passo 6 por m
~~~~~~~

No R:
```{r}
# passo 1
result1 = ifelse(true_labels&pred_labels,1,0)
print(result1)

# passo 2
result2 = 2 * result1
print(result2)

# passo 3
result3 = true_labels + pred_labels
print(result3)

# passo 4
result4 = result2/result3
print(result4)

# passo 5
result4 <- result4 %>% mutate_if(is.numeric, function(x) ifelse(is.nan(x), 0, x))
print(result4)

# passo 6
result4 <- result4 %>% mutate_if(is.numeric, function(x) ifelse(is.infinite(x), 0, x))
print(result4)

# passo 7
result5 = sum(result4)
print(result5)

# passo 8
f1 = result5/m
print(f1)

# Ou pode você encurtar fazendo da seguinte forma
res = (2*ifelse(true_labels&pred_labels,1,0)) / (true_labels + pred_labels)
res <- res %>% mutate_if(is.numeric, function(x) ifelse(is.nan(x), 0, x))
res <- res %>% mutate_if(is.numeric, function(x) ifelse(is.infinite(x), 0, x))
f1 = sum(res)/m
print(f1)
```

## Baseadas em Rótulos

As medidas de avaliação Macro-Precisão, Macro-Revocaçãoe e Macro-f1 primeiro calculam cada rótulo individualmente e somente depois calculam a média entre todos os rótulos, atribuindo assim pesos iguais para os rótulos - independente se o rótulo é frequente, infrequente ou raro. Já as medidas Micro-Precisão, Micro-Revocação e MicroF1 os rótulos são calculados todos juntos. O desempenho de rótulos raros acaba por influenciar medidas macro média, enquanto que as medidas micro média são mais influenciadas pelos rótulos mais comuns.

Para mensurar predições errôneas de rótulos três medidas foram propostas por Rivolli2018. A primeira medida apresentada denominada Wrong Label Prediction (WLP), mede quando o rótulo pode ser predito para algumas instâncias, mas essas predições estão sempre erradas. A Missing Label Prediction (MLP) permite calcular a proporção de rótulos que nunca são preditos. A terceira métrica, Constant Label Problem (CLP), mede quando o mesmo rótulo é predito para todas as instâncias. Para CLP (rótulos preditos incorretamente), WLP (rótulos nunca preditos) e MLP (rótulos sempre preditos) o valor de retorno ideal é zero, indicando que não há ocorrências destes problemas nas predições dos rótulos.

### Macro-Precisão

$$
\begin{equation}\label{eq:pma}
    \uparrow MaP = \frac{1}{ \mid l \mid } \sum_{j=1}^{ \mid l \mid } \frac{tp_{j}}{tp_{j} + fp_{j}}
\end{equation}
$$

Ou seja:

$$
MaP = \frac{1}{4} * \Big( \Big(\frac{TP_{1}}{TP_{1}+FP_{1}}\Big) + \Big(\frac{TP_{2}}{TP_{2}+FP_{2}}\Big) + \Big(\frac{TP_{3}}{TP_{3}+FP_{3}}\Big) + \Big(\frac{TP_{4}}{TP_{4}+FP_{4}}\Big) \Big)
$$

Para o nosso exemplo ficará assim:

```{r}
print(matriz_confusao_por_rotulos)
```

$$
MaP = \frac{1}{4}* \Big( \Big(\frac{18}{18+0}\Big) + \Big(\frac{1}{1+1}\Big) + \Big(\frac{20}{20+1}\Big) + \Big(\frac{12}{12+0}\Big) \Big) = 0,8630952
$$

No R:

```{r}
# Passo 1: somar TPL com FPL
soma_0 = (TPl+FPl)
print(soma_0)

# Passo 2: dividir TPL pelo passo 1
divide_8 = (TPl/soma_0)
print(divide_8)

# Passo 3: somar o resultado do passo 2
soma_9 = sum(divide_8)
print(soma_9)

# multiplicar (1/l) pelo passo 3
map = (1/l)*soma_9
print(map)

# ou simplesmente
map_2 = (1/l)* sum(TPl/(TPl+FPl))
print(map_2)

```

### Macro-Revocação

$$
\begin{equation}\label{eq:rma}
    \uparrow MaR = \frac{1}{\mid l \mid} \sum_{j=1}^{\mid l \mid} \frac{tp_{j}}{tp_{j} + fn_{j}}
\end{equation}
$$

Ou seja:

$$
MaR = \frac{1}{4} * \Big( \Big(\frac{TP_{1}}{TP_{1}+FN_{1}}\Big) + \Big(\frac{TP_{2}}{TP_{2}+FN_{2}}\Big) + \Big(\frac{TP_{3}}{TP_{3}+FN_{3}}\Big) + \Big(\frac{TP_{4}}{TP_{4}+FN_{4}}\Big) \Big)
$$

Para o nosso exemplo ficará assim:

```{r}
print(matriz_confusao_por_rotulos)
```

$$
MaR = \frac{1}{4}*\Big( \Big(\frac{18}{18+1}\Big) + \Big(\frac{1}{1+1}\Big) + \Big(\frac{20}{20+0}\Big) + \Big(\frac{12}{12+0}\Big) \Big) = 0,8618421 \\
$$

No R:

```{r}
# Passo 1: somar TPL com FNL
soma_3 = (TPl+FNl)
print(soma_3)

# Passo 2: dividir TPL pelo passo 1
divide_3 = (TPl/soma_3)
print(divide_3)

# Passo 3: somar o resultado do passo 2
soma_4 = sum(divide_3)
print(soma_4)

# multiplicar (1/l) pelo passo 3
mar = (1/l)*soma_4
print(mar)

# ou simplesmente
mar_2 = (1/l)* sum(TPl/(TPl+FNl))
print(mar_2)
```

### Macro-F1

$$ 
\begin{equation}\label{eq:f1ma}
    \uparrow MaF1 = \frac{2 \times MaP \times MaR}{MaP + MaR}
\end{equation}
$$

Para o nosso exemplo ficará assim:

$$
MaF1 = \frac{2*0,8630952*0.8618421}{0,8630952+0.8618421} = 0,8624682\\
$$

No R:

```{r}
maf1 = (2*map*mar) / (map+mar)
print(maf1)
```

### Micro-Precisão

$$
\begin{equation} \label{eq:pmi}
    \uparrow MiP = \frac{\sum_{j=1}^{l}tp_{j}}{\sum_{j=1}^{l}tp_{j} + \sum_{j=1}^{l}fp_{j}}
\end{equation}
$$

Ou seja:

$$
MiP = \frac{(TP1 + TP2 + TP3 + TP4)}{(TP1+TP1+TP3+TP4)+(FP1+FP2+FP3+FP4)}
$$

Para o nosso exemplo ficará assim:

```{r}
print(matriz_confusao_por_rotulos)
```

$$
MiP = \frac{18 + 1 + 20 + 12}{(18 + 1 + 20 + 12) + (0 + 1 + 1 + 0) } = 0,9622642
$$

No R:

```{r}
# soma TPL
res_a = sum(TPl)
print(res_a)

# soma FNL
res_b = sum(FPl)
print(res_b)

# soma TPL com FNL
res_c = res_a + res_b
print(res_c)

# divide RES1 por RES3
mip = res_a/res_c
print(mip)

# ou simplesmente:
mip2 = sum(TPl)/sum(sum(TPl) + sum(FPl))
print(mip2)
```

### Micro-Revocação

$$
\begin{equation} \label{eq:rmi}
    \uparrow MiR = \frac{\sum_{j=1}^{l}tp_{j}}{\sum_{j=1}^{l}tp_{j} + \sum_{j=1}^{l}fn_{j}}
\end{equation}
$$

Ou seja:

$$
MiR = \frac{(TP1 + TP2 + TP3 + TP4)}{(TP1+TP1+TP3+TP4)+(FN1+FN2+FN3+FN4)}
$$

Para o nosso exemplo ficará assim:

```{r}
print(matriz_confusao_por_rotulos)
```

$$
MiR = \frac{18 + 1 + 20 + 12}{(18 + 1 + 20 + 12) + (1 + 1 + 0 + 0) } = 0,9622642
$$

No R:

```{r}
# soma TPL
res_d = sum(TPl)
print(res_d)

# soma FNL
res_e = sum(FNl)
print(res_e)

# soma TPL com FNL
res_f = res_d + res_e
print(res_f)

# divide RES1 por RES3
mir = res_d/res_f
print(mir)

# ou simplesmente:
mir2 = sum(TPl)/sum(sum(TPl) + sum(FNl))
print(mir2)
```

### Micro-F1

$$
\begin{equation} \label{eq:f1mi}
    \uparrow MiF1 = \frac{2 \times MiP \times MiR}{MiP + MiR}
\end{equation}
$$  

Para o nosso exemplo ficará assim:

$$
MiF1 = \frac{2*0.9622642*0.9622642}{0.9622642+0.9622642} = 0,9622642 \\
$$

No R:

```{r}
mif1 = (2*mip*mir) / (mip+mir)
print(mif1)
```

### CLP

$$
\begin{equation} \label{eq:clp}
    \downarrow CLP = \frac{1}{l} \sum_{j=1}^{l} I \Big(tn_{j} + fn_{j} == 0 \Big)
\end{equation}
$$

Para o nosso exemplo isso significa fazer:

```{r}
print(matriz_confusao_por_rotulos)
```

$$
((TN1 + FN1) == 0) = ((33 + 1) == 0) = (34 == 0) = 0\\
((TN2 + FN2) == 0) = ((49 + 1) == 0) = (50 == 0) = 0\\
((TN3 + FN3) == 0) = ((30 + 0) == 0) = (30 == 0) = 0\\
((TN4 + FN4) == 0) = ((39 + 0) == 0) = (39 == 0) = 0\\
CLP = \frac{1}{4}*(0 + 0 + 0  + 0) = 0
$$


No R:

```{r}
# primeiro somar TNl com FNl e verificar quando o resultado é igual a zero
res_6 = ifelse((TNl+FNl)==0,1,0)
print(res_6)

# somar o resultado
res_7 = sum(res_6)
print(res_7)

# dividir pelo número de rótulos
clp = res_7/l
print(clp)

# ou simplesmente:
clp2 = sum((TNl+FNl)==0)/l
print(clp2)
```


### MLP

$$
\begin{equation} \label{eq:mlp}
    \downarrow MLP = \frac{1}{l} \sum_{j=1}^{l} I \Big(tp_{j} + fp_{j} == 0 \Big)
\end{equation}
$$

Para o nosso exemplo isso significa fazer:

```{r}
print(matriz_confusao_por_rotulos)
```


$$
((TP1 + FP1) == 0) = ((18 + 0) == 0) = (18 == 0) = 0\\
((TP2 + FP2) == 0) = ((1 + 1) == 0) = (2 == 0) = 0 \\
((TP3 + FP3) == 0) = ((20 + 1) == 0) = (21 == 0) = 0\\
((TP4 + FP4) == 0) = ((12 + 0) == 0) = (12 == 0) = 0\\
MLP = \frac{1}{4}*(0 + 0 + 0  + 0) = 0
$$

No R:
```{r}
# primeiro somar TPl com FPl e verificar quando o resultado da soma é igual a zero
res_4 = ifelse((TPl+FPl)==0,1,0)
print(res_4)

# depois somar o resultado
res_5 = sum(res_4)
print(res_5)

# e então dividir pelo número total de rótulos
mlp = res_5/l
print(mlp)

# ou simplesmente:
mlp2 = sum((TPl+FPl)==0)/l
print(mlp2)
```

### WLP

$$
\begin{equation} \label{eq:wlp}
    \downarrow WLP = \frac{1}{l} \sum_{j=1}^{l} I \Big(tp_{j} == 0 \Big)
\end{equation}
$$

Para o nosso exemplo isso significa fazer:

```{r}
print(matriz_confusao_por_rotulos)
```

$$
(TP1 == 0) = (18 == 0) = 0 \\
(TP2 == 0) = (1 == 0) = 0 \\
(TP3 == 0) = (20 == 0) = 0 \\
(TP4 == 0) = (12 == 0) = 0 \\
WLP = \frac{1}{4}*(0 + 1 + 0  + 0) = 0
$$

No R:

```{r}
# primeiro verificar quando Tl é igual a 0
res_1 = ifelse(TPl==0,1,0)
print(res_1)

# depois somar o resultado
res_2 = sum(res_1)
print(res_2)

# dividir pelo número total de rótulos
wlp = res_2/l
print(wlp)

# ou simplesmente:
wlp2 = sum(TPl==0)/l
print(wlp2)
```

# Conclusão

Neste artigo apresentei apenas as medidas para bipartições. O artigo ficaria imenso se incluísse também as medidas de Ranking, portanto, essas ficarão para um outro artigo. Espero que tenham gostado. Qualquer dúvida entrem em contato: elainececiliagatto@gmail.com

# Referências

1. CHARTE, F.; RIVERA, A. J.; CHARTE, D.; JESUS, M. J. del; HERRERA, F. [Tips, guidelinesand tools for managing multi-label datasets: The mldr.datasets r package and the cometa datarepository](https://www.sciencedirect.com/science/article/abs/pii/S0925231218301401).Neurocomputing, 2018. ISSN 0925-2312.

2. GIBAJA, E.; VENTURA, S. [Multi-label learning: A review of the state of the art and ongoingresearch](https://onlinelibrary.wiley.com/doi/abs/10.1002/widm.1139). Wiley Interdiscip. Rev. Data Min. Knowl. Discov., v. 4, n. 6, p. 411–444, 2014. ISSN19424795.

3. GIBAJA, E.; VENTURA, S. [A tutorial on multilabel learning](https://dl.acm.org/doi/10.1145/2716262). ACM Comput. Surv., Associationfor Computing Machinery, New York, NY, USA, v. 47, n. 3, abr. 2015. ISSN 0360-0300.

4. HERRERA, F.; CHARTE, F.; RIVERA, A. J.; JESUS, M. J. del. [Multilabel Classification:Problem Analysis, Metrics and Techniques](https://www.springer.com/gp/book/9783319411101). 1st. ed. [S.l.]: Springer Publishing Company, Incor-porated, 2016. ISBN 3319411101.

5. MADJAROV, G.; KOCEV, D.; GJORGJEVIKJ, D.; DžEROSKI, S. [An extensive experimentalcomparison of methods for multi-label learning](https://www.sciencedirect.com/science/article/abs/pii/S0031320312001203). In: . [S.l.: s.n.], 2012. v. 45, p. 3084–3104. ISSN00313203.

6. TSOUMAKAS, G.; KATAKIS, I. [Multi-label classification: An overview](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.104.9401). Int J Data Warehou-sing and Mining, v. 2007, p. 1–13, 2007.

7. READ, J. [Scalable Multi-label Classification](https://researchcommons.waikato.ac.nz/handle/10289/4645). Tese (Doutorado) — University of Waikato, 2010.

8. PEREIRA, R. B.; PLASTINO, A.; ZADROZNY, B.; MERSCHMANN, L. H. [Correlationanalysis  of  performance  measures  for  multi-label  classification](https://www.sciencedirect.com/science/article/abs/pii/S0306457318300165). Information  Processing  &Management, v. 54, n. 3, p. 359 – 369, 2018. ISSN 0306-4573.

9. RIVOLLI, A.; SOARES, C.; CARVALHO, A. C. P. d. L. F. d. [Enhancing multilabel classificationfor food truck recommendation](https://onlinelibrary.wiley.com/doi/abs/10.1111/exsy.12304). Expert Systems, Wiley-Blackwell, 2018.

10. ZHANG, M. L.; ZHOU, Z. H. [A review on multi-label learning algorithms](https://ieeexplore.ieee.org/document/6471714). IEEE ComputerSociety, v. 26, n. 8, p. 1819–1837, 2014.

11. WU, X.-Z.; ZHOU, Z.-H. [A unified view of multi-label performance measures](https://arxiv.org/abs/1609.00288). In:Proceedingsof the 34th International Conference on Machine Learning. [S.l.]: JMLR.org, 2017. (ICML17,v. 70), p. 3780–3788.

