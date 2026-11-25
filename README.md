# HLA Consenso · NGSengine × AlloSeq Assign

Aplicativo web (100% no navegador) para a etapa **pós-análise** da tipagem HLA por NGS:
compara o resultado do **NGSengine** (export TARR em XML) com o do **AlloSeq Assign**
(relatórios Excel), associa métricas de qualidade a cada tipagem, registra a decisão do
analista em cada divergência, confere a hemizigose de DRB3/4/5 pelo DRB1 e gera o
**resultado final (.xls)** e o **consenso Engine × Assign** no formato do app R.

Nenhum arquivo sai do computador: tudo é lido e processado no navegador. As decisões ficam
salvas no `localStorage`, por corrida, e podem ser exportadas e importadas como JSON.

## Testar uma versão nova sem mexer na que está em uso

O GitHub Pages publica **um site por repositório**, a partir de uma única branch. Para testar:

- **Repositório novo (mais simples):** por exemplo `hla-consenso-teste`. Envie o
  `index.html` e o `referencia_ld.json` (os dois na raiz) e ative o Pages. O endereço fica separado
  (`https://<usuario>.github.io/hla-consenso-teste/`).
- **Branch separada:** serve para guardar o código, mas não ganha um endereço próprio no
  Pages sem configurar um GitHub Actions.

As decisões salvas no navegador são guardadas por endereço do site. Por isso, a versão de
teste não mistura sessões com a versão em uso.

## Publicar no GitHub Pages

O jeito mais simples: publique só o **`index.html` de arquivo único** (`dist/index.html`), que
já traz o CSS e o JavaScript embutidos. Assim não há caminhos de pasta que possam se perder no
upload.

1. Crie um repositório e envie para a **raiz**, sem pastas, só estes arquivos:
   - `index.html`: o arquivo único, com tudo embutido (é o `dist/index.html` do pacote);
   - `validacao.html`: a página de validação (é o `dist/validacao.html`);
   - `referencia_ld.json`: a referência de LD, quando você já tiver gerado.
   Se preferir publicar as fontes, envie `index.html`, `css/`, `js/` e `.nojekyll` mantendo as
   pastas; se o upload pelo navegador achatar as pastas, a página abre sem estilo.
2. No repositório, abra **Settings → Pages → Build and deployment → Deploy from a branch**,
   escolha a branch `main` e a pasta `/ (root)`.
3. Em cerca de 1 minuto, o app fica disponível em `https://<usuario>.github.io/<repositorio>/`.

Também funciona offline: basta abrir o `index.html` direto no Chrome ou no Edge.
Não há dependências externas (nem CDN).

## Dois modos

- **NGSengine × Assign:** é o modo usado quando você importa o XML do NGSengine e os Excel do
  Assign. Faz a comparação completa descrita abaixo.
- **Só NGSengine:** é ativado automaticamente quando você importa **só o XML**, a rotina
  padrão sem o Assign.
  - Cada tipagem fica **verde** (aprovada no NGSengine, liberada automaticamente),
    **amarela** (aprovada, mas com alerta vermelho de QC ou código NMDP com “?”) ou
    **listrada em vermelho** (rejeitada, sai NT).
  - QC, dosagem gênica, DRB3/4/5, desequilíbrio de ligação, API MAC, revisão em lote e todos
    os arquivos continuam disponíveis.
  - O consenso Engine × Assign não é gerado nesse modo.

**Report do REDOME (opcional).** Carregue a lista `Patient;DMR;ORIGEM;NOME;` (CSV com `;`,
UTF-8 ou latin-1) na tela inicial ou na aba Exportar. Só então é gerado o
`resultados_<lote>_redome_format.csv`, no layout do app antigo, com a coluna `ORIGEM` a mais:

```
00_DMR;01_date_redome;02_alelo01_HLA-A_redome … 23_alelo02_HLA-DPB1_redome;ORIGEM;Patient;fastq_filename;sample_name;software_name;software_version
```

- **Associação da amostra ao `Patient` (número IL):** primeiro pelo ID igual; depois pelos
  dígitos, ignorando zeros à esquerda e prefixos como "IL"; por fim, por um trecho do nome da
  amostra no NGSengine.
- **Amostras sem DMR na lista:** aparecem no checklist de exportação.
- **Tipagens:** são as mesmas do resultado final, com a hemizigose aplicada. NT e pendentes
  saem em branco.
- **Privacidade:** a lista (que tem nomes) fica só na memória da página; ela não é salva no
  navegador.

## Fluxo

