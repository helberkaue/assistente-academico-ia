# Assistente Acadêmico Inteligente

Aplicação que transforma fotografias de materiais didáticos impressos em conteúdo pesquisável. O estudante envia uma imagem de um slide, apostila ou página de livro e faz perguntas em linguagem natural. O sistema extrai o texto, identifica a matéria e procura respostas no próprio material, apresentando o trecho utilizado. Quando não encontra informação suficientemente relevante, pode informar que o conteúdo não foi localizado, em vez de completar a resposta com informações externas.

Projeto Integrador Final da disciplina de Inteligência Artificial — AFYA / UNIMA Maceió.  
Professor: Ícaro Ferreira.

**Repositório:** <https://github.com/helberkaue/assistente-academico-ia>  
**Artigo:** <https://www.overleaf.com/project/6affaef395a4b47f2ecba948/share#e1163ee1d7ff7d2bd0d91e642d679afac5ecdb7e6beda45b>

**Público-alvo:** estudantes do ensino médio e da graduação.

## Integrantes e responsabilidades

| Integrante | Responsabilidade principal |
|---|---|
| Gabriel Eduardo de Oliveira Rosa | Machine Learning — classificação de matéria |
| Ephrem Matheus Gonçalves de Melo | Processamento de Linguagem Natural — intenção, busca e resposta |
| Helber Kauê Melo Avelino | Visão Computacional e integração dos módulos |
| Keivid dos Santos Stanziano | Agente de decisão e elaboração do artigo |

## Problema e objetivo

Estudantes acumulam fotografias de slides, apostilas e páginas de livros, mas o conteúdo dessas imagens não é diretamente pesquisável. Para localizar uma informação, muitas vezes é necessário abrir e reler as imagens uma a uma.

O projeto busca facilitar essa consulta: a partir de uma fotografia, o sistema extrai o texto, identifica a matéria e permite que o usuário faça perguntas sobre o conteúdo reconhecido. As respostas são baseadas no material enviado. Quando a informação não é encontrada com relevância suficiente, o agente pode recusar a resposta, reduzindo o risco de apresentar conteúdo que não está na imagem.

## Funcionalidades

- Envio de imagens nos formatos JPG, JPEG e PNG.
- Pré-processamento para reduzir efeitos de iluminação, ruído e inclinação.
- Extração de texto por reconhecimento óptico de caracteres (OCR) com EasyOCR.
- Identificação automática da matéria por um modelo de Machine Learning.
- Reconhecimento da intenção da pergunta, como definição, lista, resumo ou pergunta factual.
- Recuperação de frases e trechos relacionados à pergunta.
- Resposta baseada no texto extraído, geração de resumo ou solicitação de uma nova fotografia.
- Recusa quando a informação solicitada não apresenta relevância suficiente no material enviado.
- Exibição do texto reconhecido, da matéria prevista e das pontuações de confiança na interface.

## Tecnologias utilizadas

| Tecnologia | Aplicação |
|---|---|
| Python | Linguagem principal |
| OpenCV | Processamento e melhoria das imagens |
| EasyOCR | Reconhecimento de texto nas fotografias |
| scikit-learn | Vetorização TF-IDF e modelos de classificação |
| Naive Bayes, SVM e Regressão Logística | Treinamento e comparação de classificadores |
| TF-IDF | Recuperação lexical de trechos relacionados à pergunta |
| Streamlit | Interface web da aplicação |
| pandas, NumPy e Matplotlib | Manipulação dos dados, análise e geração de gráficos |
| pytest | Testes automatizados |

## Arquitetura

![Diagrama da arquitetura do sistema](resultados/graficos/arquitetura.png)

| Módulo | Localização | Responsabilidade | Técnica principal |
|---|---|---|---|
| Visão Computacional | `src/visao_computacional/` | Pré-processar a imagem e extrair o texto | OpenCV + EasyOCR |
| Machine Learning | `src/machine_learning/` | Classificar o texto de acordo com a matéria | TF-IDF + Naive Bayes; comparação com SVM e Regressão Logística |
| Processamento de Linguagem Natural | `src/pln/` | Reconhecer a intenção, extrair palavras-chave e recuperar trechos | Regras, TF-IDF e heurísticas |
| Agente de decisão | `src/decisao/` | Escolher a ação a partir das condições definidas | Agente reativo baseado em regras R1–R5 |
| Integração | `src/assistente.py` | Coordenar o funcionamento dos módulos | Fluxo integrado |
| Interface | `app.py` | Permitir o envio da imagem e a interação com o usuário | Streamlit |

