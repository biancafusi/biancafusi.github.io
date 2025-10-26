---
layout: post
title: Tutorial - Como gerar condições iniciais para o MONAN/MPAS
permalink: /documentacao/condicoes_iniciais/
---

# Tutorial de como produzir condições iniciais utilizando o módulo `monan_automation`

## Requisitos

Para realizar este tutorial será necessário:

1. Instalar a versão módulo `monan_automation` com as opções de download GFS e ERA5;
2. Instalar e compilar o WPS (WRF Preprocessing System);
3. Criar uma conta no ECMWF (caso não possua);
4. Baixar o Python;
5. Baixar o ambiente conda;
6. Configurar um arquivo CDSAPI com a conta do ECMWF.

-------------------------------------------------------------------------------------

## Módulo `monan_automation`

O módulo atualizado (versão referente à data 10.10.2025) está disponível no caminho `/mnt/beegfs/op_monan/monan_automation`.

Para acompanhar as atualizações, verifique o GitHub: https://github.com/otaviomf123/monan_automation

## Instalando o WPS

Focaremos em ensinar como instalar e compilar o WPS dentro do sistema EGEON.

1) Baixar as depências necessárias; para isso recomendamos o uso do módulo disponível em https://github.com/otaviomf123/MONAN_MPAS_auto_install
2) Baixar o WPS, disponível através do GitHub: https://github.com/wrf-model/WPS
3) Compilar o WPS como indica o README no repositório do GitHub.

Ao final, deverá conter pelo menos o arquivo `ungrib.exe`.

## Criando uma conta no ECMWF

1) Acesse o site: https://cds.climate.copernicus.eu/
2) Clique em Register e preencha os dados pessoais.
3) Confirme a conta pelo e-mail enviado.

> Esta conta será usada para gerar o arquivo CDSAPI necessário para acessar os dados ERA5.

## Baixando o python

No EGEON, o Python já está disponível como módulo do sistema. Para verificar e explorar a instalação, você pode usar os seguintes comandos:

```bash
python
```
> Abre o interpretador Python, permitindo que você execute comandos e scripts interativamente. Para sair, digite `exit()` ou pressione Ctrl+D.

```bash
which python
```
> Mostra o caminho completo do executável Python que está sendo usado. Isso ajuda a confirmar onde o Python está instalado no sistema.

```
python --version
```
> Exibe a versão do Python instalada, garantindo que você esteja usando uma versão compatível (3.11 ou superior).

Para computadores pessoais:

1) Acesse https://www.python.org/downloads/
2) Escolha a versão 3.11 ou superior.
3) Siga o instalador padrão para seu sistema operacional.

## Baixando o ambiente conda

O sistema EGEON já possui esse módulo baixado. Para ativá-lo, basta executar:

```bash
module load anaconda3-2022.05-gcc-11.2.0-q74p53i
```

Assim, todos os comandos `conda` passam a ficar disponíveis no terminal. O usuário vai notar um `(base)` juntamente a linha do prompt:

```bash
(base) usuario@egeon:
```

### Recomendações

Sugerimos adicionar este comando ao arquivo .bashrc, de modo que o Conda seja carregado automaticamente sempre que abrir o terminal:

```bash
echo "module load anaconda3-2022.05-gcc-11.2.0-q74p53i" >> ~/.bashrc
source ~/.bashrc
```

### Configurando um ambiente conda

Uma das principais vantagens do Conda é criar ambientes isolados, evitando conflitos entre versões de bibliotecas. Para configurar um ambiente próprio para o `monan_automation`:

```bash
# Inicializar o Conda:
conda init

# Criar um novo ambiente chamado "monan_env" com Python 3.11
conda create -n monan_env python=3.11

# Ativar o ambiente
conda activate monan_env

# Exemplo instalando o pacote xarray
conda install -c conda-forge xarray
```
> Procure sempre utilizar o comando conda install ao invés de pip install dentro do seu ambiente conda

Se quiser verificar os ambientes existentes:

```bash
conda env list
```

E para remover um ambiente que não usa mais:

```bash
conda remove -n nome_do_ambiente --all
```

### Instalando Conda em um computador pessoal

Caso esteja configurando o ambiente fora do EGEON, você pode instalar o Miniconda (versão mais leve) ou o Anaconda (versão completa).

Instalação do Miniconda (recomendado):

1) Baixe o instalador em: https://docs.conda.io/en/latest/miniconda.html
2) Execute o instalador correspondente ao seu sistema operacional.
    Linux/MacOS (via terminal): `bash Miniconda3-latest-Linux-x86_64.sh`
3) Reinicie o terminal e teste a instalação com: `conda --version`

Se o comando retornar a versão, o Conda foi instalado corretamente.

## Configurando um arquivo CDSAPI

O CDSAPI (Climate Data Store API) é a interface que permite baixar dados do ERA5 e outros produtos do ECMWF diretamente pelo Python. Essa configuração precisa ser feita apenas uma vez por usuário.

