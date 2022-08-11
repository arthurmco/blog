---
title: "Formatinhos: BMP - Parte 2: Outras profundidades de cor"
date: "2022-08-14T14:40:54-03:00"
author: "Arthur Mendes"
authorTwitter: "usrbinarthur"
showFullContent: false

tags: ["code", "formats"]
keywords: ["formatinhos", "bmp", "imagem", "bitmap", "microsoft"]
archive: "2022@08"
description: |
    O segundo episódio da série de um dos formatos de imagens historicamente mais fáceis
    de parsear.

    Nesse post você vai conseguir ler todas as profundidades de cor, pelo menos do bitmap RGB.
---

Na [primeira parte]({{<ref "/posts/formatinhos-bmp-parte-1">}}), aprendemos o básico sobre como
ler um arquivo de bitmap.

Mas nós lemos apenas uma profundidade de cor

Nesse post, aprenderemos a ler as outras profundidades

O arquivo bitmap suporta 6 profundidades de cor: 32bpp, 24bpp, 16bpp, 8bpp, 4bpp, e 1bpp.

Elas são divididas em três grupos:
 - 16 bpp e maiores têm as informações de cor escritas diretamente no array de pixels.
 - 8 bpp e menores têm, no lugar das cores, um índice para uma paleta de cores, que fica entre
   o header DIB e o array de pixels.

As três primeiras são _bem fáceis_

{{< code language="javascript" title="bmp.js: Parsing dos pixels por profundidade de cor">}}
function getColors32(y, x, buffer, pixelpos) {
    // format RGBA
    return [
        get8(buffer, pixelpos(y, x) + 2),
        get8(buffer, pixelpos(y, x) + 1),
        get8(buffer, pixelpos(y, x) + 0),
        0xff
    ];
}

 function getColors24(y, x, buffer, pixelpos) {
    // format RGBA
    return [
        get8(buffer, pixelpos(y, x) + 2),
        get8(buffer, pixelpos(y, x) + 1),
        get8(buffer, pixelpos(y, x) + 0),
        0xff
    ];
}

function getColors16(y, x, buffer, pixelpos) {
    const value = get16(buffer, pixelpos(y, x));

    const cast16to24 = (v) => Math.floor(v / 32 * 255);

    const b = cast16to24((value >>> 0) & 0x1f);
    const g = cast16to24((value >>> 5) & 0x1f);
    const r = cast16to24((value >>> 10) & 0x1f);

    return [r, g, b, 0xff];
}

{{< /code >}}

> (Juntar essas funções no código anterior fica como um exercício para o leitor. Mas _não é difícil_.
> É só prestar atenção no conteúdo das funções.)

A mais complicada é 16bpp, já que, nela, cada canal de cor tem menos do que um byte.

Antes de falar dos canais com profundidade de cor menor, eu preciso falar da **paleta**

# Paleta

Obrigatoriamente, se a profundidade de cor de sua imagem for menor ou igual a 8 bits por pixel, a
paleta será sempre definida.

O tamanho da paleta é definida pelo campo `blClrUsed` (definida no código como `paletteColors`).\
Se ele for 0, e a imagem tiver 8 bits por pixel ou menor, é a quantidade máxima permitida (2 
elevado a quantidade de bits por pixel).\
Se ele for maior que 0, a quantidade de cores ali vai ser o tamanho da paleta.

Logo depois do header (ou seja, você vai até o começo do header e soma `biSize`), teremos um array
de pixels em formato BGRX.  \
Isso significa que primeiro vem o azul, depois verde, depois vermelho e depois um byte de _padding_,
que **não é o canal alfa**.

Pra quem parseou um header, ler essas cores é muito simples. É a mesma coisa que ler o array de 
pixels, na verdade.

{{< code language="javascript" title="bmp.js: Parsing da paleta de cores">}}
function getColorTable(header, buffer) {
    if (header.dibHeader.bpp > 8)
        return null;

    const colorTableOffset = header.fileHeaderSize + header.dibHeader.size;
    const tableSize = (header.dibHeader.paletteColors === 0 ) ? 
        2 ** header.dibHeader.bpp :
        header.dibHeader.paletteColors;

    let table = [];

    const getColor = (i) => {
        const off = colorTableOffset + (i * 4);
        return {
            r: get8(buffer, off),
            g: get8(buffer, off + 1),
            b: get8(buffer, off + 2),
            a: get8(buffer, off + 3)
        };

    }

    for (let i = 0; i < tableSize; i++) {
        table.push(getColor(i));
    }

    return table;
}
{{< /code >}}

# As profundidades de cor finais

Agora que você tem a tabela de cores, ler os outros formatos fica muito mais fácil.

{{< code language="javascript" title="bmp.js: Parsing dos pixels por profundidade de cor II">}}
function getColors8(y, x, buffer, pixelpos, table) {
    const { r, g, b } = table[get8(buffer, pixelpos(y, x))];
    return [b, g, r, 0xff];
}


function getColors4(y, x, buffer, pixelpos, table) {
    const byte = get8(buffer, pixelpos(y, x));
    const value = (x % 2 == 0) ? (byte >> 4) : (byte & 0xf);

    const { r, g, b } = table[value];
    return [b, g, r, 0xff];
}

function getColors1(y, x, buffer, pixelpos, table) {
    const byte = get8(buffer, pixelpos(y, x));
    const bit = x % 8;

    const value = (byte >> (7-bit)) & 1;

    const { r, g, b } = table[value];
    return [b, g, r, 0xff];
}
{{< /code >}}

Novamente, juntar essas funções vai ficar a cargo do leitor :)

-------

Agora, qualquer bitmap que o Paint gerar você consegue abrir. Mas e quanto os outros programas?

Você deve ter reparado que, por exemplo, alguns bitmaps salvos pelo Gimp não conseguem ser abertos
corretamente. Isso porque eles usam um método de "compressão" (que não é compressão, é só a 
definição do formato) chamado `BITFIELDS`. Esse método nos permite especificar exatamente quanto de
verde, vermelho e azul a imagem vai ter,

Na próxima parte, iremos ler as informações de bitfields do arquivo de bitmap.


# Referências
 - https://docs.microsoft.com/pt-br/windows/win32/gdi/bitmap-storage
 - https://datatracker.ietf.org/doc/html/rfc7903#section-1.2