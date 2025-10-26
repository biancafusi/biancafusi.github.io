---
layout: default
title: Tutorial - Geração de Malhas MPAS/MONAN
permalink: /documentacao/geracao_malha/
---

# Geração de malhas

Este guia detalha o fluxo de trabalho para a geração de malhas não estruturadas e o pré-processamento estático (Static File) utilizando ferramentas como Jigsaw (desenvolvida pelo professor Pedro Peixoto/USP), MPAS-Limited-Area e módulos auxiliares em Python.

![Workflow Geração da Malha](/home/bianca/Documentos/cempa/github_page/biancafusi.github.io/assets/malha.jpg)

1. Instalação das Ferramentas Essenciais

Para iniciar o processo de geração e manipulação de grades MPAS, é necessário configurar um ambiente Conda que inclua o `jigsaw` e o pacote `mpas-tools`.

1.1. Configuração do Ambiente Conda

Crie o arquivo `dev-spec.txt` com os requisitos (obtidos do repositório MPAS-Tools) e execute os comandos de instalação:

```bash
conda config --add channels conda-forge
conda create --name mpas-tools --file dev-spec.txt
```

1.2. Instalação do Pacote `mpas_tools`:

```bash
conda install mpas_tools
```

1.3. Ativação:

```bash
conda activate mpas-tools
```

2. Verificação dos Pacotes

Para garantir que os módulos principais estejam instalados, verifique a lista de pacotes. Os seguintes devem estar presentes:

```bash
$ conda list | grep -E 'jigsaw|mpas_tools'
# Output esperado (as versões podem variar):
jigsaw                    0.9.14               hcb278e6_1  conda-forge
jigsawpy                  0.3.3                pyhd8ed1ab_3  conda-forge
mpas_tools                1.2.2                py313h3982432_0  conda-forge
```

3. Geração da Malha (Grid) Utilizando Jigsaw

A malha não estruturada é gerada utilizando scripts da pasta `grids/utilities/jigsaw` do repositório MPAS-BR, focando no script `spherical` para grades globais ou regionais.

(a) *Obter Scripts:* Clone ou baixe os scripts necessários da pasta de utilitários do Jigsaw (ex: `spherical.py`).

(b) *Execução:* A geração da malha é realizada através de um script de submissão (ex: `submitt.... .sh`), que invoca o programa do Jigsaw com as configurações desejadas (ex: resolução global de 30km com refinamento de 5km sobre Goiás).

> Exemplo de script de subimissão:

```bash
#!/bin/bash
#SBATCH --job-name=grid_goias
#SBATCH --partition=fat
#SBATCH --time=2-00:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=32G
#SBATCH --output=grid_goias.out
#SBATCH --error=grid_goias.err

# Roda o script para gerar a malha refinada em Goiás
python spherical_grid.py \
    -g localref \
    -r 5 \              # resolução mais fina, aqui 5km
    -l 30 \             # resolução mais grossa, aqui 30km
    -rad 500 \          # raio interno, aqui 500km
    -tr 750 \           # raio externo, aqui 750km
    -clat -16.00 \      # lat central
    -clon -49.27 \      # lon central
    -o mesh_goias_v2 \
    -p 0 >> grid_goias.log 2>&1
```

4. Geração do Static File (utilizando o `init_atmosphere_model`)

O Static File contém as informações geográficas e de relevo necessárias para a simulação. Esta etapa utiliza o executável `init_atmosphere_model` do MPAS.

4.1. *Configuração da Área de Trabalho:* Crie uma pasta de trabalho e linke os arquivos essenciais.