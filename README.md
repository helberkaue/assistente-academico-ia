# Assistente Acadêmico Inteligente

Aplicação que lê fotos de material didático impresso (slides, apostilas, páginas de livro) e responde perguntas sobre o conteúdo, sempre mostrando o trecho do material de onde a resposta foi tirada.

Projeto Integrador Final da disciplina de Inteligência Artificial, AFYA / UNIMA Maceió. Professor: Ícaro Ferreira.

**Artigo:** _(link do Overleaf)_

## Integrantes

| Nome | Responsabilidade |
|---|---|
| Gabriel Eduardo de Oliveira Rosa | Machine Learning (classificação de matéria) |
| Ephrem Matheus Gonçalves de Melo | PLN (intenção, busca e resposta) |
| Helber Kauê Melo Avelino | Visão Computacional e integração |
| Keivid dos Santos Stanziano | Agente de decisão e artigo |

## Problema

Estudantes acumulam fotos de slides e apostilas, mas o texto de uma foto não é pesquisável. Para achar uma informação é preciso reler as imagens uma a uma. O assistente transforma a foto em texto, identifica a matéria e responde perguntas em linguagem natural usando **apenas** o conteúdo da imagem. Quando a resposta não está no material, ele diz isso em vez de inventar.

**Público-alvo:** estudantes de ensino médio e graduação.

## Arquitetura

![Arquitetura](resultados/graficos/arquitetura.png)

| Módulo | Pasta | O que faz | Técnica |
|---|---|---|---|
| Visão Computacional | `src/visao_computacional/` | Corrige a foto e extrai o texto | OpenCV (ruído, iluminação, inclinação) + EasyOCR |
| Machine Learning | `src/machine_learning/` | Identifica a matéria do texto | TF-IDF + Naive Bayes (comparado a Regressão Logística e SVM) |
| PLN | `src/pln/` | Entende a pergunta e busca o trecho | Intenção por regras, palavras-chave, busca TF-IDF com heurísticas |
| Agente de decisão | `src/decisao/` | Escolhe a ação | Agente reativo com regras R1–R5 |
| Integração | `src/assistente.py`, `app.py` | Fluxo único e interface | Streamlit |

### Fluxo de uma pergunta

1. A foto é pré-processada e lida pelo OCR, que devolve o texto e uma confiança de leitura.
2. O classificador identifica a matéria do texto.
3. O texto é dividido em frases, e cada frase guarda o título da sua seção.
4. A intenção da pergunta é reconhecida (definição, lista, resumo ou factual) e as palavras-chave são extraídas.
5. As frases são pontuadas por similaridade TF-IDF com a pergunta.
6. O agente decide a ação:

| Regra | Condição | Ação |
|---|---|---|
| R1 | Menos de 20 caracteres lidos | Pedir nova foto |
| R2 | Confiança do OCR < 30% | Pedir nova foto |
| R3 | Intenção = resumo | Resumir (3 frases mais representativas) |
| R4 | Relevância do melhor trecho < 0,30 | "Não encontrei essa informação no material" |
| R5 | Caso contrário | Responder com o trecho |

O agente também emite **avisos** quando a confiança do OCR está entre 30% e 60% e quando a pergunta parece ser de outra matéria.

## Instalação

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux / macOS
source .venv/bin/activate

pip install -r requirements.txt
```

Python 3.10 ou superior (testado no 3.13). Não precisa de GPU. Na primeira execução, o EasyOCR baixa os pesos da rede (~100 MB) para `~/.EasyOCR`.

## Como executar

```bash
# Interface (demonstração)
streamlit run app.py
```

Na interface, envie uma foto (por exemplo, uma das páginas de `dados/material_impressao/material_para_imprimir.pdf`, impressa e fotografada) e faça perguntas como *"O que é a fotossíntese?"*, *"Quais são as fases da mitose?"* ou *"Resuma o texto"*.

```bash
# Ler o texto de uma imagem pela linha de comando
python -m src.visao_computacional.ocr caminho/da/imagem.jpg

