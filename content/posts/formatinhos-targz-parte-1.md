---
title: "Formatinhos: TAR.GZ - Parte 1: TAR padrão"
date: "2022-08-19T23:22:11-03:00"
author: "Arthur Mendes"
authorTwitter: "usrbinarthur"
showFullContent: false

tags: ["code", "formats"]
keywords: ["formatinhos", "arquivamento", "compactação", "tar", "gzip", "gnu"]
archive: "2022@08"
description: |
    O segundo formato da série é um dos formatos mais tradicionais de arquivamento e compressão
    dos Unixes, o `tar`, mais especificamente a sua variante comprimida, `.tar.gz`.

    O parsing desse formato, surpreendentemente, é extremamente simples, o que me faz pensar
    o motivo de eu não ter aberto a série com ele.
---

O `tar` é o formato mais tradicional de arquivamento do universo Unix, sendo usado desde o final da
década de 70, muito antes do ZIP sequer ser pensado.

O `tar` é diferente de outros formatos de arquivamento porque ele não comprime arquivos individualmente.
Todos o arquivo tar é comprimido junto. Para extrair os arquivos, você deve primeiro descomprimir o
arquivo `.tar` inteiro.

Para identificar o formato de compressão, você adiciona um sufixo no arquivo. Os formatos e sufixos
 suportados são:

Sufixo  | Formato
--------|--------
‘.gz’	| gzip
‘.tgz’	| gzip
‘.taz’	| gzip
‘.Z’	| compress
‘.taZ’	| compress
‘.bz2’	| bzip2
‘.tz2’	| bzip2
‘.tbz2’	| bzip2
‘.tbz’	| bzip2
‘.lz’	| lzip
‘.lzma’	| lzma
‘.tlz’	| lzma
‘.lzo’	| lzop
‘.xz’	| xz
‘.zst’	| zstd
‘.tzst’	| zstd

O sufixo mais visto e o mais suportado é o .gz. Por isso que você vê muito `.tar.gz`.

# O formato

O lado bom do arquivo .tar é que não existem valores binários: os valores binários dele estão nos
arquivos, não nos metadados.

Todos os números são salvos em octal. Por exemplo, se você ver um `10` no arquivo, significa que
o valor que está ali é 8, pois `10` em octal é 8 em decimal. Assim como `11`=9, `12`=10, `13`=11, 
`17`=15 e `20`=16.

As _strings_ têm um tamanho fixo. Se o tamanho delas no arquivo for menor que o tamanho fixo, vários
bytes `0` são adicionados até chegarem nesse tamanho.

O arquivo tar é dividido em blocos. Cada bloco possui 512 bytes. Esse também é o tamanho do header
contendo os metadados dos arquivos. 

O arquivo já começa com o header do primeiro arquivo:

```
00000000  67 72 61 70 68 69 63 61  6c 73 62 6f 75 6e 64 69  |graphicalsboundi|
00000010  6e 67 2e 72 73 00 00 00  00 00 00 00 00 00 00 00  |ng.rs...........|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00000060  00 00 00 00 30 30 30 30  37 37 37 00 30 30 30 31  |....0000777.0001|
00000070  37 35 30 00 30 30 30 31  37 35 30 00 30 30 30 30  |750.0001750.0000|
00000080  30 30 30 37 36 30 33 00  31 34 32 37 33 30 30 32  |0007603.14273002|
00000090  32 37 35 00 30 31 35 37  37 34 00 20 30 00 00 00  |275.015774. 0...|
000000a0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00000100  00 75 73 74 61 72 20 20  00 61 72 74 68 75 72 6d  |.ustar  .arthurm|
00000110  63 6f 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |co..............|
00000120  00 00 00 00 00 00 00 00  00 61 72 74 68 75 72 6d  |.........arthurm|
00000130  63 6f 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |co..............|
00000140  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000001f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
```

|    Posição    | Tamanho |    Campo    | Valor                   |
|---------------|---------|-------------|-------------------------|
| `0x0 (0)`     |  100    | name        | `graphicalsbounding.rs` |
| `0x64 (100)`  |  8      | mode        | `0000777`               |
| `0x6c (108)`  |  8      | uid         | `0001750`               |
| `0x74 (116)`  |  8      | gid         | `0001750`               |
| `0x7c (124)`  |  12     | size        | `00000007603`           |
| `0x88 (136)`  |  12     | mtime       | `14273002275`           |
| `0x94 (148)`  |  8      | chksum      | `015774`                |
| `0x9c (156)`  |  1      | typeflag    | `30` = `REGTYPE`        |
| `0x9d (157)`  |  100    | linkname    | ` `                      |
| `0x101 (257)` |  6      | magic       | `ustar `                |
| `0x107 (263)` |  2      | version     | `0x20 0x00`             |
| `0x109 (265)` |  32     | uname       | `arthurmco`             |
| `0x129 (297)` |  32     | gname       | `arthurmco`             |
| `0x149 (329)` |  8      | devmajor    | ` `                      |
| `0x151 (337)` |  8      | devminor    | ` `                      |
| `0x159 (345)` |  155    | prefix      | ` `                      |

