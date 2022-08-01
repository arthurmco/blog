+++
title = "Como meter a mão no banco de dados com segurança"
date = "2021-04-24T00:34:30-03:00"
author = "Arthur Mendes"
authorTwitter = "usrbinarthur" #do not include @
cover = ""
tags = ["hint"]
keywords = ["banco de dados", "database", ]
description = """Digamos que, por alguma razão do destino, você precise logar no banco de dados pra fazer alguma coisa. <br /> 

Por exemplo, seu chefe te pediu educadamente para alterar um dado para ontem e o cidadão que fez o sistema deixou o dado como somente leitura. <br/> 

Como fazer isso com mais segurança?"""
showFullContent = false
archive = "2021@04"
+++

18h de uma sexta feira.

Você está tranquilo no seu computador, fazendo o seu último trabalho do dia. Você já até abriu o seu
joguinho, porque assim que você fizer seu commit, o seu expediente acabou.

De repente, algum chefe lhe pede para alterar dados de um cliente. Seria muito fácil, mas aí você se
lembra que o animal que fez esse sistema deixou esses dados como somente leitura.

Você não tem tempo pra alterar o código do sistema do animal, já que isso vai levar mais de um dia
e o seu chefe pediu essa alteração, educadamente, para *ontem*

Você vai ter que alterar eles diretamente no banco.

Você se lembra de todas as piadas de `UPDATE` sem `WHERE`, e como a falta de algumas palavras fez o seu
colega perder um final de semana inteiro com o suporte atrás de backups.

Como fazer isso sem ter medo desses problemas?

A resposta é **TRANSAÇÕES**

-----

Quando você logar no seu banco (eu vou usar o postgres, mas hoje em dia qualquer banco* de dados 
decente suporta isso), você dá de cara com isso daqui:

```
meushell% psql -h IP -U usuario -d banco
Password for user usuario: 
psql (13.2 (Debian 13.2-1), server 11.9 (Raspbian 11.9-0+deb10u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

banco=# 
```

> Se você usa SQLite, você pode simplesmente copiar o banco antes da alteração :)

Para iniciar uma transação, você digita BEGIN

```
banco=# BEGIN;
BEGIN
banco=*# 
```

Observe que o _prompt_ mudou. Ele vai ficar com esse asterisco durante a transação.

Enquanto ele tiver esse asterisco, você digita os comandos que você quiser. Por exemplo, o update 
que o seu chefe pediu, pra abreviar o nome do maior cliente dele, que pode ser considerado um pouco
constrangedor.

```
banco=*# update usuarios_do_sistema_do_animal set nome_do_usuario='Tomás T. Pinto' where id=2020;
UPDATE 1
banco=*#
```

**PRESTE MUITA ATENÇÃO** no `UPDATE 1`. Esse 1 é a quantidade de linhas que o seu comando atualizou.

Caso isso seja o que você esperava, você digita `COMMIT`

```
banco=*# COMMIT;
COMMIT
banco=# 
```

Com o comando `COMMIT`, as suas alterações serão salvas no banco de dados.

> Se você não sabe o que esperar, você precisa primeiro analisar o que você quer. Dê um `SELECT` 
> primeiro, para você encontrar as linhas que você precisa alterar


Caso isso **não** seja o que você esperava, digite `ROLLBACK`

```
banco=*# ROLLBACK;
ROLLBACK
banco=#
```

As suas alterações serão revertidas.

**Cuidado:** Transações têm um limite de tamanho. Esse limite é bem grande, mas pode ser atingido
(já atingi esse limite no trabalho uma vez). Provavelmente você não vai atingir, mas, caso você não
consiga aplicar sua transação por causa de _timeout_, o problema pode ser esse

> P.S: Caso você tenha gostado do post, agradeça à Giulia, porque foi ela que deu a ideia de fazer 
> ontem à tarde.


