# Clarke-Wright Savings Paralelo — PPC

Projeto prático da disciplina de **Programação Paralela e Concorrente (PPC)**.

## 1. Sobre o projeto

O projeto tem como objetivo aplicar os conceitos de programação paralela e concorrente estudados na disciplina para desenvolver e avaliar uma implementação eficiente do algoritmo de economias de Clarke-Wright (*Clarke-Wright Savings*).

O problema escolhido é o roteamento de entregas urbanas. O cenário de motivação considera o aumento do volume de entregas e a necessidade de calcular rotas eficientes para uma frota de veículos, reduzindo a distância total percorrida e respeitando as restrições operacionais.

A motivação utiliza o Mercado Livre como exemplo de contexto real. O trabalho acadêmico, entretanto, será delimitado inicialmente a uma versão capacitada do Problema de Roteamento de Veículos, sem assumir que todos os aspectos de uma operação logística real serão modelados.

## 2. Problema e algoritmo escolhidos

### 2.1 Problema

Será abordado o **Capacitated Vehicle Routing Problem (CVRP)**, ou Problema de Roteamento de Veículos com Capacidade.

O cenário inicial considera:

- um depósito, identificado pelo nó `0`;
- um conjunto de clientes, cada um com uma demanda;
- uma frota com quantidade fixa de veículos;
- veículos com capacidade máxima definida;
- uma matriz de distâncias entre o depósito e os clientes;
- atendimento de cada cliente exatamente uma vez;
- respeito à capacidade dos veículos e à quantidade disponível de veículos.

O objetivo é **minimizar a soma das distâncias percorridas por todas as rotas**, incluindo os deslocamentos de saída do depósito e de retorno a ele.

A distância máxima por rota não foi definida como restrição obrigatória da primeira versão. Poderá ser considerada posteriormente, caso seja útil para os experimentos.

### 2.2 Algoritmo

O algoritmo escolhido é o **Clarke-Wright Savings**, utilizando a variante conhecida como *Parallel Savings*.

Apesar do termo *Parallel* no nome da variante, isso não significa que a implementação será executada com threads ou processos. A variante clássica calcula as economias dos pares de clientes, ordena os resultados e percorre a lista global para decidir quais rotas podem ser unidas. A execução concorrente será investigada separadamente como parte do trabalho de PPC.

Para cada par de clientes distintos `i` e `j`, a economia é calculada por:

\[
S_{ij} = d_{0i} + d_{0j} - d_{ij}
\]

Em que:

- `0` representa o depósito;
- `d_0i` é a distância do depósito ao cliente `i`;
- `d_0j` é a distância do depósito ao cliente `j`;
- `d_ij` é a distância entre os clientes `i` e `j`.

As economias são ordenadas em ordem decrescente. Em seguida, o algoritmo avalia se cada par permite unir duas rotas, considerando as extremidades das rotas e as restrições do problema.

O Clarke-Wright é uma heurística: pode produzir uma solução viável sem garantir que ela seja a solução ótima global.

## 3. Objetivos do trabalho

O trabalho pretende:

1. implementar uma versão sequencial de referência em C++;
2. desenvolver uma estrutura de código que permita comparar diferentes estratégias de paralelização;
3. identificar as etapas do algoritmo com potencial de execução paralela;
4. reduzir o tempo de execução sem comprometer a correção da solução;
5. analisar o impacto do número de threads e, quando aplicável, de outras configurações de execução;
6. comparar o desempenho das versões sequencial e paralelas por meio de experimentos reproduzíveis;
7. documentar e justificar as decisões de projeto com base nos conceitos estudados na disciplina.

## 4. Tecnologias e limites do projeto

### Decisões confirmadas

- **Linguagem:** C++.
- **Ambiente inicial:** um computador multicore, com memória compartilhada.
- **Bibliotecas de paralelização:** serão escolhidas entre as estudadas na disciplina, como OpenMP e POSIX Threads. A escolha será justificada tecnicamente e pelos resultados experimentais.
- **Entrada:** matriz de distâncias fornecida como parte da instância, além das demandas, da capacidade dos veículos e da quantidade de veículos.
- **Referência de desempenho:** implementação sequencial do mesmo algoritmo.