Como eu falei, **todos os números estão em formato octal**.

Depois desse header, no próximo bloco está o conteúdo do arquivo. 

Quando o arquivo termina, o header começa no próximo bloco, no próximo byte múltiplo de 512.

Ou seja, se o arquivo acabar no byte 5090, o header só vai começar no byte 5120, já que é o próximo
múltiplo de 512.

Para sinalizar o final do arquivo, dois blocos vazios (ou seja, cheios de bytes 0) são escritos.

O arquivo `tar` é identificado pelo valor no campo `magic`, ou seja `ustar`. Existem mais tipos de
arquivos tar, e eles serão discutidos nas próximas partes.

O checksum (escrito no campo `chksum`) é calculado somando todos os bytes do header. Os bytes que
estariam no checksum são substituídos, na função do cálculo, por espaços (` `, byte 32). O resultado
é escrito no campo (no caso da escrita) ou comparado com o valor do campo (no caso da leitura)

`uname` e `gname` são os nomes do usuário e do grupo que criaram o arquivo. `uid` e `gid` são os IDs
de usuário e grupo.

`mtime` é a data da última modificação do arquivo, ou de criação se o arquivo nunca foi alterado. O
valor é a quantidade de segundos desde 01/01/1970.

`name` é o nome do arquivo. Se ele é um link que aponta pra outro arquivo, o nome desse arquivo
vai estar em `linkname`.

`devmajor` e `devminor` só fazem sentido se o arquivo for um device, tipo aqueles arquivos que 
estão dentro da pasta `/dev` nos linuxes e unixes da vida.

## Códigos

Inicialmente, definiremos os valores possíveis nos campos `mode` e `typeflag`:

{{< code language="python" title="tarfile.py: Estruturas possíveis nos valores">}}
from enum import Enum, IntFlag

# Isso são flags.
# Significa que mais de um valor pode ser possível aqui.
class FileMode(IntFlag):
    # O modo do arquivo
    # Quando você dá `ls -l` no terminal, tem um monte de rwxrwxrwx. 
    # Isso é codificado aqui.'
    # O significado está claro se você manja um pouco de inglês. 
    # Se não, separa as duas palavras e joga no google tradutor que vai estar 
    # correto.

    OtherExec = 1
    OtherWrite = 2
    OtherRead = 4

    GroupExec = 8
    GroupWrite = 16
    GroupRead = 32

    OwnerExec = 64
    OwnerWrite = 128
    OwnerRead = 256

    SetGid = 1024
    SetUid = 2048

class FileType(Enum):
    ## O tipo de arquivo

    # Um arquivo comum
    Regular = 0            

    # Um link pra outro arquivo
    Link = 1

    # Um link simbólico. Esse valor está depreciado e não é usado
    Symlink = 2

    # Se você sabe o que são essas duas coisas abaixo, então você sabe o que 
    # significa tudo aqui nesse enum, e não está nem lendo esses comentários.
    CharacterDevice = 3
    BlockDevice = 4

    # Representa uma pasta. Uma coisa engraçada é que, dentro do arquivo tar, 
    # logo depois da pasta,  vêm os arquivos que estão dentro dela. 
    # Isso pode ser útil pra você.
    Directory = 5

    # Se você sabe o que é isso daqui, então você nem precisa desses comentários
    FIFOPipe = 6

    # Campo reservado.
    Reserved = 7

    # Como converter do valor do arquivo pro valor do enum
    # O `val` é o caractere que vem do arquivo.
    @staticmethod
    def from_value(val):
        selections = {
            "0": FileType.Regular,
            "\0": FileType.Regular,
            "1": FileType.Link,
            "2": FileType.Symlink,
            "3": FileType.CharacterDevice,
            "4": FileType.BlockDevice,
            "5": FileType.Directory,
            "6": FileType.FIFOPipe,
            "7": FileType.Reserved
        }

        return selections.get(val, FileType.Reserved)
{{< /code >}}

Algumas funções auxiliares, que vão ajudar a ler o arquivo
{{< code language="python" title="tarfile.py: Funções auxiliares">}}
# Isso aqui embaixo vem no final:

