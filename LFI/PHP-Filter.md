# Filtros PHP e Exploração de Vulnerabilidades LFI

Aplicativos web PHP, incluindo aqueles construídos com frameworks como Laravel ou Symfony, frequentemente contêm vulnerabilidades de Inclusão de Arquivo Local (LFI). A exploração dessas vulnerabilidades pode ser estendida usando Wrappers PHP, que permitem acessar diferentes fluxos de entrada/saída (E/S) no nível do aplicativo, como entrada/saída padrão e fluxos de memória. Isso possibilita ler arquivos de código-fonte PHP ou até mesmo executar comandos do sistema.

## Wrappers e Filtros PHP

Filtros PHP são um tipo de wrapper que permite passar e filtrar diferentes tipos de entrada. Usamos o esquema `php://` para acessar esses wrappers e o `php://filter/` para os filtros específicos. Os principais parâmetros utilizados são `resource` e `read`:

- **resource**: Especifica o fluxo no qual aplicar o filtro (por exemplo, um arquivo local).
- **read**: Aplica diferentes filtros no recurso de entrada.

## Tipos de Filtros

Existem quatro tipos principais de filtros:

1. **Filtros de String**
2. **Filtros de Conversão**
3. **Filtros de Compressão**
4. **Filtros de Criptografia**

Para ataques LFI, o filtro de conversão `convert.base64-encode` é especialmente útil, pois permite codificar o conteúdo do arquivo em base64, facilitando a leitura do código-fonte PHP.

## Fuzzing para Arquivos PHP

Para identificar páginas PHP disponíveis, utilizamos ferramentas como `ffuf` ou `gobuster`. Mesmo arquivos com códigos de resposta HTTP como `301`, `302`, e `403` são relevantes, pois podem ser acessados via LFI e inspecionados para revelar mais arquivos PHP referenciados.

## Inclusão Padrão de Arquivos PHP

Ao incluir arquivos PHP via LFI, o código PHP é normalmente executado e renderizado como HTML. No entanto, ao usar o filtro base64, podemos codificar o conteúdo PHP em base64, permitindo a leitura do código-fonte. Por exemplo:
```
http://teste.com/index.php?language=php://filter/read=convert.base64-encode/resource=config
```

Divulgação do Código-Fonte
Após identificar arquivos PHP de interesse, podemos usar o filtro base64 para ler o código-fonte:
```
php://filter/read=convert.base64-encode/resource=config
```
Decodificamos a string base64 resultante para obter o conteúdo do arquivo:
```
echo 'PD9waHAK...SNIP...KICB9Ciov' | base64 -d
```
### Dica
Ao copiar strings codificadas em base64, certifique-se de copiar a string inteira para garantir uma decodificação completa. Verificar o código-fonte da página pode ajudar a garantir que a string completa foi copiada.

### Resumo
Explorar vulnerabilidades LFI em aplicativos PHP pode ser significativamente potenciado pelo uso de filtros e wrappers PHP. Usar o filtro base64 permite a leitura do código-fonte PHP, fornecendo informações críticas sobre o funcionamento interno do aplicativo, que podem ser usadas para etapas subsequentes de exploração, como a execução remota de código.