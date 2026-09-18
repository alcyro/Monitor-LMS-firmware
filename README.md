# Monitor-LMS — firmware público

Este repositório é **público de propósito**. Ele existe por uma razão técnica concreta: o Gateway
LMS baixa o firmware por HTTPS **sem mandar cabeçalho de autenticação**. Um repositório privado
exigiria um token que o equipamento não tem como apresentar, e a atualização por URL simplesmente
não funcionaria.

## O que pode entrar aqui, e o que não pode

**Regra única, e ela não é negociável: só entra binário que não carrega segredo nenhum.**

| firmware | pode ser publicado? | por quê |
|---|---|---|
| **Filho** (nó sensor) | ✅ sim | conversa por ESP-NOW com o Gateway; não tem chave de nuvem |
| **Gateway / Pai** | ✅ desde a v27.96W | as chaves saíram do binário e vivem só no NVS |

Isso é **medido a cada publicação**, não presumido. Até a v27.95W, `strings firmware.bin` encontrava
as chaves dentro do binário do Pai — publicá-lo equivaleria a publicar a chave de escrita do canal de
comandos, que permite comandar a frota. A v27.96W as tirou de lá: os valores vivem só no NVS.

⚠️ **A regra não mudou, só o binário.** Antes de subir qualquer `.bin` do Pai, confira de novo —
um `#define` preenchido por engano recoloca a chave no arquivo.

O Filho publica só o SSID da malha, que é transmitido pelo ar de qualquer forma.

> Antes de subir qualquer `.bin` aqui, rode a verificação:
> `npm run fw:github -- <caminho.bin> <alvo>` no repositório do aplicativo. Ela confere papel,
> variante, tamanho e cabeçalho ESP32 — é a única guarda contra gravar o firmware errado no
> equipamento errado.

## Por que ser público não é o mesmo que ser inseguro

Quem baixar um `.bin` daqui **não consegue instalá-lo em equipamento nenhum**. O Gateway só aceita
comando assinado com HMAC e número de sequência monotônico, e confere o **SHA-256** do que baixou
contra o que veio no comando. Sem o segredo de assinatura, um binário baixado daqui é um arquivo
inerte.

O que o SHA protege é o outro sentido: ninguém consegue trocar o conteúdo deste repositório por
outro binário e fazer o seu equipamento instalá-lo, porque o hash não bateria.

## Catálogo

`catalogo.json` mapeia cada nome de build para URL, SHA-256, versão, papel e tamanho. É lido
diretamente pelo MATLAB Analysis do ThingSpeak, que assim resolve um nome curto
(`filho-wroom-21.36`) sem precisar de servidor nenhum.
