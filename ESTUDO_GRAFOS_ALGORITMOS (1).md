# Caderno de estudo — Teoria dos Grafos e Algoritmos

> **Revisão da conversa de 19/10/2026** · Preparação para entrevista de Cientista de Dados com foco em Grafos.  
> Este arquivo reúne **o conteúdo estudado e corrigido na nossa conversa**. Não é uma arquitetura definida para a pesquisa de homeostase da memória.

## 1. Conceito central: grafo não é pipeline

Um **grafo** é a estrutura matemática \(G=(V,E)\), onde \(V\) é o conjunto de vértices e \(E\) o conjunto de arestas. Vértices representam entidades; arestas representam relações entre elas. Um grafo pode ser usado para modelar um fluxo, mas **não é, por definição, uma sequência de execução**.

- **Estrutura:** quais vértices e arestas existem.
- **Topologia:** como essas conexões estão organizadas.
- **Execução/travessia:** procedimento que um algoritmo aplica à estrutura, começando de um vértice escolhido quando necessário.
- **Histórico temporal:** a ordem em que as arestas foram criadas **não é recuperável apenas pela topologia**, salvo se for registrada como metadado/evento.

### Exemplo discutido — amizades

Você já possui amizade com sua mãe (A—C). Depois surge amizade com o presidente (A—B) e, em seguida, amizade entre presidente e sua mãe (B—C). **O processo de criação teve uma sequência temporal**, mas o triângulo resultante A—B—C—A **não tem um vértice intrinsecamente inicial**: um ciclo pode ser percorrido a partir de A, B ou C.

Atenção: amizades recíprocas podem ser modeladas por **arestas não direcionadas**; solicitações de amizade, por relações **direcionadas**. É preciso explicitar qual relação a aresta representa.

## 2. Principais classificações

| Conceito | O que significa | Lembrete de entrevista |
|---|---|---|
| Direcionado (dígrafo) | Aresta \(A\to B\) tem orientação. | Direção não implica, por si só, execução nem precedência. |
| Não direcionado | Aresta \(A-B\) conecta dois vértices sem orientação. | Ideal para relações recíprocas. |
| Ponderado | Arestas possuem pesos (custo, distância, similaridade etc.). | O significado do peso depende do problema. |
| Não ponderado | Sem pesos explícitos. | Para menor **número de arestas**, BFS é adequada. |
| Cíclico | Contém ao menos um ciclo. | Não é necessário que **todos** os vértices participem de ciclos. |
| Acíclico | Não possui ciclos (segundo a noção aplicável ao tipo de grafo). | DAG = direcionado + acíclico. |
| Conexo (não direcionado) | Todo par de vértices tem um caminho. | Em dígrafos: distinguir conexão **forte** e **fraca**. |
| Simples | Sem laços nem arestas paralelas. | Multigrafos podem ter arestas paralelas, conforme a convenção. |
| Árvore | Grafo não direcionado conexo e acíclico. | Se tem \(V\) vértices, tem \(V-1\) arestas. |

**Uma classificação não exclui necessariamente outra:** um grafo pode ser **direcionado, ponderado e cíclico** ao mesmo tempo.

## 3. Vértices, arestas, grau e fontes

- **Origem da aresta:** em \(A\to B\), A é a origem **dessa aresta**, independentemente do seu grau de entrada.
- **Destino da aresta:** em \(A\to B\), B é o destino.
- **Grau de entrada** \(deg^-(v)\): quantidade de arestas que chegam a v.
- **Grau de saída** \(deg^+(v)\): quantidade de arestas que saem de v.
- **Vértice fonte:** \(deg^-(v)=0\). Não é sinônimo de "qualquer vértice escolhido para iniciar uma busca".
- **Vértice sumidouro:** \(deg^+(v)=0\).
- **Vértice isolado:** nenhuma aresta incidente (num dígrafo, entrada e saída iguais a 0).
- **Grau em grafo não direcionado:** total de arestas incidentes, contando um laço duas vezes.
- **Identidade útil:** num dígrafo, \(\sum_v deg^-(v)=\sum_v deg^+(v)=|E|\).

### Exercício já respondido

Grafo \(A\to B\to C\to A\): A tem **grau de entrada 1** e **grau de saída 1**. Ao adicionar a aresta \(A\to C\), a entrada de A continua 1 e sua saída passa a **2**.

