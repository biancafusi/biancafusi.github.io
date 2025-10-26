---
layout: page
title: Tutorial - Geração de Malhas MPAS/MONAN
permalink: /documentacao/geracao_malha/
---

# Geração de malhas

Este guia detalha o fluxo de trabalho para a geração de malhas não estruturadas e o pré-processamento estático (Static File) utilizando ferramentas como Jigsaw (desenvolvida pelo professor Pedro Peixoto/USP), MPAS-Limited-Area e módulos auxiliares em Python.

(inserir figura com o workflow)

1. Instalação das Ferramentas Essenciais

Para iniciar o processo de geração e manipulação de grades MPAS, é necessário configurar um ambiente Conda que inclua o `jigsaw` e o pacote `mpas-tools`.

1.1. Configuração do Ambiente Conda

Crie o arquivo `dev-spec.txt` com os requisitos (obtidos do repositório MPAS-Tools) e execute os comandos de instalação:

```bash
conda config --add channels conda-forge
conda create --name mpas-tools --file dev-spec.txt
```

1.2. 