## Introdução ao Jooby

# 1. Visão Geral
Jooby é uma micro estrutura da web escalonável e rápida construída sobre os servidores da web NIO mais usados. É muito simples e modular, claramente projetado para a arquitetura da web dos dias modernos. Ele vem com suporte para Javascript e Kotlin também.

Por padrão, Jooby vem com ótimo suporte para Netty, Jetty e Undertow.

Neste artigo, aprenderemos sobre a estrutura geral do projeto Jooby e como construir um aplicativo da web simples usando Jooby.

# 2. Arquitetura do aplicativo
Uma estrutura de aplicativo simples do Jooby será semelhante a abaixo:

```
├── public
|   └── welcome.html
├── conf
|   ├── application.conf
|   └── logback.xml
└── src
|   ├── main
|   |   └── java
|   |       └── com
|   |           └── isaccanedo
|   |               └── jooby
|   |                   └── App.java
|   └── test
|       └── java
|           └── com
|               └── isaccanedo
|                   └── jooby
|                       └── AppTest.java
├── pom.xml
```

O ponto a ser observado aqui é que no diretório público podemos colocar arquivos estáticos como css/js/html etc. No diretório conf, podemos colocar qualquer arquivo de configuração que um aplicativo precise, como logback.xml ou application.conf etc.

# 3. Dependência Maven
Podemos criar um aplicativo Jooby simples adicionando a seguinte dependência em nosso pom.xml:

```
<dependency>
    <groupId>org.jooby</groupId>
    <artifactId>jooby-netty</artifactId>
    <version>1.1.3</version>
</dependency>
```

Se quisermos escolher Jetty ou Undertow, podemos usar a seguinte dependência:

```
<dependency>
    <groupId>org.jooby</groupId>
    <artifactId>jooby-jetty</artifactId>
    <version>1.1.3</version>
</dependency>
<dependency>
    <groupId>org.jooby</groupId>
    <artifactId>jooby-undertow</artifactId>
    <version>1.1.3</version>
</dependency>
```

Você pode verificar a versão mais recente do projeto Jooby no Repositório Central Maven.

Jooby também tem um arquétipo Maven dedicado. Podemos usá-lo para criar um projeto de amostra com todas as dependências necessárias pré-construídas.

Podemos usar o seguinte script para gerar o projeto de amostra:

```
mvn archetype:generate -B -DgroupId=com.isaccanedo.jooby -DartifactId=jooby 
-Dversion=1.0 -DarchetypeArtifactId=jooby-archetype 
-DarchetypeGroupId=org.jooby -DarchetypeVersion=1.1.3
```

# 4. Construindo um aplicativo
### 4.1. Iniciando o servidor
Para iniciar o servidor incorporado, precisamos usar o seguinte snippet de código:

```
public class App extends Jooby {
    public static void main(String[] args) {
        run(App::new, args);
    }
}
```

Uma vez iniciado, o servidor estará rodando na porta padrão 8080.

Também podemos configurar o servidor back-end com porta personalizada e porta HTTPS personalizada:

```
{
    port( 8081 );
    securePort( 8443 );
}
```

### 4.2. Implementando o Roteador
É muito fácil criar um roteador baseado em caminho no Jooby. Por exemplo, podemos criar um roteador para o caminho '/login' da seguinte maneira:

```
{
    get( "/login", () -> "Hello from isaccanedo");
}
```

De forma semelhante, se quisermos lidar com outros métodos HTTP como POST, PUT, etc, podemos usar o snippet de código abaixo:

```
{
    post( "/save", req -> {
        Mutant token = req.param( "token" );
        return token.intValue();
    });
}
```

Aqui, estamos obtendo o token de nome do parâmetro de solicitação da solicitação. Por padrão, todos os parâmetros de solicitação são inseridos no tipo de dados Mutant de Jooby. Com base na expectativa, podemos convertê-lo em qualquer tipo de dados primitivo com suporte.

Podemos verificar qualquer parâmetro de url da seguinte maneira:

```
{
    get( "/user/{id}", req -> "Hello user : " + req.param("id").value() );
    get( "/user/:id", req -> "Hello user: " + req.param("id").value() );
}
```

Podemos usar qualquer uma das opções acima. Também é possível encontrar parâmetros começando com um conteúdo fixo. Por exemplo, podemos encontrar um parâmetro de URL começando com 
'uid:' da seguinte maneira:

```
{
    get( "/uid:{id}", req -> "Hello User with id : uid" + 
        req.param("id").value());
}
```

### 4.3. Implementando controlador de padrão MVC
Para um aplicativo corporativo, Jooby vem com uma API MVC muito parecida com qualquer outra estrutura MVC como Spring MVC.

Por exemplo, podemos lidar com um caminho chamado '/hello':

```
@Path("/hello")
public class GetController {
    @GET
    public String hello() {
        return "Hello isaccanedo";
    }
}
```

De maneira muito semelhante, podemos criar um manipulador para manipular outros métodos HTTP com a anotação @POST, @PUT, @DELETE, etc.

### 4.4. Tratamento de conteúdo estático
Para servir qualquer conteúdo estático como HTML, Javascript, CSS, imagem, etc., precisamos colocar esses arquivos no diretório público.

Uma vez colocado, a partir do roteador, podemos mapear qualquer url para estes recursos:

