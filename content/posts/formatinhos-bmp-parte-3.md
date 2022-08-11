---
title: "Formatinhos: BMP - Parte 3: Layout do pixel"
date: "2022-08-16T15:59:20-03:00"
author: "Arthur Mendes"
authorTwitter: "usrbinarthur"
showFullContent: false
draft: true

tags: ["code", "formats"]
keywords: ["formatinhos", "bmp", "imagem", "bitmap", "microsoft"]
archive: "2022@08"
description: |
    O terceiro episódio da série de um dos formatos de imagens historicamente mais fáceis
    de parsear.

    Nesse post você vai conseguir ler bitmaps com diferentes layouts do pixel.
---

Na [primeira parte]({{<ref "/posts/formatinhos-bmp-parte-1">}}), aprendemos o básico sobre como
ler um arquivo de bitmap.

Na [segunda parte]({{<ref "/posts/formatinhos-bmp-parte-2">}}), aprendemos a ler bitmaps com 
diferentes densidades de pixel.

Agora, iremos ver diferentes densidades de pixel.

# Densidades de pixel

Por padrão, o arquivo de bitmap possui uma densidade e organização padrão para o pixel.\
As funções `getColors` da parte anterior mostram isso.

Porém, alguns programas podem mudar essa ordem, por exemplo, para otimizar certas cores. O arquivo
que tem a "compressão" `BITFIELDS` possui essa capacidade.

Essa ordem é especificada em um tipo de valor chamado de 'bitmask'.

## Bitmask

Vem do inglês e significa, literalmente, _máscara de bits_. É basicamente um jeito de "cortar" um
valor: bits 1 são mantidos e bits 0 sao cortados. Por exemplo:

> Valor: 42069\
> Máscara: 496

Convertemos esses valores para binário


`Valor__: 1010 0100 0101 0101`\
`Máscara: 0000 0001 1111 0000`

Fazemos uma operação AND. Isso significa que, se os dois valores forem `1`, o resultado será `1`,
do contrário o resultado será `0`.

`Valor resultante: 0000 0000 0101 0000`

Convertemos de volta para decimal:

> Valor resultante: 80

Além disso, iremos fazer mais uma operação, a operação de shifting. Contamos quantos bits temos
até o primeiro valor 1 e shiftamos esse valor para a direita.

No caso, o valor é 4.

`Valor final: 0000 0000 0101 0000` >> 4 = `Valor final: 0000 0000 0000 0101`

> Valor final: 5

## Leitura

Agora que você já sabe como a operação de bitmask funciona, iremos ler esse valor.

Logo depois do header DIB (lembre-se da parte 1), se a compressão for `BITFIELDS`, temos quatro 
campos a mais:

|    Posição    | Tamanho |    Campo          | Valor          |
|---------------|---------|-------------------|----------------|
| _`0x20 (32)`_ |  _4_    | _biClrUsed_       | _`0`_          |
| _`0x24 (36)`_ |  _4_    | _biClrImportant_  | _`0`_          |
| 0x28 (40)     |  4      | bV5RedMask        | `0000 ff00`    |
| 0x2c (44)     |  4      | bV5GreenMask      | `00ff 0000`    |
| 0x30 (48)     |  4      | bV5BlueMask       | `ff00 0000`    |
| 0x34 (52)     |  4      | bV5AlphaMask      | `0000 00ff`    |

O tamanho desses valores é sempre 32 bits, mas ele é usado para extrair valores em densidades
de bits menores. Você simplesmente ignora os outros bits.

O `red mask` é usado para extrair o vermelho, o `green` é usado pra extrair o verde, o `blue` é
usado para extrair o azul e o `alpha` é usado para extrair o canal de transparência.