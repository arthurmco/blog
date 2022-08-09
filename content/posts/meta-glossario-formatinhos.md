---
title: "Formatinhos: Prelúdio"
date: "2022-08-11T14:25:28-03:00"
author: "Arthur Mendes"
authorTwitter: "usrbinarthur"
showFullContent: false

tags: ["meta", "code"]
keywords: ["formatinhos", "estrutura"]
archive: "2022@08"
---

Amanhã é o lançamento da minha série dos formatinhos, com o primeiro episódio sendo o dos
arquivos **.bmp**

Os posts da série terão descrições padronizadas de algumas estruturas de dados. Alguns de vocês
podem estar familiarizados com esse tipo de descrição, mas, como eu tento escrever para os
júniores e as sandys, eu vou explicar.

Se você começar a ler esse post e se pegar pensando "Eu já sei isso", então você já pode começar
a ver a série dos formatinhos.

<!--more-->

-------

> Antes de mais nada, leia isso:
> - [Numeração hexadecimal](https://pt.wikipedia.org/wiki/Sistema_de_numera%C3%A7%C3%A3o_hexadecimal)
> - [Extremidade (ou _endianness_)](https://pt.wikipedia.org/wiki/Extremidade_(ordena%C3%A7%C3%A3o))
> 
> Vai ser importante lá na frente!


O código será mostrado assim (aprendi a formatar o código desse jeito nessa semana, e fica bem
mais bonito, dá pra expandir e fica melhor em mobile.)

{{< code language="c++" title="Código exemplo">}}
#include <cstdint>
#include <string>

struct Teste {
    int8_t val1;
    int8_t val2;
    std::string str;
}

int testa_o_teste(const Teste& teste, int valor) {
    // aaaaaaa
}

{{< /code >}}

Eu vou tentar ser o mais didático possível no código. Se algo não ficar claro, você pode comentar.

Em algumas partes do post, eu vou precisar mostrar dumps hexadecimais e estruturas de dados, 
geralmente juntos.

Um dump hexadecimal é algo parecido com isso:\
(Ele quebra em mobile :(. Até eu resolver isso, veja esse dump com o celular na horizontal)

```
00000000  45 58 45 4d 00 04 00 00  00 04 00 00 20 03 8f 3d  |EXEM........ ..=|
00000010  18 00 00 00 20 03 00 00  41 69 20 71 75 65 20 73  |.... ...Ai que s|
00000020  61 63 6f 00                                       |aco.|
00000024
```

O dump acima está em um formato padrão (de um programa chamado `hexdump`). Todos os valores
das três primeiras colunas estão em hexadecimal.

A primeira coluna mostra um valor que representa a posição do byte logo depois dela, 
em hexadecimal. (`00000010` = 16 em decimal, `00000020` = 32 em decimal, `00000024` = 36).

A segunda coluna mostra os bytes 0 até 7 da linha. Ou seja: 

```
00000000  45 58 45 4d 00 04 00 00 
```
O primeiro byte é `45`, o segundo é `58`, o terceiro é `45` denovo...

```
00000010  18 00 00 00 20 03 00 00
```

O décimo sexto byte é `18`, do 17 ao 19 são `00`, o vigésimo byte, por coincidéncia, é `20`...

E por aí vai

A terceira coluna mostra os bytes 8 até 15. Mesmo esquema.

A quarta coluna é a representação, em ASCII, dos bytes daquela linha:

```
00000000  45 58 45 4d 00 04 00 00  00 04 00 00 20 03 8f 3d  |EXEM........ ..=|
```

'E' = `45`, 'X' = `58`, 'M' = `4d`... você entendeu. Os valores que não têm um caractere 
correspondente printável são representados com um ponto.


Geralmente, o dump vai vir acompanhado de uma tabela mostrando o que tem dentro do arquivo.\
Cada linha da tabela é um **campo**, um valor que significa alguma coisa.\
Por exemplo:

|    Posição   | Tamanho |    Campo    | Valor           |
|--------------|---------|-------------|-----------------|
| `0x0 (0) `   |  4      | MAGIC       | `45 58 45 4d`   |
| `0x4 (4) `   |  4      | width       | `0x400` = 1024  |
| `0x8 (8) `   |  4      | height      | `0x400` = 1024  | 
| `0xC (12) `  |  1      | bit depth   | `0x20`  = 32    |
| `0xD (13) `  |  1      | compression | `0x3` = RLE8    |
| `0xE (14) `  |  2      | checksum    | `8f 3d`         |
| `0x10 (16) ` |  4      | name offset | `0x18`          |
| `0x14 (20) ` |  4      | data offset | `0x320`         |

Na coluna **Posição** eu mostro em qual posição (tanto em decimal quanto em hexadecimal) o campo 
fica dentro do arquivo. A posição sempre vai ser em bytes.

O **Tamanho** é o tamanho, em bytes, daquele campo.

O **campo** é o nome do campo, como ele está na documentação. Se a documentação for informal, vai
ser um nome que eu vou inventar e que descreve o campo.

O **valor** é o valor atual do campo, pra você não precisar gastar neurônios caçando a posição 
exata do campo. Os valores que significam números estão convertidos, e os zeros extras são 
removidos.

Eu geralmente vou mostrar campos com tamanho variável separadamente. Por exemplo:

- O campo `name offset` aponta para uma _string_ terminada em `NUL` (`"Ai que saco"`).

Uma string terminada em `NUL` é uma string que termina em um byte `0`. É a string que o C usa.

-----

E é isso.

Se você sentiu que eu não expliquei algo direito, comente aí que eu altero a postagem explicando 
melhor.

Amanhã a gente começa!