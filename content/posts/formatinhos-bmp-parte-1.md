---
title: "Formatinhos: BMP - Parte 1: Abrindo um bitmap"
date: "2022-08-12T14:26:18-03:00"
author: "Arthur Mendes"
authorTwitter: "usrbinarthur"
showFullContent: false

tags: ["code", "formats"]
keywords: ["formatinhos", "bmp", "imagem", "bitmap", "microsoft"]
archive: "2022@08"
description: |
    O primeiro episódio da série começa com um dos formatos de imagens historicamente mais fáceis
    de parsear.

    Nesse post você vai printar um bitmap na tela usando Javascript.
---

O primeiro episódio da série começa com um dos formatos de imagens historicamente mais fáceis de
parsear.\
Isso é bom, já que eu posso testar a qualidade da minha explicação e ir melhorando conforme
os formatos forem ficando cada vez mais complexos.

O formato de arquivos BMP (bitmap, literalmente _mapa de bits_) é um dos formatos mais antigos de
armazenamento de imagens, criado para que máquinas mais fracas pudessem renderizar imagens com 
facilidade. Ele foi criado pela Microsoft e pela IBM para os x86. Portanto, todas as estruturas
estão em _little endian_, já que o x86 é _little endian_.

Você provavelmente já criou um arquivo BMP quando você era criança, já que era o formato principal
do **PaintBrush**, do Windows 7 pra trás.

Ele também é, se você usar os modos de cor e implementação padrões, um dos formatos mais fáceis 
de fazer _parsing_.

Ele suporta até 32 bits de cor por pixel (bpp), embora os formatos mais comuns sejam 24, 8 e 1bpp.
Inicialmente, iremos fazer o parsing de arquivos com 24 bits por pixel, depois expandiremos para
as outras profundidades de cor.

O arquivo BMP também suporta alguns métodos de organização de pixel que ele chama de _compressão_, 
embora nem todos sejam compressão:
 - RGB
 - RLE (apenas com 8 e 4 bpp)
 - BITFIELDS (uma máscara de bits define onde ficam as cores)
 - JPEG
 - PNG

Destes, os programas que eu uso apenas escrevem arquivos com **RGB** e **BITFIELDS**.\
Eu achei um programa _bem_ antigo que suporta RLE8, mas eu ainda não consegui fazer ele rodar. 
Quando eu conseguir, a gente adiciona RLE8 na lista de formatos suportados.

Mas, por ora, apenas RGB e BITFIELDS. RGB primeiro.

{{< figure src="/blog/images/formats/bmp/paint.png" caption="PaintBrush, o maior popularizador dos arquivos BMP. Fonte: bc-programming.com" >}}

