---
title: Clickjacking - Entendendo, Testando e Corrigindo
date: 2023-12-29 12:12:12 +/-TTTT
categories: [Sharing Knowledge, Cybersecurity, Web]
img_path: ../../assets/images
image:
  path: thumb_post_clickjacking.png
  alt: Iframes do Clickjacking
tags: [cybersecurity,web]     # TAG names should always be lowercase
---


<!-- {% include embed/youtube.html id='ORRaHV9HSrI' %} -->


---

- [Objetivo do Artigo](#objetivo-do-artigo)
- [1 - O que é clickjacking](#1---o-que-é-clickjacking)
- [2 - Cenário de ataque Clickjacking](#2---cenário-de-ataque-clickjacking)
  - [2.1 - Clickjacking utilizando Imagens](#21---clickjacking-utilizando-imagens)
  - [2.2 - Clickjacking para Download de malwares](#22---clickjacking-para-download-de-malwares)
  - [2.3 - Clickjacking Clássico (iframes)](#23---clickjacking-clássico-iframes)
- [Bypassing de CSRF](#bypassing-de-csrf)
- [Como Corrigir Clickjacking?](#como-corrigir-clickjacking)
- [Conclusão](#conclusão)
- [Fontes e Referências](#fontes-e-referências)



## Objetivo do Artigo

{: .prompt-warning }
> Qual problema esse artigo resolve ?

Clickjacking por vezes parece ser algo abstrato e de dificil compreensão. A solução costuma ter relação com implementação de headers de segurança, entretanto não é tão simples de compreender o que de fato é o clickjacking, **como se explora, como se testa a vulnerabilidade e principalmente: como se corrige?**

---

{: .prompt-tip }
> O que vamos fazer?

Nesse artigo vamos demonstrar como funciona o clickjacking, quais os principais cenários de ataque, como testar aplicações para clickjacking e quais o headers de segurança precisamos implementar para eliminar essa vulnerabilidades.

---

## 1 - O que é clickjacking

Clickjacking tem em sua etimologia a descrição de "roubo de clicks" onde basicamente um atacante se esforça para "roubar" ou coletar o click que iria para outro site ou outro local. 

Imagine que você abre seu navegador e quer acessar o seu internet banking. E em algum momento você vai precisar clicar em botões, links, campos de texto, etc... 

**Toda a ideia do clickjacking é fomentar um ambiente para que seja possível que a vítima acredite que legitimamente está clicando em algo**, nesse caso, o internet banking, ela consegue até visualizar com certa familiaridade onde está clicando, mas existe um "botão" transparente na frente em que vai roubar o click da vítima e direciona-la para um download de um malware, ou para um site ficticio.

Geralmente o clickjacking envolve a incorporação de um site em um "iframe" e a sobreposição de um botão invisível em cima de desse iframe. 

Dito isso, é possível afirmar que o **clickjacking geralmente vem acompanhado de um site malicioso** que contém em algum lugar da tela o clickjacking.

## 2 - Cenário de ataque Clickjacking

Ok, essa é a teoria. Vamos ver alguns cenários de utilização de clickjacking para absorver melhor o conhecimento.

### 2.1 - Clickjacking utilizando Imagens

Imagine que Jonas está em uma conversa informal pela internet quando percebe poderia utilizar um meme que está no "hype" popularmente conhecido como "meme do pão francês brasileiro" encaixaria muito bem no contexto da conversa, Jonas lembra do meme mas não sabe exatamente onde ele está armazenado para compartilhar.

Então o que ele faz ? Procura no google por "meme do pão francês brasileiro". E aqui começa o ataque.

Ao pesquisar por meme do pão francês brasileiro, o google naturalmente vai retornar diversos links de diversos sites.

Agora imagine que você como atacante se aproveita da popularidade desse meme e cria um site malicioso que contém o meme, pronto para buscar pessoas como Jonas. O site aparenta como a imagem abaixo:

![memes_fake](memes1.png){: .normal w="700" h="400" }

Jonas naturalmente clica aleatoriamente no link que aparece no resultado das pesquisas e acaba indo parar no site do atacante.

Ao passar o mouse em cima do meme, percebe que é "clicável", prontamente Jonas imagina que deve leva-lo até a origem do meme então clica em cima da imagem.

![clickjacking](clickjacking.gif){: .normal w="700" h="400" }

Aqui temos um exemplo de clickjacking. No gif acima é possível ver que Jonas foi redirecionado para um link arbitrariamente escolhido pelo atacante. Ao pensar que estava clicando no meme quando na verdade estava clicando em uma camada posta propositalmente na frente do meme.

  ```html
    <img src="images/memes1.jpeg" alt="Meme 1">
  ```

<br>

E a outra camada, que fica por cima da imagem é um link que leva a uma origem arbitrária. Atenção para a class "invisible-link":

```html
<body>
    <h1>Top 15 Memes 2023</h1>
    <div class="iframe-container">
        <img src="images/memes1.jpeg" alt="Meme 1">
	      <a href="https://media.tenor.com/He2W0AQvZfsAAAAC/hacked-hack.gif" class="invisible-link" target="_blank" rel="noopener noreferrer"></a>
    </div>
</body>
```

CSS:

```css
        .invisible-link {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 999;
            cursor: pointer;
        }
        .iframe-container {
            display: inline-block;
            margin: 0 10px;
            position: relative;
        }
```

Essa é uma forma possível do clickjacking ser utilizado, mas apenas redirecionar links pode não ser muito efetivo, então vamos ver outros cenários.

---

### 2.2 - Clickjacking para Download de malwares

Usando o mesmo princípio da busca pelo meme em alta, ao invés de redirecionar a vítima para um link arbitrário, iremos fazer a vítima realizar o download involuntário ao clicar na imagem.


![clickjacking2](download_malware.gif){: .normal w="700" h="400" }

```html
<body>
    <h1>Top 15 Memes 2023</h1>
    <div class="iframe-container">
        <img src="images/memes1.jpeg" alt="Meme 1">
	<a href="images/malware.txt" class="invisible-link" rel="noopener noreferrer" download></a>
    </div>
</body>
```
---

Aqui está o código completo:

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meme Page</title>
    <style>
        body {
            font-family: 'Courier New', monospace;
            text-align: left;
            margin: 0;
            padding: 0;
            background-color: #255255255;
        }

        h1 {
            color: #000;
            margin-bottom: 20px;
        }

        .iframe-container {
            display: inline-block;
            margin: 0 10px;
            position: relative;
        }

        /* Link Invisível */
        .invisible-link {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 999;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <h1>Top 15 Memes 2023</h1>
    <div class="iframe-container">
        <img src='images/memes1.jpeg' alt="Meme 1">
        <img src='images/memes2.jpeg' alt="Meme 2">
	<!-- redireciona arbitrariamente -->
<!-- <a href="https://media.tenor.com/He2W0AQvZfsAAAAC/hacked-hack.gif" class="invisible-link" target="_blank" rel="noopener noreferrer"></a>-->
	<!-- causa o download de malware -->
	<a href="{{ url_for('static',filename='images/malware.txt') }}" class="invisible-link" rel="noopener noreferrer" download></a>
    </div>
</body>
</html>
```

Com isso, o atacante pode até imaginar que baixou o próprio meme para máquina, e ao clicar é executado o malware. 

Podemos observar nesse cenário que o clickjacking pode ser bem perigoso dependendo da criatividade do atacante. **Se utilizado junto com outras técnicas como pharming pode ser desastroso.**

---

### 2.3 - Clickjacking Clássico (iframes)

O ataque clássico de clickjacking basicamente envolve incorporar uma página legítima em um frame, e induzir a vítima a clicar em determinados locais.

Para explicar o ataque clássico, vamos imaginar uma aplicação web que contém uma página de reset de senha. Para resetar a senha obviamente você precisa estar com a sessão ativa (logado) na aplicação.

O formulário de reset de senha é protegido por token único para cada sessão (CSRF token), então um ataque do tipo CSRF não é viável.

A aplicação não configurou os headers de segurança **x-frame-options ou CSP**, Então nesse caso o ataque de clickjacking é uma boa alternativa para os atacantes. 

App vulneravel:

![reset_senha](reset_senha_1.png){: .normal w="700" h="400" }

Para testar se essa app é de fato vulneravel a um ataque de clickjacking, vamos tentar incorporar essa página em um iframe de outro domínio. Para isso vamos montar um html local e tentar incorpora-lo.

E para comparação, iremos tentar incorporar uma página que sabemos que é segura, o google. E colocaremos os dois iframes um do lado do outro:

![comparacao_iframe](comparacao_iframe.png){: .normal w="700" h="400" }

Abaixo tem o código html utilizado, repare que a incorporação acontece na tag "<iframe>":

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Clickjacking Hacker Lab</title>
    <style>
        body {
            font-family: 'Courier New', monospace;
            text-align: center;
            margin: 0;
            padding: 0;
            background-color: #000;
            color: #0f0;
        }

        h1 {
            color: #0f0;
            margin-bottom: 20px;
        }

        .iframe-container {
            display: inline-block;
            margin: 0 10px;
            position: relative;
        }

        .iframe-container iframe {
            width: 100%;
            height: 450px;
            border: 2px solid #0f0;
            background-color: #000;
        }

        .iframe-description {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            color: #0f0;
            font-size: 18px;
            padding: 10px;
            box-sizing: border-box;
            z-index: 1;
        }

        .impact-section {
            margin-top: 50px;
        }

        .impact-section h2 {
            color: #0f0;
            margin-bottom: 10px;
        }

        .impact-text {
            text-align: left;
            max-width: 600px;
            margin: 0 auto;
            color: #0f0;
        }
        /* Link Invisível */
        .invisible-link {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 999;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <h1>Clickjacking Hacker Lab</h1>

    <div class="iframe-container">
        <iframe src="http://app.absolem00.com/"></iframe>
        <div class="iframe-description">Vulneravel a clickjacking</div>
    </div>

    <div class="iframe-container">
        <iframe src="https://www.google.com.br"></iframe>
        <div class="iframe-description">Não vulneravel a clickjacking</div>
    </div>

    <div class="impact-section">
        <h2>Impactos do Clickjacking:</h2>
        <div class="impact-text">
            <p>O Clickjacking é uma técnica maliciosa que engana o usuário ao clicar em algo diferente do que percebem.</p>
            <p>Impacto: Clicar em botões não intencionais, baixar malwares, ser direcionado para páginas de phishing, etc...</p>
            <p>É importante implementar medidas de segurança, usualmente habilita-se o header de resposta Content-Security-Policy (CSP) com a diretiva "frame-ancestors". Ou o X-Frame-Options, para prevenir ou mitigar os riscos associados ao Clickjacking.</p>
        </div>
    </div>

    <div class="impact-section">
        <h2>Cenário Real. Clique abaixo</h2>
    </div>

    <div class="iframe-container">
        <iframe src="http://app.absolem00.com/" allowtransparency="true"></iframe>
        <a href="https://media.tenor.com/He2W0AQvZfsAAAAC/hacked-hack.gif" class="invisible-link" target="_blank" rel="noopener noreferrer"></a>
    </div>

</body>
</html>
```

Maravilha, agora sabemos que a app de fato pode ser incorporada em um iframe e pode ser utilizada em um ataque de clickjacking.

A partir daqui, o principio é o mesmo. Adicionar uma camada transparente na frente do iframe para induzir o atacante a clicar no "Redefinir Senha" para redefinir a senha para vazio, ou em casos mais elaborados iludir o usuário a preencher campos. Enfim, aqui a criatividade é a principal arma para o ataque ser bem sucedido.

![clickjacking_lab](lab_clickjacking.gif){: .normal w="700" h="400" } 

---

## Bypassing de CSRF

Como comentado anteriormente, um atacante pode utilizar o clickjacking para **bypassar o controle CSRF**. Como muitos de vocês devem saber, CSRF é um token único para ser utilizado principalmente em formularios. E o clickjacking permite que o próprio usuário clique, com seu "próprio csrf". Dessa forma não importa se o formulario possui ou não o token CSRF pois quem irá preencher o formulario é a própria vitima ao ser enganada pelo clickjacking.

---

## Como Corrigir Clickjacking?

A solução mais simples envolve a **adição do header de segurança "CSP" que significa Content-Security-Policy com a diretiva "frame-ancestors"**. A qual permite gerenciar quando a página pode ser incorporada em um frame por outros domínios.

Também é possível com o header "X-FRAME-OPTIONS" entretanto esse header está obsoleto e já foi substituido totalmente pelo CSP com a chegada do html5

Ou seja, CSP que é um header de resposta que permite adicionar políticas de segurança aos sites. Dependendo da diretiva utilizada você pode assegurar contra clickjacking ou XSS.

>OBS: O CSP é suportado para ser utilizado em meta tags, entretanto a diretiva frame-ancestors não tem essa capacidade ainda.

```text
frame-ancestors 'none';
```

Vou deixar dois exemplos de servidores simples utilizando python que fazem a injeção do header CSP. Atenção que a linha que injeta o x-frame-options está comentada pois está obsoleto.

```python
from flask import Flask, render_template, make_response
from flask_socketio import SocketIO

app = Flask(__name__)
socketio = SocketIO(app)

@app.route('/')
def index():
    html_content = render_template("memes.html")
    response = make_response(html_content)
    response.headers['Content-Security-Policy'] = "frame-ancestors 'none'"
#    response.headers['X-Frame-Options'] = 'SAMEORIGIN'
    return response

if __name__ == '__main__':
    # Use a porta 80 para HTTP (pode exigir privilégios de administrador)
    socketio.run(app,host='0.0.0.0',port=80, debug=True)

```

E pelo simples fato de adicionar o header de segurança, o navegador não permite a incorporação de iframes que não estão especificados.

Veja como no resultado como ao tentar importar novamente o site dos memes recebemos um "refused to connect"

![corrigido_csp](corrigido_csp.png){: .normal w="700" h="400" }

---

## Conclusão

Clickjacking é uma vulnerabilidade que alguns programas de bugbounty desconsideram, e talve o clickjacking só pelo clickjacking não até não aparentar  grandes ameaças, mas quando combinado com pharming, phishing, xss e outras vulnerabilidades pode ser uma perigosa alavanca para o sucesso dos ataques.

Dito isso, vale lembrar que é sempre válido analisar os headers de resposta da aplicação. Procure pelos headers CSP com a diretiva frame-ancestors e pelo x-frame-options também. Se não houver um desses especificados, ou conter apenas os "*-report-only", vale a pena o teste para clickjacking.

Espero que depois desse artigo o conceito de clickjacking tenha ficado mais claro e seu apetite por conhecer mais headers de resposta dos servidores tenha aumentado! Obrigado.

---

## Fontes e Referências

Fonte sobre CSP: https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/Content-Security-Policy

Fonte sobre x-frame-options estar obsoleto: https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/X-Frame-Options

Mais sobre implementação do CSP: https://content-security-policy.com/