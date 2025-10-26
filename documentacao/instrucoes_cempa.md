---
layout: default
title: Tutorial - Instruções para a operação do projeto CEMPA
permalink: /documentacao/instrucoes_cempa/
---

# Passo 1. Copiar e compilar o MONAN

`/home/bianca.fusinato/MONAN-workspace/MONAN-latest/MONAN_Phys_SRF_v1` -> Já compilado

Para baixar e compilar:

```bash
git clone https://github.com/saulorfreitas/MONAN_Phys_SRF_v1
cd MONAN_Phys_SRF_v1
git checkout develop # usar para ficar na branch develop
```

Para compilar:

```bash
cd MONAN_Phys_SRF_v1
make clean CORE=init_atmosphere
make gfortran CORE=init_atmosphere
make clean CORE=atmosphere
make gfortran CORE=atmosphere
```

## Passo 1.1. Instalação das dependências

Caso o usuário opte por baixar e compilar, uma possibilidade é seguir os passos disponíveis [aqui](https://github.com/otaviomf123/MONAN_MPAS_auto_install)

## Passo 1.2. Geração das tabelas auxiliares para utilizar a física convection_permitting_monan

Executar:

```bash
cd MONAN_Phys_SRF_v1
./build_tables 
```

# Passo 2. Copiar a versão mais atual do módulo `monan_automation`

Executar:

```bash
cp /home/bianca.fusinato/MONAN-workspace/monan_automation.zip
unzip monan_automation.zip
```

Para baixar as versões mais atualizadas:

```bash
cd dir_trabalho
git clone https://github.com/otaviomf123/monan_automation.git
```

## Passo 2.1. Configurando o módulo `monan_automation`

Antes de utilizar o módulo, instale as dependências necessárias.

```bash
cd monan_automation
pip install -r requirements.txt
```

Também é possível instalar em um ambiente conda. A principal vantagem é manter os pacotes isolados do sistema.

```bash
cd monan_automation
conda create -n monan_env --file requirements.txt
conda activate monan_env
```

# Passo 3. Fazer uma cópia do arquivo config.yml

Executar:

```bash
cd monan_automation
cp config.yml config_teste_op.yml
```

# Passo 4. Realizar as mudanças no arquivo copiado

Bloco 1: 

```python
# Configurações gerais
general:
  base_dir: "/home/bianca.fusinato/MONAN-workspace/EXPERIMENTS" #-> caminho para o diretório onde ficaram as rodadas
  forecast_days: 1  # Dias de previsão
```

Bloco 2: 

```python
# Configurações de data
dates:
  run_date: "20250916"  # Data da rodada (AAAAmmdd)
  cycle: "00"           # Hora da rodada (00, 06, 12, 18)
  start_time: "2025-09-16_00:00:00"
  end_time: "2025-09-17_00:00:00"
```

Bloco 3: 

```python
# URLs e dados de entrada
data_sources:
  gfs_base_url: "https://noaa-gfs-bdp-pds.s3.amazonaws.com"
  forecast_hours:
    start: 0
    end: 24  # dias * 24 horas 
    step: 3   # Intervalo de 3 horas
```

Bloco 4: 

```python
# Caminhos dos executáveis e arquivos
paths:
  # WPS -> é necessário ter esse módulo instalado e compilado, sugerimos criar uma pasta só para ele
  wps_dir: "/home/bianca.fusinato/WPS"
  link_grib: "/home/bianca.fusinato/WPS/link_grib.csh"
  ungrib_exe: "/home/bianca.fusinato/WPS/ungrib.exe"
  vtable_gfs: "/home/bianca.fusinato/WPS/ungrib/Variable_Tables/Vtable.GFS"
  
  # MPAS/MONAN
  mpas_init_exe: "/home/bianca.fusinato/MONAN-workspace/MONAN-latest/MONAN_Phys_SRF_v1/init_atmosphere_model"
  monan_exe: "/home/bianca.fusinato/MONAN-workspace/MONAN-latest/MONAN_Phys_SRF_v1/atmosphere_model"
  monan_dir: "/home/bianca.fusinato/MONAN-workspace/MONAN-latest/MONAN_Phys_SRF_v1"
  
  # Dados geográficos e malha
  geog_data_path: "/home/bianca.fusinato/MONAN-workspace/mpas_static"
  wps_geog_path: "/home/bianca.fusinato/WPS/geogrid/" #-> é uma pasta com todas essas informações geofísicas
  static_file: "/home/bianca.fusinato/MONAN-workspace/GRIDS/goias_mesh_5km/goias5km_global30km_mpas.static.nc" #-> informações da malha
  decomp_file_prefix: "/home/bianca.fusinato/MONAN-workspace/GRIDS/goias_mesh_5km/goias.graph.info.part." #-> arquivo de partição da malha
  
  # Arquivo de condição inicial
  init_filename: "brasil_circle.init.nc" #-> nome da condição inicial, "brasil_circle.init.nc" é a default
  
  # Arquivos de streams (templates)
  stream_diagnostics: "/home/bianca.fusinato/MONAN-workspace/EXPERIMENTS/streams/stream_list.atmosphere.diagnostics"
  stream_output: "/home/bianca.fusinato/MONAN-workspace/EXPERIMENTS/streams/stream_list.atmosphere.output"
  stream_surface: "/home/bianca.fusinato/MONAN-workspace/EXPERIMENTS/streams/stream_list.atmosphere.surface"
  streams_atmosphere: "/home/bianca.fusinato/MONAN-workspace/EXPERIMENTS/streams/streams.atmosphere"
  
  # Templates de streams
  stream_init_template: "./templates/streams.init_atmosphere"
  stream_atmosphere_template: "./templates/streams.atmosphere"
  stream_diagnostics_template: "./templates/stream_list.atmosphere.diagnostics"
```

Bloco 5: O objetivo desse bloco é passar as informações do recorte espacial das condições iniciais para o WPS para que esses arquivos ocupem somente o espaço necessário. Atualmente **não está funcionando**

```python
# Configurações do domínio
domain:
  dx: 15000
  dy: 15000
  ref_lat: 33.00
  ref_lon: -79.00
  truelat1: 30.0
  truelat2: 60.0
  stand_lon: -79.0
  e_we: 150
  e_sn: 130
```

Bloco 6:

```python
# Configurações de física
physics:
  nvertlevels: 55
  nsoillevels: 4
  nfglevels: 34
  dt: 30.0 #-> a escolha desse parâmetro deve levar em conta a quantidade de pontos
  physics_suite: "convection_permitting_monan" # opções: mesoscale_refence (dx > 10km) e a convection_permitting_monan (dx < 10km)
                                                # nesse convection_permitting_monan está já configurado para ter o efeito das cold pools
```

Bloco 7:

```python
# Configurações de execução
execution:
  mode: "slurm"       # Modo de execução: "slurm" ou "mpirun" - se colocar o mpirun roda no headnote
  cores: 256           # Número de processos MPI; note que precisa ter a mesma informação que no Bloco 8
```

Bloco 8: Caso a escolha seja *slurm* anteriormente

```python
# Configurações do SLURM
slurm:
  partition: "fat"
  nodes: 2
  ntasks_per_node: 128 #-> ter em mente que ao mudar aqui precisa ter o arquivo de decomposição da malha
  memory: "300G"
  job_name: "MPAS_model" 
```

Bloco 9: Caso a escolha seja mpirun no Bloco 7

```python
# Configurações do mpirun
mpirun:
  host: "localhost"            # Host de destino para execução
  mpi_extra_args: ""           # Argumentos adicionais do mpirun (opcional)
  timeout_hours: 24            # Timeout de execução em horas
```

Bloco 10: Informações para criação de um arquivo tipo log para acompanhar os processos

```python
# Logging
logging:
  level: "INFO"
  format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
  file: "monan_execution.log"
```

Bloco 11: Informações que serão utilizadas na última etapa do `monan_automation`, dentro do conversor de malha não estruturada para um arquivo netcdf com a malha estruturada

```python
# Configurações de conversão para grade regular
conversion:
  enabled: true  # Habilitar conversão automática
  grid: #-> a configuração padrão é para a América do Sul, poderá ser diminuída.
    lon_min: -90
    lon_max: -20
    lat_min: -45
    lat_max: 25
    resolution: 0.046  # Resolução em graus que sairá o dado
    max_dist_km: 30  # Distância máxima para interpolação em km; i.e, irá procurar o vizinho mais próximo em até 30km
```

## Passo 4.1. Instalando o WPS

Um tutorial se encontra em https://github.com/wrf-model/WPS.

É necessário gerar o arquivo `ungrib.exe` para produzir as condições iniciais e de contorno.

## Passo 4.2. Copiando as informaçẽs do `geogrid`

Os dados são disponibilizados em: https://www2.mmm.ucar.edu/wrf/users/download/get_sources_wps_geog.html#mandatory

O recomendado é baixar os dados de maior resolução (*Download Highest Resolution of each Mandatory Field*)

## Passo 4.3. Copiando a malha e os arquivos de particionamento

Para executar o MONAN, é necessário ter acesso a dois tipos de arquivos:  

- **Arquivo *static*** (`goias5km_global30km_mpas.static.nc` → Combina informações da malha com dados geográficos.)  
- **Arquivos de particionamento** (`goias.graph.info.part.*` → O número após o sufixo (32, 40, 64, 128, 192, 256, 384) indica a quantidade de partições, igual ao número de processadores que serão utilizados)  

Ainda, caso o usuário deseje o arquivo com as informações da grade: `goias.region.nc` → Arquivo da grade gerada e recortada.

Todos eles estão disponíveis em:

```bash
cd /mnt/beegfs/bianca_fusinato/MONAN-operacao/grade
```

## Passo 4.3. Copiando a malha e os arquivos de particionamento

Para ter acesso ao arquivo que contém a grade gerada, ao arquivo *static* que combina as informações da grade gerada com a parte geográfica do modelo e aos arquivos de particionamento da malha, basta acessar e copiar os arquivos disponíveis em:

```bash
cd /mnt/beegfs/bianca_fusinato/MONAN-operacao/grade
```

Os arquivos do tipo `goias.graph.info.part.` são os arquivos de particionamento e o número no final é o tanto de partições que há conforme o número de processadores que serão utilizados (32, 40, 64, 128, 192, 256, 384). Para produzir mais arquivos desses, é necessário a ferramenta METIS disponível nesse [link](https://github.com/KarypisLab/METIS.git).

Já o arquivo `goias5km_global30km_mpas.static.nc` é o arquivo *static* necessário no `stream.atmosphere` do MONAN.

Por fim, o arquivo `goias.region.nc` é a grade gerada e recortada.

## Passo 4.4. Escolha do parâmetro *dt* em *physics*

O parâmetro **`dt`** representa o passo de integração do modelo (em segundos) e deve ser definido em função da resolução horizontal da grade (**`dx`**, em quilômetros).  

A regra utilizada é: dt = 6 * dx

### Exemplo prático
No caso da malha `goias_region.nc`, a resolução horizontal é de **5 km**.  
Aplicando a regra: dt = 6 * 5 = 30 segundos

Portanto, o valor adequado de **`dt`** a ser configurado é **30 segundos**.

## Passo 4.5. Desligando o efeito das cold pools dentro da parametrização de convecção Grell-Freitas

Atualmente, a parametrização de convecção está com a seguinte configuração no `namelist.atmosphere`:

```bash
'gf_monan': {
    'config_gf_pcvol': 0,
    'config_gf_cporg': 1, # -> cold pool ligada; 0 para desligar
    'config_gf_gustf': 1,
    'config_gf_sub3d': 0
},
```

Para realizar a ateração do parâmetro `'config_gf_cporg'` para 0, ou seja, desligando as cold pools, o usuário deverá entrar em:

```bash
cd monan_automation/src/
```

E alterar a função `_generate_model_namelist` dentro do arquivo `model_runner.py`:

# Passo 5. Executando o módulo `monan_automation`

Faça as verificações básicas:

```bash
cd monan_automation/src
python verify_setup.py
```

(RECOMENDADO PARA PRIMEIRA VEZ) Execute cada um dos processos do módulo separadamente:

```bash
cd monan_automation
python main.py --config config_teste_op.yml --step download
```

```bash
cd monan_automation
python main.py --config config_teste_op.yml --step wps
```

```bash
cd monan_automation
python main.py --config config_teste_op.yml --step init
```

```bash
cd monan_automation
python main.py --config config_teste_op.yml --step boundary
```

```bash
cd monan_automation
python main.py --config config_teste_op.yml --step run
```

```bash
cd monan_automation
python main.py --config config_teste_op.yml --step convert
```

Posteriomente, poderá ser realizado esses processos de uma vez só, uma vez que já foi garantido que cada um dos passos foram executados sem erro:

```bash
cd monan_automation
python main.py --config config_teste_op.yml
```

## Passo 5.1. Caso em que já estão baixadas as condições iniciais

Caso o usuário já tenha as condições iniciais baixadas, deverá colocar ela dentro da pasta criada pelo módulo:

```bash
cd dir_experimento/YYYYMMDD/gfs/
cp dados_ja_baixados .
```

Então, deverá ser executado por passos, iniciando a partir do `--step wps`:

```bash
cd monan_automation
python main.py --config config_teste_op.yml --step wps
```

```bash
cd monan_automation
python main.py --config config_teste_op.yml --step init
```

```bash
cd monan_automation
python main.py --config config_teste_op.yml --step boundary
```

```bash
cd monan_automation
python main.py --config config_teste_op.yml --step run
```

```bash
cd monan_automation
python main.py --config config_teste_op.yml --step convert
```