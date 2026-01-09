# Guia de Build (Linux e Windows)

Este documento explica como compilar e gerar o binário do servidor para esta distribuição baseada em TFS 1.7 (Protocolo 8.60).

## Requisitos
- Compilador C/C++ moderno
  - Linux: GCC 10 ou superior (g++-10)
  - Windows: MSVC (Visual Studio) com CMake
- CMake com suporte a Presets
- Ninja (recomendado)
- vcpkg para dependências C/C++
- Ferramentas de compressão: zip e unzip

## Linux (Ubuntu/Debian)

1. Instale ferramentas básicas:

```bash
sudo apt-get update
sudo apt-get install -y g++-10 gcc-10 ninja-build zip unzip
```

2. Clone vcpkg e faça o bootstrap:

```bash
git clone https://github.com/microsoft/vcpkg.git ~/vcpkg
~/vcpkg/bootstrap-vcpkg.sh
```

3. Configure variáveis de ambiente no terminal:

```bash
export VCPKG_ROOT=~/vcpkg
export CMAKE_TOOLCHAIN_FILE=$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake
export CC=gcc-10
export CXX=g++-10
```

4. Configure o projeto com o preset do vcpkg:

```bash
cmake --preset vcpkg
```

5. Compile:

```bash
cmake --build --preset vcpkg -j $(nproc)
```

6. Binário gerado:
- `build/tfs`

7. Rodar o servidor:

```bash
./build/tfs
```

Observações:
- O primeiro build via vcpkg baixa e compila dependências (Boost, OpenSSL, PugiXML, Lua, MariaDB), levando alguns minutos.
- Caso utilize uma libstdc++ sem `std::binary_semaphore`, o código possui fallback para compilar normalmente.
- Para execução correta, é necessário um `config.lua` válido e a estrutura `data/` do servidor.

## Windows (MSVC + vcpkg)

1. Instale o Visual Studio (Desktop development with C++) e CMake.
2. Instale vcpkg:
   - `git clone https://github.com/microsoft/vcpkg`
   - `.\vcpkg\bootstrap-vcpkg.bat`
3. Configure o CMake usando a toolchain do vcpkg:
   - `-DCMAKE_TOOLCHAIN_FILE=%VCPKG_ROOT%\scripts\buildsystems\vcpkg.cmake`
4. Gere e compile pelo CMake ou pela solução gerada:
   - `cmake -S . -B build -G "Ninja"`
   - `cmake --build build --config Release`
5. O executável será gerado em `build\tfs.exe`.

## CI (GitHub Actions)

Este repositório inclui um workflow que compila no Ubuntu e publica o artefato `build/tfs`:
- Arquivo: `.github/workflows/build-artifacts.yml`
- Acessar em “Actions” → execução “Build and Upload Artifacts” → seção “Artifacts”.

## Dicas
- Tenha o MySQL/MariaDB acessível e configure as credenciais no `config.lua`.
- Verifique se a versão da Lua e as dependências instaladas pelo vcpkg estão coerentes.
- Utilize `-j` com `cmake --build` para paralelismo ajustando ao número de núcleos.
