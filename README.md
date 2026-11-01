# Imagens Docker para desenvolvimento

Imagens com Node.js, Go, Python, Rust ou Tauri pré-instalados, disponíveis em
Debian, Ubuntu e Alpine. Cada arquivo seleciona uma linha de versão, sem
depender da versão da linguagem oferecida pelo gerenciador de pacotes do sistema.

## Estrutura

Os Dockerfiles seguem o padrão `<tecnologia>/<distribuição>/dockerfile.v<versão>`:

```text
node/debian/dockerfile.v24
go/ubuntu/dockerfile.v1.27
python/alpine/dockerfile.v3.12
rust/debian/dockerfile.v1.99
tauri/ubuntu/dockerfile.v2
```

Todas as versões da tabela possuem variantes `debian`, `ubuntu` e `alpine`.

| Tecnologia | Versões |
| --- | --- |
| Node.js | 16, 18, 19, 20, 21, 22, 23, 24, 25, 26 |
| Go | 1.18, 1.19, 1.20, 1.21, 1.22, 1.23, 1.24, 1.25, 1.26, 1.27 |
| Python | 3.8, 3.9, 3.10, 3.11, 3.12, 3.13, 3.14, 3.15 |
| Rust | 1.90, 1.91, 1.92, 1.93, 1.94, 1.95, 1.96, 1.97, 1.98, 1.99 |
| Tauri | 1, 2 |

São 120 Dockerfiles. A matriz inclui versões históricas para compatibilidade;
nem todas as versões listadas ainda recebem atualizações dos projetos originais.

## Construir uma imagem

Execute os comandos na raiz do repositório, com Docker em execução:

```sh
docker build --pull -f node/debian/dockerfile.v24 -t kleberholtz/base-node:v24-debian node/debian

docker build --pull -f go/ubuntu/dockerfile.v1.27 -t kleberholtz/base-go:v1.27-ubuntu go/ubuntu

docker build --pull -f python/alpine/dockerfile.v3.12 -t kleberholtz/base-python:v3.12-alpine python/alpine

docker build --pull -f rust/debian/dockerfile.v1.99 -t kleberholtz/base-rust:v1.99-debian rust/debian

docker build --pull -f tauri/ubuntu/dockerfile.v2 -t kleberholtz/base-tauri:v2-ubuntu tauri/ubuntu
```

Use o padrão `kleberholtz/base-<tecnologia>:v<versão>-<distribuição>` para nomear
as imagens. Para reinstalar também os pacotes do sistema sem reutilizar camadas
em cache, acrescente `--no-cache` ao comando de build.

O build cria a imagem localmente. Para publicá-la após autenticar no Docker Hub:

```sh
docker push kleberholtz/base-node:v24-debian
```

## Usar como base

Depois de construir ou publicar a imagem desejada:

```dockerfile
FROM kleberholtz/base-node:v24-debian

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .

CMD ["node", "index.js"]
```

Cada imagem define `/app` como diretório de trabalho. As imagens Python incluem
pip e suporte a ambientes virtuais; Node inclui npm; Rust inclui Cargo e
ferramentas de compilação. Go usa `GOTOOLCHAIN=local` para evitar que um projeto
baixe e selecione automaticamente outra versão do compilador.

## Versões e atualização do sistema

Os runtimes são copiados das imagens oficiais das linguagens. As imagens de
origem e as bases do sistema estão fixadas por digest em cada Dockerfile;
isso preserva o patch selecionado mesmo quando uma tag upstream é atualizada.
O nome do Dockerfile indica a linha da versão, por exemplo `v24` ou `v1.99`.
Durante o build, cada arquivo verifica a linha instalada e as ferramentas incluídas.

As bases comuns são snapshots das tags `debian:latest`, `ubuntu:latest` e
`alpine:latest`, registrados nos próprios Dockerfiles. Primeiro são atualizados
os pacotes do sistema; no Debian e Ubuntu, dependências sem uso são removidas antes da
instalação. Os caches dos gerenciadores de pacotes são limpos ao final.

Os pacotes APT/APK continuam seguindo os repositórios da distribuição. O digest
fixa cada imagem de origem, mas não congela o conteúdo desses repositórios.
`--pull` respeita os digests definidos nos Dockerfiles e não muda a versão do runtime.

## Tauri

| Variante | CLI | Node.js | Rust | WebKitGTK | Bases |
| --- | --- | --- | --- | --- | --- |
| `v1` | 1.6.3 | 22 | 1.99 | 4.0 | Debian 12, Ubuntu 22.04, Alpine 3.20 |
| `v2` | 2.12.4 | 26 | 1.99 | 4.1 | Snapshots atuais fixados por digest |

Tauri 1 usa bases mais antigas para manter a compatibilidade com WebKitGTK 4.0.
A CLI global é chamada com `tauri`; as dependências da aplicação continuam sendo
definidas pelo próprio projeto.

As imagens preparam builds desktop Linux. Para executar uma interface gráfica,
é necessário configurar o acesso ao display. SDKs Android e iOS não estão incluídos.
No Alpine, os binários usam musl e Rust está configurado para vincular às
bibliotecas dinâmicas do sistema; Debian e Ubuntu usam glibc.

## Atualizar versões

Para atualizar um patch, altere a tag e o digest da imagem de origem no
Dockerfile correspondente e valide o build. Para adicionar uma linha de versão,
crie um arquivo em cada distribuição, ajustando a imagem de origem e a
verificação da versão instalada.

## Referências

- Imagens oficiais: [Node.js](https://hub.docker.com/_/node), [Go](https://hub.docker.com/_/golang), [Python](https://hub.docker.com/_/python) e [Rust](https://hub.docker.com/_/rust).
- Dependências Linux: [Tauri 1](https://v1.tauri.app/v1/guides/getting-started/prerequisites/) e [Tauri 2](https://v2.tauri.app/start/prerequisites/).