# Lê uma string do arquivo
def read_string(value):
    return value.decode('utf-8').rstrip('\x00 ')

# Lê um número do arquivo. Ele está em octal, então devemos converter.
def read_number(value, default=None):
    try:
        return int(read_string(value), base=8)
    except ValueError:
        return default

{{< /code >}}


Depois, vamos ler o arquivo TAR:
{{< code language="python" title="tarfile.py: Estruturas do arquivo tar">}}
## Coloque isso no começo do arquivo, antes do `from enum`
from dataclasses import dataclass

## Isso aqui embaixo vem no final:

# O tamanho do bloco:
BLOCK_SIZE = 512

@dataclass
class TarFile:
    # Você já conhece esses campos :)
    name: str
    mode: FileMode
    uid: int
    gid: int
    size: int
    mtime: dt.datetime
    checksum: int
    typeflag: FileType
    linkname: str
    magic: str
    version: int
    uname: str
    gname: str
    devmajor: int
    devminor: int
    prefix: str

    # O offset do arquivo que segue esse header, pra gente poder ler ele
    # depois
    offset: int

    # O offset do próprio header.
    header: int

    def read_file(self, fileobject):
        # Lê o conteúdo do arquivo que esse header representa
        tell = fileobject.tell()

        fileobject.seek(self.offset)
        data = fileobject.read(self.size)
        fileobject.seek(tell)

        return data

    def get_header_offset(self):
        return self.offset - BLOCK_SIZE

    def verify_checksum(self, fileobject):
        # Verifica o checksum desse header, pra ver se ele é válido ou não.
        tell = fileobject.tell()

        fileobject.seek(self.get_header_offset())
        data = fileobject.read(BLOCK_SIZE)

        checksum = sum([v if i not in range(148, 156) else ord(' ')
                        for i, v in enumerate(data)])

        fileobject.seek(tell)

        return checksum == self.checksum

    @staticmethod
    def from_file(fileobject):
        # Lê um arquivo.
        #
        # Esse `fileobject` é um objeto de arquivo, gerado pelo método `open`, ou
        # por qualquer método que represente um arquivo, como `GzipFile` e outros
        # similares.
        #
        # Esse método vai alterar o fileobject, fazendo ele apontar pro próximo 
        # bloco, que na maior parte das vezes vai ser o conteúdo do arquivo.

        offset = fileobject.tell()

        name = read_string(fileobject.read(100))
        if name == "":
            return None

        mode = FileMode(read_number(fileobject.read(8)))
        uid = read_number(fileobject.read(8))
        gid = read_number(fileobject.read(8))
        size = read_number(fileobject.read(12))
        mtime = dt.datetime.fromtimestamp(read_number(fileobject.read(12)))
        checksum = read_number(fileobject.read(8))
        typeflag = FileType.from_value(read_string(fileobject.read(1)))
        linkname = read_string(fileobject.read(100))

        magic = read_string(fileobject.read(6))
        version = read_number(fileobject.read(2), 0)

        if not magic.startswith("ustar"):
            raise ValueError(f"Invalid header at offset {offset}")

        uname = read_string(fileobject.read(32))
        gname = read_string(fileobject.read(32))
        devmajor = read_number(fileobject.read(8))
        devminor = read_number(fileobject.read(8))
        prefix = fileobject.read(155)

        pad = fileobject.read(12)

        return TarFile(name, mode, uid, gid, size, mtime, checksum, typeflag, 
                       linkname, magic, version, uname, gname, devmajor, devminor,
                       prefix, offset + BLOCK_SIZE, offset)

{{< /code >}}

Essa função acima só vai ler o header. Depois do header vem o arquivo.

E é isso. O necessário para ler o formato está descrito

Se você quiser, você pode transformar esse código que eu mostrei em um parser de arquivo `.tar`:
 - Você pode simplesmente usar `open` para ler um arquivo `tar` puro. O método `TarFile.from_file`
   aceita um objeto de arquivo que a função `open` retorna
 - Para ler um arquivo `.tar.gz`, use a classe `GzipFile` para ler o arquivo, já que esse é o 
   formato que o tar está comprimido
 - Outros formatos têm outras bibliotecas de suporte. Se você quiser, pode suportá-las.
 - Lembre-se que dois blocos vazios, cheios de 0, identificam o final do arquivo...
 - Converta esse código para a sua linguagem favorita

Na próxima parte, iremos ver um `tar` com mais funcionalidades, conhecida como formato PAX.

# Referências:
 - https://www.gnu.org/software/tar/manual/html_node/Standard.html
 - https://www.gnu.org/software/tar/manual/html_chapter/Formats.html#Compression