### Não confundir grau com relevância

Grau convencional **conta conexões**. Um vértice B com três arestas **continua com grau 3**, mesmo que suas relações sejam mais pertinentes à pergunta que as 500 arestas de A. Em grafos ponderados, pode-se calcular **força/grau ponderado** agregando pesos, mas **relevância dependente da consulta** é outra grandeza: não altera automaticamente o grau topológico.

## 4. Adjacência versus alcançabilidade

- **Adjacência entre vértices:** há uma **aresta direta** entre eles (em dígrafos, atenção ao sentido).
- **Alcançabilidade:** existe um **caminho**, direto ou passando por outros vértices.
- **Arestas adjacentes/incidentes a um mesmo vértice:** arestas que compartilham pelo menos um extremo; não são a mesma coisa que *vértices adjacentes*.

### Exemplo dos voos

Se existirem apenas **Rio → Salvador** e **Salvador → GRU**, **Rio e GRU não são adjacentes**, mas **GRU é alcançável a partir do Rio**. Se houver voo direto Rio → GRU, passam a ser adjacentes também.

O grafo básico modela **rotas**; quando horários importam, a viabilidade temporal de conexões exige uma modelagem adicional.

## 5. Caminhos, ciclos e grafos acíclicos

- **Caminho:** sequência de vértices ligados por arestas, normalmente sem repetição de vértices; em dígrafos, deve respeitar o sentido.
- **Ciclo dirigido:** sequência fechada de arestas, retornando ao início sem repetir outros vértices no caso de ciclo simples.
- **Grafo cíclico:** basta **um** ciclo em qualquer parte do grafo.
- **Um ciclo pode começar a ser descrito em qualquer de seus vértices:** A→B→C→A, B→C→A→B e C→A→B→C são descrições do mesmo ciclo.

### Exemplo discutido

Arestas: A→B, A→C, B→D, C→D, D→E. É **um único DAG**, não cinco grafos separados: cada linha é uma aresta. Ao adicionar **D→A**, surgem **dois ciclos simples dirigidos**: A→B→D→A e A→C→D→A. O vértice E não participa, mas o grafo completo é cíclico.

## 6. Ordenação topológica e algoritmo de Kahn

**Ordenação topológica** é uma **ordem linear de todos os vértices de um DAG** tal que, para cada aresta \(u\to v\), **u aparece antes de v**. Não equivale a seguir fisicamente um caminho: vértices consecutivos na lista podem não ser adjacentes.

- **Só existe para o grafo direcionado inteiro quando ele é acíclico** (DAG; assumindo grafos finitos, como em algoritmos usuais).
- Um DAG finito tem **ao menos um vértice fonte**, mas pode ter **vários**.
- Uma ordem topológica pode **não ser única**; ter só uma fonte inicial não garante unicidade.
- A origem **de uma aresta** não precisa ser uma fonte.
- Um vértice que não era fonte pode se **tornar fonte no grafo restante** durante Kahn.

### Exemplo A→C e B→C

Ordens válidas: **A, B, C** e **B, A, C**. A e B são fontes; ambos precisam vir antes de C. C não pode ser selecionado após apenas A porque ainda possui uma entrada vinda de B.

### Exemplo com fonte única

A→B e A→C tem **uma única fonte, A**, mas admite as ordens **A, B, C** e **A, C, B**. Se acrescentarmos B→C, a ordem torna-se **A, B, C**.

### Passo a passo — Kahn

1. Calcular os graus de entrada.
2. Colocar em uma fila/conjunto todos os vértices de grau de entrada zero.
3. Retirar um deles e acrescentá-lo à ordenação.
4. Remover conceitualmente suas arestas de saída, decrementando os graus de entrada dos destinos.
5. Adicionar os destinos que chegarem a grau de entrada zero e repetir.
6. Se foram processados menos de \(|V|\) vértices, **existe um ciclo**.

**Complexidade:** \(O(V+E)\) com lista de adjacência e controle apropriado de graus de entrada.

**Subgrafo:** mesmo quando o grafo original contém ciclos, um **subgrafo induzido ou escolhido que seja DAG** pode ser ordenado topologicamente. Essa ordem não será uma ordenação válida do grafo original inteiro.

