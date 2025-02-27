
# Introdução às Inclusões de Arquivos

As inclusões de arquivos locais (LFI) representam uma vulnerabilidade comum em muitas linguagens de programação, incluindo PHP, Node.js, Java e .NET. Essa vulnerabilidade ocorre quando um aplicativo da web permite que os usuários manipulem parâmetros de entrada para incluir conteúdo de arquivos locais no servidor, potencialmente expondo dados sensíveis e permitindo a execução de código malicioso.

## PHP
Em PHP, a função `include()` é comumente usada para carregar arquivos locais ou remotos durante o processamento de uma página da web. Se um parâmetro controlado pelo usuário, como um parâmetro GET, for passado diretamente para a função `include()` sem filtragem adequada, isso pode levar a uma vulnerabilidade de LFI. Por exemplo:

```php
if (isset($_GET['language'])) {
    include($_GET['language']);
}
```

Neste exemplo, se o parâmetro GET `'language'` não for validado corretamente, um invasor pode manipulá-lo para incluir arquivos arbitrários no servidor.

## Node.js
Em Node.js, assim como em PHP, os desenvolvedores podem encontrar vulnerabilidades de LFI se os parâmetros de entrada não forem devidamente validados. Por exemplo, se um parâmetro GET for usado para determinar qual arquivo ler e exibir, sem a devida verificação, isso pode levar a uma vulnerabilidade. Um exemplo simplificado pode ser:

```javascript
if(req.query.language) {
    fs.readFile(path.join(__dirname, req.query.language), function (err, data) {
        res.write(data);
    });
}
```

Se o parâmetro `language` não for validado adequadamente, um invasor pode manipulá-lo para acessar arquivos fora do diretório esperado.

## Java
Em aplicativos Java baseados em servidor, a inclusão de arquivos locais também pode ocorrer se os parâmetros de entrada não forem verificados corretamente. Por exemplo, em páginas JSP (JavaServer Pages), o seguinte código pode ser vulnerável:

```jsp
<c:if test="${not empty param.language}">
    <jsp:include file="<%= request.getParameter('language') %>" />
</c:if>
```

Se o parâmetro `language` não for validado de maneira adequada, um invasor pode explorar essa vulnerabilidade para incluir arquivos arbitrários no servidor.

## .NET
No ambiente .NET, a vulnerabilidade de LFI pode ocorrer se os parâmetros de entrada não forem verificados adequadamente antes de serem usados para incluir arquivos. Por exemplo:

```csharp
@if (!string.IsNullOrEmpty(HttpContext.Request.Query['language'])) {
    <% Response.WriteFile("<% HttpContext.Request.Query['language'] %>"); %> 
}
```

Se o parâmetro GET `'language'` não for validado corretamente, um invasor pode manipulá-lo para incluir arquivos arbitrários no servidor.

Em resumo, em todas essas linguagens de programação, a inclusão de arquivos locais pode levar a vulnerabilidades de segurança se os parâmetros de entrada não forem validados corretamente antes de serem usados para incluir arquivos no servidor. É crucial que os desenvolvedores implementem medidas de segurança, como filtragem e validação de entrada, para mitigar essas vulnerabilidades.


# A inclusão de arquivos locais (LFI) 

A inclusão de arquivos locais (LFI) é uma vulnerabilidade comum em aplicações web que permite que um invasor leia arquivos no servidor remoto. Aqui está um resumo dos conceitos discutidos juntamente com exemplos de código:

1. LFI básico:
   - Exemplo de URL: `http://teste.com.br/index.php?language=/etc/passwd`
   - Exploração: Alterar o parâmetro `language` para o caminho absoluto do arquivo desejado.
   - Exemplo de código: `include($_GET['language']);`

2. Travessia de diretório:
   - Exemplo de URL: `http://teste.com.br//index.php?language=../../../../etc/passwd`
   - Exploração: Usar `../` para percorrer diretórios e alcançar o arquivo desejado.
   - Exemplo de código: `include("./languages/" . $_GET['language']);`

3. Prefixo do nome do arquivo:
   - Exemplo de URL: `http://teste.com.br/index.php?language=/../../../etc/passwd`
   - Exploração: Adicionar `/` antes da carga útil para considerar o prefixo como um diretório.
   - Exemplo de código: `include("lang_" . $_GET['language']);`

4. Extensões anexadas:
   - Exemplo de URL: `http://teste.com.br/extension/index.php?language=/etc/passwd`
   - Exploração: Bypass adicionando uma extensão válida ao caminho do arquivo.
   - Exemplo de código: `include($_GET['language'] . ".php");`

5. Ataques de segunda ordem:
   - Exploração: Inserir uma carga útil LFI em uma entrada de banco de dados que será usada posteriormente por outra funcionalidade para extrair um arquivo.
   - Exemplo de código: Depende do contexto específico da aplicação.

6. Data:
   - Exemplo de URL: `http://teste.com.br/index.php?language=data:text/plant,123`
