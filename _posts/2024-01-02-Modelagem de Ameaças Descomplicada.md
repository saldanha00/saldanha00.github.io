---
title: Modelagem de Ameaças (Threat Modelling) Descomplicada
categories: [cibersecurity, appsec]
img_path: ../../assets/images
image:
  path: Threat_Graph.gif
  alt: Threat Modelling
tags: [ciberecurity,appsec]     # TAG names should always be lowercase
---

- [Modelagem de Ameaças ( Threat Modelling )](#modelagem-de-ameaças--threat-modelling-)
  - [O que é a Modelagem de Ameaças?](#o-que-é-a-modelagem-de-ameaças)
    - [Diferenciação entre Ameaça x Vulnerabilidade.](#diferenciação-entre-ameaça-x-vulnerabilidade)
      - [Exemplos de cenários entre Ameaça x Vulnerabilidade](#exemplos-de-cenários-entre-ameaça-x-vulnerabilidade)
    - [Diferenciação entre Modelagem de Ameaças e Gerenciamento de Risco](#diferenciação-entre-modelagem-de-ameaças-e-gerenciamento-de-risco)
    - [Por que devo incluir uma modelagem de ameaças?](#por-que-devo-incluir-uma-modelagem-de-ameaças)
      - [Resumindo as principais vantagens:](#resumindo-as-principais-vantagens)
  - [Começando a Modelar: Decomposição do Sistema](#começando-a-modelar-decomposição-do-sistema)
    - [1 - Quais as dependências externas?](#1---quais-as-dependências-externas)
    - [2 - Quais os Pontos de Entrada? (EntryPoints)](#2---quais-os-pontos-de-entrada-entrypoints)
    - [3 - Quais os Pontos de Saída? (ExitPoints)](#3---quais-os-pontos-de-saída-exitpoints)
    - [4 - Quais os Assets?](#4---quais-os-assets)
    - [5 - Quais os Níveis de acesso? (Trust Levels)](#5---quais-os-níveis-de-acesso-trust-levels)
  - [Classificação das Ameaças](#classificação-das-ameaças)
    - [Tipos de Ameaças](#tipos-de-ameaças)
    - [Tabela Informativa STRIDE](#tabela-informativa-stride)
    - [Mais exemplos do STRIDE](#mais-exemplos-do-stride)
    - [Análise de Ameaças](#análise-de-ameaças)
  - [Contramedidas e Recomendações](#contramedidas-e-recomendações)
    - [STRIDE - Técnicas de mitigação de ameaças](#stride---técnicas-de-mitigação-de-ameaças)
    - [ASF -  Ténicas de mitigação de Ameaças](#asf----ténicas-de-mitigação-de-ameaças)
  - [Conclusão](#conclusão)
  - [Referências](#referências)



# Modelagem de Ameaças ( Threat Modelling )


## O que é a Modelagem de Ameaças?

É uma etapa no processo de desenvolvimento de software seguro, **idealmente realizada durante a fase de planejamento do software** (antes de iniciar a codificação) mas nada impede de fazer em outros momentos. Essa etapa **visa identificar as os tipos de ataques que aquele software poderá encontrar durante o seu ciclo de vida**, além de **pontuar as contramedidas necessárias para evitar ou mitigar que aquelas ameaças se tornem vulnerabilidades**.

Pode-se dizer que **A modelagem parte de uma observação do sistema a partir de uma perspectiva atacante**. Se eu fosse atacar esse sistema amanhã, por onde eu começaria ? É com essa linha de raciocínio que a modelagem deve fluir.

Em sumo, a Modelagem ocorre em três etapas que estão resumidas abaixo, mas iremos abordar com mais profundidade no decorrer do artigo.

<details>
<summary> 1 - Decomposição do sistema </summary>
Ocorre o entendimento completo e detalhado do funcionamento do sistema e todos os fluxos envolvidos entre os componentes.<br><br>Por exemplo, fluxo de comunicação entre uma aplicação spring boot e um serviço de mensageria rabbitMQ ou Kafka.<br><br>Ou entender que tipo de dados trafegam entre minha api e o banco de dados. Esse tipo de entendimento que consiste na decomposição do sistema.
</details>

<details>
<summary> 2 - Classificação das ameaças </summary>
Uma vez compreendido o sistema, é possível identificar quais as ameaças são possíveis e classifica-las. Geralmente utilizando algum framework.<br><br>Veremos mais a frente o framework STRIDE que nos ajuda a classificar as ameaças.
</details>

<details>
<summary> 3 - Determinação de contramedidas e Mitigações </summary>
Com as ameaças devidamente identificadas e classificadas é possível pensar em recomendações de segurança.<br><br>As Recomendações devem ser adaptadas para cada caso, entretanto algumas são de bases fundamentais como tráfego de dados utilizando protocolo seguro (TLS) ou criptografia de dados sensíveis em repouso.
</details>

### Diferenciação entre Ameaça x Vulnerabilidade.

Uma Ameaça é uma vulnerabilidade em Potencial. É uma ação ou evento que pode ocorrer com intuito de causar danos a um ativo, geralmente um sistema. Já a Vulnerabilidade é a certeza de uma falha em um sistema que leva a exploração de uma ameaça, geralmente contém evidências da sua existência como um printscreen ou uma PoC.

#### Exemplos de cenários entre Ameaça x Vulnerabilidade

{: .prompt-warning }
>"Exemplo 01 - Diferença entre Ameaça x Vulnerabilidade?"
    Considere o cenário: "Meu time planeja subir uma API com autenticação por tokens que irá realizar consultas no banco de dados, esses tokens são armazenados no código da aplicação".<br><br>Nesse cenário a ameaça seria um atacante adquirir o token de forma ilícita e realizar a autenticação indevida. E uma vulnerabilidade seria o armazenamento inseguro desses tokens. E a contramedida seria utilizar um cofre de segredos para armazenamento seguro de credenciais.

{: .prompt-warning }
>"Exemplo 02 - Diferença entre Ameaça x Vulnerabilidade? - Descarte de vulnerabilidades"
    Considere o cenário: "Estou planejando desenvolver um sistema que irá desacoplar uma feature do monolito e passará a receber dados bancários e indexar em uma base de dados. A aplicação irá conter 03 componentes: kafka, redshift e spring boot application".<br><br>Nesse cenário não há vulnerabilidades, pois o sistema ainda não existe, portanto não há falha existente. Entretanto existem ameaças como: Um atacante pode interceptar os dados bancários entre o kafka e o springboot ou entre o springboot e o redshift, a contramedida seria utilizar protocolo de comunicação seguro (TLS), outra ameaça possível seria que um atacante poderia negar o serviço do spring boot application utilizando ataques DoS, a contramedida seria adicionar um rate-limite na aplicação.

{: .prompt-warning }
>"Exemplo 03 - Diferença entre Ameaça x Vulnerabilidade? - Descarte de Ameaças"
    Considere o mesmo cenário: "Estou planejando desenvolver um sistema que irá desacoplar uma feature do monolito e passará a receber dados bancários e indexar em uma base de dados. A aplicação irá conter 03 componentes: kafka, redshift e spring boot application".<br><br> Como mencionei antes, não há vulnerabilidades, mas quais as ameaças não entrariam nesse cenário ? Um exemplo poderia ser uma ameaça do tipo "Escalação de privilégios" pois provavelmente esse spring boot aplication teria apenas um nível de acesso, então não faria sentido uma ameaça de escalação de privilégios, o qual iria precisar de pelo menos dois níveis de acesso, geralmente um nível de usuário e outro de administrador.

### Diferenciação entre Modelagem de Ameaças e Gerenciamento de Risco

A modelagem tem o intuito de identificar os tipos de ataques que podem comprometer o sistema e geralmente é realizado por pela engenharia de cibersegurança ou pela engenharia de software.

Gerenciamento de risco ou Risk Management, como é mais falado, vai tratar sobre decisões e "trade-offs" sobre como lidar com os riscos e ameaças identificados. Geralmente é realizado por profissionais de gestão de risco e advogados.

### Por que devo incluir uma modelagem de ameaças?

Agora que você já sabe a relação entre ameaça e vulnerabilidade, deve ter percebido que **A vulnerabilidade nada mais é do que a ameaça que não foi tratada**. Então imagine que você tem duas opções:

<details>
<summary>1 - Não Identificar as ameaças que sua aplicação estará sujeita a enfrentar. (Não fazer a modelagem)</summary>
Consequência 1: Quando a aplicação estiver rodando em produção, a chance do sistema estar inseguro é grande. Por mais experiente que seja o seu time de desenvolvimento, todo sistema está sujeito a falhas. Então um atacante poderá explorar uma vulnerabilidade na sua aplicação.<br><br>

Consequência 2:  Se houver um processo de auditoria envolvendo SoX, PCI, Sigilo Bancário ou LGPD, provavelmente sua aplicação precisará de muito mais esforço para se adequar as normas de compliance. Apesar das normas serem diferentes, muitas práticas de segurança são gerais e se repetem entrem as normativas.<br><br>

Consequência 3:  Se um time não tem o hábito de desenvolver software seguro, não desenvolve o mindset de segurança o que acaba dificultando a evolução do ecossistema da empresa, além de dar exemplos ruins para os recém chegados e outros times.
</details>

<details>
<summary>2 - Fazer o processo de modelagem de ameaças e identificar as ameaças que a sua aplicação estará sujeita a enfrentar.</summary>

Consequência 1: Sua aplicação recebe um artefato de segurança (um documento oficial) que sua aplicação passou pela etapa de modelagem de ameaças e que já identificamos as ameaças envolvidas.<br><br>

Consequência 2: Todo o processo de desenvolvimento do software vai levar em consideração a modelagem realizada, ou seja, não vamos gastar tempo desenvolvendo uma feature insegura só para ter que refazer mais tarde.<br><br>

Consequência 3: Seu time ganha mais expertiese com threat modelling, aumenta a colaboração entre times (Engenharia de software e Appsecs) e sua aplicação fica mais próxima de alcançar o ideal do SSDLC (Secure Software Develoment Life Cycle)
</details>

#### Resumindo as principais vantagens:

- Melhor entendimento e clareza dos pontos referentes a segurança da sua aplicação.
- Entrega de produtos mais seguros e melhores.
- Força o time a pensar em possíveis brechas da aplicação, levando a proatividade na correção de bugs.
- Um passo mais próximo de estar na esteira segura (S-SDLC).

---

## Começando a Modelar: Decomposição do Sistema

Como citamos anteriormente, A modelagem pode ser separada em 03 etapas, vamos falar da primeira etapa aqui.

**O objetivo principal dessa etapa é adquirir conhecimento do sistema e como ele interage com os componentes externos.**

Idealmente, utilizamos um Diagrama, ou uma documentação que possamos visualizar a arquitetura da aplicação e todos os seus componentes, preferencialmente um DFD (Data Flow Diagram). Caso não tenha um diagrama, é possível realizar a modelagem pois no final o resultado será um diagrama também, entretanto um DFD pré-existente torna o processo menos oneroso.

A partir de agora iremos seguir uma sequência lógica do que devemos analisar durante essa etapa.

### 1 - Quais as dependências externas?

A primeira ação é identificar no sistema o que é externo e que não é controlável pelo time.

Por exemplo: Se a minha aplicação dispara uma trigger que envia mensagens para um canal do Microsoft Teams, o componente "Microsoft Teams" é uma dependência externa. Isso é importante porque se eu identificar ameaças nesse componente externo, o nosso time não irá conseguir trata-la pois o controle é externo a nós.

Outro exemplo são APIs de outros times, se eu consumo uma API do monolito, não vale a pena mapear as ameaças desse componente externo nessa modelagem de ameaças, pois pertence a outro time e cabe a eles o desenvolvimento da solução. Nesse caso o mais correto seria fazer outra modelagem para essa API externa. Dessa forma conseguimos manter o escopo fechado e bem organizado.

### 2 - Quais os Pontos de Entrada? (EntryPoints)

Agora que sabemos o que exatamente é do nosso escopo, podemos partir para os pontos de entrada do sistema, ou seja: **Por onde é possível um atacante interagir com aquele sistema ?**

Por exemplo, imagine um ibanking onde temos a **página de login** que é um ponto de entrada, tem a página principal onde é possível interagir com os **botões** de descrição dos itens, tem o **campo de busca** onde é possível interagir provavelmente com o banco de dados através das consultas no campo "search".

Algumas APIs possuem o **swagger** que documentam os entrypoints. 

É importante mapear quais entrypoints o atacante poderia utilizar.

Falando em entrypoints é crucial verificar se existe segregação de acessos através de níveis de acesso, por exemplo: No site do ibanking ibanking.com todos tem acesso, é público. Entretanto para entrar no ibanking é preciso ter um login, o qual requer um nível de acesso diferente.

### 3 - Quais os Pontos de Saída? (ExitPoints)

Tão importante quanto os pontos de entrada, são os de saída, principalmente em ataques que ocorrem no lado do cliente como XSS (Cross-Site Scripting) e Information Disclosure.

Imagine que alguém está tentando fazer login na sua conta. E essa pessoa tem uma arquivo .txt que contém 10 mil possíveis combinações de usuários com o seu nome, e que toda vez que esse atacante erra a tentativa de login o sistema informa: "Login e senha incorretos". Entretanto quando o atacante acertar o seu login corretamente o sistema informa: "Senha incorreta". Isso significa que o atacante conseguiu enumerar o seu usuário a partir do ExitPoint da tela de login. Nesse caso o correto seria ter uma mensagem padrão "Usuário ou Senha Incorretos" tanto para erro no campo usuário quanto no campo de senha.

Também **pode levar a SQLInjection**.

Então, mapeamos também os ExitPoints.

### 4 - Quais os Assets?

Um sistema pode conter 10 vulnerabilidades e nenhuma ser atrativa para um atacante, por exemplo: Se um blog que dispõe poemas sobre flores tiver vulnerabilidades, não necessariamente ele irá ser atacado porque os assets podem não ser atrativos para o atacante.

Então, para a modelagem de ameaças os Assets são como os "Alvos", o tesouro que o atacante busca conseguir. Geralmente são listas de clientes, dados bancários, transações financeiras, credenciais de acesso... As vezes o alvo é o frontpage da página apenas para fazer um anúncio (popularmente chamado de defacement). As vezes o objetivo é apenas causar lentidão e assim alterar a experiência dos usuários. As variações dependem muito da origem do ataque e da experiência do atacante.

Mas via de regra o asset é aquilo que o atacante buscaria.

No [NIST SP 800-154](https://csrc.nist.gov/files/pubs/sp/800/154/ipd/docs/sp800_154_draft.pdf) é disposto sobre modelagem de ameaças "data-centric" que leva em consideração dados PII o principal alvo do atacante. Vale a pena a leitura se esse for o seu caso.

É importante ressaltar que asset nem sempre é relacionado a dados, pode ser simplesmente uma mensagem de erro que permite enumeração de usuário ou a possibilidade de deletar usuários no banco de dados.

Tenha em mente que assets é aquilo que é "atrativo" para o atacante.

O OWASP tem uma lista bem robusta de exemplos de assets [aqui](https://owasp.org/www-community/Threat_Modeling_Process#stride)

Ou seja, identifique no sistema quais são os Assets que o atacante buscaria.

### 5 - Quais os Níveis de acesso? (Trust Levels)

Identifique quais e quantos níveis de acesso existem no sistema, seguem alguns exemplos:

Acesso anônimo (usualmente de páginas publicadas na internet)<br>
Acesso de leitura (Read-Only)<br>
Acesso de leitura e escrita<br>
Acesso Administrativo<br>
Acesso segregado (funções específicas)

---

## Classificação das Ameaças

Nessa segunda etapa, com todo o conhecimento adquirido sobre o sistema identificamos quais ameaças o sistema está sujeito.

### Tipos de Ameaças

Por ser um conceito genérico, existem diversas ameaças no mundo de cibersegurança, para ajudar a não se perder adotamos o **framework STRIDE** que classifica as ameaças em tipos de ameaças. STRIDE é a junção das iniciais dos 06 principais tipos de ameaças. Leia com carinho a tabela abaixo para entender cada tipo de ameaça:

### Tabela Informativa STRIDE

| Inicial | Tipo de Ameaça |  Descrição | Exemplos de ameaças  |
|:-:|:--:|:-----------------------------|:--------------------:|
| **S** | Spoofing  | **Fingir ser quem você não é (Falsificação)**  |  Exemplo 1: Um atacante se faz passar por um usuário legítimo para obter acesso a informações confidenciais.<br>Exemplo 2: Um atacante manipula o endereço IP de origem de uma comunicação para parecer que está vindo de uma fonte confiável.<br>Exemplo 3: Um hacker falsifica um certificado SSL para enganar os usuários e interceptar comunicações seguras.   |
| **T** | Tampering  | **Alteração indevida de dados ferindo a integridade do componente (Alteração)** |  Exemplo 1: Um invasor modifica o conteúdo de uma mensagem enviada pela rede para alterar seu significado ou executar ações indesejadas.<br>Exemplo 2: Um atacante altera os dados armazenados em um banco de dados, comprometendo sua integridade.<br>Exemplo 3: Um hacker modifica o código-fonte de um aplicativo para introduzir comportamentos maliciosos.   |
| **R** | Repudiation  | **O ato de não conseguir provar um ataque por insuficiente de logs (Repúdio)**       |  Exemplo 1: Um usuário mal-intencionado realiza uma transação financeira e nega ter feito a operação, alegando que foi um erro do sistema.<br>Exemplo 2: Um invasor altera os logs de auditoria para remover evidências de suas atividades maliciosas.<br>Exemplo 3: Um hacker envia uma mensagem de e-mail em nome de outra pessoa e depois nega ter enviado a mensagem.  |
| **I** | Information Disclosure  | **Vazamento indevido de informações sensíveis (Vazamento de dados)**       |  Exemplo 1: Um funcionário desonesto obtém acesso não autorizado a informações confidenciais da empresa e as vende para concorrentes.<br>Exemplo 2: Um atacante explora uma vulnerabilidade em um aplicativo web para obter acesso a dados pessoais de usuários.<br>Exemplo 3: Um invasor intercepta uma comunicação sem fio não criptografada para obter informações sensíveis.  |
| **D** | Denial of Service  | **Sobrecarrega o sistema a ponto de afetar a experiência dos clientes (Negação de Serviço)**       |  Exemplo 1: Um ataque de inundação de tráfego sobrecarrega um servidor, tornando-o inacessível para usuários legítimos.<br>Exemplo 2: Um invasor utiliza técnicas de ataque distribuído de negação de serviço (DDoS) para sobrecarregar uma rede ou serviço.<br>Exemplo 3: Um hacker explora uma vulnerabilidade de exaustão de recursos para esgotar os recursos de um sistema, tornando-o com alta lentidão ou indisponível.  |
| **E** | Elevation of Privileges  | **Um usuário consegue obter acesso a recursos não-autorizados (Elevação de Privilégios)**       |  Exemplo 1: Um invasor obtém acesso a um sistema com privilégios de usuário comum e, em seguida, explora uma vulnerabilidade para obter privilégios administrativos.<br>Exemplo 2: Um atacante manipula um cookie de autenticação para elevar seus privilégios em um aplicativo web.<br>Exemplo 3: Um atacante utiliza uma técnica de escalada de privilégios para obter acesso a informações ou recursos restritos.  |


### Mais exemplos do STRIDE

Eu acredito que os exemplos são a melhor abordagem para entender as ameaças do STRIDE, então vou deixar um link do artigo ["The Threats To Our Products"](https://adam.shostack.org/microsoft/The-Threats-To-Our-Products.docx), criado pelos *Praerit Garg and Loren Kohnfelder*, os principais criadores do STRIDE.

> Curiosidade: O artigo foi publicado dia 01 de abril 1999, bem no dia mentira D:


### Análise de Ameaças

Talvez a parte mais fundamental da modelagem de ameaças é a análise das ameaças Ou tão fundamental quanto a Decomposição da aplicação.

Nessa parte iremos analisar os dados que obtivemos até agora e iremos pensar de uma perspectiva de atacante, pensando em casos de uso e de abuso. Nas imagens abaixo podemos perceber a lógica por trás da análise de ameaças

<img src="UseAndMisuseCase.jpg"/>

<img src="Threat_Graph.gif"/>

Após as devidas análises, teremos identificados todas as ameaças para o sistema.

## Contramedidas e Recomendações

Essa etapa final é para pensar em recomendações que evitem ou mitiguem cada ameaça identificada. Na prática ao pensar na ameaça já pensamos no que fazer para evitar, mas para fins didáticos separamos em uma etapa diferente, afinal é a etapa que leva ao relatório de ameaças.

Cada recomendação deve se adaptar ao cenário do sistema, mas temos alguns pontos de partida.

Vamos ver inicialmente as mitigações gerais e depois as específicas:

### STRIDE - Técnicas de mitigação de ameaças

| Relação com STRIDE | Técnica de Mitigação |
|--------------------|----------------------|
| Spoofing | 1 - Autenticação Segura<br>2 - Proteção de credenciais de acesso como cookies senhas e tokens<br>3 - Evitar armazenamento de credenciais ou armazenar em locais seguros |
| Tampering | 1 - Autorização Segura (Níveis de acesso bem definidos)<br>2 - Utilização de Hashes <br>3 - Utilização de MACs (Mandatory Access Control)<br> 4 - Assinaturas digitais<br>5 - Protocolos de proteção contra tampering como TLS, PGP, SSH, Kerberos ou IPSec|
| Repudiation | 1 - Assinaturas digitais<br>2 - Timestamps (Data e hora de cada acesso ou ação importante) nos logs<br>3 - Trilhas de auditoria |
| Information Disclosure | 1 - Utilizar mecanismos de Autorização<br>2 - Utilizar protocolos do tipo Privacy-enhanced como ZKP (Zero-Knowledge Proof)<br>3 - A boa e velha Criptografia dos dados<br> 4 - Proteção de dados sensíveis<br>5 - Não armazenar dados sensíveis ou armazenar em locais seguros (não é no código da aplicação) |
| Denial of Service | 1 - Autenticação Segura<br> 2 - Autorização Segura<br>3 - Filtering<br>4 - Throttling<br>5 - Utilizar QoS (Quality of Service) |
| Elevation of Privilege | 1 - Utilizar o princípio do menor privilégio, sempre.<br>2 - Segregação de funções bem definidas |


### ASF -  Ténicas de mitigação de Ameaças

Relacionado a Contramedidas e Recomendações, utilizar mais de um framework pode ajudar a dar ideias personalizadas, vamos trazer só alguns exemplos de recomendações de um outro framework, diferente do STRIDE que é o **ASF (Application Security Frame)**.

| Tipo de Ameaça ASF | Contramedida |
|--------------------|--------------|
|Autenticação|1 - Credenciais e tokens de autenticação protegidos com criptografia em trânsito e em repouso como TLS e SSE<br>2 - Mecanismos de proteção contra ataques de dicionário ou força bruta como Captcha ou bloqueio de sessão por x minutos<br>3 - Política forte de senhas<br>4 - Credenciais salvas em formato de hash<br>5 - Resets de senhas não revelam dicas ou validam logins<br> |
|Autorização|1 - ACLs fortes usadas para reforçar autorização a recursos específicos<br>2 - Roles de controle de acesso utilizadas para restringir operações<br>3 - Seguir o princípio do mínimo privilégio<br>4 - Segregação de privilégios devidamente configurada.|
|Configuração de Sistema|1 - Princípio do privilégio mínimo reforçado<br>2 - Auditoria e Logs de todos os acessos administrativos habilitado<br>3 - Acesso a páginas e entrypoints de configuração de sistema sejam restritos apenas a administradores|
|Proteção de Dados em Base de dados ou Trânsito|1 - Utilizar os padrões de algoritmos de criptografia mais seguros como SHA512 ou AES256<br>2 - Utilização de HMACs (Hashed Message Authentication Codes), geralmente utilizado para verificar integridade dos dados durante um transporte.<br>3 - Dados confidenciais devidamente protegidos em transporte e em repouso<br>4 - Não enviar nenhum tipo de dado sensível em texto claro (plaintext) |
|Validação de Dados / Validação de Parâmetros|1 - Validação do tipo de dados, tamanho, IP de origem devem ser reforçados<br>2 - Todo dado recebido do cliente deve ser validado (Entrypoints)<br>3 - Não utilizar parâmetros que podem ser manipulados pelo cliente diretamente em consultas SQL ou outros campos de decisão.<br>4 - Validar entrada de dados utilizando allowlist<br>5 - Utilizar codificação de output (evitando XSS) |
|Tratamento de Erros e Gereciamento de Exceções|1 - Todas as exceções são tratadas de maneira estruturada<br>2 - Privilégios são restauradas para o nível apropriado após erros ou exceções<br>3 - Mensagens de erros devem ser genéricas para o cliente, evitando conceder informações importantes para o atacante como versão de banco ou outras tecnologias|
|Gerenciamento de Usuários e Sessões|1 - Nenhuma informação sensível deve ser armazenada em texto claro (principalmente nos cookies)<br>2 - O conteúdo dos cookies de autenticação deve ser criptografado<br>3 - Cookies devem ser configurados para expirar em tempo hábil (não deixar vários dias)<br>4 - Sessões devem ser testadas contra "replay attacks" que basicamente envolve um atacante repetindo mensagens capturadas anteriormente. Para evitar é interessante utilizar timestamps, https, nonce (número usado uma única vez) ou tokens de sessão<br>5 - Re-autenticação do usuário quando utilizar funções críticas, muito utilizado em ibankings antes de transferir dinheiro por exemplo.<br>6 - Sessões devem ser expiradas após o logout.|
|Auditoria e Logs|1 - Informações sensíveis não devem ser logadas, ou logadas de forma mascarada ou criptografada.<br>2 - Controle de acesso reforçado para visualizar os logs da aplicação<br>3 - Controles de integridade nos arquivos de logs para evitar não-repudiação<br>4 - Sempre logar eventos chaves (principalmente administrativos e financeiros)<br>5 - Ter uma trilha de auditoria bem definida|

---

## Conclusão

Modelagem de ameaças é um passo importante se você deseja construir sistemas seguros, pode ser de grande valia no ínicio do projeto, mas também pode ser aproveitado nos outros momentos do ciclo de vida do software.

Sun Tzu na arte da guerra explica que a vitória está diretamente ligada ao conhecimento que temos de nós mesmos e do seu inimigo. A modelagem exige um brando conhecimento da sua aplicação e do seu atacante.

## Referências

- NIST SP 800-154 sobre Threat Modelling - https://csrc.nist.gov/files/pubs/sp/800/154/ipd/docs/sp800_154_draft.pdf
- OWASP Threat Modelling STRIDE - https://owasp.org/www-community/Threat_Modeling_Process#stride