### Fluxo de funcionamento

1. O usuário envia uma fotografia do material didático.
2. A imagem passa por etapas de pré-processamento, que podem corrigir iluminação, ruído e inclinação.
3. O EasyOCR identifica o texto e fornece uma medida de confiança da leitura.
4. O classificador estima a matéria relacionada ao texto extraído.
5. O conteúdo é dividido em frases e seções para permitir a busca.
6. O módulo de PLN identifica a intenção da pergunta e recupera os trechos lexicalmente mais relevantes usando TF-IDF.
7. O agente de decisão seleciona entre responder, resumir, solicitar outra fotografia ou informar que não encontrou a informação.

### Regras do agente de decisão

| Regra | Condição | Ação |
|---|---|---|
| R1 | Menos de 20 caracteres foram reconhecidos | Solicitar uma nova fotografia |
| R2 | Confiança média do OCR inferior a 30% | Solicitar uma nova fotografia |
| R3 | A intenção identificada é resumo | Gerar um resumo com três frases representativas |
| R4 | Similaridade do melhor trecho inferior a 0,30 | Informar que a informação não foi encontrada |
| R5 | Nenhuma das condições anteriores se aplica | Responder com base no trecho recuperado |

O sistema também pode emitir avisos quando a confiança do OCR está entre 30% e 60% ou quando a pergunta aparenta tratar de uma matéria diferente daquela identificada no material.

## Requisitos

- Python 3.10 ou superior. O projeto foi testado com Python 3.13.
- Git para clonar o repositório.
- Conexão com a internet na primeira execução para que o EasyOCR possa baixar os pesos do modelo, caso ainda não estejam disponíveis localmente.
- GPU não é obrigatória. O arquivo `requirements.txt` configura a instalação do PyTorch para CPU.

## Instalação

Clone o repositório e entre na pasta do projeto:

```bash
git clone https://github.com/helberkaue/assistente-academico-ia.git
cd assistente-academico-ia
```

Crie e ative um ambiente virtual.

**Windows (PowerShell):**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**Linux / macOS:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Instale as dependências:

```bash
pip install -r requirements.txt
```

Na primeira inicialização, o EasyOCR pode baixar seus arquivos de modelo. Esse processo exige conexão com a internet e ocorre apenas quando os pesos ainda não estão instalados na máquina.

## Como executar

Com o ambiente virtual ativado e as dependências instaladas, execute:

```bash
streamlit run app.py
```

O Streamlit abrirá a aplicação no navegador. Envie uma fotografia do material didático e faça uma pergunta, por exemplo:

- `O que é a fotossíntese?`
- `Quais são as fases da mitose?`
- `Resuma o texto.`

O sistema exibirá a matéria identificada, a probabilidade associada à classe prevista, a confiança da leitura do OCR e a resposta ou ação escolhida pelo agente. O texto reconhecido também pode ser consultado na própria interface.

### Leitura de uma imagem pela linha de comando

Também é possível executar o OCR diretamente, sem abrir a interface:

```bash
python -m src.visao_computacional.ocr caminho/da/imagem.jpg
```

## Testes automatizados

Para executar os testes que não dependem do processamento completo do OCR:

```bash
pytest -m "not lento"
```

Para executar todos os testes, inclusive os marcados como lentos:

```bash
pytest
```

## Experimentos e reprodutibilidade

Os scripts abaixo reproduzem os experimentos descritos neste README. Execute os comandos a partir da raiz do projeto, com o ambiente virtual ativado.

### Experimento 1 — Avaliação do OCR

Compara configurações de pré-processamento usando imagens sintéticas com texto de referência:

```bash
python -m src.visao_computacional.experimento_ocr
```

Se as imagens sintéticas ainda não existirem, o script as gera antes da avaliação. O experimento compara o reconhecimento sem pré-processamento, com correção de iluminação e inclinação, com remoção de ruído e com binarização.

### Experimento 2 — Classificação de matéria

O conjunto de dados de treinamento já está incluído em `dados/wikipedia_materias.csv`. Para treinar novamente os modelos:

```bash
python -m src.machine_learning.treinar
```

A coleta dos parágrafos da Wikipédia pode ser refeita, quando necessário, com:

```bash
python -m src.machine_learning.coletar_dados
```

### Experimentos 3 e 4 — PLN e agente de decisão

Para avaliar reconhecimento de intenção, recuperação de trechos e decisão usando o texto de referência das páginas:

