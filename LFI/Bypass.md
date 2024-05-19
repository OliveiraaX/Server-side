## Bypasses Básicos

O filtro de passagem de caminho não recursivo é uma técnica comum para mitigar vulnerabilidades de inclusão de arquivo local (LFI) em aplicativos da web. No entanto, esse tipo de filtro pode ser facilmente contornado se não for implementado corretamente.

### Exemplo de Filtro Básico em PHP:

```php
$language = str_replace('../', '', $_GET['language']);
```

Neste exemplo, o filtro simplesmente remove todas as ocorrências de "../" da
entrada do usuário, impedindo a navegação para cima no sistema de arquivos. No
entanto, esse filtro é inadequado, pois não é recursivo e pode ser facilmente
contornado.

Desvio com "../":

```php
/index.php?language=....//....//....//etc/passwd
```
A presença de múltiplas ocorrências de "../" não é tratada pelo filtro, permitindo a navegação para cima no sistema de arquivos.

Codificação de URL:
Alguns filtros podem ser contornados usando codificação de URL. Por exemplo, a entrada "../" pode ser codificada como "%2e%2e%2f", evitando a detecção pelo filtro.

Exemplo com Codificação de URL:
```php
/index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc/passwd
```
A codificação de URL pode ignorar filtros que tentam detectar padrões específicos na entrada.

Caminhos Aprovados:
Alguns aplicativos da web só aceitam arquivos de um diretório específico. No entanto, esse filtro pode ser contornado iniciando a entrada com o caminho aprovado e navegando para cima no sistema de arquivos.

Exemplo de Caminho Aprovado:
```php
/index.php?language=./languages/../../../../etc/passwd
```
Iniciando com o caminho aprovado, podemos navegar para cima no sistema de arquivos e acessar arquivos fora do diretório esperado.

Truncamento de Caminho:
Em versões antigas do PHP, strings tinham um comprimento máximo e eram truncadas se ultrapassassem esse limite. Esse comportamento pode ser explorado para contornar filtros de caminho.

Exemplo de Truncamento de Caminho:
```php
/index.php?language=non_existing_directory/../../../etc/passwd/./././[...]
```
Explorando o truncamento de caminho, podemos acessar arquivos fora do diretório esperado.

Bytes Nulos:
Versões antigas do PHP eram vulneráveis ​​a injeção de byte nulo, permitindo que a entrada fosse truncada no byte nulo.

Exemplo com Byte Nulo:
```php
/index.php?language=/etc/passwd%00
```
Inserindo um byte nulo, podemos evitar a detecção de filtros e acessar arquivos fora do diretório esperado.

Essas técnicas demonstram como filtros de passagem de caminho podem ser contornados, destacando a importância de implementar medidas de segurança mais robustas para mitigar vulnerabilidades de LFI.