1. **Arraste os arquivos:** `*_tarr.xml` (ou um `.zip` com ele) e os `*_report_assign.xlsx`
   (p1, p2…).
2. **Placa:** mapa amostra × locus. Cor e ícone mostram a situação da comparação, e os
   marcadores mostram a decisão (`auto`, `E`, `A`, `M`, `✕`, `!` = pendente) e os alertas de QC.
3. **Revisão:** as pendências vêm agrupadas por locus e situação. Dá para decidir um grupo
   inteiro de uma vez ou abrir uma tipagem por vez.
4. **Painel da tipagem:** genótipos lado a lado (em verde, os pares presentes nos dois
   softwares), a prévia exata do que será liberado por cada opção, as métricas de qualidade
   contra os limites, o status de revisão no NGSengine, os alelos anotados e as edições.
   Atalhos: `E` NGSengine, `A` Assign, `M` manual, `X` não liberar, `J`/`K` navegar,
   `P` próxima pendente, `Ctrl+Z` desfazer.
5. **Qualidade:** gráficos da placa e os limites de QC, editáveis e recalculados na hora.
6. **DRB3/4/5:** confirme quantos alelos de cada gene hemizigoto vão para o resultado, com
   a sugestão do DRB1 e da dosagem gênica lado a lado.
7. **Ligação:** confira os haplótipos DRB1~DQB1 e B~C da corrida contra a referência do
   repositório (gerada pelo `ld_referencia.py`).
8. **Exportar:** checklist de conferência, resultado final `.xls`, consenso no formato do
   app R, conferência detalhada e relatório para imprimir ou salvar em PDF.

## Regras

**Comparação.** Do lado do NGSengine, usa `ConclusiveTypingResult/Typing/GLString`
(`|` separa genótipos, `+` separa os alelos, `/` separa as alternativas). Do lado do Assign,
usa a aba *Ambiguities* (genótipos `X + Y` separados por `/`). Os dois viram conjuntos de
pares, reduzidos a 3 campos e comparados:

| Situação | Critério |
|---|---|
| Concordante | conjuntos de genótipos idênticos |
| Concordante só no 1º genótipo (**amarelo, pendente**) | mesmo genótipo principal, mas as listas de genótipos possíveis diferem (ex.: Assign inclui `DQB1*03:276N+DQB1*03:276N`) |
| Concordância parcial | há genótipos em comum, mas também alternativas exclusivas |
| Concordante só em 2 campos | nenhum par em comum em 3 campos, mas há em 2 campos |
| Divergente | nenhum par em comum |
| Só NGSengine / Só Assign | apenas um software tipou o locus |

- Hemizigoto (`DRB3*02:02:01`) equivale a homozigoto (`DRB3*02:02:01+DRB3*02:02:01`).
- Por padrão, **todas** as alternativas entram na comparação, inclusive as que têm alelo nulo
  (N). Assim, qualquer diferença entre os genótipos brutos aparece em amarelo. A opção
  "Ignorar alternativas com alelo nulo" existe na aba Qualidade, mas fica desligada.

**Liberação automática.** É feita só quando a comparação é **concordante** (verde: listas de
genótipos idênticas), o locus está **Approved** no NGSengine e não há alerta vermelho de QC (essa última
condição é configurável). Todo o resto fica pendente para o analista.

**Rejeitados.** Locos rejeitados no NGSengine que têm tipagem saem como **NT** (automático,
configurável). O analista pode trocar por Assign ou por tipagem manual. Na placa, esses locos
aparecem com fundo listrado em vermelho.

**Código NMDP incompleto (“?”).** A tipagem fica pendente de revisão. O botão
**Consultar API MAC** faz a mesma chamada do `mac_request_api.py`: envia a
`TypingResult/GLString` para `hml.nmdp.org/mac/api/encode`, com a versão IMGT do locus, e
libera o código devolvido.

**Assign ambíguo.** O Assign não usa “?”. A tipagem dele precisa de código NMDP quando a aba
*Ambiguities* tem mais de um genótipo distinto em 3 campos; o *Summary* mostra só o primeiro.
- **Ao escolher o Assign num locus assim** (um por um ou em lote), a tipagem fica pendente (marca
  `A?` na placa) e entra na lista da API MAC.
- **O que vai para a API:** a ambiguidade do Assign, convertida para GL (genótipos separados por
  `|`), com a versão IMGT do Assign.
- **Alternativa:** o painel permite liberar o *Summary* assim mesmo, sem código. Fica registrado
  como decisão explícita do analista.
