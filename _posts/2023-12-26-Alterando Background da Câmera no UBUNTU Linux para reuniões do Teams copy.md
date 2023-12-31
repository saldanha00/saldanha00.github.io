---
title: Adicionando Filtro de Tela de Fundo Customizado direto na câmera do UBUNTU Linux para reuniões do Teams
date: 2023-12-26 12:12:12 +/-TTTT
categories: [Sharing Knowledge, Linux]
img_path: ../../assets/images
image:
  path: teste-ok.png
  alt: Background Personalizado com device virtual
tags: [linux,webcam,teams]     # TAG names should always be lowercase
---

{% include embed/youtube.html id='ORRaHV9HSrI' %}
<!-- https://youtu.be/ORRaHV9HSrI -->

---

- [Qual a ideia?](#qual-a-ideia)
- [Instalação e Preparação do Ambiente](#instalação-e-preparação-do-ambiente)
- [Passo a Passo](#passo-a-passo)
  - [1 - Vamos criar o device virtual, para isso liste os devices que você já tem, pois cada device possui uma numeração e não podemos sobrescrever nenhum.](#1---vamos-criar-o-device-virtual-para-isso-liste-os-devices-que-você-já-tem-pois-cada-device-possui-uma-numeração-e-não-podemos-sobrescrever-nenhum)
  - [2 - Verificando qual o principal device físico da sua webcam](#2---verificando-qual-o-principal-device-físico-da-sua-webcam)
  - [3 - Depois que já identificamos qual o device real iremos utilizar para gerar o device virtual, o usaremos o v4l2loopback para criação.](#3---depois-que-já-identificamos-qual-o-device-real-iremos-utilizar-para-gerar-o-device-virtual-o-usaremos-o-v4l2loopback-para-criação)
  - [4 - Instalação do projeto Linux-Fake-Background-Webcam](#4---instalação-do-projeto-linux-fake-background-webcam)
  - [5 - Testando](#5---testando)
  - [6 - Maravilha! Se deu certo até aqui, você já pode testar no seu teams:](#6---maravilha-se-deu-certo-até-aqui-você-já-pode-testar-no-seu-teams)
- [E agora ? Criando serviço](#e-agora--criando-serviço)
- [Muito bem! Chegamos até aqui. O que falta ?](#muito-bem-chegamos-até-aqui-o-que-falta-)

---



# Qual a ideia?

{: .prompt-warning }
> Qual problema esse artigo resolve ?

Já faz algum tempo que a microsoft implementou no aplicativo de reuniões Microsoft Teams a opção de alterar o background da sua câmera. E isso permite utilizar imagens que o próprio teams disponibiliza como essa:

![Background Padrão Teams](wallpaper_teams_normal.png){: .normal w="300" h="150" }

Entretanto, para algumas empresas é interessante customizar o wallpaper do fundo da tela para alguma imagem padrão da compania, isso passa uma imagem de profissionalismo e dependendo do tipo de apresentação que você for fazer, pode ser interessante utilizar um background customizado da empresa.

O problema é: Para usuários do Ubuntu, mas vale para as outras distribuições Linux também. **A microsoft não disponibiliza a feature que envolve carregar uma imagem customizada para o background. Apenas usuários do windows premium tem essa regalia.**

Fonte: https://support.microsoft.com/en-us/office/change-your-background-in-microsoft-teams-meetings-f77a2381-443a-499d-825e-509a140f4780

---

{: .prompt-tip }
> O que vamos fazer?

Nesse artigo **vamos descer um pouco o nível**. Não vamos trabalhar na camada de aplicação do teams, vamos **criar um device de video virtual** que carrega a leitura de vídeo da câmera padrão do notebook e adiciona um fundo de tela.

Nesse device virtual que vamos criar, iremos configurar no teams para utilizar o device virtual ao invés da câmera original, sendo assim **possível ter o background que quisermos, independente do aplicativo de reuniões, seja teams, google meeting, zoom, etc.**

Por sorte, já temos na comunidade o [lfbw](https://github.com/fangfufu/Linux-Fake-Background-Webcam/tree/master) que irá nos auxiliar **MUITO** na resolução.

Testado em: Ubuntu Linux 22.04.3 LTS

---

# Instalação e Preparação do Ambiente

1 - Precisamos instalar o pacote ffmpeg e v4l-utils para testar o device virtual e facilitar a identificação:

```bash
sudo apt -y install ffmpeg v4l-utils
```

2 - Para criar o device virtual, utilizaremos o [v4l2loopback-dkms](https://github.com/umlaeute/v4l2loopback)

```bash
sudo apt -y install v4l2loopback-dkms
```

3 - Clone o repositório [Linux-Fake-Background-Webcam](https://github.com/fangfufu/Linux-Fake-Background-Webcam)

```bash
git clone https://github.com/fangfufu/Linux-Fake-Background-Webcam.git
```

4 - Certifique-se de ter instalado o [python3](https://www.python.org/downloads/)

```bash
python3 --version
```

---

# Passo a Passo

## 1 - Vamos criar o device virtual, para isso liste os devices que você já tem, pois cada device possui uma numeração e não podemos sobrescrever nenhum.

```bash
sudo v4l2-ctl --list-devices
```

```bash
sudo ls -l /dev/video*
```

A saída deve ser algo parecido com a imagem abaixo:

![listing video devices](listing_device.png){: .normal w="700" h="400" }

No meu caso, o último device físico é o /dev/video4, então eu posso criar a partir do /dev/video5 sem comprometer minha webcam. Para esse teste irei utilizar o /dev/video6

## 2 - Verificando qual o principal device físico da sua webcam

Para isso, utilizaremos o ffmpeg para testar cada um dos devices:

```bash
ffplay /dev/video1
```

Teste com todos os devices que você tiver, no meu caso o que funcionou corretamente foi o /dev/video1, note que outros podem funcionar, entretanto dependendo da sua webcam pode ter mais de um device funcional. Escolha o que for mais próximo da sua realidade.

![ffplay](ffplay.png){: .normal w="700" h="400" }

## 3 - Depois que já identificamos qual o device real iremos utilizar para gerar o device virtual, o usaremos o v4l2loopback para criação.

tenha certeza que não há nenhum device virtual já criado, o comando abaixo deleta os existentes (em memória, qualquer problema é só rebootar. Mas se você nunca criou um device virtual de video ou audio antes, não há com o que se preocupar).

```bash
sudo modprobe -r v4l2loopback
```

Supondo que você quer criar o device virtual no /dev/video6:

```bash
sudo modprobe v4l2loopback devices=1 exclusive_caps=1 video_nr=6 card_label="fake-cam"
```

Cheque se o device foi criado, conforme demonstrado no passo 1 de listagem.

## 4 - Instalação do projeto Linux-Fake-Background-Webcam 
   
Navegue até a raiz da pasta do repositório que você clonou do Linux-Fake-Background-Webcam 

Certifique-se de que o pip está atualizado

```bash
python3 -m pip install --upgrade pip .
```

Agora vamos instalar as dependencias do projeto

```bash
python3 -m pip install --upgrade .
```

>Caso aconteça algum erro na instalação das dependencias, [verifique no projeto](https://github.com/fangfufu/Linux-Fake-Background-Webcam/tree/master).

Agora iremos configurar o arquivo **config-example.ini**. Abaixo há uma sugestão,

Mas o que iremos precisar mesmo é adicionando o campo "background-image" com o path da imagem que você deseja adicionar no fundo.

E alterar o valor do "webcam-path" para o device correto que você identificou no passo 2 (utilizando ffplay)

<details>
  <summary>config-example.ini</summary>

    {% highlight text %}

      width = 640
      height = 480
      fps = 30
      ; no-background = yes
      background-keep-aspect = no
      no-foreground = yes
      webcam-path = /dev/video1
      background-image = /opt/Linux-Fake-Background-Webcam/background.jpg
      v4l2loopback-path = /dev/video6
      threshold = 50
  {% endhighlight text %}
</details>
<br>

Não se preocupe a principio com os valores de width e height, depois do teste você ajusta conforme necessário.

## 5 - Testando

Com tudo configurado corretamente, vamos testar (certifique-se de que o lfbw está devidamente instalado conforme passo 4):

Na raiz do projeto Linux-Fake-Background-Webcam execute:

```bash
lfbw -c config-example.ini
```

>Observação: Para cancelar o processo, basta pressionar "ctrl+\" que é o sinal "SIGQUIT"

se der tudo certo a seguinte mensagem irá aparecer:

```text
Real camera original values are set as: 640x480 with 30 FPS and video codec 1448695129
Real camera new values are set as: 640x480 with 30 FPS and video codec 1196444237
Running...
```

Na mensagem acima você pode conferir os valores de altura e largura da sua tela e ajustar conforme necessário. No nosso caso já está ajustado.

Agora para verificar se o device virtual está funcionando, teste utilizando ffplay conforme passo 2:

No nosso exemplo, criamos para o /dev/video6

```bash
ffplay /dev/video6
```

Com o sucesso, deve aparecer a imagem de fundo dessa forma:

![teste-ok](teste-ok.png){: .normal w="700" h="400" }

## 6 - Maravilha! Se deu certo até aqui, você já pode testar no seu teams:

Certifique-se de escolher nas configurações o device virtual criado.

![teams_ok](teams_ok.png){: .normal w="700" h="400" }

---

# E agora ? Criando serviço

Com isso você já consegue entrar em reuniões com backgrounds personalizados, entretanto  se você reiniciar sua máquina vai precisar realizar o passo 3 e 5 novamente.

Para tornar isso mais efetivo, vamos transformar em serviço.

Volte para a raiz do projeto github, dentro dele tem uma pasta "systemd-user". Sim, a comunidade pensou nisso, que maneiro né ?

vamos copiar o arquivo lfbw.service que está dentro da pasta systemd-user para o /etc/systemd/system no caso do ubuntu ou dependendo da sua distribuição, o local onde ficam os arquivos ".service", 

>Na dúvida pergunte ao chatgpt o path so serviços da sua distribuição Linux.

```bash
sudo cp -a lfbw.service /etc/systemd/system/
``````

e o outro arquivo "lfbw_start_wrapper.sh" para o path ~/.local/bin

```bash
cp -a lfbw_start_wrapper.sh ~/.local/bin/lfbw_start_wrapper.sh"
```

agora edite o arquivo lfbw.service corrigindo o path do script para sua home:

```bash
vi /etc/systemd/system/lfbw.service
```

<details>
<summary>lfbw.service</summary>

{% highlight bash %}

[Unit]
Description=Fake camera
After=network.target

[Service]
Type=simple

#Edite a linha abaixo
ExecStart=/home/{USERNAME_HOME_HERE}/.local/bin/lfbw_start_wrapper.sh

KillSignal=SIGQUIT

[Install]
WantedBy=default.target

{% endhighlight bash %}
</details>

Agora edite o arquivo lfbw_start_wrapper.sh que copiou para sua home (Se atente para o número do parâmetro "video_nr" que deve ser o número do device virtual da câmera, que irá ser criado.):

```bash
vi ~/.local/bin/lfbw_start_wrapper.sh
```

<details>
<summary>lfbw_start_wrapper.sh</summary>

{% highlight bash %}

#!/bin/bash
sudo modprobe -r v4l2loopback
sudo modprobe v4l2loopback devices=1 exclusive_caps=1 video_nr=6 card_label="fake-cam"
sudo -u {{SEU_USUARIO_DA_MAQUINA_AQUI}} /home/{{SEU_USUARIO_DA_MAQUINA_AQUI}}/.local/bin/lfbw -c "{{PATH_DO_config-example.ini}}"

{% endhighlight bash %}
</details>

Um reload para salvar o novo serviço:

```bash
systemctl daemon-reload
```

dessa forma criamos um serviço chamado lfbw.

Teste o serviço:

```bash
service lfbw start
```

```bash
service lfbw status
```

se estiver tudo certo, habilite na inicialização do sistema operacional:

```bash
systemctl enable lfbw
```

---

# Muito bem! Chegamos até aqui. O que falta ?

- Falta elaborar uma forma mais eficiente de manter o device virtual sempre funcionando, há casos onde dependendo da inicialização do sistema a webcam original não surge no sempre device (/dev/video2). Há casos onde surge no /dev/video0 ou /dev/video1, etc...

Caso isso aconteça com você, basta repetir o passo 2 e editar o arquivo config-example.ini conforme passo 4.