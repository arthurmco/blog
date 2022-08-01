+++
title = "O meu Mac está dando um erro EADDRINUSE na porta 5000!"
date = "2022-05-10T09:55:03-03:00"
author = "Arthur Mendes"
authorTwitter = "usrbinarthur" #do not include @
cover = ""
tags = ["mac", "airplay", "EADDRINUSE", "port-5000"]
keywords = ["talk"]
description = """
É mais um dos serviços que a Apple cria para melhorar sua vida, e que você provavelmente
nunca vai usar.
"""
showFullContent = false
archive = "2022@05"
+++

Ontem, no trabalho, eu fui tentar subir um serviço localmente, no Macbook que eles deram.

O serviço roda na porta 5000, então eu fui tentar rodá-lo. Para minha surpresa, apareceu
um erro assim (não vou botar ele aqui porque eu não quero ser processado):

```
Error: listen EADDRINUSE: address already in use :::5000
```

Isso significa que, obviamente, a porta 5000 está bloqueada.

Eu e o meu líder ficamos uns 15 minutos tentando descobrir o que era. E era muito simples

Era um serviço do AirPlay que rodava nessa porta. O AirPlay é um serviço de compartilhamento da 
Apple, então provavelmente a Apple deixa isso ativo por padrão, porque ela acha que você tem 
tudo compatível com esse protocolo.

O serviço fica escutando na porta 5000

```
$ lsof -i :5000

COMMAND    PID      USER   FD TYPE            DEVICE SIZE/OFF NODE NAME
ControlCe 1338 arthurmco  18u IPv4 0xdd3dfcad86ca845      0t0  TCP *:commplex-main (LISTEN)
ControlCe 1338 arthurmco  19u IPv6 0xdd3dfbc773edd3d      0t0  TCP *:commplex-main (LISTEN)

```

É esse `ControlCe` aí (deve ser de ControlCenter). Um nome extremamente sugestivo.

Infelizmente, eu só tenho só mais um produto Apple, e eu nem uso ele com esse computador.

Pra desativar isso, você:
 - abre as **Preferências do Sistema** do Mac (ou **System Preferences**, se
você prefere em inglês), 
 - vai em **Compartilhamento** (ou **Sharing**). 
 - desmarca a opção que diz "Receptor de AirPlay" (eu sei lá como é isso em inglês, mas vai 
   ter AirPlay no nome). Ela é a última opção, ou está entre as últimas.

Pronto. Agora você pode rodar serviços na 5000 à vontade.