- **Sem código:** alternativas que só diferem no 4º campo não precisam de código.

**Rejeitado com “?”.** Um locus rejeitado no NGSengine vira NT antes de qualquer consulta: o “?”
dele não vai para a API MAC, porque essa tipagem não será usada. Se o analista trocar pelo Assign,
vale a regra do Assign ambíguo.

**API MAC em lote.** Na placa e no topo da aba Revisão, o botão *Consultar todas na API MAC*
envia de uma vez todas as tipagens pendentes sem código NMDP (“?” do NGSengine ou Assign ambíguo
escolhido), indicando a origem de cada uma:
- até 3 consultas simultâneas; GLStrings repetidas são consultadas uma vez só;
- barra de progresso e resultado por amostra; dá para parar no meio;
- cada código obtido vira decisão *NGSengine + API MAC*, com o nome do analista;
- as que falharem podem ser repetidas ("Tentar de novo") ou copiadas como tabela
  (amostra, locus, IMGT, GLString) para consulta por outro meio.

**DRB3/4/5.** O NGSengine duplica genes hemizigotos (X+X). A aba DRB3/4/5 sugere quantos
alelos liberar pela associação com o DRB1:
- DR52 (DRB1\*03, 11, 12, 13, 14) → DRB3;
- DR53 (DRB1\*04, 07, 09) → DRB4;
- DR51 (DRB1\*15, 16) → DRB5;
- DRB1\*01, 08 e 10 não carregam esses genes.

O analista confirma ou troca entre 2 alelos, 1 alelo ou nenhum. Conflitos (ex.: o DRB1 indica
DR51, mas o DRB5 não foi tipado) ficam destacados.

**Dosagem gênica / detector de número de cópias (captura híbrida).** Na captura híbrida, o
número de reads mapeados a um gene (`Locus/Mappability/@TotalReads` do export TARR) é
proporcional ao número de cópias. Dentro do mesmo barcode:

- `razão` = reads do gene investigado ÷ reads do **gene de referência da mesma classe HLA**.
  - Classe I: A, B ou C. Automático: **C**, o que tem mais reads por barcode.
  - Classe II: DRB1, DQB1 ou DPB1. Automático: **DRB1**.
  - O próprio gene de referência é avaliado contra o 2º candidato da classe.
- `cópias` = 2 × razão ÷ calibração. A calibração é a mediana da razão dos **heterozigotos
  daquele gene** na placa (2 cópias certas). Cada gene tem a sua taxa de captura.
  DRB3/4/5, que são parálogos com poucos heterozigotos, são calibrados juntos.
- **Classificação (limites editáveis):** < 0,5 = gene ausente; 0,5–1,35 = 1 cópia;
  1,35–1,65 = zona cinza; > 1,65 = 2 cópias. Se o gene de referência da amostra tem menos de
  60% dos reads típicos da placa, a dosagem é marcada como pouco confiável.
- **Troca da referência:** na aba **Qualidade**, cartão "Detector de número de cópias". A
  tabela mostra, para cada candidato, a mediana de reads por barcode e o CV dos heterozigotos
  (quanto menor, mais estável). A escolha fica salva na sessão.

Usos:
- **DRB3/4/5:** segunda evidência, independente do DRB1, para decidir entre hemizigose e
  homozigose, e para separar gene ausente de falha de captura.
- **Locos clássicos homozigotos:** perto de 1 cópia gera alerta vermelho de possível perda
  alélica (*drop-out* de captura) ou deleção; perto de 2 cópias, a homozigose fica confirmada.

Validação na corrida NTVPB_51 (40 amostras), com as referências C (classe I) e DRB1
(classe II):

| Grupo | Cópias estimadas |
|---|---|
| Heterozigotos A/B/C/DQB1/DPB1 | 1,78–2,23 (CV dos heterozigotos de 4,5–4,9%) |
| Homozigotos clássicos | 1,82–2,28 (homozigose confirmada) |
| DRB3/4/5 com 1 cópia (n=42) | 0,94–1,25 |
| DRB3/4/5 com 2 cópias (n=10) | 1,89–2,43 |
| Genes ausentes | 0,01–0,26 |

- 100% de concordância com a previsão pelo DRB1.
- Na amostra 3020, o DRB5 tem 0,23 cópia (só ruído) apesar do DRB1\*15:03: deleção do gene
  ou falha de captura.
- Trocar as referências para A e DQB1 não muda nenhuma classificação.

A dosagem só vale para captura híbrida. Em kits por amplicon, o número de reads depende da
eficiência de cada PCR.