```
{
    assets( "/employee" , "form.html" );
}
```

### 4.5. Formulário de Manuseio
A interface de solicitação do Jooby, por padrão, lida com qualquer objeto de formulário sem usar qualquer conversão de tipo manual.

Vamos supor que precisamos enviar os detalhes do funcionário por meio de um formulário. Na primeira etapa, precisamos criar um objeto bean Employee que usaremos para manter os dados:

```
public class Employee {
    String id;
    String name;
    String email;

    // standard constructors, getters and setters
}
```

Agora, precisamos criar uma página para criar o formulário:

```
<form enctype="application/x-www-form-urlencoded" action="/submitForm" 
    method="post">
    <input name="id" />
    <input name="name" />
    <input name="email" />
    <input type="submit" value="Submit"/>
</form>
```

A seguir, criaremos um gerenciador de postagem para abordar este formulário e buscar os dados enviados:

```
post( "/submitForm", req -> {
    Employee employee = req.params(Employee.class);
    // ...
    return "empoyee data saved successfullly";
});
```

O ponto a ser observado aqui é que devemos declarar form enctype como 
application/x-www-form-urlencoded para suportar a vinculação dinâmica de formulário.

Por Request.file (String filename), podemos recuperar o arquivo enviado:

```
post( "/upload", req -> {
    Upload upload = req.file("file");
    // ...
    upload.close();
});
```

### 4.6. Implementando um Filtro
Fora da caixa, Jooby fornece a flexibilidade para definir filtros globais, bem como os filtros baseados em caminho.

Implementar o filtro no Jooby é um pouco complicado, pois precisamos configurar o caminho da URL duas vezes, uma para o filtro e outra para o manipulador.

Por exemplo, se tivermos que implementar um filtro para um caminho de URL chamado '/filtro', precisamos implementar o filtro explicitamente neste caminho:

```
get( "/filter", ( req, resp, chain ) -> {
    // ...
    chain.next( req, resp );
});
```

A sintaxe é muito semelhante ao filtro Servlet. É possível restringir a solicitação e enviar de volta a resposta no próprio filtro chamando o método Response.send (Result result).

Assim que o filtro for implementado, precisamos implementar o manipulador de solicitação:

```
get("/filter", (req, resp) -> {
    resp.send("filter response");
});
```

### 4.7. Sessão
Jooby vem com dois tipos de implementação de sessão; na memória e com base em cookies.

Implementar o gerenciamento de sessão na memória é bastante simples. Temos as opções de escolher qualquer um dos armazenamentos de sessão de alto rendimento disponíveis com Jooby, como EhCache, Guava, HazleCast, Cassandra, Couchbase, Redis, MongoDB e Memcached.

Por exemplo, para implementar um armazenamento de sessão baseado em Redis, precisamos adicionar a seguinte dependência Maven:

```
<dependency>
    <groupId>org.jooby</groupId>
    <artifactId>jooby-jedis</artifactId>
    <version>1.1.3</version>
</dependency>
```

Agora podemos usar o snippet de código a seguir para habilitar o gerenciamento de sessão:

```
{
    use(new Redis());
    session(RedisSessionStore.class);

    get( "/session" , req -> {
        Session session = req.session();
        session.set("token", "value");
        return session.get("token").value();
    });
}
```

O ponto a ser observado aqui é que podemos configurar o url do Redis como a propriedade 'db' no application.conf.

Para habilitar o gerenciamento de sessão baseado em cookie, precisamos declarar 
cookieSession(). Se a abordagem baseada em cookie for selecionada, devemos declarar a propriedade application.secret no arquivo application.conf. Uma vez que cada cookie que será assinado será assinado com esta chave secreta, é sempre aconselhável usar um fragmento de string aleatório longo como chave secreta.

Tanto na abordagem na memória quanto na baseada em cookies, devemos declarar o parâmetro de configuração necessário no arquivo application.conf, caso contrário, o aplicativo lançará uma IllegalStateException na inicialização.

# 5. Teste
Testar a rota MVC é realmente fácil, pois uma rota está vinculada a uma estratégia para alguma classe. Isso facilita a execução de testes de unidade em todas as rotas.

Por exemplo, podemos criar rapidamente um caso de teste para o URL padrão:

```
public class AppTest {
 
    @ClassRule
    public static JoobyRule app = new JoobyRule(new App());

    @Test
    public void given_defaultUrl_expect_fixedString() {
 
        get("/").then().assertThat().body(equalTo("Hello World!"))
          .statusCode(200).contentType("text/html;charset=UTF-8");
    }
}
```

O ponto a ser observado aqui é que usar a anotação @ClassRule criará apenas uma instância do servidor para todos os casos de teste. Se precisarmos construir instâncias separadas dos servidores para cada caso de teste, temos que usar a anotação @Rule sem o modificador estático.

Também podemos usar o MockRouter do Jooby para testar o caminho da mesma maneira:

```
@Test
public void given_defaultUrl_with_mockrouter_expect_fixedString() 
  throws Throwable {
 
    String result = new MockRouter(new App()).get("/");
 
    assertEquals("Hello World!", result);
}
```

# 6. Conclusão
Neste tutorial, exploramos o projeto Jooby e sua funcionalidade essencial.