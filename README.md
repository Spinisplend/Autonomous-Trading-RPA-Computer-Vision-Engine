# Autonomous Trading RPA & Computer Vision Engine

> **Nota de Engenharia e Segurança:** Este repositório documenta a arquitetura de um agente de automação de processos (RPA) para terminais financeiros. Por conter lógicas de execução proprietárias e credenciais sensíveis, o código-fonte original é mantido em repositório privado.

## Visão Geral
O projeto consiste em um motor de automação robótica (RPA) especializado na replicação de ordens em terminais de trading (Copy Trading). Diferente de integrações via API tradicional, este sistema opera via Visão Computacional, sendo capaz de interagir com interfaces gráficas legadas ou protegidas, interpretando dados visuais em tempo real para tomada de decisão e execução de ordens.

## Stack Tecnológica
* **Linguagem:** Python
* **Visão Computacional:** OpenCV (Template Matching), PyAutoGUI
* **OCR (Optical Character Recognition):** Tesseract OCR (com pré-processamento via PIL)
* **Manipulação de Imagem:** Pillow (Resizing, Grayscale, Binarização)

## Arquitetura e Soluções de Engenharia

### 1. Processamento de Imagem e Otimização de OCR
A leitura de dados financeiros diretamente da tela apresenta desafios de fidelidade. 
* **Pipeline de Tratamento:** Antes de submeter o fragmento da tela ao Tesseract, o motor executa um redimensionamento via interpolação Lanczos (3x) e aplica um limiar de binarização (Thresholding). Isso elimina artefatos de antialiasing da interface, elevando drasticamente a acurácia do reconhecimento de strings numéricas de volumes e IDs.

### 2. Template Matching e Navegação Heurística
O robô localiza elementos de interface (botões, rótulos e campos) utilizando algoritmos de correlação de padrões.
* **Busca por Região:** Para otimizar a performance e evitar falsos positivos, o sistema limita a área de busca dinamicamente (Bounding Boxes) com base em âncoras visuais detectadas anteriormente (como o cabeçalho da lista de ativos).

### 3. Lógica de Fechamento Inteligente (Spatial Logic)
Um dos maiores desafios técnicos resolvidos foi a correlação entre uma ordem específica e seu respectivo botão de fechamento em listas dinâmicas.
* **Solução:** O sistema realiza o parsing de toda a grade de ordens via OCR (`image_to_data`), extrai as coordenadas (x, y, w, h) do ticket desejado e projeta um vetor de busca lateral para localizar o elemento interativo de fechamento ("X") na mesma linha lógica, garantindo que a ordem correta seja encerrada mesmo com a lista em constante atualização.

### 4. Gerenciamento de Estado e Ciclo de Vida (Monitoring)
* **State Persistence:** O robô mantém um dicionário em memória (`trades_mapeadas`) para rastrear o vínculo entre a "Trade Fonte" e a "Trade Copiada".
* **Ciclo de Monitoramento:** Implementação de um loop assíncrono que diferencia estados de tela (Login vs. Dashboard), tratando exceções de rede e expiração de sessão de forma autônoma.
