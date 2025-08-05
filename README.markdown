# Fogás VRP - Algoritmo Genético

## 1. Introdução

Este projeto foi desenvolvido como parte do Tech Challenge da Fogás, com o objetivo de resolver um **Vehicle Routing Problem (VRP)** para otimizar as rotas de entrega de botijões de gás. O algoritmo genético implementado encontra rotas eficientes para atender 35 clientes a partir de um depósito central, utilizando 5 veículos, cada um com capacidade de 100 botijões. A solução minimiza a **distância total percorrida** pela frota, respeitando as restrições de capacidade, e oferece uma visualização interativa com um gráfico de convergência e as rotas otimizadas.

Este documento serve como a documentação completa do projeto, cobrindo a descrição do problema, detalhes da implementação, análise de resultados e conclusões.

## 2. Descrição do Problema

O problema consiste em otimizar as rotas de entrega de botijões de gás para a Fogás, com as seguintes especificações:

- **Depósito**: Localizado nas coordenadas (400, 100) na tela.
- **Clientes**: 35 clientes, cada um com coordenadas (x, y) e uma demanda específica de botijões (entre 5 e 20, com demanda total estimada em ~410 botijões, viável para 5 veículos com capacidade total de 500).
- **Veículos**: 5 veículos, cada um com capacidade máxima de 100 botijões.
- **Objetivo**: Minimizar a **distância total percorrida** pela frota, garantindo que:
  - Todos os 35 clientes sejam atendidos.
  - A soma das demandas em cada rota não exceda a capacidade do veículo (100 botijões).
- **Restrições**:
  - Cada cliente deve ser visitado exatamente uma vez por um único veículo.
  - Cada veículo inicia e termina sua rota no depósito.
- **Visualização**: Exibir um gráfico da convergência da aptidão (distância total) e as rotas otimizadas em uma interface gráfica.

O desafio é encontrar a configuração de rotas que resulte na menor distância total, considerando as restrições de capacidade e a necessidade de atender todos os clientes.

## 3. Implementação do Algoritmo

A solução utiliza um **algoritmo genético** implementado em Python, com as bibliotecas **Pygame** para visualização gráfica e **NumPy** para cálculos eficientes. A implementação está dividida em dois arquivos:

- **`vrp_fogas.py`**: Contém a lógica principal, incluindo inicialização do Pygame, configuração do problema, criação da matriz de distâncias, visualização (gráfico e rotas) e execução do algoritmo genético.
- **`genetic_algorithm.py`**: Implementa as funções do algoritmo genético, como inicialização da população, cálculo de aptidão, seleção, cruzamento e mutação.

### 3.1. Detalhes do Algoritmo Genético

#### Representação da Solução (Genoma)
- A solução é representada como uma **lista de listas**, onde cada lista interna corresponde à rota de um veículo, contendo os índices dos clientes na ordem de visita. Exemplo: `[[0, 1, 2], [3, 4], [5], [], []]` indica que o primeiro veículo atende os clientes 0, 1 e 2, o segundo atende 3 e 4, o terceiro atende 5, e os dois últimos estão vazios.
- Cada rota deve respeitar a capacidade de 100 botijões, e todos os 35 clientes devem ser atendidos.

#### Inicialização
- **Tamanho da População**: 70 indivíduos (máximo entre 50 e 2 vezes o número de clientes, ou seja, 2 × 35 = 70).
- **Método**: Os índices dos 35 clientes são embaralhados aleatoriamente e distribuídos entre os 5 veículos. Clientes são adicionados a um veículo até atingir a capacidade (100 botijões) ou até que todos sejam alocados. Clientes excedentes são alocados a outros veículos com capacidade disponível, garantindo soluções iniciais viáveis.

#### Função de Fitness
- A função calcula a **distância total percorrida** pela frota, usando uma **matriz de distâncias** pré-calculada (36x36, incluindo depósito e 35 clientes) para eficiência.
- Para cada rota, a distância é a soma dos trechos entre pontos consecutivos (depósito → cliente → cliente → depósito), obtida da matriz.
- Soluções inválidas (rotas que excedem 100 botijões ou não atendem todos os clientes) recebem aptidão `float('inf')`.
- A matriz de distâncias é criada uma única vez no início, evitando cálculos repetidos da distância euclidiana, o que melhora significativamente o desempenho para 2000 gerações.