# Testes (o marcador "lento" roda o OCR de verdade)
pytest -m "not lento"
pytest
```

### Reproduzir os experimentos

```bash
python -m src.visao_computacional.experimento_ocr        # Exp. 1: OCR x pré-processamento (~30 min em CPU)
python -m src.machine_learning.coletar_dados             # baixa os parágrafos da Wikipédia (já incluídos no repositório)
python -m src.machine_learning.treinar                   # Exp. 2: classificadores de matéria
python -m src.experimento_perguntas --fonte gabarito     # Exp. 3 e 4: PLN e calibração do agente
python -m src.experimento_fotos_reais                    # Exp. 5: fotos reais
python -m src.experimento_perguntas --fonte ocr          # Exp. 5: perguntas sobre o texto das fotos
```

## Dados

| Conjunto | Origem | Uso |
|---|---|---|
| 60 imagens sintéticas | Geradas por `gerar_imagens_sinteticas.py` (semente 42) a partir de 12 textos-base, com inclinação, desfoque, ruído, sombra, baixo contraste e JPEG | Exp. 1 |
| 1.607 parágrafos, 119 artigos | Wikipédia em português (CC BY-SA 4.0), 6 matérias | Exp. 2 (treino e validação) |
| 6 páginas impressas | `dados/material_impressao/`, com os 12 textos-base | Fotos reais e perguntas |
| 51 perguntas | `dados/perguntas.csv`: 38 com resposta no material, 5 de resumo e 8 sem resposta | Exp. 3, 4 e 5 |
| Fotos reais | `dados/imagens_reais/`, celular, em diferentes condições | Exp. 5 |

## Resultados

### Exp. 1: pré-processamento no OCR (EasyOCR, 60 imagens sintéticas)

| Configuração | CER médio | CER mediana | WER médio | Tempo |
|---|---|---|---|---|
| Sem pré-processamento | 31,1% | 39,0% | 47,5% | 6,6 s |
| Iluminação + inclinação | 4,5% | 3,3% | 20,6% | 7,3 s |
| Completo (+ remoção de ruído) | **4,5%** | **2,4%** | **19,2%** | 7,5 s |
| Completo + binarização | 8,9% | 4,0% | 29,0% | 7,4 s |

![CER em função da inclinação](resultados/graficos/ocr_easyocr_cer_vs_inclinacao.png)

- **A correção de inclinação explica quase todo o ganho, pela ordem de leitura.** Acima de ~1,8° de inclinação, o CER sem pré-processamento salta para 35–60%. O EasyOCR continua reconhecendo os caracteres, mas os segmentos de linhas vizinhas se misturam ao reconstruir a ordem de leitura.
- **A binarização piora o EasyOCR**, provavelmente porque a rede foi treinada com imagens em tons de cinza. Por isso ela fica desligada por padrão.

### Exp. 2: classificação de matéria (validação cruzada de 5 partes, separada por artigo)

| Modelo | Acurácia | F1-macro |
|---|---|---|
| **Naive Bayes (palavras)** | **0,879 ± 0,028** | **0,869 ± 0,033** |
| SVM linear (caracteres) | 0,867 ± 0,040 | 0,857 ± 0,039 |
| SVM linear (palavras) | 0,863 ± 0,031 | 0,854 ± 0,030 |
| Regressão Logística (palavras) | 0,860 ± 0,029 | 0,850 ± 0,026 |

![Matriz de confusão](resultados/graficos/ml_matriz_confusao.png)

- A maior confusão é **Geografia → História** (47 parágrafos), matérias que compartilham vocabulário de território, povos e economia.
- O modelo final acertou **100%** dos 12 textos-base e do texto lido pelo OCR nas 60 imagens sintéticas, inclusive **sem pré-processamento** (CER de 31%). Os 60 textos derivam dos mesmos 12 conteúdos, então o resultado mostra robustez a erros de OCR, não generalização para 60 textos independentes.

### Exp. 3: PLN (51 perguntas sobre o texto exato das páginas)

- **Intenção:** 98,0% de acerto (50/51). O único erro foi "Quais fenômenos…", que não casa com a regra "quais são/os/as".
- **Busca** (38 perguntas com resposta):

| Método | Recall@1 | Recall@3 | MRR |
|---|---|---|---|
| **Caracteres (3–5)** | **0,895** | **1,000** | **0,943** |
| Palavras | 0,868 | 1,000 | 0,930 |

### Exp. 4: calibração do agente de decisão

![Calibração do limiar](resultados/graficos/decisao_limiar_gabarito.png)

Com o limiar **0,30**, o agente responde corretamente **89,5%** das perguntas que têm resposta e recusa **87,5%** (7/8) das que não têm, com 89,1% de acerto total.

No sistema completo, foram 34 respostas corretas, 4 erradas, 5 resumos gerados, 7 recusas corretas e 1 resposta inventada. Os erros têm três causas:
- **Heurística de anáfora exagerando:** em "O que é a mitose?", a frase seguinte, "Ela é dividida…", passa a definição.
- **Lacuna lexical:** "**Onde** ocorre a fotossíntese?" tem como resposta "**Nos cloroplastos**…", sem palavras em comum com a pergunta.
- **Coincidência de palavra:** "**Segunda** Guerra Mundial" casou com "**segunda** metade do século".

### Exp. 5: fotos reais

Foram utilizadas imagens reais de materiais didáticos
para testar o funcionamento do reconhecimento de texto (OCR).

**Imagem 1**

![Foto real 1](dados/imagens_reais/foto%1.jpg)

**Imagem 2**

![Foto real 2](dados/imagens_reais/foto%2.jpg)

**Imagem 3**

![Foto real 3](dados/imagens_reais/foto%3.jpg)

**Imagem 4**

![Foto real 1](dados/imagens_reais/foto%4.jpg)

**Imagem 5**

![Foto real 2](dados/imagens_reais/foto%5.jpg)

**Imagem 6**

![Foto real 3](dados/imagens_reais/foto%6.jpg)

## Limitações

- O escopo é material **impresso**. Manuscrito, fórmulas e diagramas não foram tratados.
- A correção de inclinação cobre ±10° no plano. Perspectiva forte não é corrigida.
- A busca é **lexical** (TF-IDF): perguntas sem palavras em comum com a resposta falham. Embeddings semânticos resolveriam parte disso.
- As heurísticas da busca e o limiar do agente foram ajustados com as mesmas 51 perguntas usadas na avaliação, então os números do Exp. 3 e 4 tendem a ser otimistas. O Exp. 5, com o texto das fotos reais, é uma verificação mais independente.
- O conjunto de perguntas é pequeno (51) e cobre um único material de 6 páginas.
- O OCR em CPU leva ~7 s por imagem.

## Estrutura do repositório

```
app.py                   interface Streamlit
src/
  assistente.py          integração dos módulos (fluxo único)
  visao_computacional/   pré-processamento, OCR, métricas, geradores de imagens, Exp. 1
  machine_learning/      coleta da Wikipédia, classificador, Exp. 2
  pln/                   texto, intenção, busca, resposta
  decisao/               agente baseado em regras
  experimento_perguntas.py   Exp. 3 e 4
  experimento_fotos_reais.py Exp. 5
