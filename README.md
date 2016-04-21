# Politicos-brasileiros

API com todas as informações dos nossos queridos políticos.

Iniciarei esse projeto como exemplo de uma API feita com Node.js e MongoDB utilizando a arquitetura atômica ensinada no [Be MEAN](http://webschool.io/bemean).

Antes de metermos o dedo precisamos conhecer melhor a área onde iremos atuar e para isso copiei o texto abaixo que acredito explicar bem nossa situação atual:

> Pelas regras atuais, as eleições para presidente, governador, prefeito e senador seguem o sistema majoritário. No caso de deputados federais, estaduais, distritais e vereadores, o sistema utilizado hoje é o proporcional com lista aberta.
> 
> - Sistema Majoritário: Pelas regras atuais, as eleições para presidente, governador, prefeito e senador seguem o sistema majoritário. Geralmente, é eleito o candidato que receber a maioria absoluta dos votos válidos (mais da metade dos votos apurados, excluídos os votos em branco e os nulos). Se nenhum candidato atingir o número na primeira votação, realiza-se um segundo turno entre os dois mais votados.
> 
> No caso de eleição de prefeitos de municípios com menos de 200 mil eleitores, exige-se apenas a maioria relativa dos votos (o maior número dos votos apurados) e não há segundo turno.
> 
> - Sistema proporcional com lista aberta: No caso de deputados federais, estaduais, distritais e vereadores, o sistema utilizado hoje é o proporcional com lista aberta. É possível votar tanto no candidato como na legenda. Na apuração, deve-se contabilizar o total de votos obtidos por cada partido, somando os votos de legenda e os votos dos candidatos dessa legenda. As vagas são distribuídas de forma proporcional aos votos totais obtidos por cada partido. A partir daí, os partidos preenchem suas vagas conquistadas com seus candidatos com maior votação. É por isso que um candidato com muitos votos, ajuda a eleger candidatos de sua legenda ou coligação que tenha obtido menos votos.

*fonte: [http://www.ebc.com.br/noticias/politica/2013/07/como-funciona-o-sistema-eleitoral-brasileiro](http://www.ebc.com.br/noticias/politica/2013/07/como-funciona-o-sistema-eleitoral-brasileiro)*

## Entidades

Inicialmente definimos qual são as entidades do nosso sistema, você pode pensar nisso como *classes*:

- Partido
- Político
    + Denúncias
    + Condenações
    + Projetos

Percebeu que temos 3 entidades que estão diretamente ligadas à `Político` caso utilizássemos um banco relacional elas seriam tabelas separadas, porém como usaremos o MongoDb que é baseado em documento essas 3 entidades irão virar documentos *embedados* dentro da coleção de `politicos`.

Após definirmos as entidades precisamos definir cada campo(atômico) que os compõe.

### Partido

Pegando os dados [daqui http://www.tse.jus.br/partidos/partidos-politicos/registrados-no-tse](http://www.tse.jus.br/partidos/partidos-politicos/registrados-no-tse) temos:

- sigla
- nome
- numero
- presidente

Não adicionei o deferimento pois acho que de nada influencia.

Porém se lermos os dados disponibilizados pela [Wikipedia](https://pt.wikipedia.org/wiki/Lista_de_partidos_pol%C3%ADticos_no_Brasil) podemos adicionar:

- dataCriacao
- dataRegistroDefinitivo
- numeroAfiliados
- espectroPolitico
- ideologia

E se entramos na página de cada partido como em [http://www.tse.jus.br/partidos/partidos-politicos/partido-do-movimento-democratico-brasileiro](http://www.tse.jus.br/partidos/partidos-politicos/partido-do-movimento-democratico-brasileiro) podemos adicionar os seguintes dados:

- Endereço: Câmara dos Deputados, Ed. Principal, Sl. T6, Praça dos Três Poderes, Brasília-DF
- CEP: 70160-900
- Telefone: (61) 3215.9206/9209/9211/9215
- FAX: (61) 3215.9220
- Endereço Internet: www.pmdb.org.br
- Email: pmdb@pmdb.org.br / diretorionacional.pmdb@uol.com.br

Logo nossa Molécula(Schema) ficará:

```js
{
  sigla: String
, nome: String
, numeroPartido: Number
, presidente: String
, dataCriacao: Date
, dataRegistroDefinitivo: Date
, numeroAfiliados: Number
, espectroPolitico: String
, ideologia: String
, site: String
, email: String
, telefoneCompleto: String
, faxCompleto: String
, enderecoPartido: Object
, impeachment: Boolean
}
```

Não iremos separar o DDD do Telefone como seria o comum só porque não há necessidade de termos esses dados em separado e adicionei diretamente a *flag* `impeachment` para definir diretamente na sua entidade qual foi seu voto pois inicialmente é o foco principal do nosso sistema, cruzar os dados dos políticos e suas votações.

Perceba ali que uso `enderecoPartido` em vez de `endereco` pois o Átomo que temos de endereço é diferente do que precisamos aqui, assim como `telefoneCompleto` e ele é um `Object` pois na verdade ali nós teremos um outro *Schema* que será:

```js
{
  completo: String
, cep: String
}
```


### Político - Molécula

Agora chegamos no principal onde iremos definir quais Átomos(campos) compõe essa Molécula, para termos apenas os dados que interessa não colocaremos tudo, ficando assim:

```js
{
  partido_id: String
, nome: String
, cidade: String
, estado: String
, votosRecebidos: Number
, partidosAnteriores: Array
, denuncias: Array
, condenacoes: Array
, projetos: Array
, votacoes: Array
}
```

A informação dos `votosRecebidos` é muito importante para vermos quais são os políticos que realmente se elegeram com seus votos e a informação sobre os `partidosAnteriores` para sabermos se ele não honra sua legenda visto que o formato eleitoral brasileiro os votos recebidos antes vão para o Partido e depois para os Políticos.

Eu iria colocar em `votacoes` os dados de cada sessão, porém se formos cruzar dados de votantes de cada votação será melhor que separemos ela em uma coleção nova e nesse *Array* em Político precisaremos apenas adicionar `votacao_id` nesse *Array*, entretanto como não é nosso foco inicial irei deixar sem por hora.

## Campos - Átomos

> Como vimos anteriormente definimos já quais são nossos campos então o que eu preciso definir nessa etapa?

**A definição de cada campo/átomo.**

Então vamos separar cada átomo das 2 entidades!

```js
{
  sigla: String
, nome: String
, numeroPartido: Number
, presidente: String
, dataCriacao: Date
, dataRegistroDefinitivo: Date
, numeroAfiliados: Number
, espectroPolitico: String
, ideologia: String
, site: String
, email: String
, telefoneCompleto: String
, faxCompleto: String
, endereco: Object
, impeachment: Boolean
}

{
  partido_id: String
, nome: String
, cidade: String
, estado: String
, votosRecebidos: Number
, partidosAnteriores: Array
, denuncias: Array
, condenacoes: Array
, projetos: Array
, votacoes: Array
}
```

Para definirmos cada átomo usamos a seguinte estrutura para o *Mongoose*:

```js
{
  type: String
, validate: require('')
, required: true
}
```

### sigla

```js
{
  type: String
, validate: require('./../hadrons/siglaValidateMongoose')
, required: true
}
```

### nome

```js
{
  type: String
, validate: require('./../hadrons/nomeValidateMongoose')
, required: true
}
```

### numeroPartido

```js
{
  type: Number
, validate: require('./../hadrons/numeroPartidoValidateMongoose')
, required: true
}
```

### presidente

```js
{
  type: String
, validate: require('./../hadrons/presidenteValidateMongoose')
}
```

### dataCriacao

```js
{
  type: Date
, validate: require('./../hadrons/dataCriacaoValidateMongoose')
}
```

### dataRegistroDefinitivo

```js
{
  type: Date
, validate: require('./../hadrons/dataRegistroDefinitivoValidateMongoose')
}
```


### numeroAfiliados

```js
{
  type: Number
, validate: require('./../hadrons/numeroAfiliadosValidateMongoose')
}
```


### espectroPolitico

```js
{
  type: String
, validate: require('./../hadrons/espectroPoliticoValidateMongoose')
}
```


### ideologia

```js
{
  type: String
, validate: require('./../hadrons/ideologiaValidateMongoose')
}
```


### site

```js
{
  type: String
, validate: require('./../hadrons/siteValidateMongoose')
}
```


### email

```js
{
  type: String
, validate: require('./../hadrons/emailValidateMongoose')
}
```


### telefoneCompleto

```js
{
  type: String
, validate: require('./../hadrons/telefoneValidateMongoose')
}
```


### faxCompleto

```js
{
  type: String
, validate: require('./../hadrons/faxValidateMongoose')
}
```


### enderecoPartido

Esse Átomo na verdade é uma composição de outros 2 átomos:

- completo
- cep

Por isso ficará assim:

```js
{
  completo: {
    type: String
  , validate: require('./../hadrons/enderecoPartidoCompletoValidateMongoose')
  }
, cep: {
    type: String
  , validate: require('./../hadrons/enderecoPartidoCEPValidateMongoose')
  }
}
```

Porém são 2 átomos independentes.

### enderecoPartidoCompleto

```js
{
  type: String
, validate: require('./../hadrons/enderecoPartidoCompletoValidateMongoose')
}
```

### enderecoPartidoCEP

```js
{
  type: String
, validate: require('./../hadrons/enderecoPartidoCEPValidateMongoose')
}
```

### impeachment

```js
{
  type: String
, validate: require('./../hadrons/impeachmentValidateMongoose')
}
```


### partido_id

```js
{
  type: String
}
```


### cidade

```js
{
  type: String
, validate: require('./../hadrons/siglaValidateMongoose')
}
```


### estado

```js
{
  type: String
, validate: require('./../hadrons/siglaValidateMongoose')
}
```


### votosRecebidos

```js
{
  type: String
, validate: require('./../hadrons/siglaValidateMongoose')
}
```


### partidosAnteriores

```js
{
  type: String
, validate: require('./../hadrons/siglaValidateMongoose')
}
```


### denuncias

```js
{
  type: String
, validate: require('./../hadrons/siglaValidateMongoose')
}
```


### condenacoes

```js
{
  type: String
, validate: require('./../hadrons/siglaValidateMongoose')
}
```


### projetos

```js
{
  type: String
, validate: require('./../hadrons/siglaValidateMongoose')
}
```


### votacoes

```js
{
  type: String
, validate: require('./../hadrons/siglaValidateMongoose')
}
```

## Moléculas

### Partido

```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const Molecule = {
  sigla: require('./../atoms/sigla')
, nome: require('./../atoms/nome')
, numero: require('./../atoms/numero')
, presidente: require('./../atoms/presidente')
, dataCriacao: require('./../atoms/dataCriacao')
, dataRegistroDefinitivo: require('./../atoms/dataRegistroDefinitivo')
, numeroAfiliados: require('./../atoms/numeroAfiliados')
, espectroPolitico: require('./../atoms/espectroPolitico')
, ideologia: require('./../atoms/ideologia')
, site: require('./../atoms/site')
, email: require('./../atoms/email')
, telefoneCompleto: require('./../atoms/telefoneCompleto')
, faxCompleto: require('./../atoms/faxCompleto')
, enderecoPartido: require('./../atoms/enderecoPartido')
, impeachment: require('./../atoms/impeachment')
}

module.exports = new Schema(Molecule);
```




