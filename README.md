# Tradução Automática com D2L

Notebook baseado na Seção 9.5 do *Dive into Deep Learning* (PyTorch) com foco em download, preparação e análise do dataset `fra-eng`. Todo o fluxo roda tanto no Google Colab quanto localmente.

Notebook principal: `traducao_automatica.ipynb`.

## Estrutura do notebook

1. **Download robusto e extração** – cria a pasta `data/`, baixa `fra-eng.zip` de múltiplos espelhos com tentativas e extrai `fra.txt`.
2. **Pré-processamento e tokenização** – normaliza texto, remove pontuação e separa sentenças em tokens.
3. **Vocabulário e DataLoader** – constrói vocabulários fonte/alvo, aplica padding/truncamento e devolve `DataLoader` PyTorch com `(X, X_valid_len, Y, Y_valid_len)`.
4. **Smoke test** – executa um minibatch para validar shapes e comprimentos válidos.
5. **Experimento opcional** – reproduz a seção 9.5.7 com variação de `num_examples` e gráfico do crescimento dos vocabulários.

## Como executar no Google Colab

1. Abra o link direto: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://drive.google.com/file/d/1zX7bbXJGRAIss037IDRH7PDIzRczx599/view?usp=sharing).
2. Vá em **Runtime → Run all** (Executar tudo).
3. O notebook cria automaticamente a pasta `data/`, baixa o dataset, gera vocabulários e retorna o `DataLoader`.

## Como executar localmente

1. Clone este repositório:
   ```bash
   git clone git@github.com:ColettoG/pond_traducao_automatica_d2l.git
   cd pond_traducao_automatica_d2l
   ```
2. Prepare um ambiente Python 3.10+ e instale as dependências mínimas:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # (Windows: .venv\Scripts\activate)
   pip install torch requests numpy matplotlib
   ```
3. Abra o notebook com Jupyter ou VS Code e execute todas as células:
   ```bash
   jupyter notebook
   ```
   O download do dataset e a geração do `DataLoader` acontecem automaticamente.

## Saída esperada

- `train_iter`: `DataLoader` PyTorch com os lotes `(X, X_valid_len, Y, Y_valid_len)`.
- `src_vocab` e `tgt_vocab`: vocabulários mapeando tokens ↔ índices.
- Gráfico opcional mostrando o crescimento dos vocabulários conforme `num_examples`.

## Perguntas e Respostas



1. Tente valores diferentes do argumento num_examples na funçãoload_data_nmt. Como isso afeta os tamanhos do vocabulário do idioma de origem e do idioma de destino?

- À medida que você aumenta num_examples em load_data_nmt, os tamanhos dos vocabulários de origem (inglês) e de destino (francês) crescem de forma monotonicamente não-decrescente. O crescimento tende a desacelerar conforme mais sentenças são incluídas (efeito de “retornos decrescentes”), porque muitos tokens frequentes já apareceram cedo; os novos exemplos contribuem, em média, com tokens cada vez mais raros, vários dos quais podem ficar abaixo do limiar min_freq=2 e virar `<unk>`. Isso vale para ambos os idiomas pela mesma razão: tokenização em nível de palavra + filtro de frequência mínima. (A própria seção destaca que vocabulários em nível de palavra crescem bem mais do que em nível de caractere.  

2. O texto em alguns idiomas, como chinês e japonês, não tem indicadores de limite de palavras (por exemplo, espaço). A tokenização em nível de palavra ainda é uma boa ideia para esses casos? Por que ou por que não?
- A tokenização em nível de palavra não é adequada para idiomas como chinês e japonês, já que esses idiomas não utilizam espaços para separar palavras. Nesses casos, simplesmente dividir o texto por espaços não funciona. Uma alternativa seria usar segmentadores lexicais específicos, como Jieba para o chinês e MeCab para o japonês. Porém, a abordagem mais adotada atualmente em tradução automática é a tokenização em subwords, como BPE (Byte Pair Encoding) ou SentencePiece, que não dependem de segmentação prévia e reduzem problemas com palavras raras ou fora do vocabulário.  
Pesquisas de Sennrich et al. (2016) mostraram que o uso de subwords melhora a qualidade da tradução ao lidar melhor com palavras raras, enquanto Kudo e Richardson (2018) apresentaram o SentencePiece, um tokenizador independente de idioma que pode treinar diretamente em texto cru e obteve bons resultados em traduções envolvendo japonês. Assim, para chinês e japonês, a melhor prática é usar subwords em vez de palavras inteiras, garantindo mais robustez e generalização nos modelos de tradução automática.


## Referências

- *Dive into Deep Learning*, Capítulo 9 – Tradução Automática.
- Dataset paralelo Inglês-Francês disponível nos espelhos da AWS usados pela D2L.
