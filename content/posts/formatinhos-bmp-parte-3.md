---
title: "Formatinhos: BMP - Parte 3: Layout do pixel"
date: "2022-08-16T15:59:20-03:00"
author: "Arthur Mendes"
authorTwitter: "usrbinarthur"
showFullContent: false

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

`Valor final: 0000 0000 0101 0000` >> 4 = `0000 0000 0000 0101`

> Valor final: 5

## Leitura

Agora que você já sabe como a operação de bitmask funciona, iremos ler esse valor.

Logo depois do header DIB (lembre-se da parte 1), se a compressão for `BITFIELDS`, temos quatro 
campos a mais:

```
00000020  00 00 00 00 10 00  d7 0d  00 00 d7 0d 00 00 [00 00  |................|
00000030  00 00 00 00 00 00  00 00  ff 00 00 ff 00 00  ff 00  |................|
00000040  00 00 00 00 00 ff] 00 00  00 00 00 00 00 00  00 00  |................|
00000050  00 00 00 00 00 00  00 00  00 00 00 00 00 00  00 00  |................|
```

|    Posição    | Tamanho |    Campo          | Valor                         |
|---------------|---------|-------------------|-------------------------------|
| _`0x20 (32)`_ |  _4_    | _biClrUsed_       | _`0`_                         |
| _`0x24 (36)`_ |  _4_    | _biClrImportant_  | _`0`_                         |
| 0x28 (40)     |  4      | bV5RedMask        | `00 00 ff 00`  (0x00ff_0000)  |
| 0x2c (44)     |  4      | bV5GreenMask      | `00 ff 00 00`  (0x0000_ff00)  |
| 0x30 (48)     |  4      | bV5BlueMask       | `ff 00 00 00`  (0x0000_00ff)  |
| 0x34 (52)     |  4      | bV5AlphaMask      | `00 00 00 ff`  (0xff00_0000)  |

O tamanho desses valores é sempre 32 bits, mas ele é usado para extrair valores em densidades
de bits menores. Você simplesmente ignora os outros bits.

O `red mask` é usado para extrair o vermelho, o `green` é usado pra extrair o verde, o `blue` é
usado para extrair o azul e o `alpha` é usado para extrair o canal de transparência.

Por padrão, cada pixel é organizado desse jeito:

16 bits:
```
15 14 13 12 11 10 09 08 | 07 06 05 04 03 02 01 00
-  B  B  B  B  B  G  G  | G  G  G  R  R  R  R  R
``` 


24 bits: 
```
------------------------| 23 22 21 20 19 18 17 16
-  -  -  -  -  -  -  -  | R  R  R  R  R  R  R  R

15 14 13 12 11 10 09 08 | 07 06 05 04 03 02 01 00
G  G  G  G  G  G  G  G  | B  B  B  B  B  B  B  B
``` 

32 bits: 
```
31 30 29 28 27 26 25 24 | 23 22 21 20 19 18 17 16
-  -  -  -  -  -  -  -  | R  R  R  R  R  R  R  R

15 14 13 12 11 10 09 08 | 07 06 05 04 03 02 01 00
G  G  G  G  G  G  G  G  | B  B  B  B  B  B  B  B
``` 

Traduzir isso para bitmasks será um exercício para o leitor.

Em código, a leitura e interpretação das bitmasks é a seguinte:

{{< code language="javascript" title="bmp.js: Parsing da máscara de bits">}}
function openDIBHeader(buffer) {
    /* ... */
    const dibHeader = {
        compression: parseCompression(get32(buffer, 30)),
        bitmapSize: get32(buffer, 34),
        horizontalRes: get32(buffer, 38),
        verticalRes: get32(buffer, 42),
        paletteColors: get32(buffer, 46),
        importantColors: get32(buffer, 50)
    };

    /* adicione essa linha aqui: */
    const headerBitfields = {
        red: get32(buffer, 54),
        green: get32(buffer, 58),
        blue: get32(buffer, 62),
        alpha: get32(buffer, 66),
    }
    /* é esse código acima que você tem que botar */

    return {
        size: headerSize,
        ...baseHeader,
        ...(headerSize >= 40 ? dibHeader : {}),
        // isso aqui embaixo também: 
        bitfields: (dibHeader.compression === 'bitfields' ? headerBitfields : null)
    }
}

/* Pega a bitmask e te diz quanto você precisa shiftar e em qual valor
 * você precisa dar AND para ter o valor de cor baseado na bitmask
 *
 * Não é o jeito mais otimizado de fazer isso, mas é um jeito correto.
 */
function bitmaskIntoShiftAndMask(bitmask) {
    let shift = 0;
    let mask = bitmask;

    while (!(mask & 0x1)) {
        if (mask === 0)
            break;

        mask = mask >>> 1;
        shift++;
    }

    return { shift, mask };
}

/* Pega a bitmask dos quatro canais */
function getBitmasks(bitfields) {
    if (!bitfields)
        return null;

    return {
        r: bitmaskIntoShiftAndMask(bitfields.red),
        g: bitmaskIntoShiftAndMask(bitfields.green),
        b: bitmaskIntoShiftAndMask(bitfields.blue),
        a: bitmaskIntoShiftAndMask(bitfields.alpha),
    }
}

/* Meio auto-explicativo essa :P */
function getValueFromBitmask(value, { shift, mask }) {
    return (value >>> shift) & mask;
}

/**
 * Vou deixar como exemplo uma das densidades de cor aqui, para você ver como 
 * extrair a cor do pixel baseados nas bitmasks.
 *
 * O parâmetro bitmask tem como formato o resultado da função getBitmasks()
 *
 * As outras densidades ficam a seu cargo implementar, se você quiser
 */
function getColors32(y, x, buffer, pixelpos, bitmask) {
    const value = get32(buffer, pixelpos(y, x));
    const bits = getBitmasks(bitmask);
 
    const r = getValueFromBitmask(value, bits.r);
    const g = getValueFromBitmask(value, bits.g);
    const b = getValueFromBitmask(value, bits.b);
    
    // Se não tiver alfa, faz ele ser completamente opaco.
    const a = bits.a.mask ? getValueFromBitmask(value, bits.a) : 0xff;

    // format RGBA
    return [
        r, g, b, a
    ];
}

{{< /code >}}

-----

E concluimos esse formato. Por enquanto.

O próximo formato sai daqui a alguns dias, e o repositório com um parser BMP em Javascript 
sai também daqui a uns dias

Vou começar a estudar a compressão RLE, mas ela vai vir mais como um bônus.