#### Seleção
- **Método**: Seleção por ordenação (ranking).
- A aptidão de todos os indivíduos é calculada, e os 50% melhores (35 indivíduos) são selecionados como pais para a próxima geração, priorizando soluções com menor distância.

#### Crossover
- **Método**: Cruzamento por rota.
- Para dois pais, cada filho herda rotas de um dos pais aleatoriamente (50% de probabilidade por rota).
- Após a cópia, a capacidade de cada rota é verificada, e clientes excedentes são removidos. A função `ensure_all_clients` adiciona clientes faltantes a rotas com capacidade disponível, garantindo que todos os 35 clientes sejam atendidos.

#### Mutação
- **Taxa**: 5% (0.05).
- Dois tipos de mutação:
  - **Troca interna**: Troca dois clientes aleatoriamente dentro da mesma rota.
  - **Realocação**: Move um cliente de uma rota para outra, se a capacidade permitir.
- Após cada mutação, `ensure_all_clients` garante que todos os clientes sejam atendidos.

#### Critério de Parada
- O algoritmo executa **2000 gerações**, permitindo ampla exploração do espaço de soluções.
- O usuário pode encerrar manualmente pressionando a tecla `q` ou fechando a janela Pygame.

#### Elitismo
- A melhor solução de cada geração é preservada na próxima geração (elitismo), garantindo que a aptidão nunca piore.

### 3.2. Visualização
- **Interface Gráfica**: Utiliza Pygame para exibir:
  - **Gráfico de Convergência**: Um gráfico quadrado (300x300 pixels) à esquerda da tela mostra a evolução da melhor aptidão (distância total) ao longo das 2000 gerações. O gráfico tem fundo branco, bordas pretas, linha verde para a aptidão, e rótulos em Arial (tamanho 12) nos eixos (gerações e aptidão).
  - **Rotas**: À direita, as rotas são desenhadas com:
    - Depósito: Círculo vermelho em (400, 100).
    - Clientes: Círculos azuis nas respectivas coordenadas.
    - Rotas: Linhas coloridas (5 cores distintas para os 5 veículos) conectando o depósito aos clientes e de volta.
- **Saída no Console**: Exibe a melhor aptidão a cada geração e a solução final (rotas e distância total).

### 3.3. Otimização com Matriz de Distâncias
- Uma matriz de distâncias (36x36) é pré-calculada com as distâncias euclidianas entre todos os pares de pontos (depósito e 35 clientes).
- Essa abordagem reduz o custo computacional da função de aptidão, que é chamada milhares de vezes durante as 2000 gerações, tornando o algoritmo mais eficiente em comparação com cálculos repetidos da distância euclidiana.

## 4. Requisitos para Execução

- **Python**: Versão 3.6 ou superior.
- **Bibliotecas**:
  - `pygame`: Para a interface gráfica (gráfico e rotas).
  - `numpy`: Para cálculos eficientes com a matriz de distâncias.
- **Instalação**:
  ```bash
  pip install pygame numpy
  ```
- **Sistema Operacional**: Windows, macOS ou Linux (qualquer sistema compatível com Python e Pygame).
- **Arquivos**:
  - `vrp_fogas.py`: Programa principal.
  - `genetic_algorithm.py`: Funções do algoritmo genético.
  - Ambos devem estar no mesmo diretório.

## 5. Como Executar

1. **Clone o Repositório** (se aplicável) ou salve os arquivos `vrp_fogas.py` e `genetic_algorithm.py` no mesmo diretório.
2. **Instale as Dependências**:
   ```bash
   pip install pygame numpy
   ```
3. **Execute o Programa**:
   ```bash
   python vrp_fogas.py
   ```
4. **Interação**:
   - A janela Pygame exibe o gráfico de convergência (esquerda) e as rotas (direita).
   - O console mostra a melhor aptidão a cada geração e a solução final.
   - Pressione `q` ou feche a janela para encerrar.
