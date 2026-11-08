<img alt="Logo da Universidade Federal do Ceará" src="../assets/logo-ufc.svg">

# **Guia dos dados pré-processados**

Este guia tem o objetivo de orientar quem vai consumir os arquivos de saída, gerados pela etapa de pré-processamento. Ele descreve o conteúdo de cada arquivo, o significado das colunas, as chaves de junção e os cuidados de interpretação. Os números de referência citados (contagens, limiares) valem para a versão atual dos dados e estão em `metadados.json`.

## **Visão geral**

| Arquivo | Unidade | Linhas | Uso |
|---|---|---|---|
| `setores.csv` | Setor censitário | 4.268 | Tabela principal: população, renda, distâncias e classes de desassistência |
| `setores.geojson` | Setor censitário | 4.268 | Geometria dos setores (somente chaves) |
| `bairros.geojson` | Bairro | 121 | Geometria dos bairros com indicadores agregados |
| `unidades.csv` | Unidade de saúde | 193 | UBS, UPAs e hospitais, com a carga de cada um |
| `cobertura.csv` | Base × nível × faixa × distância | 3.510 | Curva de cobertura acumulada |
| `metadados.json` | — | — | Parâmetros e contagens da geração |
| `conferir_hospitais.csv` | Hospital | 35 | Apoio à conferência manual |

Os níveis de saúde aparecem nos nomes de colunas com os sufixos `ubs`, `upa` e `hosp`.

Para análise em SIG, o notebook também gera `data/output/qgis/analise_fortaleza.gpkg`.

## **Convenções**

* **Sistemas de coordenadas.** Os GeoJSON e as colunas `lon`/`lat` estão em WGS 84 (EPSG:4326), para desenho no D3. As colunas `x`/`y` estão em metros, em SIRGAS 2000 / UTM 24S (EPSG:31984), para distâncias e simulações.
* **Distâncias** em metros, inteiras.
* **Códigos como texto.** `CD_SETOR` e `CD_BAIRRO` são identificadores. Em D3, é recomendado ler como texto para que as junções com as propriedades do GeoJSON funcionem.
* **Booleanos** no CSV aparecem como `True` e `False` (com inicial maiúscula), que o D3 não converte; compare com o texto, como no exemplo abaixo.
* **Valores ausentes** aparecem como campo vazio (por exemplo, `renda` de setores com dado suprimido).

## `setores.csv`

Um registro por setor com população maior que zero. A chave é `CD_SETOR`.

| Coluna | Descrição |
|---|---|
| `CD_SETOR` | Código do setor (texto) |
| `CD_BAIRRO`, `nome_bairro`, `nome_regional` | Bairro e regional do setor |
| `populacao` | População residente (Censo 2022) |
| `renda` | Rendimento nominal médio mensal dos responsáveis por domicílio no setor, em R$. Vazio quando suprimido |
| `renda_sm` | `renda` em salários mínimos (referência: R$ 1.212) |
| `faixa_renda` | `Até 2 SM`, `De 2 a 5 SM`, `Mais de 5 SM` ou `Sem dado` |
| `x`, `y`, `lon`, `lat` | Centroide do setor (metros UTM e graus) |
| `dist_ubs`, `dist_upa`, `dist_hosp` | Distância **em linha reta** até a unidade mais próxima do nível |
| `un_ubs`, `un_upa`, `un_hosp` | `id` (de `unidades.csv`) dessa unidade |
| `dist_via_*`, `un_via_*` | Equivalentes, **pela malha viária** (OpenStreetMap): a pé para UBS e de veículo para UPA e hospital |
| `vulneravel` | `True` se a renda média do setor é de até 1 SM. Vazio de renda resulta em `False` |
| `cls_*` | Classe de desassistência em linha reta |
| `cls_via_*` | Classe de desassistência pela via |

### **Classes de desassistência**

| Classe | Significado |
|---|---|
| 0 | Adequado: distância até o limiar moderado |
| 1 | Desassistência moderada: acima do moderado, até o severo |
| 2 | Desassistência severa: acima do severo |

Os limiares (em metros) são:

| Nível | Moderado | Severo |
|---|---|---|
| UBS | 1000 | 1300 |
| UPA | 3500 | 4500 |
| Hospital | 3000 | 4000 |

Nos setores com `vulneravel = True`, os limiares são multiplicados por 0,8 (`limiar_rigido.fator` em `metadados.json`), ou seja, UBS 800/1040, UPA 2800/3600 e Hospital 2400/3200. As colunas `cls_*` já incorporam essa regra. Para recalcular com outros limiares, por exemplo em um controle da interface, use as colunas `dist_*` e aplique o limiar (e o fator, se for o caso) no cliente.

## `setores.geojson`

Contém apenas a geometria e as chaves `CD_SETOR` e `CD_BAIRRO`. Os atributos ficam em `setores.csv`, e a junção é feita por `CD_SETOR`. A geometria é simplificada em 10 m e arredondada na 5ª casa decimal, o que é adequado à escala da cidade.

## `bairros.geojson`

Propriedades por bairro. Os indicadores são calculados a partir dos setores do bairro, ponderados pela população.

| Propriedade | Descrição |
|---|---|
| `CD_BAIRRO`, `nome_bairro`, `nome_regional` | Identificação (um bairro cruzado por duas regionais fica com a de maior área) |
| `populacao` | População do bairro |
| `renda_media_sm` | Renda média em SM, ponderada pela população, considerando só setores com renda |
| `pct_pop_baixa_renda` | % da população (com renda informada) que mora em setores da faixa `Até 2 SM` |
| `dist_{ubs,upa,hosp}` | Distância mediana (m) em linha reta, ponderada pela população |
| `dist_via_{...}` | Idem, pela via |
| `carga_{...}` | Carga média das unidades que atendem o bairro, ponderada pela população (carga em linha reta) |
| `pct_mod_{...}`, `pct_sev_{...}` | % da população em desassistência moderada (classe 1 ou 2) e severa (classe 2), em linha reta |
| `pct_mod_via_{...}`, `pct_sev_via_{...}` | Idem, pela via |