```bash
python -m src.experimento_perguntas --fonte gabarito
```

O script também aceita `--fonte ocr`, mas essa opção depende da execução prévia do experimento de fotografias reais e da existência de imagens identificadas com a condição `boa` (por exemplo, `p1_boa.jpg`, `p2_boa.jpg`).

### Experimento 5 — Fotografias reais

Para executar a avaliação quantitativa com as imagens reais armazenadas em `dados/imagens_reais/`:

```bash
python -m src.experimento_fotos_reais
```

O script compara o texto extraído com o texto de referência das páginas e gera arquivos CSV com métricas de CER, WER, confiança do OCR, classificação de matéria e tempo de processamento. Os arquivos são gravados em `resultados/`. Para identificar automaticamente a página e a condição da fotografia pelo nome, utilize o padrão `p<numero>_<condicao>.jpg`, como `p1_boa.jpg`, `p2_sombra.jpg` ou `p3_inclinada.jpg`. Quando o nome não segue esse padrão, o código tenta identificar a página pelo texto reconhecido e registra a condição como `desconhecida`.

**Observação:** os valores de confiança observados na interface nas seis fotografias reais são apresentados na seção de resultados. Eles não substituem as métricas de CER e WER geradas pela comparação com o texto de referência.

## Conjuntos de dados

| Conjunto | Composição | Finalidade |
|---|---|---|
| Imagens sintéticas | 60 imagens geradas a partir de 12 textos-base, com variações de inclinação, desfoque, ruído, sombra, contraste e compressão JPEG | Avaliação do OCR |
| Base de classificação | 1.607 parágrafos provenientes de 119 artigos da Wikipédia em português, distribuídos em seis matérias | Treinamento e avaliação dos classificadores |
| Material impresso | Seis páginas com textos de referência | Testes de OCR, classificação e perguntas |
| Perguntas | 51 perguntas: 38 com resposta no material, cinco de resumo e oito sem resposta | Avaliação do PLN e do agente de decisão |
| Fotografias reais | Seis imagens de materiais didáticos capturadas por celular | Verificação funcional com imagens do mundo real |

Os textos da Wikipédia utilizados no conjunto de classificação são distribuídos sob a licença CC BY-SA 4.0. Consulte a fonte original para conhecer as condições de atribuição e compartilhamento.

## Resultados experimentais

### Experimento 1 — OCR em imagens sintéticas

Avaliação realizada com 60 imagens sintéticas e o EasyOCR. CER (*Character Error Rate*) e WER (*Word Error Rate*) representam taxas de erro; portanto, valores menores são melhores.

| Configuração | CER médio | CER mediano | WER médio | Tempo médio |
|---|---:|---:|---:|---:|
| Sem pré-processamento | 31,13% | 39,0% | 47,5% | 6,6 s |
| Iluminação + inclinação | 4,5% | 3,3% | 20,6% | 7,3 s |
| Completo, com remoção de ruído | **4,45%** | **2,4%** | **19,16%** | **7,51 s** |
| Completo + binarização | 8,9% | 4,0% | 29,0% | 7,4 s |

![CER em função da inclinação](resultados/graficos/ocr_easyocr_cer_vs_inclinacao.png)

A correção de inclinação explica grande parte do ganho: acima de aproximadamente 1,8° de inclinação, o CER sem pré-processamento chegou a variar entre 35% e 60%. Nesses casos, o OCR ainda reconhece caracteres, mas segmentos de linhas vizinhas podem se misturar durante a reconstrução da ordem de leitura.

A binarização apresentou resultado inferior ao pré-processamento completo sem essa etapa. Uma possível explicação é que a rede do EasyOCR lida melhor com as imagens em tons de cinza; por isso, a binarização permanece desativada na configuração padrão.

### Experimento 2 — Classificação de matéria

A avaliação utilizou validação cruzada em cinco partes, separando os dados por artigo para reduzir a possibilidade de parágrafos do mesmo artigo aparecerem simultaneamente no treinamento e na validação.

| Modelo | Acurácia média | F1-macro médio |
|---|---:|---:|
| **Naive Bayes (palavras)** | **0,8786 ± 0,0285** | **0,8694 ± 0,0332** |
| SVM linear (caracteres) | 0,8674 ± 0,0399 | 0,8572 ± 0,0395 |
| SVM linear (palavras) | 0,8631 ± 0,0311 | 0,8536 ± 0,0295 |
| Regressão Logística (palavras) | 0,8599 ± 0,0290 | 0,8497 ± 0,0259 |

