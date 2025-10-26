---
layout: default
title: Tutorial - Geração de Malhas MPAS/MONAN
permalink: /documentacao/geracao_malha/
---

# Geração de malhas para o sistema MONAN/MPAS

Este guia detalha o fluxo de trabalho para a geração de malhas não estruturadas e o pré-processamento estático (Static File) utilizando ferramentas como Jigsaw (desenvolvida pelo professor Pedro Peixoto/USP), MPAS-Limited-Area e módulos auxiliares em Python.

![Workflow Geração da Malha](/assets/malha.jpg)

## 1. Instalação das Ferramentas Essenciais

Para iniciar o processo de geração e manipulação de grades MPAS, é necessário configurar um ambiente Conda que inclua o `jigsaw` e o pacote `mpas-tools`.

### 1.1. Configuração do Ambiente Conda

Crie o arquivo `dev-spec.txt` com os requisitos (obtidos do repositório MPAS-Tools) e execute os comandos de instalação:

```bash
conda config --add channels conda-forge
conda create --name mpas-tools --file dev-spec.txt
```

### 1.2. Instalação do Pacote `mpas_tools`:

```bash
conda install mpas_tools
```

### 1.3. Ativação:

```bash
conda activate mpas-tools
```

## 2. Verificação dos Pacotes

Para garantir que os módulos principais estejam instalados, verifique a lista de pacotes. Os seguintes devem estar presentes:

```bash
$ conda list | grep -E 'jigsaw|mpas_tools'
# Output esperado (as versões podem variar):
jigsaw                    0.9.14               hcb278e6_1  conda-forge
jigsawpy                  0.3.3                pyhd8ed1ab_3  conda-forge
mpas_tools                1.2.2                py313h3982432_0  conda-forge
```

## 3. Geração da Malha (Grid) Utilizando Jigsaw

A malha não estruturada é gerada utilizando scripts da pasta `grids/utilities/jigsaw` do repositório MPAS-BR, focando no script `spherical` para grades globais ou regionais.

(a) **Obter Scripts:** Clone ou baixe os scripts necessários da pasta de utilitários do Jigsaw (ex: `spherical.py`).

(b) **Execução:** A geração da malha é realizada através de um script de submissão (ex: `submitt.... .sh`), que invoca o programa do Jigsaw com as configurações desejadas (ex: resolução global de 30km com refinamento de 5km sobre Goiás).

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

## 4. Geração do Static File (utilizando o `init_atmosphere_model`)

O Static File contém as informações geográficas e de relevo necessárias para a simulação. Esta etapa utiliza o executável `init_atmosphere_model` do MPAS.

### 4.1. Configuração da Área de Trabalho: 

Crie uma pasta de trabalho e linke os arquivos essenciais.

```bash
# 1. Crie e acesse o diretório de trabalho
mkdir goias_mesh_5km
cd goias_mesh_5km

# 2. Linke os arquivos de malha gerados
ln -s /caminho/para/mesh_goias5km_global30km/mesh_goias5km_global30km_mpas.nc .
ln -s /caminho/para/mesh_goias5km_global30km/mesh_goias5km_global30km_graph.info .

# 3. Linke o executável MPAS e os arquivos de configuração
ln -s /caminho/para/MONAN-Model/init_atmosphere_model .
cp /caminho/para/MONAN-Model/namelist.init_atmosphere .
cp /caminho/para/MONAN-Model/streams.init_atmosphere .
```

### 4.2 **Modificação do `namelist.init_atmosphere`:** 

Edite o arquivo para configurar os caminhos de dados geográficos e ativar as etapas de processamento.

```bash

# Ajuste o caminho para os dados geográficos:
config_geog_data_path = '/home/bianca.fusinato/MONAN-workspace/GRIDS/geog_data' # (Ajustar este caminho!)

# Certifique-se que apenas as etapas 'static_interp' e 'native_gwd_static' estão ativas:
&preproc_stages
    config_static_interp = true
    config_native_gwd_static = true
    config_vertical_grid = false
    config_met_interp = false
/
# ... outras configurações 
# Ajuste o nome do arquivo de partição gerado no jigsaw:
/
&decomposition
    config_block_decomp_file_prefix = 'mesh_rgs_graph.info'
/
```

### 4.3 **Execução:** 

Utilize o script de submissão (`run_static.slurm`) do seu ambiente HPC para rodar o `init_atmosphere_model`.

## 5. Recorte de Malha (Limitando a Área de Simulação)

Para simulações de área limitada, utiliza-se o pacote **MPAS-Limited-Area** para recortar o campo estático (`static file`) em uma região de interesse, otimizando o custo computacional.

### 5.1. Instalação do MPAS-Limited-Area

(a) Clone o Repositório e Instale as Dependências:

```bash
git clone https://github.com/MPAS-Dev/MPAS-Limited-Area.git
cd MPAS-Limited-Area
pip install -r requirements.txt
```

(b) Adicione ao PATH: Certifique-se que o executável create_region está acessível.

```bash
export PATH="${PATH}:/path/to/MPAS-Limited-Area"
```

(c)Teste de Instalação:

```bash
./create_region
# Output esperado: usage: create_region... error: the following arguments are required: points
```

### 5.2. Definição e Execução do Corte

Defina a Região (`.pts` file): Copie um exemplo (ex: `docs/points-examples/india.circle.pts`) e edite para definir sua região:

```bash
# Arquivo: goias_5km.pts
Name: goias
Type: circle
Point: -16.68, -49.27  # Latitude e Longitude (Centro da Região)
radius: 1000000.0      # Raio do corte em Metros
```

### 5.3 Execute o Recorte e Plote a Imagem: 

Use o argumento `--plot` para visualização.

```bash
./create_region --plot goias_5km.pts /local/para/campo_estatico.nc
```

*Isso gerará uma imagem (.png) para visualização. Ao retirar o `--plot`, um novo arquivo de malha recortada será gerado.*

## 6. Geração do Módulo de Partição (Para Simulação Paralela)

Para executar o modelo em múltiplos núcleos de processamento (parallelism), o arquivo de malha deve ser particionado utilizando o **METIS**.

### 6.1. Instalação das Dependências (GKlib e METIS)

Esta é uma etapa crítica que exige que as bibliotecas sejam compiladas com caminhos específicos.

(a) Instalação do GKlib (Dependência do METIS):

```bash
git clone https://github.com/KarypisLab/GKlib.git
cd GKlib
make config
make
make install
```

(b) Instalação do METIS: Ajuste o caminho do `gklib_path` conforme o diretório onde o GKlib foi instalado.

```bash
cd METIS
make config shared=1 cc=gcc prefix=~/local gklib_path=/caminho/para/GKlib/
make install
```

(c) Correção de Caminhos (Se necessário): Se o instalador do METIS procurar por bibliotecas em caminhos incorretos:

```bash
ln -s ~/local/lib64/libGKlib.a ~/local/lib/libGKlib.a
```

### 6.2. Particionamento da Malha

Use o executável `gpmetis` para gerar o arquivo de partição (`.part.XXX`):

(a) Configure o PATH e LD_LIBRARY_PATH (se necessário):

```bash
export PATH=$HOME/local/bin:$PATH
export LD_LIBRARY_PATH=$HOME/local/lib:$LD_LIBRARY_PATH
```

(b) Execute o Particionamento: (O número 256 indica o número de partições/núcleos)

```bash
cd /caminho/para/o/arquivo/goias.graph.info/
gpmetis goias.graph.info 256
```

*Isso gerará o arquivo de partição (ex: goias.graph.info.part.256).*