Esses sete indicadores por nível, mais `renda_media_sm`, permitem montar as coordenadas paralelas por bairro (distância e carga de cada nível, e renda).

## `unidades.csv`

A chave é `id` (código CNES), referenciada por `un_*` e `un_via_*` em `setores.csv`.

| Coluna | Descrição |
|---|---|
| `id`, `nome` | Código CNES e nome fantasia |
| `nivel` | `UBS`, `UPA` ou `Hospital` |
| `em_fortaleza` | `False` para unidades do entorno de 5 km, mantidas só para não criar vazios na borda. As do entorno que não são a mais próxima de nenhum setor de Fortaleza foram removidas |
| `lon`, `lat`, `x`, `y` | Coordenadas (graus e metros UTM) |
| `carga` | População dos setores cuja unidade mais próxima, **em linha reta**, é esta |
| `classe_carga` | Quintil da carga dentro do nível (1 = menor, 5 = maior) |

## `cobertura.csv`

Curva de cobertura acumulada: a cada 100 m, a parcela da população que está a até aquela distância da unidade mais próxima.

| Coluna | Descrição |
|---|---|
| `base` | `linha_reta` ou `via` |
| `nivel` | `UBS`, `UPA` ou `Hospital` |
| `faixa` | `Todas`, as três faixas de renda e `Sem dado` |
| `distancia_m` | Distância, de 0 até o máximo observado, em passos de 100 m |
| `pct_pop_ate` | % da população da faixa (0 a 100) a até `distancia_m` da unidade mais próxima |

A parcela *além* de um limiar é `100 - pct_pop_ate` no ponto do limiar. A faixa `Sem dado` tem população muito pequena, e seus percentuais são instáveis.

## `metadados.json`

Registra a data de geração, os CRS, a contagem de setores e a população, os parâmetros de renda (referência de salário mínimo, cortes, faixas, parcela sem dado), os limiares, a regra do limiar rígido (`limiar_rigido`), as classes, o modo da rede viária por nível (`rede_viaria`), as unidades por nível, as unidades do entorno removidas (`unidades_entorno_removidas`) e o raio do entorno. Convém ler os limiares e o fator daqui, em vez de repeti-los no código da aplicação.

## **Cuidados de interpretação**

* **Renda é média do setor.** Ela vale para todos os moradores do setor, e as faixas medem quantas pessoas moram em setores de cada faixa, e não quantas pessoas ganham cada faixa. Cerca de 0,6% dos setores (0,01% da população) não têm renda.

* **A linha reta subestima o percurso.** Em mediana, a distância pela via é cerca de 1,4 vez a de linha reta. Os mesmos limiares são aplicados às duas bases, e por isso as parcelas desassistidas pela via são maiores.

* **Carga não é capacidade.** `carga` mede demanda geográfica (população atribuída por proximidade em linha reta), e não leitos, equipes ou profissionais. Ela inclui unidades do entorno que atendem setores de Fortaleza, e todas as unidades do arquivo têm carga maior que zero.

* **Simulação de nova unidade.** Deve usar a distância em linha reta (colunas `x`/`y` e `dist_*`). As colunas pela via não se recalculam no navegador.

* **População.** Setores sem população foram removidos. Além disso, 45 setores do Censo (cerca de 0,7% da população) não existem na malha de setores e ficam fora dos dados.

* **Bairros.** Os setores cujo nome de bairro não existe na malha de bairros foram associados ao bairro que contém o seu ponto interno (por exemplo, o Dendê consta em Rachel de Queiroz).

## **Exemplo de leitura em D3**

```js
const setores = await d3.csv("setores.csv", (d) => ({
  ...d3.autoType(d),
  CD_SETOR: d.CD_SETOR,       // identificadores como texto
  CD_BAIRRO: d.CD_BAIRRO,
  vulneravel: d.vulneravel === "True",   // booleano em texto
}));
const bairros = await d3.json("bairros.geojson");
const unidades = await d3.csv("unidades.csv", (d) => ({ ...d3.autoType(d), em_fortaleza: d.em_fortaleza === "True" }));
const unidadePorId = new Map(unidades.map((u) => [u.id, u]));

// População em desassistência severa de UPA, pela via
const pop = d3.sum(setores.filter((s) => s.cls_via_upa === 2), (s) => s.populacao);
```

O `d3.autoType` converte números e transforma campos vazios em `null`.

## **Uso no QGIS**

`data/output/qgis/analise_fortaleza.gpkg` reúne, em um só arquivo, três camadas já unidas e prontas para abrir (Camada > Adicionar camada > Adicionar camada vetorial):

| Camada | Geometria | Conteúdo |
|---|---|---|
| `setores` | Polígono | Todas as colunas de `setores.csv`, mais `nome_un_*`, `carga_un_*`, `nome_un_via_*` e `carga_un_via_*` (nome e carga da unidade mais próxima de cada nível, em linha reta e pela via) |
| `bairros` | Polígono | As propriedades de `bairros.geojson` |
| `unidades` | Ponto | As colunas de `unidades.csv`, mais `tipo` (código de tipo de unidade do CNES) |

Diferenças em relação aos arquivos da aplicação: a geometria é a original (sem simplificação), todas as camadas estão em SIRGAS 2000 / UTM 24S (EPSG:31984, em metros), e os booleanos são booleanos de verdade, e não texto.