## 7. BFS — busca em largura

**Analogia usada no estudo:** a "vizinha fofoqueira": conversa com os vizinhos, depois com os vizinhos dos vizinhos, por níveis.

- Geralmente utiliza uma **fila FIFO**.
- Explora os vértices em ordem de **distância em número de arestas** a partir da origem.
- Encontra **caminhos com menor número de arestas** em grafos não ponderados, ou quando todas as arestas têm o mesmo custo.
- Não garante o menor **peso acumulado** quando os pesos são diferentes.
- Pode ser utilizada para testar alcançabilidade, distâncias por saltos, expansão em níveis.

**Complexidade:** \(O(V+E)\) em lista de adjacência, para percorrer os elementos alcançáveis (ou todas as componentes, se executada para cada origem necessária).

### Exemplo de entrevista

A→B peso 100; A→C peso 10; C→B peso 10. **BFS escolhe A→B**, porque tem **1 aresta** em vez de 2. Se nosso objetivo é minimizar soma de pesos, **A→C→B custa 20**, melhor que 100. "Melhor" depende da função objetivo: se a meta fosse menor número de transferências, o caminho direto seria adequado.

## 8. DFS — busca em profundidade

**Analogia usada no estudo:** o "detetive": segue um ramo tão longe quanto possível e, quando não consegue avançar, **retrocede** (*backtracking*) para explorar outros ramos.

- Geralmente utiliza **pilha LIFO** ou recursão.
- Não garante o caminho mínimo.
- É útil para exploração, detecção de ciclos (com controle de estados), conectividade, componentes, ordenação topológica via tempos de término em DAGs.

**Complexidade:** \(O(V+E)\) em lista de adjacência.

**Exemplo do estudo:** exploração de dependências/relações em código e repositórios pode usar DFS; não afirmar que uma biblioteca específica usa DFS sem verificar sua implementação.

### BFS versus DFS

| Pergunta | BFS | DFS |
|---|---|---|
| Como avança? | Por camadas | Por um ramo, com retrocesso |
| Estrutura típica | Fila | Pilha / recursão |
| Menor número de arestas? | Sim, em grafo não ponderado | Não garante |
| Detecta ciclos? | Com lógica adequada | Com lógica adequada |
| Complexidade típica | \(O(V+E)\) | \(O(V+E)\) |

## 9. Dijkstra — menor custo com pesos não negativos

Dijkstra encontra caminhos mínimos **por soma dos pesos** a partir de uma origem, com garantia quando todas as arestas têm **peso ≥ 0**. **Zero é permitido.**