dados/                   textos-base, Wikipédia, material para impressão, perguntas, fotos reais
modelos/                 classificador treinado
resultados/              tabelas (.csv), gráficos e exemplos
testes/                  testes automatizados (pytest)
artigo/                  fonte LaTeX do artigo
```

## Referências

- BAEK, Y. et al. Character Region Awareness for Text Detection. *CVPR*, 2019.
- BUADES, A.; COLL, B.; MOREL, J.-M. A non-local algorithm for image denoising. *CVPR*, 2005.
- JOACHIMS, T. Text Categorization with Support Vector Machines. *ECML*, 1998.
- LEVENSHTEIN, V. I. Binary codes capable of correcting deletions, insertions, and reversals. *Soviet Physics Doklady*, 1966.
- MANNING, C. D.; RAGHAVAN, P.; SCHÜTZE, H. *Introduction to Information Retrieval*. Cambridge University Press, 2008.
- OTSU, N. A Threshold Selection Method from Gray-Level Histograms. *IEEE Trans. SMC*, 1979.
- PEDREGOSA, F. et al. Scikit-learn: Machine Learning in Python. *JMLR*, 2011.
- RUSSELL, S.; NORVIG, P. *Artificial Intelligence: A Modern Approach*. 4. ed. Pearson, 2020.
- SALTON, G.; BUCKLEY, C. Term-weighting approaches in automatic text retrieval. *Information Processing & Management*, 1988.
- SHI, B.; BAI, X.; YAO, C. An End-to-End Trainable Neural Network for Image-based Sequence Recognition. *IEEE TPAMI*, 2017.
- SMITH, R. An Overview of the Tesseract OCR Engine. *ICDAR*, 2007.
- JaidedAI. EasyOCR. <https://github.com/JaidedAI/EasyOCR>.
- Textos de treino: Wikipédia em português, licença CC BY-SA 4.0.
