+++
title = "Problemas com `git pull` no Windows?"
date = "2022-03-06T14:47:11-03:00"
author = "Arthur M"
authorTwitter = "usrbinarthur" #do not include @
cover = ""
tags = ["ssh", "windows", "git"]
keywords = ["short"]
description = "Talvez você esteja usando o cliente SSH errado"
showFullContent = false
+++

Você está tendo problemas de autenticação com o `git pull` no Windows?

Se você deu `git clone` ou `git pull` e se deparou com um erro mais ou menos assim:

```
Permission denied (publickey).: exit status 255
```

e as suas chaves SSH estão registradas no Git certinhas e você conseguiu dar 
`ssh -vT git@github.com` sem problema nenhum, então talvez o problema seja
que você está usando o cliente SSH do git ao invés do cliente do Windows.

O cliente do Windows tem um bônus que ele funciona dentro do vscode também

Para você fazer isso, rode esse comando:

```
git config --global core.sshCommand C:/Windows/System32/OpenSSH/ssh.exe
```

Lembre-se que você deve instalar o OpenSSH.

Para isso, abra as configurações do Windows, procure naquela barra de pesquisa
por "Recursos Opcionais", vá em "Adicionar recurso opcional", clique no
botão de "Exibir recursos" e procure por "Cliente OpenSSH". Instale e reinicie.
