---
title: "Criando um Cinto de Utilidades CLI"
date: "2023-08-24"
draft: false
tags: ["typescript", "javascript", "node", "npm"]
slug: "cinto-de-utilidades-cli"
---

Olá, leitores!

Como um viciado em teclado, eu não consigo ficar muito tempo longe do computador. Na verdade, estou na segunda metade da minha licença-paternidade e ansioso para construir algo e ter um descanso das intermináveis fraldas sujas.

## A Ideia

Então, aqui está a ideia simples: Eu quero criar um programa _[CLI](https://pt.wikipedia.org/wiki/Interface_de_linha_de_comandos)_ para fornecer utilitários que eu uso diariamente como engenheiro de software. A linguagem que vou usar é o Typescript. Os utilitários, ou recursos, que eu quero incluir na primeira versão são:

1. Codificação/decodificação de strings Base64
2. _Pretty printing_/minificação de payloads JSON
3. Comparação de dois arquivos

Estes têm um escopo aceitável para começar, mas a intenção aqui é ter um conjunto em constante crescimento de utilitários que serão adicionados ao programa ao longo do tempo.

> Mas, Gabe... Typescript? Sério mesmo? Eu sei que você não é fã desse ecossistema. Por quê então?

Além da interface CLI que me permitirá integrá-lo ao meu fluxo de trabalho, que é fortemente baseado em terminal, quero criar esse "pacote" com interfaces bem definidas, para que os utilitários também possam ser consumidos por uma interface de usuário na web. E você sabe... quando se trata do navegador, precisamos recorrer ao Typescript/Javascript.

> E quanto ao WASM?

Bem, eu ainda não me aprofundei nisso. Na verdade, isso poderia ser algo interessante para experimentar no futuro.

> Esse conjunto de ferramentas não vai contra a filosofia minimalista do UNIX de ter uma ferramenta específica para uma tarefa específica?

Sim.

## O Planejamento

### Nomenclatura

Há um ditado de Phil Karlton no mundo da programação que diz que "há apenas duas coisas difíceis na Ciência da Computação: invalidação de cache e dar nomes às coisas". Eu diria que erros _off by one_ também são difíceis, embora isso seja uma conversa diferente.

Eu não acredito que essa iniciativa envolva lidar com cache, mas eu precisava de um nome para esse conjunto de ferramentas e depois de um pouco de reflexão (não muito profunda), decidi escolher `ubt`. Algumas razões por trás dessa escolha são:

- Eu não conheço outra ferramenta chamada `ubt`, embora provavelmente já exista uma. (Note que se você for o criador do `ubt` genuíno, por favor, perdoe minha falta de originalidade!)
- Se você se esforçar o suficiente, isso é um mnemônico para "cinto de utilidades"

### Ergonomia

Ao ter apenas algumas letras, isso pode ser rapidamente invocado no prompt de comando e tem o potencial de criar alguma memória muscular para os datilógrafos.

Além disso, enquanto no mundo da CLI, quero que cada utilitário seja invocado como um subcomando. Portanto, se considerarmos o conjunto de 3 utilitários que [liste acima](#a-ideia), esses podem ser `b64`, `json` e `diff`.

De fato, é possível dividir esses subcomandos em categorias distintas:

1. Utilitários que recebem um parâmetro: `b64`, `json`
2. Utilitário que recebe dois parâmetros: `diff`

Para o primeiro caso, quero que o programa leia a entrada do [stdin](https://pt.wikipedia.org/wiki/Fluxos_padr%C3%A3o#Entrada_padr%C3%A3o_(stdin)), para que possa interagir melhor com outras ferramentas. Por exemplo:

```shell
$ echo "string a ser codificada em base64" | ubt b64 -e
```
Por outro lado, para o segundo caso, usaremos argumentos posicionais:

```shell
$ ubt diff <arquivo1> <arquivo2>
```
## A Estruturação Inicial

Para começar, vale mencionar que [trabalhar em uma base de código Typescript já existente e configurar uma do zero são duas abordagens diferentes](https://www.linkedin.com/pulse/greenfield-vs-brownfield-giorgi-bastos/). Em minha opinião, o ecossistema Javascript é tremendamente fragmentado quando comparado a outros ecossistemas, como Go ou .NET — existem diversas maneiras e ferramentas para alcançar praticamente qualquer coisa que você possa imaginar. Até mesmo existe uma [piada constante sobre o número de dias desde o último lançamento de um framework Javascript](https://dayssincelastjavascriptframework.com/)! Mas isso não é necessariamente um problema, desde que você esteja familiarizado com as opções disponíveis.

Dito isso, tenho que admitir que me diverti muito configurando o projeto.

Quanto às ferramentas, escolhi utilizar:

- [Yarn](https://yarnpkg.com/) para gerenciamento de dependências
- [ESLint](https://eslint.org/) para análise de código
- [Prettier](https://prettier.io/) para formatação
- [Jest](https://jestjs.io/pt-BR/) para testes
- [Husky](https://typicode.github.io/husky/) para ganchos do git bacanas

Além disso, há também a transpilação do Typescript e suas configurações.

## A Execução

Temos todos os recursos e funcionalidades no lugar, mas ainda não temos código. Vamos analisar como as coisas foram construídas e por que foram construídas dessa maneira.

### Estrutura do Projeto

Optei por uma estrutura de diretórios "o mais plana possível". Dadas as poucas restrições para esse domínio de problema, não quis superengenheirar o projeto.

Aqui está como fica o diretório `src`, com os arquivos de teste omitidos:

```shell
$ tree src -I '*.test.ts'
src
├── base64.ts
├── cli.ts
├── diff
│   ├── index.ts
│   └── stringFormatter.ts
├── index.ts
└── json.ts

2 directories, 6 files
```
O arquivo `index.ts` é o ponto de entrada do programa. Ele importa a fábrica de análise de linha de comando definida em `cli.ts`, cria uma instância dela para analisar os argumentos fornecidos.

O arquivo `cli.ts` também é responsável por unir a funcionalidade definida em `base64.ts`, `diff` e `json.ts`.

A única razão pela qual o `diff` difere (jogo de palavras intencional!) das outras funcionalidades, tendo uma pasta em vez de um único arquivo, é porque ele foi construído de maneira que permite [extensões futuras](https://pt.wikipedia.org/wiki/Princ%C3%ADpio_do_aberto/fechado). Suponha que queiramos que esse pacote seja usado nos bastidores por uma interface de usuário da web — poderíamos criar um `htmlFormatter.ts`, ou algo semelhante, e herdar das estruturas exportadas em `diff/index.ts` para permitir _[DRYness](https://pt.wikipedia.org/wiki/Don%27t_repeat_yourself)_.

### Desenvolvimento Orientado a Testes

Decidi adotar um estilo de desenvolvimento [TDD](https://pt.wikipedia.org/wiki/Test-driven_development) para este pequeno projeto.

Além de toda a evangelização que as pessoas fazem sobre o TDD na internet, para mim, o benefício mais subestimado é a sensação de progresso obtida pelo rápido feedback que você recebe. Você vê os testes falharem, implementa um pouco de funcionalidade, executa os testes novamente e depois os vê passar. Quando se trabalha sozinho em um projeto, isso é essencial.

O segundo melhor efeito colateral dessa técnica é que você acaba com interfaces melhor definidas, já que você é obrigado, _a priori_, a escrever código de teste que atua como o artefato de software que vai consumir e invocar as funções/métodos em teste.

No entanto, para que o TDD seja eficaz, você precisa ter requisitos sólidos e bem definidos. Idealmente, você só começaria a implementá-lo após modelar o sistema ou passar por uma etapa de prototipagem, caso contrário, você acabaria tendo que gastar tempo descobrindo como seus componentes vão interagir uns com os outros e teria muito retrabalho corrigindo testes à medida que avança em direção à solução ideal.

## O Resultado

<script async id="asciicast-O3Q731849e7MSwUm1FmGBQ0dG" src="https://asciinema.org/a/O3Q731849e7MSwUm1FmGBQ0dG.js"></script>

## Utilização

Caso você queira usar esta ferramenta, você precisará de um gerenciador de pacotes Javascript.

Para o `yarn`, você precisará executar o seguinte comando:

```shell
$ yarn global add ubt
```
No caso de estar usando o `npm`, você precisará executar o seguinte comando:
```shell
$ npm install -g ubt
```

Isso deve adicionar o script `ubt` como um executável no diretório de instalação do seu gerenciador de pacotes. Certifique-se de que esse diretório esteja disponível em seu `$PATH`, para que você possa invocar o comando de qualquer diretório em seu sistema de arquivos.

## Referências

- [Repositório no Github](https://github.com/ewilazarus/ubt)
- [Pacote no Npm](https://www.npmjs.com/package/ubt)

-----

Obrigado por ler! Eu me diverti muito escrevendo este post no blog e criando essa ferramenta.

Fique ligado para mais!