![Matriz de confusão](resultados/graficos/ml_matriz_confusao.png)

O Naive Bayes com representação TF-IDF por palavras apresentou o melhor resultado médio entre os modelos comparados. A maior confusão observada foi entre **Geografia e História**, com 47 parágrafos classificados de forma equivocada nessa direção. Essas matérias compartilham vocabulário relacionado a território, povos e economia.

O modelo final também classificou corretamente os 12 textos-base e os textos extraídos das 60 imagens sintéticas, inclusive no cenário sem pré-processamento, em que o CER médio foi de aproximadamente 31%. Como as 60 imagens foram derivadas dos mesmos 12 conteúdos, esse resultado indica resistência a erros de OCR dentro desse conjunto, mas não comprova generalização para 60 textos independentes.

### Experimento 3 — Processamento de Linguagem Natural

Foram utilizadas 51 perguntas relacionadas ao material didático.

- **Reconhecimento de intenção:** 98,0% de acerto (50 de 51 perguntas). O erro ocorreu na pergunta “Quais fenômenos…”, que não correspondeu às regras implementadas para expressões como “quais são”, “quais os” ou “quais as”.
- **Recuperação de trechos:** avaliação das 38 perguntas que possuíam resposta no material.

| Método de busca | Recall@1 | Recall@3 | MRR |
|---|---:|---:|---:|
| N-gramas de caracteres (3–5) | **0,8947** | **1,0000** | **0,9430** |
| N-gramas de palavras | 0,8680 | 1,0000 | 0,9300 |

O método baseado em caracteres teve o melhor resultado nas métricas de recuperação apresentadas. O Recall@3 de 1,0 indica que, nesse conjunto, o trecho esperado apareceu entre os três primeiros resultados para todas as perguntas respondíveis.

### Experimento 4 — Agente de decisão

Com o limiar de similaridade definido em **0,30**, os resultados registrados foram:

- **89,5%** de acerto nas perguntas que possuíam resposta no material.
- **87,5%** de recusas corretas entre as perguntas sem resposta (7 de 8).
- **89,1%** de acerto total na avaliação considerada.

No fluxo completo, foram registradas 34 respostas corretas, quatro respostas incorretas, cinco resumos gerados, sete recusas corretas e uma resposta indevida para uma pergunta sem resposta no material.

![Calibração do limiar](resultados/graficos/decisao_limiar_gabarito.png)

Os erros observados ajudam a identificar limitações concretas da abordagem lexical:

- **Heurística de anáfora:** em uma pergunta como “O que é a mitose?”, a frase seguinte, iniciada por “Ela é dividida…”, pode ser selecionada como se apresentasse a definição.
- **Lacuna lexical:** a pergunta “Onde ocorre a fotossíntese?” pode não encontrar adequadamente o trecho “Nos cloroplastos…”, pois as expressões têm poucas palavras em comum.
- **Coincidência de palavra:** a expressão “Segunda Guerra Mundial” pode ser associada incorretamente a um trecho que contém “segunda metade do século”.

**Limitação da avaliação:** as heurísticas de busca e o limiar do agente foram ajustados utilizando o mesmo conjunto de 51 perguntas usado na avaliação. Por esse motivo, os resultados dos experimentos 3 e 4 podem ser otimistas e não devem ser interpretados como desempenho comprovado em um conjunto externo independente.

### Experimento 5 — Teste funcional com fotografias reais

Foram analisadas seis fotografias reais de materiais didáticos. Durante os testes na aplicação:

- O classificador exibiu **100% de probabilidade para a matéria prevista** nas seis imagens analisadas.
- A confiança média apresentada pelo EasyOCR ficou entre **82% e 84%**.

Esses valores representam as pontuações exibidas pelos modelos durante o teste. **A probabilidade de 100% não equivale a 100% de acurácia do classificador**, e a confiança do OCR não equivale à taxa de caracteres ou palavras reconhecidos corretamente. Para afirmar acurácia, CER ou WER das fotografias reais, é necessário comparar as previsões e o texto extraído com rótulos e transcrições de referência. Por isso, essas medidas não são apresentadas aqui como métricas de acurácia do experimento com fotos reais.

As seis imagens estão disponíveis na pasta `dados/imagens_reais/`:

- [Foto 1](dados/imagens_reais/foto_1.jpg)
- [Foto 2](dados/imagens_reais/foto_2.jpg)
- [Foto 3](dados/imagens_reais/foto_3.jpg)
- [Foto 4](dados/imagens_reais/foto_4.jpg)
- [Foto 5](dados/imagens_reais/foto_5.jpg)
- [Foto 6](dados/imagens_reais/foto_6.jpg)