MPI e outras abordagens estudadas poderão ser consideradas se houver justificativa e condições experimentais adequadas. Ainda não foi decidido implementar uma versão distribuída.

Não está prevista a adoção de bibliotecas de paralelização externas ao conteúdo trabalhado na disciplina. Bibliotecas padrão da linguagem poderão ser utilizadas para estruturas de dados, ordenação, entrada e saída.

## 5. Arquitetura inicial proposta

O programa será dividido em componentes para facilitar os testes, a medição de desempenho e a comparação das estratégias.

### 5.1 Leitura e validação da entrada

Responsável por ler e validar os dados da instância:

- quantidade de clientes;
- quantidade de veículos;
- capacidade dos veículos;
- demanda de cada cliente;
- matriz de distâncias.

### 5.2 Cálculo das economias

Calcula as economias de todos os pares distintos de clientes. Para `n` clientes, considerando pares não ordenados e sem incluir pares de um cliente consigo mesmo, são calculadas `n(n - 1) / 2` economias.

Essa etapa é uma candidata natural à paralelização, pois o cálculo de cada par pode ser feito independentemente. A implementação paralela deverá evitar escritas concorrentes na mesma posição de memória.

### 5.3 Ordenação das economias

Ordena as economias em ordem decrescente. Será definido um critério determinístico de desempate, para favorecer a reprodutibilidade dos resultados.

Inicialmente, a ordenação poderá utilizar um algoritmo da biblioteca padrão do C++. Uma implementação paralela de ordenação só será considerada se os experimentos indicarem que essa etapa representa um custo relevante.

### 5.4 Construção das rotas

Percorre a lista ordenada e avalia as uniões possíveis. Uma união só poderá ser aplicada quando:

- os clientes pertencerem a rotas diferentes;
- os clientes ocuparem extremidades compatíveis para a união;
- a capacidade da rota resultante não for excedida;
- a união respeitar as demais condições do algoritmo.

Como cada união altera o estado das rotas, essa etapa apresenta dependências que tornam a paralelização mais delicada. A primeira versão será sequencial e servirá de referência para futuras otimizações.

### 5.5 Validação e apresentação dos resultados

Ao final, o programa verificará se:

- todos os clientes foram atendidos exatamente uma vez;
- nenhuma rota excede a capacidade;
- a quantidade de rotas não excede a frota disponível;
- a distância total foi calculada corretamente.

Se a heurística não produzir uma solução viável, o programa deverá informar isso explicitamente. A ausência de uma solução viável encontrada pela heurística não prova, por si só, que a instância do CVRP seja matematicamente inviável.

## 6. Estratégia de paralelização

A estratégia definitiva ainda será estabelecida após a implementação de referência e a análise dos custos de cada etapa.

As principais possibilidades a investigar são:

1. **Paralelização do cálculo das economias:** distribuir os pares de clientes entre threads.
2. **Balanceamento do trabalho:** avaliar a distribuição dos pares e o custo de criação e coordenação das threads.
3. **Ordenação:** medir o custo da ordenação e verificar se uma alternativa paralela se justifica.
4. **Construção das rotas:** investigar representações de dados e formas de reduzir verificações desnecessárias, preservando a validade das uniões.
5. **Sincronização:** avaliar o custo de mutexes, operações atômicas, barreiras ou outros mecanismos estudados, apenas quando necessários à estratégia escolhida.

A paralelização não será considerada bem-sucedida apenas por utilizar várias threads. Será necessário demonstrar que ela reduz o tempo de execução em condições comparáveis e verificar o impacto sobre a qualidade e a validade das rotas.

Uma implementação concorrente que altere a ordem de aceitação das economias poderá produzir rotas diferentes da versão sequencial. Nesse caso, além do tempo, será necessário comparar a distância total e a viabilidade das soluções.

## 7. Plano de implementação

O desenvolvimento será conduzido de forma incremental:

