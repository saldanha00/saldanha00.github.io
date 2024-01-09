---
title: Clickjacking - Entendendo, Testando e Corrigindo
date: 2023-12-29 12:12:12 +/-TTTT
categories: [Sharing Knowledge, Cybersecurity, Web]
img_path: ../../assets/images
image:
  path: thumb_post_clickjacking.png
  alt: Iframes do Clickjacking
tags: [cybersecurity,web,clickjacking,securityheaders]     # TAG names should always be lowercase
---


<!-- {% include embed/youtube.html id='ORRaHV9HSrI' %} -->


- [O que é clickjacking](#o-que-é-clickjacking)
- [Cenário de ataque Clickjacking](#cenário-de-ataque-clickjacking)
  - [Clickjacking utilizando Imagens](#clickjacking-utilizando-imagens)
  - [Clickjacking para Download de malwares](#clickjacking-para-download-de-malwares)
  - [Clickjacking Clássico (iframes)](#clickjacking-clássico-iframes)
    - [Testando para clickjacking](#testando-para-clickjacking)
- [Bypassing de CSRF](#bypassing-de-csrf)
- [Como Corrigir Clickjacking?](#como-corrigir-clickjacking)
  - [CSP (Content-Security-Policy)](#csp-content-security-policy)
- [response.headers\['X-Frame-Options'\] = 'SAMEORIGIN'](#responseheadersx-frame-options--sameorigin)
  - [X-FRAME-OPTIONS](#x-frame-options)
  - [Técnicas de Frame-Busting](#técnicas-de-frame-busting)
  - [Atributo SameSite](#atributo-samesite)
- [Conclusão](#conclusão)
- [Fontes e Referências](#fontes-e-referências)


---





{: .prompt-warning }
> Qual problema esse artigo resolve ?

Clickjacking por vezes parece ser algo abstrato e pelo fato de usualmente ser utilizado em conjunto com outras vulnerabilidades pode ser de dificil compreensão.

Em alguns casos o clickjacking é resumido apenas em sites incorporados em iframes e isso não é 100% a realidade do clickjacking.

Vamos entender um pouco sobre CSP e X-Frame-Options, mas não somente isso. Vamos tentar simplificar com vários exemplos o que de fato é o clickjacking, **como se explora?, como se testa a vulnerabilidade? e como se corrige?**

---

{: .prompt-tip }
> O que vamos fazer?

Nesse artigo vamos demonstrar como funciona o clickjacking, quais os principais cenários de ataque, como testar aplicações para clickjacking e quais o headers de segurança precisamos implementar para eliminar essa vulnerabilidade.

---

# O que é clickjacking

Clickjacking tem em sua etimologia a tradução para "roubo de clicks" onde basicamente um atacante se esforça para "roubar" ou coletar o click que iria para outro site ou outro local.

Imagine que você abre seu navegador e quer acessar o seu internet banking. E em algum momento você vai precisar clicar em botões, links, campos de texto, etc... 

**Toda a ideia do clickjacking é fomentar um ambiente para que seja possível que a vítima acredite que legitimamente está clicando em algo**, nesse caso do internet banking, a vítima consegue até visualizar com certa familiaridade onde está clicando, mas existe um "botão" transparente na frente em que vai roubar o click da vítima e direciona-la para um download de um malware, ou para um site ficticio por exemplo.

Geralmente o clickjacking envolve a incorporação de um site em um "frame" e a sobreposição de um botão invisível em cima de desse frame. 

Dito isso, é possível afirmar que o **clickjacking geralmente vem acompanhado de um site malicioso** que contém em algum lugar da tela o clickjacking.

![[Clickjacking Iphone](https://blog.intigriti.com/hackademy/clickjacking/)](clickjacking_iphone.jpeg "https://blog.intigriti.com/hackademy/clickjacking/")

# Cenário de ataque Clickjacking

Ok, essa é a teoria. Vamos ver alguns cenários de utilização de clickjacking para absorver melhor o conhecimento.

## Clickjacking utilizando Imagens

Imagine que Jonas está em uma conversa informal pela internet quando percebe poderia utilizar um meme que está no "hype" popularmente conhecido como "meme do pão francês brasileiro" encaixaria muito bem no contexto da conversa, Jonas lembra do meme mas não sabe exatamente onde ele está armazenado para compartilhar.

Então o que ele faz ? Procura no google por "meme do pão francês brasileiro". E aqui começa o ataque.

Ao pesquisar por meme do pão francês brasileiro, o google naturalmente vai retornar diversos links de diversos sites.

Agora imagine que você como atacante se aproveita da popularidade desse meme e cria um site malicioso que contém o meme, pronto para buscar pessoas como Jonas. O site aparenta como a imagem abaixo:

![memes_fake](memes1.png){: .normal w="700" h="400" }

Jonas naturalmente clica aleatoriamente no link que aparece no resultado das pesquisas e acaba indo parar no site do atacante.

Ao passar o mouse em cima do meme, percebe que é "clicável", prontamente Jonas imagina que deve leva-lo até a origem do meme então clica em cima da imagem.

![clickjacking](clickjacking.gif){: .normal w="700" h="400" }

Aqui temos um exemplo de clickjacking. No gif acima é possível ver que Jonas foi redirecionado para um link arbitrariamente escolhido pelo atacante. Ao pensar que estava clicando no meme quando na verdade estava clicando em uma camada invisivel posta propositalmente na frente do meme.

<details>
    <summary>Código HTML</summary>
    
    {% highlight html %}
        <body>
            <h1>Top 15 Memes 2023</h1>
            <div class="iframe-container">
                <img src="images/memes1.jpeg" alt="Meme 1">
                <a href="https://media.tenor.com/He2W0AQvZfsAAAAC/hacked-hack.gif" class="invisible-link" target="_blank" rel="noopener noreferrer"></a>
            </div>
        </body>
    {% endhighlight html %}

</details>
<details>
    <summary>Código CSS</summary>
    
    {% highlight css %}

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
    {% endhighlight css %}
</details>

<br>
Essa é uma forma possível do clickjacking ser utilizado, mas apenas redirecionar links pode não ser muito efetivo, para esse ataque funcionar o atacante precisa de bastante criativade e trabalho no CSS da página. Vamos ver outros cenários.

---

## Clickjacking para Download de malwares

Usando o mesmo princípio da busca pelo meme em alta, ao invés de redirecionar a vítima para um link arbitrário, iremos fazer a vítima realizar o download involuntário ao clicar na imagem.


![clickjacking2](download_malware.gif){: .normal w="700" h="400" }

<details>
    <summary>Código HTML</summary>
    
    {% highlight html %}
        <body>
            <h1>Top 15 Memes 2023</h1>
            <div class="iframe-container">
                <img src="images/memes1.jpeg" alt="Meme 1">
            <a href="images/malware.txt" class="invisible-link" rel="noopener noreferrer" download></a>
            </div>
        </body>
    {% endhighlight html %}
</details>
<br>
<details>
    <summary>Aqui está o código completo (HTML + CSS)</summary>
    {% highlight html %}
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
            <a href="{ { url_for('static',filename='images/malware.txt') } }" class="invisible-link" rel="noopener noreferrer" download></a>
            </div>
        </body>
        </html>
    {% endhighlight html %}
</details>
<br>
Com isso, seria até plausível imaginar que a vítima pensou que baixou a própria imagem do meme para máquina, e ao clicar é executado o malware. 

Existe a possibilidade do atacante forçar o download do malware e logo em seguida redirecionar para o site original, mascarando o download.

>Podemos observar nesse cenário que o clickjacking pode ser bem perigoso dependendo da criatividade do atacante. **Se utilizado junto com outras técnicas como pharming pode ser desastroso.**

---

## Clickjacking Clássico (iframes)

O ataque clássico de clickjacking basicamente envolve incorporar uma página legítima em um frame, e induzir a vítima a clicar em determinados locais.

Esse tipo de ataque é muito utilizado para bypassar formulários que tem a proteção [CSRF](https://owasp.org/www-community/attacks/csrf).

Para explicar o ataque clássico, vamos imaginar uma aplicação web que contém uma página de troca de senha. Para trocar a senha naturalmente você precisa estar com a sessão ativa (logado) na aplicação.

O formulário de reset de senha é protegido por token único para cada sessão (CSRF token), então um ataque do tipo CSRF não é viável.

A aplicação não configurou os headers de segurança **x-frame-options ou CSP**, Então nesse caso o ataque de clickjacking é uma boa alternativa para os atacantes. 

App vulneravel:

![reset_senha](reset_senha_1.png){: .normal w="700" h="400" }

### Testando para clickjacking

Para testar se essa app é de fato vulneravel a um ataque de clickjacking, vamos tentar incorporar essa página em um iframe de outro domínio. Para isso vamos montar um html local e tentar incorpora-lo.

E para comparação, iremos tentar incorporar uma página que sabemos que é segura, o google. E colocaremos os dois iframes um do lado do outro:

![comparacao_iframe](comparacao_iframe.png){: .normal w="700" h="400" }

Abaixo tem o código html utilizado, repare que a incorporação acontece na tag "<iframe>":

<details>
<summary>Código HTML</summary>

{% highlight html %}
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
{% endhighlight html %}
</details>
<br>
Maravilha, agora sabemos que a app de fato pode ser incorporada em um iframe e pode ser utilizada em um ataque de clickjacking.

A partir daqui, o principio é o mesmo. Adicionar uma camada transparente na frente do iframe para induzir o atacante a clicar no "Redefinir Senha" para redefinir a senha para vazio, ou em casos mais elaborados iludir o usuário a preencher campos. Enfim, aqui a criatividade é a principal arma para o ataque ser bem sucedido.

![clickjacking_lab](lab_clickjacking.gif){: .normal w="700" h="400" } 

<br>

>Perceba que o fato da página permitir a incorporação em um iframe de outro domínio possibilita ataques de clickjacking, portanto podemos inferir que a página incorporada é vulneravel a ataques de clickjacking.

---

# Bypassing de CSRF

Como comentado anteriormente, um atacante pode utilizar o clickjacking para **bypassar o controle CSRF**. Como muitos de vocês devem saber, CSRF é um token único para ser utilizado principalmente em formularios. E o clickjacking permite que o próprio usuário clique, com seu "próprio csrf". Dessa forma não importa se o formulario possui ou não o token CSRF pois quem irá preencher o formulario é a própria vitima ao ser enganada pelo clickjacking.

>Até aqui você já deve ter percebido que 90% do sucesso do clickjacking depende do CSS desenvolvido pelo atacante.

---

# Como Corrigir Clickjacking?

## CSP (Content-Security-Policy)

A solução mais simples envolve a **adição do header de segurança "CSP" que significa Content-Security-Policy com a diretiva "frame-ancestors"**. A qual permite gerenciar quando a página pode ser incorporada em um frame por outros domínios.

Ou seja, CSP que é um header de resposta que permite adicionar políticas de segurança aos sites. Dependendo da diretiva utilizada você pode assegurar contra clickjacking ou XSS.

```text
frame-ancestors 'none';
```

>OBS: O CSP é suportado para ser utilizado em meta tags, entretanto a diretiva frame-ancestors não tem essa capacidade ainda.

Vou deixar um exemplo de um servidor simples utilizando python que fazem a injeção do header CSP. Atenção que a linha que injeta o x-frame-options está comentada pois está obsoleto.

<details>
<summary>Código python3</summary>
{% highlight python %}
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
{% endhighlight python %}
</details>
<br>

E pelo simples fato de adicionar o header de segurança, o navegador não permite a incorporação de iframes que não estão especificados.

Veja como no resultado como ao tentar importar novamente o site dos memes recebemos um "refused to connect"

![corrigido_csp](corrigido_csp.png){: .normal w="500" h="400" }




## X-FRAME-OPTIONS

Também é possível com o header "X-FRAME-OPTIONS" entretanto **esse header está obsoleto e já foi substituido totalmente pelo CSP**. Mas como tudo em tecnologia, o "legado" ainda persiste em larga escala na internet, então vamos entender um pouco o XFO:

> O Header X-Frame-Options tem três opcões: DENY, SAMEORIGIN, and ALLOW-FROM. 
> 
> O "DENY" previne que o conteúdo da sua página seja incorporado dentro de qualquer frame. 
> Enquanto a opção "SAMEORIGIN" permite que seja incorporado pelos frames apenas do mesmo domínio da página. 
> E o "ALLOW-FROM" basicamente permite que você especifique quais domínios tem autorizão para incorporar sua página em frames.

## Técnicas de Frame-Busting

Frame-busting envolve a criação de scripts que realizam validações recorrentes para descobrir se a página está sendo incorporada em algum frame, se estiver então geralmente o browser é forçado a redirecionar ou simplesmente quebrar o frame.

É a correção mais "manual" e antiga que temos. Portanto é pouco recomendada. Mas em casos onde a inserção dos headers de segurança não são aplicáveis esse tipo de solução se encaixa bem. Vou deixar dois exemplos abaixo de códigos frame-busting padrões.

<details>
<summary>frame-busting padrão 1</summary>

{% highlight javascript %}

<script>
    if (top != self) {
        top.location.replace(self.location.href);
    }
</script>

{% endhighlight %}

</details>

<details>
<summary>frame-busting padrão 2</summary>

{% highlight javascript %}

<script>
    if (window != window.top) {
        window.top.location = window.location;
    }
</script>

{% endhighlight %}

</details>

## Atributo SameSite

Apesar da flag SameSite com a diretiva "Lax" ou "Strict" ser bastante utilizada para prevenir contra roubo de cookies pelo XSS, também é possível utilizar essa flag para mitigar o clickjacking.

Se levarmos em consideração que boa parte dos ataques de clickjacking são relacionados a usuários interagindo com alguma sessão ativa como por exemplo uma sessão do seu ibanking, sabemos que por comportamento padrão do HTTP, que é um protocolo stateless, os dados de sessão estarão nos cookies.

Dito isso, ao restringir o acesso do iframe aos cookies, mitigamos os ataques de clickjacking que envolvem que a vítima esteja logada em algum site.

Para mais detalhes sobre esse recurso interessante recomendo a leitura: https://www.invicti.com/blog/web-security/same-site-cookie-attribute-prevent-cross-site-request-forgery/

---

# Conclusão

Clickjacking é uma vulnerabilidade que alguns programas de bugbounty desconsideram, e talvez o clickjacking só pelo clickjacking não até não aparentar  grandes ameaças, mas quando combinado com pharming, phishing, xss e outras vulnerabilidades pode ser uma perigosa alavanca para o sucesso dos ataques.

Dito isso, vale lembrar que é sempre válido analisar os headers de resposta da aplicação. Procure pelos headers CSP com a diretiva frame-ancestors e pelo x-frame-options também. Se não houver um desses especificados, ou conter apenas os "*-report-only", vale a pena o teste para clickjacking.

Espero que depois desse artigo o conceito de clickjacking tenha ficado mais claro e seu apetite por conhecer mais headers de resposta dos servidores tenha aumentado! Obrigado.

---

# Fontes e Referências

Fonte sobre CSP: https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/Content-Security-Policy

Fonte sobre x-frame-options estar obsoleto: https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/X-Frame-Options

Mais sobre implementação do CSP: https://content-security-policy.com/

OWASP sobre Clickjacking: https://owasp.org/www-community/attacks/Clickjacking

Como a flag "SameSite" pode mitigar o clickjacking: https://www.invicti.com/blog/web-security/same-site-cookie-attribute-prevent-cross-site-request-forgery/