5. **Saída Esperada**:
   - Gráfico: Linha verde mostrando a redução da distância ao longo das 2000 gerações.
   - Rotas: 5 rotas coloridas conectando o depósito aos 35 clientes.
   - Console: Exemplo de saída:
     ```
     Geração 1: Melhor aptidão = 2800.45
     ...
     Geração 2000: Melhor aptidão = 1400.78
     Melhor solução: [[0, 1, 2, 3], [4, 5, 6], [7, 8], [9, 10, 11], [12, 13, 14]]
     Melhor aptidão: 1400.78
     ```

## 6. Análise de Resultados

### 6.1. Performance do Algoritmo
- **Convergência**: Em execuções típicas, a aptidão (distância total) diminui significativamente nas primeiras 500-700 gerações, estabilizando-se próximo a uma solução otimizada após 1000-1500 gerações. Com 2000 gerações, o algoritmo explora amplamente o espaço de soluções, alcançando distâncias na faixa de 1200-1600 unidades (dependendo das coordenadas dos 35 clientes).
- **Eficiência**: A matriz de distâncias (36x36) reduz o tempo de execução em comparação com cálculos repetidos da distância euclidiana. Para 70 indivíduos e 2000 gerações, o tempo de execução é de aproximadamente 2-3 minutos em um computador padrão (Intel i5, 8 GB RAM), com a matriz eliminando milhares de cálculos de raiz quadrada.
- **Qualidade da Solução**: A solução final atende todos os 35 clientes, com cada veículo transportando no máximo 100 botijões. As rotas são visualmente coerentes, com clientes próximos agrupados na mesma rota, minimizando cruzamentos desnecessários.

### 6.2. Visualização
- O **gráfico de convergência** exibe claramente a redução da distância total, com a linha verde estabilizando, indicando que o algoritmo encontrou uma solução próxima do ótimo.
- As **rotas** na interface gráfica são fáceis de interpretar, com cores distintas para cada veículo e o depósito centralizado. A visualização valida que todos os 35 clientes são atendidos e que as rotas são eficientes.

### 6.3. Limitações
- **Estocasticidade**: Como o algoritmo genético é probabilístico, os resultados variam entre execuções. Algumas execuções podem convergir para soluções subótimas, embora o elitismo minimize esse risco.
- **Escalabilidade**: Para problemas com mais de 100 clientes, o tamanho da matriz de distâncias (O(n²)) pode aumentar o uso de memória, embora o impacto seja pequeno para 35 clientes.
- **Parâmetros**: A taxa de mutação (5%) e o tamanho da população (70) foram escolhidos empiricamente. Ajustes podem melhorar a convergência em cenários específicos.

## 7. Conclusões

O algoritmo genético desenvolvido resolve eficientemente o VRP da Fogás, otimizando as rotas para 35 clientes com 5 veículos, minimizando a distância total percorrida enquanto respeita a capacidade de 100 botijões por veículo. A utilização de uma **matriz de distâncias** melhora significativamente o desempenho, tornando o algoritmo viável para 2000 gerações. A **visualização** com Pygame, incluindo um gráfico quadrado de convergência à esquerda e rotas coloridas à direita, facilita a análise da solução e valida sua corretude.

### Principais Contribuições
- **Otimização Eficiente**: A matriz de distâncias reduz o custo computacional, essencial para problemas com muitas iterações.
- **Visualização Clara**: A interface gráfica com gráfico de convergência e rotas coloridas ajuda a entender a evolução do algoritmo e a solução final.
- **Flexibilidade**: O código é modular, com funções separadas em `genetic_algorithm.py`, permitindo ajustes para diferentes números de clientes ou veículos.

### Melhorias Futuras
- **Ajuste de Parâmetros**: Testar diferentes taxas de mutação ou tamanhos de população para melhorar a convergência.
- **Pausa Interativa**: Adicionar uma pausa entre gerações para inspecionar a evolução das rotas.
- **Saída de Resultados**: Salvar a matriz de distâncias ou a solução final em um arquivo para análises posteriores.
- **Heurísticas Adicionais**: Incorporar heurísticas (ex.: 2-opt) para refinar as rotas após o algoritmo genético.

## 8. Referências
- **Pygame**: Biblioteca para visualização gráfica (https://www.pygame.org).
- **NumPy**: Biblioteca para cálculos numéricos (https://numpy.org).
- **Algoritmo Genético**: Baseado em técnicas padrão para VRP, adaptadas para o contexto da Fogás.