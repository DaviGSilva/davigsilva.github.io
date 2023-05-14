---
author: Davi Silva
---
# Stateless ou não?
Recentemente levantei uma questão no trabalho sobre aplicações que trabalhei, quais são **statefull** e quais são **stateless**. Isso levantou vários pontos interessantes que gostaria de registrar, segue o fio da meada:

# Uma aplicação é statefull por que guarda sessão no próprio servidor?
**Sim**, quando uma aplicação salva a "sessão" do cliente na mesma máquina que ela está rodando, além de ser uma má prática de aplicações web (12factor), também está salvando o estado da requisição anterior na atual.
Isso tudo é no contexto Web :point_up

# Então quando uso $_SESSION do Php, meu app nunca vai ser stateless?
Não necessariamente, se você utilizar a $_SESSION na configuração padrão do Php, então sim, você está salvando sua sessão na mesma máquina do servidor. Só que é possível desacoplar sua session php da máquina, utilizando drivers de sessão.

### Driver de sessão
A maioria das linguagens e frameworks tem a possibilidade de configurar drivers de sessão, no universo php é muito comum utilizarmos o [Memcached](https://memcached.org/) ou [Redis](https://redis.io/).
Quando utilizamos o `$_SESSION`, podemos utilizar algum driver para armazenar essa sessão em um servidor à parte que está rodando Memcached, por exemplo.

Para detalhes de implementação com php acesse a documentação oficial:
- https://www.php.net/manual/en/function.session-set-save-handler.php
- https://www.php.net/manual/en/memcached.sessions.php

Notes: Se você utilizar Laravel, Yii, Zend ou qualquer outro framework, provavelmente tem uma forma super simples de configurar esse driver de sessão.

# Não consigo escalar uma aplicação porque ela é statefull?
Então... você consegue sim escalar sua aplicação, mesmo guardando estado nela.
Imagina que você tem uma aplicação em Php e MySQL, bem simples, rodando junto em um único servidor, atualmente essa aplicação só tem 1 requisição por segundo, daqui 1 ano tem mil, como que escala? Como não deixar meu servidor sobrecarregado?
Existem algumas possibilidades:
1. Aumentando recurso da máquina (Memória RAM e CPU)
2. Duplicando seu servidor (Ctrl C + Ctrl V da sua máquina)

### 1. Sobre aumentar recurso da máquina
Essa primeira opção é o que chamamos de escalar a aplicação verticalmente (Scaling up), imagina um servidor que você só vai adicionando mais e mais recursos, ele vai ficando mais robusto, com mais memória e mais processador, por exemplo. Ou seja, aumenta ele, vai "crescendo pra cima".

### 2. Sobre Ctrl C + Ctrl V do servidor
Essa segunda opção é o que chamamos de escalar a aplicação horizontalmente (Scaling out), imagina o horizonte, vai pros lados...
O cenário ideal é que essa aplicação esteja seguindo boas práticas de desenvolvimento web, 12 fatores (https://12factor.net), etc. Mas o Joãozinho tava preocupado em entregar o projeto o mais rápido possível, então essa aplicação monolítica tá usando session padrão do php, se eu duplicar o servidor o cliente X pode acessar qualquer um dos dois servidores e vai perder a sessão de login.
Quando temos esse cenário, podemos resolver com uma configuração chamada Sticky Sessions:

### Sticky Sessions
Sticky sessions é uma configuração de load balance que direciona todo o tráfego de um determinado cliente para uma máquina específica, exemplo: Existe os servidores A e B, que são os servidores duplicados, quando o cliente Fulaninho tenta acessar o app, primeiro bate no load balance e ele decide quem vai receber essa requisição, após isso ele pode "sticky"(grudar/salvar) esse cliente e sempre direcionar todas as requisições para o servidor A.

Esse tipo de estratégia é um paliativo, pois nada garante que o balanceamento de carga esteja 50/50 (50% para o servidor A e 50% para B), então algum pode ser sobrecarregado por exemplo. Isso é excelente para suprir a necessidade de escalar rapidamente, mas é importante freezar que é necessário resolver esse problema, existem duas soluções que poderiam ser utilizadas nesse caso:
1. Não utilizar sessão no mesmo servidor (Utilizar Memcached em outro servidor)
2. Mudar estratégia de login (Utilizar JWT, ao invés de session)

# Referências:
- Tudo começou por esse simples e excelente vídeo: https://youtu.be/SkXbGKeQPGM
- https://www.redhat.com/pt-br/topics/cloud-native-apps/stateful-vs-stateless , acesso em 13/05/2023 22h.
- https://stackoverflow.com/questions/13946033/is-it-recommended-to-store-php-sessions-in-memcache#answer-13946354
- https://12factor.net/pt_br/