> **Nota sobre os arquivos:** as seis fotografias foram adicionadas por meio da [PR #2](https://github.com/helberkaue/assistente-academico-ia/pull/2). Caso essa alteração ainda não esteja integrada à branch principal, os links das imagens passarão a funcionar no README da branch principal após a integração da PR.

## Limitações e trabalhos futuros

- O escopo atual contempla materiais impressos. Escrita manual, fórmulas matemáticas complexas e diagramas não foram avaliados de forma específica.
- A correção de inclinação foi desenvolvida para inclinações no plano; fotografias com perspectiva muito acentuada podem continuar apresentando erros.
- A recuperação de respostas é lexical, baseada em TF-IDF. Perguntas e respostas formuladas com vocabulários muito diferentes podem não apresentar similaridade suficiente. Embeddings semânticos podem ajudar a reduzir esse problema.
- A avaliação de PLN e decisão utiliza um conjunto relativamente pequeno de perguntas e o mesmo material de seis páginas.
- O limiar e as heurísticas foram ajustados com o conjunto de perguntas avaliado, o que pode superestimar os resultados. Uma avaliação futura deve separar dados de calibração e teste.
- A confiança retornada pelo OCR e a probabilidade do classificador são pontuações dos modelos e não substituem uma avaliação com dados rotulados independentemente.
- O OCR em CPU leva, em média, alguns segundos por imagem, conforme a resolução e a condição da fotografia.

## Estrutura do repositório

```text
assistente-academico-ia/
├── app.py                         # Interface Streamlit
├── requirements.txt               # Dependências do projeto
├── pytest.ini                     # Configuração dos testes
├── dados/
│   ├── imagens_reais/             # Fotografias reais
│   ├── material_impressao/         # Material e textos de referência
│   ├── perguntas.csv              # Perguntas usadas nos experimentos
│   ├── textos_base.json            # Textos-base
│   └── wikipedia_materias.csv      # Base para classificação
├── modelos/                        # Modelo de classificação treinado
├── resultados/                     # CSVs, gráficos e exemplos dos experimentos
├── src/
│   ├── assistente.py               # Integração dos módulos
│   ├── caminhos.py                 # Caminhos e diretórios do projeto
│   ├── decisao/                    # Agente baseado em regras
│   ├── machine_learning/           # Classificação de matérias
│   ├── pln/                        # Intenção, busca e resposta
│   └── visao_computacional/        # Pré-processamento e OCR
└── testes/                         # Testes automatizados
```

## Referências

- BAEK, Y. et al. Character Region Awareness for Text Detection. *Proceedings of CVPR*, 2019.
- BUADES, A.; COLL, B.; MOREL, J.-M. A non-local algorithm for image denoising. *Proceedings of CVPR*, 2005.
- JOACHIMS, T. Text Categorization with Support Vector Machines: Learning with Many Relevant Features. *ECML*, 1998.
- LEVENSHTEIN, V. I. Binary codes capable of correcting deletions, insertions, and reversals. *Soviet Physics Doklady*, 1966.
- MANNING, C. D.; RAGHAVAN, P.; SCHÜTZE, H. *Introduction to Information Retrieval*. Cambridge University Press, 2008.
- OTSU, N. A Threshold Selection Method from Gray-Level Histograms. *IEEE Transactions on Systems, Man, and Cybernetics*, 1979.
- PEDREGOSA, F. et al. Scikit-learn: Machine Learning in Python. *Journal of Machine Learning Research*, 2011.
- RUSSELL, S.; NORVIG, P. *Artificial Intelligence: A Modern Approach*. 4. ed. Pearson, 2020.
- SALTON, G.; BUCKLEY, C. Term-weighting approaches in automatic text retrieval. *Information Processing & Management*, 1988.
- SHI, B.; BAI, X.; YAO, C. An End-to-End Trainable Neural Network for Image-based Sequence Recognition. *IEEE Transactions on Pattern Analysis and Machine Intelligence*, 2017.
- SMITH, R. An Overview of the Tesseract OCR Engine. *Proceedings of ICDAR*, 2007.
- JaidedAI. EasyOCR. Disponível em: <https://github.com/JaidedAI/EasyOCR>.
- Wikipédia em português. Conteúdo disponibilizado sob licença CC BY-SA 4.0.