- Mantém uma **distância provisória** para cada vértice.
- Seleciona repetidamente o vértice ainda não finalizado com a **menor distância provisória** (estratégia gulosa/*greedy*).
- **Relaxa** as arestas de saída, tentando melhorar as distâncias dos vizinhos.
- Com pesos não negativos, a distância do vértice extraído como mínimo pode ser finalizada com segurança.

**Complexidade típica:** \(O((V+E)\log V)\) com lista de adjacência e heap binário; depende da implementação.

**Greedy não significa "necessariamente mais lento".** É um princípio de escolha local, não uma definição de complexidade.

### Por que a restrição de pesos?

A→B peso 5; A→C peso 10; C→B peso −8. Há um caminho A→C→B de custo **2**. Uma versão clássica de Dijkstra que finalize B ao extrair sua menor distância provisória pode fixar B com custo 5 **antes** de explorar C. Logo, **arestas negativas quebram a garantia de correção**; não significa que toda instância com pesos negativos falhará.

## 10. Bellman-Ford — pesos negativos

- Pode encontrar caminhos mínimos com **arestas negativas**, desde que não exista **ciclo de peso total negativo alcançável a partir da origem** que torne o custo indefinidamente reduzível.
- Realiza relaxamentos sucessivos das arestas (até \(|V|-1\) rodadas completas na versão básica).
- Uma rodada adicional identifica a possibilidade de **ciclo negativo alcançável**.
- **Complexidade usual:** \(O(VE)\).

**Não confundir:** um grafo **cíclico** não impede automaticamente o cálculo de caminhos mínimos. O problema especial de Bellman-Ford são **ciclos negativos alcançáveis**, não qualquer ciclo.

## 11. Diferenças que o entrevistador pode explorar

| Confusão comum | Distinção correta |
|---|---|
| "Todo grafo direcionado tem ordem topológica." | Só todo **DAG** finito admite essa ordenação. |
| "A tem origem, então é fonte." | Ser origem de **uma aresta** não implica grau de entrada 0. |
| "Ciclo tem um começo obrigatório." | Há um ponto de início escolhido para descrevê-lo, não um início estrutural único. |
| "Um grafo direcionado é um pipeline." | Arestas orientadas podem representar inúmeras relações, não apenas execução. |
| "A tem 500 conexões, então é mais relevante." | **Grau** é contagem; **relevância** depende do problema e da medida escolhida. |
| "BFS acha a menor distância de GPS." | BFS minimiza **número de arestas** em grafos não ponderados; não quilômetros variáveis. |
| "Dijkstra só aceita peso positivo." | Aceita peso **não negativo** (incluindo zero). |
| "Dijkstra é mais lento porque é greedy." | Greedy é uma estratégia; desempenho depende das operações e estruturas. |
| "Visitar A, B, C significa ter arestas A→B→C." | Uma **ordem topológica** não precisa ser um caminho. |
| "Adicionar D→A fecha um único ciclo." | No exemplo com ramificações por B e C, fecha **dois** ciclos simples. |

## 12. Perguntas de entrevista COM respostas

As respostas abaixo foram formuladas para você **conseguir explicar oralmente**. Os exemplos e observações mostram onde o entrevistador pode aprofundar.

### 1) O que é um grafo? O que são vértices e arestas?

**Resposta:** Um grafo é uma estrutura matemática definida por **\(G=(V,E)\)**, em que **\(V\)** é o conjunto de vértices e **\(E\)** o conjunto de arestas. Os vértices representam entidades e as arestas representam relações entre elas. Na modelagem de voos, aeroportos podem ser vértices e rotas podem ser arestas; o grafo descreve as conexões, não necessariamente um pipeline de execução.

**Se aprofundarem:** direção, peso e significado de uma aresta dependem da relação modelada.

### 2) Um grafo pode ser direcionado, ponderado e cíclico ao mesmo tempo?

**Resposta:** Sim. As propriedades não são mutuamente exclusivas. Considere **A→B (peso 5), B→C (peso 3), C→A (peso 2)**. As setas indicam direção, os números são pesos e as arestas formam um ciclo dirigido. **Cíclico** significa que existe pelo menos um ciclo; não significa que todos os vértices do grafo participem dele.

**Se aprofundarem:** A→B→C→A pode ser descrito começando por B ou C também; o ciclo não possui início intrínseco.

### 3) Qual a diferença entre origem da aresta, vértice fonte e início de travessia?

**Resposta:** Em **A→B**, A é a **origem daquela aresta**, mesmo que receba outras arestas. Um **vértice fonte** é definido pelo grau de entrada igual a zero. Já o **ponto de partida de uma BFS ou DFS** é um vértice escolhido para iniciar o algoritmo e não precisa ser fonte. No grafo A→B→C, B é a origem da aresta B→C, mas não é uma fonte.

**Se aprofundarem:** Um vértice pode tornar-se fonte **no grafo restante** durante a execução do algoritmo de Kahn.

### 4) Rio→Salvador→GRU: Rio e GRU são adjacentes? GRU é alcançável?

**Resposta:** Rio e GRU **não são adjacentes**, porque não existe aresta direta Rio→GRU. Contudo, GRU **é alcançável a partir do Rio** pelo caminho Rio→Salvador→GRU. Adjacência exige conexão direta; alcançabilidade exige um caminho, possivelmente com vértices intermediários.

**Se aprofundarem:** Em dígrafos, o sentido importa. A alcançabilidade Rio→GRU não garante GRU→Rio.

### 5) O que caracteriza um ciclo dirigido?

**Resposta:** É um percurso fechado que parte de um vértice e retorna a ele respeitando a orientação das arestas, sem repetir vértices intermediários quando falamos de **ciclo simples**. Por exemplo, A→B→C→A. As conexões A→B, B→C e A→C **não** formam ciclo dirigido, porque não existe retorno orientado até A.

**Se aprofundarem:** Um ciclo é propriedade da estrutura, mesmo que nunca executemos uma travessia.

### 6) Todos os vértices precisam estar em ciclos para o grafo ser cíclico?

**Resposta:** Não. Basta existir **um único ciclo** em qualquer parte do grafo. Se temos A→B→C→A e C→D, o grafo é cíclico apesar de D não pertencer a nenhum ciclo.

**Se aprofundarem:** Um grafo pode combinar regiões cíclicas com regiões acíclicas.

### 7) Quando existe ordenação topológica? Ela pode ter mais de uma resposta?

**Resposta:** Para um **grafo direcionado finito**, a ordenação topológica existe se, e somente se, ele é **acíclico (DAG)**. É uma lista de todos os vértices em que, para cada aresta u→v, u aparece antes de v. Pode haver várias listas válidas: com A→C e B→C, tanto **A,B,C** quanto **B,A,C** respeitam as restrições. A lista **não precisa representar um caminho** no grafo.

**Se aprofundarem:** Um DAG tem ao menos uma fonte; ter uma única fonte inicial **não garante** ordem única. Em A→B e A→C, A,B,C e A,C,B são válidas.

### 8) Como funciona o algoritmo de Kahn? E se a fila esvaziar antes do fim?

**Resposta:** Primeiro calculamos o **grau de entrada** de cada vértice e colocamos os de grau zero numa fila. Retiramos um vértice, acrescentamos à ordenação e removemos conceitualmente suas arestas de saída, atualizando os graus. Quando novos vértices chegam a grau zero, entram na fila. Se a fila esvaziar **antes de processar todos os vértices**, há um ciclo e não existe ordenação topológica do grafo inteiro. A implementação usual roda em **\(O(V+E)\)** com listas de adjacência.

**Se aprofundarem:** Depois de processar A em A→C e B→C, C ainda recebe B; logo B deve ser processado antes de C.

### 9) Qual a diferença entre BFS e DFS e quando utilizar cada uma?

**Resposta:** **BFS** busca em largura: percorre os vértices por níveis a partir de uma origem, normalmente com uma **fila FIFO**. Ela encontra caminhos mínimos **em número de arestas** em grafos não ponderados. **DFS** busca em profundidade: segue um ramo até não conseguir avançar e então retrocede (*backtracking*), normalmente usando **pilha ou recursão**. DFS é útil em exploração de dependências, componentes, detecção de ciclos e ordenação topológica de DAGs. Ambas podem ser implementadas em **\(O(V+E)\)** com listas de adjacência.

**Analogia do estudo:** BFS é a vizinha fofoqueira que pergunta aos vizinhos e aos vizinhos dos vizinhos; DFS é o detetive que investiga um ramo antes de voltar.

### 10) O caminho com menos arestas tem sempre o menor peso acumulado?

**Resposta:** Não. Imagine A→B com peso 100 e A→C→B com pesos 10 e 10. BFS identifica o caminho direto de **uma aresta**, mas ele custa 100; o caminho de duas arestas custa 20. Se o objetivo é minimizar saltos, A→B é melhor; se é minimizar a **soma dos pesos**, A→C→B é melhor.

**Se aprofundarem:** Sempre pergunte o que o peso representa: quilômetros, tempo, custo, risco ou outra grandeza.

### 11) Como funciona Dijkstra, por que exige pesos não negativos e o que significa *greedy*?

**Resposta:** Dijkstra mantém uma distância provisória para os vértices e seleciona, em cada etapa, aquele de menor distância ainda não finalizado. Ao selecioná-lo, considera sua distância definitiva e tenta melhorar os custos de seus vizinhos por **relaxamento de arestas**. A estratégia é *greedy* porque faz uma escolha local de menor custo conhecido. Com pesos **não negativos (inclusive zero)**, essa escolha é segura. Com arestas negativas, uma distância já finalizada pode ser melhorada depois, quebrando a garantia de correção.

**Exemplo:** A→B custa 5; A→C custa 10; C→B custa −8. O caminho por C custa 2, embora B possa ser finalizado com 5 antes de C ser analisado. *Greedy* não é sinônimo de lento: com heap binário, uma complexidade usual é **\(O((V+E)\log V)\)**.

### 12) O que Bellman-Ford resolve que Dijkstra não garante?

**Resposta:** Bellman-Ford calcula caminhos mínimos mesmo com **arestas de peso negativo**, desde que não exista um **ciclo negativo alcançável da origem** que permita reduzir indefinidamente o custo. Ele relaxa as arestas repetidas vezes (na versão básica, até V−1 passadas completas), e uma passada adicional permite detectar ciclos negativos alcançáveis. Sua complexidade usual é **\(O(VE)\)**.

**Se aprofundarem:** Um ciclo positivo ou zero não impede automaticamente caminhos mínimos; o problema especial é um ciclo **de peso total negativo** relevante para a origem.

### 13) Qual a diferença entre grau, grau ponderado e relevância contextual?

**Resposta:** **Grau** é a quantidade de arestas incidentes (ou grau de entrada e saída, num dígrafo). **Grau ponderado** (*strength*) agrega os pesos das arestas incidentes, geralmente por soma. **Relevância contextual** é uma pontuação definida para o objetivo ou a pergunta atual; pode combinar conteúdo, relações e pesos, mas **não é o mesmo que grau**. Uma memória com 3 conexões pode ser mais relevante para uma consulta do que uma memória com 500, embora seu grau seja menor.

**Se aprofundarem:** Quando o peso depende da consulta, uma pontuação ponderada pode mudar sem que o número de arestas — o grau convencional — mude.

### 14) Sua pesquisa já comprovou maior velocidade de recuperação usando grafos?

**Resposta:** **Não.** A pesquisa investiga a hipótese de que representar relações explícitas entre memórias possa melhorar a **continuidade contextual**, a recuperação e a gestão de retenção, atualização e esquecimento; a eficiência computacional é uma questão **a medir**, não um resultado concluído. Uma avaliação precisará comparar a abordagem com alternativas fortes, incluindo busca vetorial indexada, considerando qualidade de recuperação, latência, custo, armazenamento e manutenção.

**Se aprofundarem:** Nem toda busca vetorial é exaustiva. Índices aproximados como **HNSW** já usam estruturas de grafos; a hipótese de grafos de memória é sobre **relações semânticas/contextuais explícitas**, não apenas navegação entre vetores próximos.

## 13. Vínculo com a pesquisa (sem fechar solução)

Grafos podem representar relações persistentes entre memórias e permitir estudar conectividade, caminhos, graus, pesos, componentes e travessias. **Isso ainda não estabelece qual algoritmo será utilizado**, como selecionar um vértice inicial nem se a recuperação ficará mais rápida. Primeiro vem o problema de pesquisa e a comparação experimental; depois, a decisão de arquitetura.

> **Frase de fechamento:** o grafo guarda a estrutura das relações; um algoritmo define como percorrê-la ou analisá-la; o desempenho deve ser medido.

## 14. Algoritmos e modelos que aparecem no roteiro do Itaú (fotos de 19/10/2026)

> **Distinção:** esta seção reúne métodos **explicitamente mencionados** nas fotos enviadas pelo Itaú. Nem todos são algoritmos únicos: alguns são técnicas, famílias de modelos ou procedimentos de avaliação. **Não são necessariamente perguntas confirmadas da entrevista.**

| Tema citado | Definição curta para responder oralmente | Ponto de atenção se aprofundarem |
|---|---|---|
| **Árvore de decisão** | Aplica divisões recursivas nos atributos para prever uma classe ou valor. | Impureza Gini/entropia na classificação; risco de overfitting. |
| **Random Forest** | Conjunto de árvores treinadas com amostragens e subconjuntos aleatórios de variáveis; agrega suas previsões. | *Bagging*, redução de variância e árvores treinadas em paralelo. |
| **Gradient Boosting** | Adiciona modelos sequencialmente para reduzir uma função de perda, tipicamente ajustando-se ao gradiente negativo. | Diferença para bagging e significado da função de perda. |
| **AdaBoost** | Combina classificadores fracos sequencialmente, enfatizando amostras difíceis conforme o erro. | Não é sinônimo de Gradient Boosting; mecanismo de atualização é diferente. |
| **Regressão logística** | Modela a probabilidade de uma classe, usualmente pela função sigmoide no caso binário. | Classifica apesar do nome "regressão"; limiar de decisão não precisa ser 0,5. |
| **KNN** | Usa os `k` exemplos de treinamento mais próximos para predizer rótulos/valores. | Supervisão, escolha de `k`, métrica de distância e escala de atributos. |
| **Naive Bayes** | Aplica o Teorema de Bayes e supõe independência condicional entre atributos dada a classe. | GaussianNB (atributos contínuos aproximados por Normais) vs BernoulliNB (binários). |
| **Redes neurais** | Aprendem transformações parametrizadas por pesos, otimizados com gradientes. | *Backpropagation*, ativação, função de perda e overfitting. |
| **SVM** | Busca uma fronteira de decisão com grande margem; kernels podem modelar separações não lineares. | Margem, vetores de suporte e parâmetro `C`. |
| **Regressão linear** | Estima uma resposta contínua como combinação linear de variáveis explicativas. | Resíduos, multicolinearidade e hipóteses para inferência. |
| **Regularização L1/L2** | Penaliza a magnitude dos coeficientes para controlar complexidade. | L1 pode zerar coeficientes; L2 tende a encolhê-los sem zerar exatamente. |
| **Árvore de regressão** | Divide atributos para prever valores numéricos, tipicamente reduzindo erro quadrático. | Difere da árvore de classificação no critério de partição/saída. |
| **GLM** | Liga o valor esperado da resposta a um preditor linear por uma função de ligação. | Distribuição, função de ligação e exemplos (logística, Poisson). |
| **K-means** | Particiona observações em `k` grupos minimizando a soma das distâncias quadráticas aos centroides. | Sensível à escala, inicialização e outliers; número `k` definido previamente. |
| **K-medoids** | Agrupa ao redor de observações reais que representam cada cluster. | Pode ser mais robusto a valores extremos que K-means. |
| **DBSCAN** | Identifica regiões densas e classifica alguns pontos como ruído. | `eps`, `min_samples`; não exige `k` e encontra clusters de formas variadas. |
| **Agrupamento hierárquico** | Constrói uma hierarquia de grupos por aglomeração/divisão. | Dendrograma, métrica de distância e critério de ligação. |
| **PCA** | Projeta dados em componentes ortogonais que maximizam variância explicada. | Redução **linear**; não é um algoritmo de seleção simples de colunas originais. |
| **Simplex** | Resolve programação linear movendo-se entre soluções básicas viáveis até encontrar ótimo ou diagnosticar situações especiais. | Função objetivo, restrições lineares e região viável. |
| **Programação inteira** | Busca soluções ótimas sujeitas a variáveis inteiras ou binárias. | *Branch-and-bound* é um método comum; relaxação linear remove temporariamente a exigência de integralidade. |

**Outros tópicos das fotos que também exigem definição:** tratamento de valores ausentes e outliers; categorização; correlação/associação; seleção de variáveis; métricas (AUC, KS, Gini, F1, RMSE, MAE, R²); validação *holdout*, *k-fold* e *out-of-time*; SQL/joins; NLP, Transformers, RAG, embeddings, *fine-tuning*, quantização, RLHF e guardrails. Esses tópicos são métodos, métricas ou conceitos, não necessariamente um algoritmo específico.

### Quatro contrastes clássicos de entrevista

- **KNN x K-means:** KNN é supervisionado e prediz por proximidade a exemplos rotulados; K-means é não supervisionado e agrupa exemplos.
- **Random Forest x Gradient Boosting:** Random Forest agrega muitas árvores com diversidade para reduzir variância; Gradient Boosting constrói modelos sequencialmente para reduzir a perda.
- **PCA x seleção de variáveis:** PCA cria novas componentes como combinações lineares; seleção de variáveis escolhe atributos já existentes.
- **DBSCAN x K-means:** DBSCAN procura densidade e pode separar ruído; K-means exige `k` e prefere agrupamentos aproximadamente compactos sob distância euclidiana.

## 15. Métodos adicionais de grafos para uma vaga especializada (não listados nominalmente nas fotos)

Esses métodos **não aparecem pelo nome no roteiro fotografado**. São sugestões para estudar por causa do foco declarado da vaga em grafos, e **não** escolhas definidas para a pesquisa de homeostase.

### 15.1 PageRank — importância estrutural em grafos direcionados

**Resposta curta:** PageRank atribui importância a um vértice levando em conta a importância de outros vértices que apontam para ele, com normalização das contribuições e amortecimento (*damping*). É um método de centralidade baseado em passeios aleatórios.

Uma forma simplificada da equação, para `N` nós e grafo com tratamento adequado de nós sem saída, é:

\[
PR(v)=\frac{1-d}{N}+d\sum_{u\to v}\frac{PR(u)}{outdeg(u)}
\]

- `d` é o fator de amortecimento (frequentemente 0,85, **não** obrigatório).
- **Grau ≠ PageRank:** muitas conexões não garantem pontuação mais alta, porque importa também de onde elas vêm.
- **PageRank ≠ relevância para uma pergunta:** é uma importância estrutural relativa ao grafo e aos parâmetros, não prova de pertinência semântica.
- Pode ser computado iterativamente até convergência.

**Se perguntarem:** "Um nó com três conexões pode ter PageRank maior que um com 500?" — **Sim, dependendo da direção, da estrutura e da importância das origens dessas conexões.** Não é garantido.

### 15.2 Louvain — detecção de comunidades

**Resposta curta:** Louvain é uma heurística que identifica grupos de vértices densamente conectados em comparação com a estrutura geral, buscando melhorar uma medida de qualidade chamada **modularidade**.

- Em geral, move vértices entre comunidades procurando aumentar a modularidade local; depois agrega comunidades para repetir o processo.
- É **não supervisionado** e não requer que os grupos correspondam a rótulos conhecidos.
- **Limitações:** resultado pode variar; maximizar modularidade não garante a "verdadeira" comunidade e pode haver limite de resolução.
- Aplicação plausível: investigar grupos de contas e relações financeiras; **comunidade não implica fraude**.

### 15.3 Node2Vec — embeddings de vértices

**Resposta curta:** Node2Vec aprende um vetor para cada vértice por meio de **caminhadas aleatórias enviesadas** no grafo e de um objetivo de predição de contexto inspirado em Skip-gram.

- Seus parâmetros `p` e `q` modulam o estilo de exploração local e mais distante nas caminhadas.
- Busca preservar padrões de vizinhança/estrutura em representações vetoriais.
- **Não é o mesmo que embedding de texto:** o sinal principal vem da **estrutura do grafo**, não necessariamente do conteúdo linguístico dos vértices.
- Pode alimentar tarefas como classificação de nós ou predição de links.

### 15.4 GNN — Graph Neural Networks

**Resposta curta:** GNN é uma **família de modelos** que aprende representações de vértices e/ou grafos agregando informações de vizinhos e combinando-as com características dos próprios vértices.

Esquema simplificado (*message passing*):

1. Cada vértice começa com características (*features*).
2. Recebe/agrega mensagens de vizinhos segundo as arestas.
3. Atualiza sua representação por uma transformação aprendida.
4. Após uma ou mais camadas, utiliza as representações na tarefa preditiva.

- **GCN:** exemplo clássico de agregação baseada em vizinhança normalizada.
- **GraphSAGE:** aprende funções de agregação que podem generalizar a nós não vistos.
- **GAT:** utiliza atenção para ponderar contribuições de vizinhos.
- Tarefas: **classificação de nós**, **predição de links** e **classificação de grafos**.
- **Limitações:** *over-smoothing* com várias camadas, escalabilidade em grafos grandes e sensibilidade à qualidade das relações e dos dados.

**Contraste importante:** uma GNN **aprende parâmetros**; BFS/DFS/Dijkstra são algoritmos de travessia/caminho mínimo, **não** redes neurais.

### 15.5 Em que ordem estudar antes da entrevista?

**Se o foco for grafos e o tempo for curto:** PageRank e medidas de centralidade → Louvain → Node2Vec → princípios de GNN (GCN/GraphSAGE/GAT). Depois revisar métodos estatísticos citados no roteiro que você não domina.

**Não confundir eficiência de recuperação com aplicação automática desses métodos.** Sua pesquisa continua em fase de hipótese; estudar PageRank, Node2Vec ou GNN não significa ter decidido usá-los.

Para uma revisão em formato pergunta e resposta, consulte **[PERGUNTAS_E_RESPOSTAS_ENTREVISTA.md](PERGUNTAS_E_RESPOSTAS_ENTREVISTA.md)**.