**Desequilíbrio de ligação (DRB1~DQB1 e B~C).** A referência é gerada **fora do app**, de
tempos em tempos, pelo script `ld_referencia.py`. O script usa só a biblioteca padrão do
Python: não precisa instalar nada.

```bat
cd C:\ngs_hla_automation\hla_consenso
..\.venv\Scripts\python.exe ld_referencia.py "D:\resultados_finais" --csv frequencias_ld.csv
```

- **Entrada:** a pasta com os `*_resultado_final` (`.xls`, `.xlsx` ou `.csv`); subpastas
  também são lidas.
- **Saída 1 — `referencia_ld.json`:** é o arquivo que vai para o repositório do GitHub
  Pages, **na raiz, ao lado do `index.html`** (sem pastas). O app lê esse arquivo sozinho a cada placa analisada. Ele traz as frequências, o D′
  e a **lista das placas incluídas**, com data de inclusão e número de doadores novos.
- **Saída 2 — `base_ld_acumulada.json`:** fica **no seu computador, fora do GitHub** (já está
  no `.gitignore`). Guarda as placas já processadas, um hash dos IDs já contados e as
  genotipagens agregadas, sem IDs. Na próxima execução, só as placas novas são lidas, e o EM
  é refeito sobre o acumulado. O resultado é idêntico ao de reprocessar tudo; isso foi
  testado. Use `--refazer` para começar do zero.
- **Regras:**
  - só IDs numéricos (doadores voluntários); famílias e controles são ignorados;
  - célula NT descarta o locus;
  - ID repetido na mesma leva conta pela ocorrência mais completa;
  - um doador já presente na base não é recontado;
  - alelos em 2 campos;
  - códigos MAC viram ambiguidade entre os alelos do mesmo 1º campo, e o EM distribui a
    probabilidade.
- **No app (aba Ligação):** mostra a fonte da referência, as placas incluídas e, para cada
  amostra da corrida, a fase mais provável dos dois haplótipos, quantas vezes cada um foi
  visto no legado e o status:
  - *esperado*;
  - *incomum* (visto menos de 3 vezes; alerta amarelo);
  - *nunca observado* (alerta vermelho e revisão);
  - *alelo sem registro*.

  Se a página for aberta direto do disco, sem servidor, importe o `.json` pela própria aba.

**Tipagem liberada.**
- *NGSengine:* `TypingResult/NMDPCodes` reduzidos a 3 campos (igual ao app anterior).
- *Assign:* aba *Summary* com o prefixo do locus (pode vir com sufixo G).
- *Manual* ou *API MAC*.
- *NT:* rejeitado ou não liberado.

Pendentes e locos não tipados saem em branco.

**Arquivos exportados.**
- `<lote>_resultado_final.xls` (Excel 97-2003, aba `resultados`), com as colunas `amostra`,
  `HLA-A_01`, `HLA-A_02`, … `HLA-DRB5_02` e `fastq_filename`, na ordem
  A, B, C, DQB1, DPB1, DRB1, DRB3, DRB4, DRB5. As amostras que contêm `1044736` são excluídas
  (configurável).
- `<projeto>_engine_assign_consensus_report.csv`: o mesmo arquivo do comparador em R,
  inclusive na comparação literal *match/divergence*. A única mudança é que o lado do NGSengine
  agora vem de `ConclusiveTypingResult/Typing/GLString`.
- `<lote>_conferencia_detalhada.csv`: decisão, analista, data, observação e métricas por
  tipagem.
- `resultados_<lote>_informacoes_adicionais.csv`: métricas por locus, como no app Python.
- Sessão (`.json`), para continuar a análise em outro computador.

## Validação a cada versão nova

Toda versão tem um número (`HLA.VERSAO` em `js/core.js`), que aparece no topo do app, no
relatório de conferência, na sessão exportada e no gabarito. A validação tem três camadas.

**1. Casos artificiais (automáticos).** Uma placa fabricada, sem dados de pacientes, com
resposta conhecida para cada regra crítica. São 62 casos em 11 grupos:
- comparação e nomenclatura;
- leitura do XML, do Excel e do `.xls`;
- decisões automáticas e opções de configuração;
- código NMDP e API MAC (NGSengine e Assign);
- dosagem gênica;
- DRB3/4/5;
- decisões do analista e resultado final;
- modo só NGSengine;
- consenso e REDOME;
- desequilíbrio de ligação;
- o próprio sistema de gabarito.

Rodam em dois lugares:
- no `montar.py`, antes de gerar o site: se um caso falhar, **nenhum arquivo é gerado**, então
  uma versão quebrada não chega a ser entregue;
