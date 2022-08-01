+++
title = "Coisas interessantes sobre patchfiles"
date = "2022-03-13T14:28:37-03:00"
author = "Arthur M"
authorTwitter = "usrbinarthur" #do not include @
cover = "images/about-patch-cover.png"
tags = ["tips"]
keywords = ["patch", "diff", "git diff"]
description = """
Hoje eu precisei editar um arquivo .patch na mão. Achei interessante
compartilhar como é o formato, já que não é algo que se vê todo dia
"""
showFullContent = false
archive = "2022@03"
+++

Hoje eu precisei editar um arquivo `patch` na mão, algo relacionado
com um projeto pessoal meu.

Para quem não sabe, arquivos `patch` são aqueles com extensão .patch.
Eles têm um monte de linhas que começam com `+` e `-`.  
A sintaxe desse arquivo é a mesma sintaxe que o comando `git diff`
mostra em um repositório válido, 

Um arquivo diff é mais ou menos assim:

```diff

diff --git a/src/common/CMakeLists.txt b/src/common/CMakeLists.txt
index edeeeec..fcbe64c 100644
--- a/src/common/CMakeLists.txt
+++ b/src/common/CMakeLists.txt
@@ -48,13 +48,11 @@ add_dependencies(familyline-common input-flatbuffer input-ser-flatbuffer network
 target_compile_features(familyline-common PRIVATE cxx_std_20)
 
 find_package(fmt 6...7.2 CONFIG REQUIRED)
-find_package(ZLIB REQUIRED)
 find_package(tl-expected 1 CONFIG REQUIRED)
 find_package(nlohmann_json REQUIRED)
 find_package(FlatBuffers REQUIRED)
 find_package(SDL2 CONFIG)
 find_package(range-v3 CONFIG)
-find_package(glm REQUIRED)
 
 if (FLINE_USE_VCPKG)
   include(${CMAKE_TOOLCHAIN_FILE})
@@ -79,8 +77,9 @@ else()
   target_link_libraries(familyline-common PUBLIC ${CURLPP_LDFLAGS})
   target_include_directories(familyline-common PUBLIC ${CURLPP_INCLUDE_DIRS})
   endif()
+  pkg_search_module(ZLIB REQUIRED zlib)
 
-  target_link_libraries(familyline-common PUBLIC fmt::fmt glm::glm ${ZLIB_LIBRARIES})
+  target_link_libraries(familyline-common PUBLIC fmt::fmt glm ${ZLIB_LIBRARIES})
   target_link_libraries(familyline-common PUBLIC tl::expected range-v3::range-v3)
   target_link_libraries(familyline-common PUBLIC nlohmann_json::nlohmann_json)
   target_include_directories(familyline-common PUBLIC
diff --git a/src/common/generated.cmake b/src/common/generated.cmake
index d30ff1c..b105a41 100644
--- a/src/common/generated.cmake
+++ b/src/common/generated.cmake
@@ -5,6 +5,8 @@
 # called INPUT_FLATBUFFER_INCLUDE, where the generated header file
 # is.
 
+# TODO: solve the issue of us having to run cmake twice to get the flatbuffers path
+
 include_guard(GLOBAL)
 
 include("${CMAKE_SOURCE_DIR}/cmake/BuildFlatBuffers.cmake")
diff --git a/tools/bump_version.py b/tools/bump_version.py
index 5b79583..46f736c 100644
--- a/tools/bump_version.py
+++ b/tools/bump_version.py
@@ -127,7 +127,7 @@ update_cmakelists(current_ver, next_ver)
 ####################
 
 os.system("git add CMakeLists.txt")
-os.system("git commit -m \"Bump version (v{} -> v{})\"".format(current_ver, next_ver))
+os.system("git commit --no-verify -m \"Bump version (v{} -> v{})\"".format(current_ver, next_ver))
 
 ####################


```

Existem vários formatos de arquivos de patch. O mais comum é esse aí de cima,
chamado de `unified format`

Um arquivo de patch pode conter vários arquivos. Eles são delimitados
por duas coisas:
 
 - o comando usado pra gerar o diff:  
 ```
diff --git a/tools/bump_version.py b/tools/bump_version.py
index 5b79583..46f736c 100644 
 ```

 - o `header` do arquivo:  
 ```
 --- a/tools/bump_version.py
 +++ b/tools/bump_version.py
 ```
 
 Opcionalmente, esse header pode conter uma data. É útil para ver se o
 seu arquivo é mais novo ou mais velho do que o patch
 
 ```
 --- a/tools/bump_version.py    2022-03-07 01:34:08.815661222 -0300
 +++ b/tools/bump_version.py    2022-03-09 18:29:20.993654217 -0300 
 ```
 
 O `---` delimita o arquivo "antigo". O `+++` delimita o arquivo
 "novo", que vai surgir depois do path.
 
 O que conta pro formato é o caminho depois do `a/` ou do `b`, esses
 dois nomes são só de referência
 
Depois disso, para cada arquivo, nós temos vários "chunks", pedaços de alteração.

```diff
@@ -127,7 +127,7 @@ update_cmakelists(current_ver, next_ver)
 ####################
 
 os.system("git add CMakeLists.txt")
-os.system("git commit -m \"Bump version (v{} -> v{})\"".format(current_ver, next_ver))
+os.system("git commit --no-verify -m \"Bump version (v{} -> v{})\"".format(current_ver, next_ver))
 
 ####################
```

O formato é mais ou menos assim:

`@@ -linhaA, quantidadeA +linhaB, quantidadeB @@`

A `linhaA` é o número da linha onde o chunk começa, onde a alteração
vai acontecer no arquivo antigo. A `quantidadeA` é a quantidade do
número de linhas nesse "chunk" no arquivo original

A `linhaB` é o número da linha onde o chunk começa, onde a alteração
vai acontecer no arquivo novo. A `quantidadeB` é a quantidade do
número de linhas nesse "chunk" depois que todas as alterações forem
aplicadas.

Por exemplo, nesse chunk aí de cima, ele começa na linha 127 do
arquivo, temos 7 linhas nele antes das alterações, e 7 linhas nele
depois

Nesse aqui: 

```diff
@@ -48,13 +48,11 @@ add_dependencies(familyline-common input-flatbuffer input-ser-flatbuffer network
 target_compile_features(familyline-common PRIVATE cxx_std_20)
 
 find_package(fmt 6...7.2 CONFIG REQUIRED)
-find_package(ZLIB REQUIRED)
 find_package(tl-expected 1 CONFIG REQUIRED)
 find_package(nlohmann_json REQUIRED)
 find_package(FlatBuffers REQUIRED)
 find_package(SDL2 CONFIG)
 find_package(range-v3 CONFIG)
-find_package(glm REQUIRED)
 
 if (FLINE_USE_VCPKG)
   include(${CMAKE_TOOLCHAIN_FILE})
```

ele começa na linha 48 do arquivo original, temos 13 linhas nele antes
das alterações e teremos 11 linhas depois.
 
Depois disso, vêm algumas linhas do arquivo original, para dar
contexto.

As linhas sem nada antes são linhas que não sofrerão alterações.

As linhas com `-` serão removidas.

As linhas com `+` serão adicionadas.

-----

E é isso.

É um formato relativamente simples, mas bem útil para dizer
alterações.

Se você quiser mais detalhes sobre o formato, pode vir [nesse post do
Guido van
Rossum](https://www.artima.com/weblogs/viewpost.jsp?thread=164293),
onde ele escreveu uma mini-especificação do formato.