Conforme a documentação oficial ([tutorial do ECMWF](https://cds.climate.copernicus.eu/how-to-api)), siga os passos abaixo:

1. Criar ou acessar sua conta no ECMWF

    1.1 Se ainda não possui conta, registre-se em: https://cds.climate.copernicus.eu/user/register

    1.2 Caso já tenha conta, faça login em: https://cds.climate.copernicus.eu/user

2. Localizar sua chave de acesso (API Key)

    Após logar, acesse a página: https://cds.climate.copernicus.eu/api-how-to
    Você verá um bloco de texto semelhante a este (com seus dados pessoais):

    ```bash
    url: https://cds.climate.copernicus.eu/api
    key: <PERSONAL-ACCESS-TOKEN>
    ```

3. Criar o arquivo de configuração .cdsapirc

    No seu ambiente Unix/Linux (ou WSL no Windows):
    ```bash
    nano ~/.cdsapirc
    ```
    Cole o conteúdo exibido na sua página do ECMWF, por exemplo:

    ```bash
    url: https://cds.climate.copernicus.eu/api
    key: <PERSONAL-ACCESS-TOKEN>
    ```
    
    Salve e feche o arquivo (Ctrl+O, Enter, Ctrl+X no nano).

4. Instalar o pacote cdsapi
    
    No ambiente Conda ou Python que estiver utilizando:
    ```bash
    pip install "cdsapi>=0.7.7"
    ```

5. Testar a configuração

    Para garantir que tudo está funcionando, execute no terminal Python:
    ```bash
    python -c "import cdsapi; print('CDSAPI configurado com sucesso!')"
    ``` 
    
    Se não houver erro, sua conta já está pronta para baixar dados ERA5 usando scripts em Python.

## Usando do módulo `monan_automation`

### Passo 1. Fazer uma cópia do arquivo config.yml

Executar:

```bash
cd monan_automation
cp config.yml config_teste_op.yml
```

### Passo 2. Realizar as mudanças no arquivo copiado

Bloco 1:

```python
# Configurações gerais
general:
  base_dir: "/home/bianca.fusinato/MONAN-workspace/EXPERIMENTS" #-> caminho para o diretório onde ficarão as rodadas
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

Bloco 3: Bloco onde o tipo de condição inicial e de contorno são escolhidas

```python
# Data source selection
data_source:
  type: "gfs"  # Options: "gfs" or "era5"
```

Bloco 4:

```python
# URLs e dados de entrada
data_sources:
  gfs_base_url: "https://noaa-gfs-bdp-pds.s3.amazonaws.com"
  forecast_hours:
    start: 0
    end: 240  # 10 dias * 24 horas
    step: 3   # Intervalo de 3 horas
```

Bloco 5: configuração para baixar os dados do ERA5

```python
# ERA5 specific configuration (only used when data_source.type = "era5")
era5:
  # Pressure levels (user configurable)
  pressure_levels:
    - '10'
    - '30'
    - '50'
    - '70'
    - '100'
    - '150'
    - '200'
    - '250'
    - '300'
    - '350'
    - '400'
    - '500'
    - '600'
    - '650'
    - '700'
    - '750'
    - '775'
    - '800'
    - '825'
    - '850'
    - '875'
    - '900'
    - '925'
    - '950'
    - '975'
    - '1000'
  
  # Grid resolution
  grid_resolution: '0.25/0.25'
  
  # Download interval in hours
  download_interval_hours: 3
```

Bloco 6:

```python
# Caminhos dos executáveis e arquivos
paths:
  # WPS -> é necessário ter esse módulo instalado e compilado, sugerimos criar uma pasta só para ele
  wps_dir: "/home/bianca.fusinato/WPS"
  link_grib: "/home/bianca.fusinato/WPS/link_grib.csh"
  ungrib_exe: "/home/bianca.fusinato/WPS/ungrib.exe"
  vtable_gfs: "/home/bianca.fusinato/WPS/ungrib/Variable_Tables/Vtable.GFS"
  vtable_ecmwf: "/home/cepel/otaviomf123/monan/WPS/ungrib/Variable_Tables/Vtable.ECMWF"
  
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
  stream_diagnostics: "./templates/stream_list.atmosphere.diagnostics"
  stream_output: "./templates/stream_list.atmosphere.output"
  stream_surface: "./templates/stream_list.atmosphere.surface"
  streams_atmosphere: "./templates/streams.atmosphere"
  
  # Templates de streams
  stream_init_template: "./templates/streams.init_atmosphere"
  stream_atmosphere_template: "./templates/streams.atmosphere"
  stream_diagnostics_template: "./templates/stream_list.atmosphere.diagnostics"
```
> Para copiar as informaçẽs do geogrid: os dados são disponibilizados em: https://www2.mmm.ucar.edu/wrf/users/download/get_sources_wps_geog.html#mandatory. O recomendado é baixar os dados de maior resolução (Download Highest Resolution of each Mandatory Field)

Bloco 7: O objetivo desse bloco é passar as informações do recorte espacial das condições iniciais para o WPS para que esses arquivos ocupem somente o espaço necessário. Atualmente não está funcionando

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

Bloco 8:

```python
# Physics configuration
physics:
  # Vertical grid
  nvertlevels: 55      # Number of vertical levels
  nsoillevels: 4       # Number of soil levels
  nfglevels: 34        # First-guess vertical levels
  
  # Time stepping
  dt: 30.0             # Model time step (seconds) -> a escolha desse parâmetro deve levar em conta a quantidade de pontos
  
  # Physics suite (pre-configured combinations)
  physics_suite: "mesoscale_reference_monan"
  # Options: "mesoscale_reference", "convection_permitting", "suite_HRRR_gf"
  # mesoscale_refence (dx > 10km) e a convection_permitting_monan (dx < 10km)
  
  # Microphysics schemes
  microphysics:
    scheme: "wsm6"     # Options: "wsm6", "thompson", "morrison"
    
  # Convection schemes
  convection:
    scheme: "grell_freitas"  # Options: "grell_freitas", "tiedtke", "kain_fritsch", "off"
    
  # Planetary Boundary Layer (PBL)
  pbl:
    scheme: "mynn"     # Options: "mynn", "ysu", "mrf"
    
  # Surface layer
  surface_layer:
    scheme: "sf_mynn"  # Options: "sf_mynn", "sf_sfclay"
    
  # Land surface model
  land_surface:
    scheme: "noah"     # Options: "noah", "noahmp"
    
  # Radiation schemes
  radiation:
    # Longwave radiation
    longwave:
      scheme: "rrtmg"  # Options: "rrtmg", "cam", "rrtmg_lw"
      interval: "00:30:00"  # How often to call LW radiation (HH:MM:SS)
      
    # Shortwave radiation  
    shortwave:
      scheme: "rrtmg"  # Options: "rrtmg", "cam", "rrtmg_sw"
      interval: "00:30:00"  # How often to call SW radiation (HH:MM:SS)
      
    # Radiation options
    cloud_fraction_scheme: "cld_fraction"  # Options: "cld_fraction", "cld_incidence"
    
  # Gravity wave drag
  gravity_wave_drag:
    enabled: true
    
  # Other physics options
  options:
    sst_update: false           # Update SST during run
    sstdiurn_update: false      # Diurnal SST update
    deepsoiltemp_update: false  # Deep soil temperature update
```

Bloco 9:

```python
# Configurações de execução
execution:
  mode: "slurm"       # Modo de execução: "slurm" ou "mpirun" - se colocar o mpirun roda no headnote
  cores: 256           # Número de processos MPI; note que precisa ter a mesma informação que no Bloco 8
```

Bloco 10: Caso a escolha seja slurm anteriormente

```python
# Configurações do SLURM
slurm:
  partition: "cpu"     # Partição do SLURM para submissão do job
  nodes: 2             # Número de nós a serem utilizados
  ntasks_per_node: 128 # Número de tasks por nó
  memory: "300G"       # Memória solicitada por nó
  job_name: "MPAS_model" # Nome do job no SLURM
  infiniband: "-iface ibp65s0"  # Flag infiniband para execução SLURM ( editável)
```

Bloco 11: Caso a escolha seja mpirun no Bloco 7

```python
# Configurações do mpirun
mpirun:
  hosts: []            # Lista de hosts para execução (se vazio, não usa --host)
  np:                  # Número total de processos (-np). Se não especificado, usa execution.cores
  extra_args: []       # Lista de argumentos extras para o comando mpirun
  infiniband: "-iface ibp65s0"  # Flag infiniband para mpirun ( editável)
  timeout_hours: 24    # Timeout de execução em horas
```

Bloco 12: Informações para criação de um arquivo tipo log para acompanhar os processos

```python
# Logging
logging:
  level: "INFO"
  format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
  file: "monan_execution.log"
```

Bloco 13: Informações que serão utilizadas na última etapa do monan_automation, dentro do conversor de malha não estruturada para um arquivo netcdf com a malha estruturada

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

### Desligando o efeito das cold pools dentro da parametrização de convecção Grell-Freitas

Atualmente, a parametrização de convecção está com a seguinte configuração no namelist.atmosphere:

```bash
'gf_monan': {
    'config_gf_pcvol': 0,
    'config_gf_cporg': 1, # -> cold pool ligada; 0 para desligar
    'config_gf_gustf': 1,
    'config_gf_sub3d': 0
},
```

Para realizar a ateração do parâmetro 'config_gf_cporg' para 0, ou seja, desligando as cold pools, o usuário deverá entrar em:

```bash
cd monan_automation/src/
```

E alterar a função `_generate_model_namelist` dentro do arquivo `model_runner.py`.

### Passo 3. Executando o módulo monan_automation (FAZENDO APENAS O PREPARO DAS CONDIÇÕES INICIAS)

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

#### Caso o usuário deseje realizar a rodada inteira:

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

#### Passo 3.1. Caso em que já estão baixadas as condições iniciais e deseja-se fazer a rodada

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