- na página `validacao.html`, sozinhos, ao abrir a página.

Para conferir que os casos pegam erros de verdade, foram plantados defeitos de propósito no
código (troca da associação DR51, comparação frouxa, "não liberar" sem NT, duplicação de DRB3/4/5
mantida). Cada um foi detectado por pelo menos um caso.

**2. Corridas reais conferidas (gabaritos).** Depois de conferir uma corrida inteira no app,
use **Exportar → Validação → Gerar gabarito de validação**. O arquivo
`<lote>_gabarito_validacao.json` guarda:
- os arquivos de entrada, pela impressão digital SHA-256;
- as suas decisões, a conferência DRB3/4/5 e a configuração;
- todas as saídas daquele momento: situação, sugestão, QC e tipagem liberada de cada locus;
  resultado final; consenso; informações adicionais; REDOME (se a lista estava carregada).

A cada versão nova, abra `validacao.html` e solte os gabaritos junto com os arquivos originais
das corridas. A página refaz cada corrida do zero, com as mesmas decisões, e compara célula por
célula. Toda diferença aparece com o local, o valor conferido e o valor da versão nova.

- **Arquivo trocado:** se um arquivo tiver o mesmo nome, mas conteúdo diferente do usado no
  gabarito, a corrida não é verificada.
- **Privacidade:** os gabaritos e os arquivos das corridas **ficam no seu computador**. Não suba
  esses arquivos para o GitHub.
- **Quais corridas usar:** é bom ter pelo menos 3: uma com Assign, uma só com NGSengine e uma
  com a lista do REDOME. Também vale ter as corridas que já deram algum problema.
- **LD:** a comparação é feita **sem** a referência de LD, porque ela muda com o tempo. As
  regras de LD são verificadas no grupo 9 dos casos artificiais.

**3. Relatório.** O botão **Relatório de validação (PDF)** imprime a página com:
- a versão;
- a data e o navegador;
- cada caso;
- cada corrida conferida.

Serve de registro da validação de cada versão.

### Roteiro a cada versão

1. Receba os arquivos da versão nova (`index.html`, `validacao.html`) e publique-os no
   **repositório de teste**.
2. Abra `…/validacao.html` no repositório de teste e solte os gabaritos com os arquivos das
   corridas.
3. **Tudo verde** ("Versão aprovada na validação"): imprima o relatório em PDF e copie os arquivos
   para o repositório em uso.
4. **Alguma diferença:** não publique. Ou é um erro (avise e mande o print da diferença), ou é
   uma mudança de regra que você pediu. Nesse caso, confira as corridas afetadas na versão nova,
   gere os gabaritos de novo e anote o motivo no campo "Observação".
5. Quando uma regra nova for implementada, ela ganha casos artificiais próprios na mesma
   entrega.

## Diferenças em relação ao app Python atual

- A redução para 3 campos é feita por truncamento, preservando o sufixo de expressão
  (`A*24:02:01:02L` → `A*24:02:01L`). Isso equivale à redução *exon* do py-ard para alelos
  isolados, que é o caso dos NMDPCodes.
- A API MAC é chamada pelo navegador. Se o servidor do NMDP recusar chamadas vindas de
  páginas web (CORS), o app avisa e copia a GLString para a área de transferência.

## Estrutura

Depois de editar as fontes, rode `python montar.py`: ele roda a validação (`node testar.js`) e,
se tudo passar, gera `dist/index.html` e `dist/validacao.html`.

```
index.html        interface
css/style.css     estilos (tema claro/escuro)
js/arquivos.js    leitor de ZIP/XLSX/XLS e gerador de ZIP e de .xls BIFF8 (sem bibliotecas)
js/ld.js          desequilíbrio de ligação: checagem contra a referência (o EM também está aqui, para testes)
ld_referencia.py  gera/atualiza referencia_ld.json a partir do legado (incremental)
js/core.js        leitura do TARR e do Assign, comparação, QC e exportação (testável no Node)
js/fluxo.js       fluxo completo sem interface (ler → consolidar → decisões → saídas) e gabaritos
js/app.js         interface: placa, painel de decisão, revisão em lote, gráficos, exportação
validacao.html    página de validação
js/validacao.js   casos artificiais (placa fabricada) e mini-framework de testes
js/validacao_ui.js interface da página de validação
testar.js         roda os casos no Node (usado pelo montar.py)
```

Os testes de bancada ficam em `teste/` e usam `node` e `playwright`; não fazem parte do site.
