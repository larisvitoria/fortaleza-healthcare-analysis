<div align="center">

<img src="assets/logo-ufc.svg" alt="Universidade Federal do Ceará" height="80">

# Acesso à Saúde em Fortaleza

## **Quem está longe do cuidado? Acesso a UBS, UPAs e hospitais em Fortaleza.**

**Graduação — Visualização de Dados (CK0266) · 2026.2**

Departamento de Computação · Prof.ª Dr.ª Emanuele Marques Rodrigues Santos

Alunas: Larissa Vitória (553875) e Maria Cecília (553577)

</div>

---

## Sobre o projeto

Projeto desenvolvido durante a disciplina de Visualização de Dados da Universidade Federal do Ceará.

A análise estima o quão longe a população de Fortaleza está dos serviços públicos de saúde. Para cada setor censitário, calcula-se a distância até a UBS, a UPA e o hospital do SUS mais próximos, em linha reta e pela malha viária. Também se avalia a carga de cada unidade e como o acesso se distribui entre faixas de renda. A intenção é produzir uma aplicação interativa que permita explorar os dados e simular o impacto de novas unidades de saúde.

## Principais características

* **Acesso por setor censitário**: distância até a unidade mais próxima de cada tipo;
* **Linha reta e rede viária**: comparação usando dados do OpenStreetMap;
* **Carga das unidades**: estimativa da população atendida por unidade;
* **Recorte por renda**: desigualdade de acesso entre faixas de renda;
* **Saídas prontas**: arquivos para a aplicação web e para análise no QGIS.

## Estrutura do repositório

```text
.
├── assets/          # Imagens e logos
├── docs/            # Documentações do projeto
├── notebooks/       # Pré-processamento e análise exploratória
├── data/
│   ├── raw/         # Dados brutos
│   └── output/      # Dados gerados
└── requirements.txt
```

| Notebook | Função |
|---|---|
| [preprocessing.ipynb](notebooks/preprocessing.ipynb) | Lê os dados brutos e gera os arquivos finais |
| [exploratory_analysis.ipynb](notebooks/exploratory_analysis.ipynb) | Estatísticas descritivas e justificativa dos limiares |

## Como executar

**1. Prepare o ambiente**

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**2. Adicione os dados brutos** em `data/raw/`. As fontes e os caminhos esperados estão descritos no início do notebook de pré-processamento.

**3. Execute os notebooks, nesta ordem:**

1. `notebooks/preprocessing.ipynb`
2. `notebooks/exploratory_analysis.ipynb`

## Fontes de dados

- **IBGE**: setores censitários e população;
- **IPLANFOR**: bairros e indicadores socioeconômicos;
- **CNES**: estabelecimentos de saúde;
- **OpenStreetMap**: malha viária.

## Documentação

* [Proposta do projeto](docs/proposta.pdf): documento orientativo com as intenções iniciais e objetivos gerais;
* [Apresentação do projeto](docs/apresentacao.pdf): slides da apresentação da proposta;
* [Guia dos dados](docs/guia_dados.md): formato e uso dos arquivos gerados.

## Licença

Distribuído sob a licença descrita em [LICENSE](LICENSE).