1. **Definir o formato da entrada** e criar instâncias pequenas para testes.
2. **Implementar a versão sequencial** do Clarke-Wright *Parallel Savings*.
3. **Criar testes de correção** para cálculo de economias, construção de rotas, capacidade, frota e distância total.
4. **Medir o desempenho sequencial**, incluindo o tempo total e, quando possível, o tempo das etapas individuais.
5. **Implementar estratégias paralelas** usando recursos estudados na disciplina.
6. **Comparar e otimizar as implementações**, investigando gargalos e overhead.
7. **Avaliar instâncias maiores e bases públicas**, após validar o funcionamento com entradas sintéticas.
8. **Documentar os experimentos** e preparar o relatório em formato de artigo.

## 8. Avaliação experimental

Os experimentos deverão permitir comparar a implementação sequencial e as versões paralelas em diferentes tamanhos de entrada e configurações de execução.

### Métricas previstas

- **Tempo de execução:** média aritmética de 10 execuções por configuração.
- **Intervalo de confiança de 95%:** para expressar a incerteza da estimativa do tempo médio.
- **Speedup:** razão entre o tempo sequencial de referência e o tempo paralelo.
- **Eficiência paralela:** speedup dividido pelo número de unidades de processamento utilizadas, respeitando a definição adotada para cada experimento.
- **Fração serial e métrica de Karp-Flatt:** para estimar o efeito combinado da parte serial e dos overheads paralelos.
- **Qualidade da solução:** distância total e viabilidade das rotas produzidas.

Também serão registrados, quando disponíveis, o processador, a quantidade de núcleos e threads de hardware, a memória, o sistema operacional, o compilador, as opções de compilação e as configurações usadas nos testes.

Para que as comparações sejam válidas, as versões deverão utilizar as mesmas instâncias e condições equivalentes. O protocolo de experimentos será detalhado antes da coleta definitiva dos resultados.

## 9. Dados de entrada

A primeira etapa utilizará instâncias sintéticas pequenas e controladas, criadas para verificar a correção do algoritmo. Depois, serão avaliadas instâncias maiores e, se possível, conjuntos públicos de referência para CVRP.

O formato de arquivo ainda será definido. A estrutura deverá representar, no mínimo:

- quantidade de clientes;
- quantidade de veículos;
- capacidade dos veículos;
- demanda de cada cliente;
- matriz de distâncias.

O leitor de entrada deverá ser separado da lógica do algoritmo para facilitar a adoção de diferentes formatos de instância futuramente.

## 10. Decisões tomadas e pontos em aberto

### Decisões tomadas

- Problema de roteamento inspirado no cenário de entregas urbanas.
- CVRP capacitado como variante inicial.
- Clarke-Wright Savings, variante *Parallel Savings*.
- Minimização da soma das distâncias percorridas.
- C++ como linguagem de implementação.
- Matriz de distâncias fornecida na entrada.
- Frota com quantidade fixa de veículos.
- Execução inicial em computador multicore.
- Implementação sequencial como referência e estrutura preparada para comparar estratégias paralelas.
- Uso de bibliotecas de paralelização estudadas na disciplina, com justificativa técnica.

### Pontos em aberto

- Formato exato da entrada.
- Tratamento de veículos com capacidades iguais ou diferentes; a primeira versão deverá explicitar a hipótese adotada.
- Inclusão ou não de uma distância máxima por rota.
- Biblioteca e estratégia de paralelização definitivas.
- Estrutura de dados utilizada para representar e unir rotas.
- Compilador, opções de compilação e ambiente de medição.
- Tamanhos das instâncias e conjuntos públicos que serão utilizados.
- Protocolo final de experimentos e análise estatística.
- Possível comparação de qualidade entre estratégias que produzam rotas diferentes.

## 11. Referências

As referências bibliográficas sobre o algoritmo Clarke-Wright, o CVRP e as estratégias de paralelização serão selecionadas e verificadas durante a elaboração do referencial teórico e dos trabalhos relacionados. As fontes efetivamente utilizadas deverão ser citadas no relatório final.

## 12. Status

-[X] **Etapa 1 — Definição do grupo e escolha do problema:** concluída.

-[ ] **Etapa 2 — Implementação e avaliação da solução paralela:** em planejamento. As decisões de escopo estão registradas neste documento; a implementação ainda será desenvolvida.

-[ ] **Etapa 3 — Relatório em formato de artigo:** a ser desenvolvido com base na implementação e nos resultados experimentais.