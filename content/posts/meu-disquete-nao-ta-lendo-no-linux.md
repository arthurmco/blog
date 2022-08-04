---
title: "Meu disquete não tá lendo!"
date: "2022-08-04T20:07:38-03:00"
author: "Arthur Mendes"
authorTwitter: "usrbinarthur"
showFullContent: false

tags: ["tips", "stories"]
keywords: ["disquete", "linux", "windows", "erro"]
description: |
    Recentemente eu precisei usar um disquete no Linux e descobri que até o básico sobre
    essas ferramentas está sendo esquecido pela nova geração.

    Se o erro `Add. Sense: Cannot read medium - unknown format` apareceu pra você, aqui está a solução.

    E se você usa Windows, calma que tem coisa pra você no final do post.
archive: "2022@08"
---

Há alguns dias atrás, eu consegui um teclado de música usado, um Yamaha PSR-350 pra aprender a 
tocar música, já que era algo que eu sempre quis aprender. O que faltava, na real, era dinheiro pra
comprar algum instrumento.

Não é um teclado muito novo, mas também não é muito antigo: é de algum lugar dos anos 2000.

{{< figure src="/blog/images/key/yamaha-psr-350.jpg" caption="O teclado aí. Fonte: fr.audiofanzine.com" >}}

Uma das coisas que me chamou a atenção foi o fato de ele ter um belo leitor de disquetes, pra você
poder salvar suas músicas pra tocar no PC, ou pra transmitir músicas pra tocá-las no teclado.

Depois de algumas semanas esperando meu leitor de disquete USB chegar da China, eu pude testá-lo com
alguns disquetes que eu tinha comprado.

Infelizmente, para minha surpresa, o disco mostrou uma mensagem nada amigável.

```
[ 1780.060701] sd 7:0:0:0: [sdc] Attached SCSI removable disk
[ 1827.483175] sd 7:0:0:0: [sdc] Read Capacity(10) failed: Result: hostbyte=DID_OK driverbyte=DRIVER_SENSE
[ 1827.483178] sd 7:0:0:0: [sdc] Sense Key : Medium Error [current] 
[ 1827.483179] sd 7:0:0:0: [sdc] Add. Sense: Cannot read medium - unknown format
```

Claro que eu pensei que o leitor estava quebrado. 

Porém, como ele estaria quebrado se ele foi reconhecido? Provavelmente era algum problema com o
disquete que eu usei.

Então eu tentei outro, novinho em folha. Mesmo erro.

----------

Irei poupar a vocês as horas de pesquisa vendo fóruns e mais fóruns, e lhes mostrar direto o que
eu fiz pra resolver:

O problema era que o disquete não estava formatado em **baixo nível**.

Você deve ter reparado, caso você tenha uns 30 e poucos anos no mínimo, que um disquete de 720KB ou
1.44MB podia ser usado em várias arquiteturas de computador diferente, mas eles eram incompatíveis
entre si: um disquete no PC não funcionava no Apple II e vice-versa.

Isso se devia ao fato que esses sistemas esperavam que *os setores estivessem em lugares diferentes
do disco*. 

O disquete é uma mídia dividida em *cabeças*, *trilhas* e *setores* (heads, tracks e 
sectors). Um disquete de 1.44 MB contém 2 cabeças, 80 trilhas e 18 setores por trilha, 
cada setor tendo 512 bytes.

{{< figure src="/blog/images/key/heads.jpg" caption="Como o disquete é organizado fisicamente. Fonte: researchgate.net" >}}

A quantidade de cabeças é pré-definida, mas, em alguns discos, as trilhas e os setores, não são. Eles
são definidos por software, em padrões de bits gravados no disquete que informam onde a trilha começa e 
o tamanho de cada setor. Essa definição é chamada de *formatação de baixo-nível*, ou *low-level formatting*

O problema é que os disquetes que eu comprei não tinham esses padrões de bits, eles não estavam 
"formatados".

## Solucionando

Para você fazer isso no Linux, você deve baixar um utilitário chamado ufiformat. 
[O link para o repositório](https://github.com/tedigh/ufiformat).

Ele não está compilado, mas compilar é tranquilo: instale o pacote `e2fs-progs` para compilar
(deve ser algo chamado `e2fsprogs-dev` no Debian e derivados), depois é só dar `./configure` e 
`make`.

Para listar os dispositivos disponíveis, digite `./ufiformat -i`.

Para rodar, digite `./ufiformat -V -v /dev/<teu disquete>`. Ele vai rodar e depois verificar para
ver se deu certo.\
Depois que ele terminar, tire o disquete e coloque denovo.\
Se isso daqui aparecer no `dmesg`:

```
[ 2213.047066] usb 3-4: reset full-speed USB device number 8 using xhci_hcd
[ 2213.273042] sd 7:0:0:0: Power-on or device reset occurred
[ 2214.872958] sd 7:0:0:0: [sdc] 2880 512-byte logical blocks: (1.47 MB/1.41 MiB)
[ 2321.016219]  sdc:
```

significa que deu certo.


# E no Windows?

No Windows é mais fácil que parece.

Quando você inserir o disquete, ele vai dar uma mensagem de erro dizendo que o disco pode não estar
formatado.

Você pode achar que ele está falando do sistema de arquivos, mas ele está falando mesmo da definição
de setores, do low level.

Clique com o botão direito no disquete e vá em "Formatar". Uma janela assim vai aparecer:

![A janela te pedindo pra formatar](/blog/images/key/winformat.png)

Sabe essa opção "Formatação Rápida"? **Não marque ela.** Se ela tiver selecionada, **desmarque**. 
Se você selecionar, ela não vai formatar o disquete em baixo nível.

Clique em OK e espere uns minutinhos.

Depois disso, corre pro abraço!