Para iniciarmos, vá até [esse repositório](https://github.com/arthurmco/formatties-base), baixe
os arquivos da pasta "bmp", vá até a pasta e rode o `npm install`

Depois disso, iremos começar.

# File Header

No arquivo BMP, o _header_ é uma das primeiras informações.

```
00000000  42 4d 56 cc 0a 00 00 00  00 00 36 00 00 00 28 00  |BMV.......6...(.|
```

|    Posição   | Tamanho |    Campo    | Valor              |
|--------------|---------|-------------|--------------------|
| `0x0 (0)`    |  2      | bfType      | `42 4d` = `BM`     |
| `0x2 (2)`    |  4      | bfSize      | `0xACC56` = 707670 |
| `0x6 (6)`    |  2      | bfReserved1 | `00 00`            |
| `0x8 (8)`    |  2      | bfReserved2 | `00 00`            |
| `0xA (10)`   |  4      | bfOffBits   | `0x36`             |

 - O campo `bfType` é o nosso "Header". Existem vários formatos, mas nós só vamos nos concentrar
   no "BM", que significa "**B**it**M**ap".
 - O `bfSize` é o tamanho do arquivo. (Pode conferir se você quiser :P)
 - Os dois reservados são, bem, reservados para o aplicativo que criou o arquivo, e nós devemos 
   ignorar
 - O `bfOffBits` mostra a posição (ou offset) do bitmap, dos "pixels" da imagem, a partir do começo
   do arquivo

Nós vamos ler esse header da seguinte forma: crie um arquivo na pasta `src` chamado de
`bmp.js`. Toda nossa lógica de parsing de bitmap ficará nesse arquivo

{{< code language="javascript" title="bmp.js: Parsing do header do arquivo bitmap">}}
/* Funções auxiliares para ler valores de 1, 2 e 4 bytes 
 * Endianness ignorada por simplicidade */
const get8 = (buffer, index) => buffer[index];
const get16 = (buffer, index) => (buffer[index] | (buffer[index+1] <<< 8));
const get32 = (buffer, index) => (get16(buffer, index) | get16(buffer, index+2) <<< 16);

const hasBMPHeader = (buffer) => get16(buffer, 0) == 0x4d42;

function openBMPHeader(buffer) {
    if (!hasBMPHeader(buffer))
        return null;

    // Nós só pegamos apenas o que precisamos.
    //
    // É importante pegar o tamanho do header, já que o próximo header vai vir logo depois
    // desse, e uma tabela de cores opcional virá logo depois desses dois. Saber onde estamos
    // é muito importante!
    return {
        fileHeaderSize: 14,
        bmpSize: get32(buffer, 2),
        pixelDataOffset: get32(buffer, 10)
    }
}

{{< /code >}}

# DIB Header

O _header DIB_ vem logo depois do _header do arquivo_, e contém informações sobre os pixels
em si.

DIB significa _Device Independent Bitmap_, ou Bitmap Independente de Dispositivo. Teoricamente
ele não vai ter diferença onde quer que você veja, seja numa tela ou numa folha.

Existem vários tipos de headers, mas eles meio que se complementam, compartilhando campos 
semelhantes. Eu vou mostrar apenas um por enquanto, que é o que o Paint suporta.

```
00000000  42 4d 56 cc 0a 00 00 00  00 00 36 00 00 00 [28 00  |BMV.......6...(.|
00000010  00 00 70 02 00 00 7a 01  00 00 01 00 18 00  00 00  |..p...z.........|
00000020  00 00 20 cc 0a 00 00 00  00 00 00 00 00 00  00 00  |.. .............|
00000030  00 00 00 00 00 00]
```
O header DIB está demarcado em colchetes nesse dump. As posições (ou offsets) são relativas
ao header DIB.

|    Posição   | Tamanho |    Campo        | Valor              |
|--------------|---------|-----------------|--------------------|
| `0x0 (0)`    |  4      | biSize          | `0x28` = 40        |
| `0x4 (4)`    |  4      | biWidth         | `0x270` = 624      |
| `0x8 (8)`    |  4      | biHeight        | `0x17a` = 378      |
| `0xC (12)`   |  2      | biPlanes        | `0x0001` = 1       |
| `0xE (14)`   |  2      | biBitCount      | `0x0018` = 24      |
| `0x10 (16)`  |  4      | biCompression   | `0x0` = `BI_RGB`   |
| `0x14 (20)`  |  4      | biSizeImage     | `0xacc20` = 707616 |
| `0x18 (24)`  |  4      | biXPelsPerMeter | `0`                |
| `0x1c (28)`  |  4      | biYPelsPerMeter | `0`                |
| `0x20 (32)`  |  4      | biClrUsed       | `0`                |
| `0x24 (36)`  |  4      | biClrImportant  | `0`                |

 - `biSize` é o tamanho da estrutura, usado para diferenciar as diferentes versões. E a estrutura
   toda menos esse campo. A versão de 40 bytes é a mais comum
    - Existe uma versão da estrutura onde `biSize` = 12. Ele só vai até `biBitCount`, a imagem é
      RGB por padrão. Essa versão é usada pelos Windows 2.x e 3.0.
 - `biWidth` e `biHeight` definem a largura e a altura, respectivamente, do bitmap, em pixels.
 - Você não precisa se preocupar com `biPlanes`, é uma herança da época em que a memória gráfica era
   dividia em partes chamadas de planos. O valor padrão é 1.
 - `biBitCount` é a quantidade de bits por pixels da imagem.
    - O padrão é **24 bits**, ou seja **3 bytes por pixel** (1 byte pra vermelho, 1 pra verde e 
      1 pra azul).
 - O campo `biSizeImage` define a quantidade de bytes do bitmap, quanto de espaço os pixels ocupam.
 - Os campos que têm `PelsPerMeter` definem a resolução em pixels por metro. Geralmente esse parâmetro
   é zerado.
 - `blClrUsed` é usado pra definir uma tabela de cores. Se o bitmap tiver 16 bits por pixel ou mais,
   esse campo pode ser ignorado.
 - `blClrImportant` é usado pra mostrar a quantidade de cores necessárias para mostrar o bitmap. 
 Geralmente é 0.
 
 O parsing desse header não é nada difícil. No arquivo bmp.js, adicione essas linhas, logo
 depois do trecho anterior

{{< code language="javascript" title="bmp.js: Parsing do header DIB">}}
function parseCompression(val) {
    if (val >= 4)
        return null;
    else
        return ["rgb", "rle8", "rle4", "bitfields"][val]

}

function openDIBHeader(buffer) {
    const headerSize = get32(buffer, 14);

    const baseHeader = {
        width: get32(buffer, 18),
        height: get32(buffer, 22),
        planes: get16(buffer, 26),
        bpp: get16(buffer, 28),
        compression: 'rgb'
    };

    const dibHeader = {
        compression: parseCompression(get32(buffer, 30)),
        bitmapSize: get32(buffer, 34),
        horizontalRes: get32(buffer, 38),
        verticalRes: get32(buffer, 42),
        paletteColors: get32(buffer, 46),
        importantColors: get32(buffer, 50)
    };

    return {
        size: headerSize,
        ...baseHeader,
        ...(headerSize >= 40 ? dibHeader : {})
    }
}

{{< /code >}}

Altere as linhas da função `openBMPHeader` pra incluir o header DIB

{{< code language="javascript" title="bmp.js: Parsing do header DIB">}}
  return {
        fileHeaderSize: 14,
        bmpSize: get32(buffer, 2),
        pixelDataOffset: get32(buffer, 10)
        // Adicione essa linha aqui embaixo:
        dibHeader: openDIBHeader(buffer),
  }  
{{< /code >}}

# Pixels

O campo `bfOffBits` (no código como `pixelDataOffset` pra ficar mais claro) aponta para uma lista
de pixels, organizadas desse jeito:

![Array de pixels, em formato BGR (Blue, Green, Yellow)](/blog/images/formats/bmp/pixelgrid.png)

Cada quadrado dessa imagem é 1 byte. Primeiro vem o valor azul do pixel, depois o verde e depois
o vermelho, e por aí vai. Em 24 bits por pixel, o formato mais comum hoje em dia, é desse jeito aí:

Essa lista de pixels é dividida em tiras. Cada tira tem um pixel de altura e a largura é a largura
da imagem. O começo de cada tira de pixels deve estar numa posição divisível por 4. Se não tiver,
bytes adicionais (chamados de _padding_ são adicionados ao final da tira.)

O código pra ler isso não é muito difícil não.

{{< code language="javascript" title="bmp.js: Parsing dos pixels">}}
/// Pega o tamanho da tira, em bytes
const getRowSize = (header) => (
  (header.dibHeader.width * (header.dibHeader.bpp / 8)) + 3) & 0xfffffffc;

/**
 * Faz o parsing dos pixels do arquivo BMP, retorna um array de arrays, em formato RGBA, o
 * padrão do `canvas` do Javascript.
 * O primeiro array é uma tira, o segundo é um pixel.
 * @param {*} header
 * @param {Uint8Buffer} buffer O buffer do arquivo BMP. Eu faço o comentário assim pro
 * VSCode fazer o typecheck do javascript e o autocomplete funcionar direito.
 * @returns
 */
function parseColors(header, buffer) {
    let pixels = [];
    const rowsize = getRowSize(header);

    const pixeloff = header.pixelDataOffset;

    // Posição do pixel.
    // O Math.floor enforça uma "divisão inteira", já que o javascript não tem um operador de divisão que
    // retorne um inteiro. Isso é importante, já que não dá pra ler o byte 16.666666, é só 16 ou 17.
    // Um valor floating também vai cagar a conta mais pra frente, quando formos ler cores de arrays de
    // pixels menores que 1 byte.
    const pixelpos = (y, x) => pixeloff + (y * getRowSize(header)) + Math.floor(x * header.dibHeader.bpp/8);

    for (let y = 0; y < header.dibHeader.height; y++) {
        let row = [];

        for (let x = 0; x < header.dibHeader.width; x++) {
            row.push([
                get8(buffer, pixelpos(y, x) + 2),
                get8(buffer, pixelpos(y, x) + 1),
                get8(buffer, pixelpos(y, x) + 0),
                0xff
            ])
        }

        pixels.push(row);
    }
    return pixels;
}
{{< /code >}}

# Mostrando os valores.

Agora chegou a hora de visualizar os pixels. Vamos alterar o código em dois lugares:

Coloque isso no final do arquivo `bmp.js`

{{< code language="javascript" title="bmp.js: Parsing dos pixels">}}
function openBMPFile(buffer) {
    const header = openBMPHeader(buffer);

    if (!header)
        return null;

    return {
        header,
        colors: parseColors(header, buffer)
    };
}
{{< /code >}}

No arquivo `index.js`, no lugar do `const idata...`, coloque essa linha:

```javascript
const idata = plot(buffer, canvas);
```

A função plot vai ficar assim:

{{< code language="javascript" title="index.js: Visualização dos pixels">}}
function plot(buffer, canvas) {
    const file = openBMPFile(buffer);

    const w = file.header.dibHeader.width;
    const h = file.header.dibHeader.height;

    let imagedata = canvas.createImageData(w, h);

    for (let y = 0; y < h; y++ ) {
        for (let x = 0; x < w; x++) {
            const pos = (y*w*4) + (x*4);

            imagedata.data[pos] = file.colors[y][x][0];
            imagedata.data[pos+1] = file.colors[y][x][1];
            imagedata.data[pos+2] = file.colors[y][x][2];
            imagedata.data[pos+3] = file.colors[y][x][3];
        }
    }

    return imagedata;
}
{{< /code >}}

Agora a imagem vai funcionar...

Ou _quase_.

Você deve ter reparado que a imagem está de ponta cabeça. No formato BMP, a primeira "tira" do arquivo
fica embaixo, e a última fica em cima.

Existe um fix para isso, e é _super_ fácil, mas eu vou deixar como um exercício para o leitor.

# Parte 2

Você pode reparar que você consegue abrir quase todos os tipos de arquivos que o Paint gera.

_Quase_.

![Nós só suportamos apenas 24bpp. E esses outros?](/blog/images/formats/bmp/others.png)

Nós só suportamos o "Bitmap de 24 bits" até agora.

Tente importar os outros formatos. Veja o que acontece.

Na próxima parte iremos ver como se abrem esses outros formatos de cor. Felizmente, todos são
"rgb", mas eles têm diferentes **densidades de pixel**

# Referências
 - https://docs.microsoft.com/pt-br/windows/win32/gdi/bitmap-storage
 - https://datatracker.ietf.org/doc/html/rfc7903#section-1.2