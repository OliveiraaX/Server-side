## O que é a Travessia de Diretório (Path Traversal)

A travessia de diretório ocorre quando alguém consegue acessar arquivos ou diretórios em um sistema de arquivos que normalmente não estariam disponíveis, pulando os caminhos ou restrições de acesso adequados. Em alguns casos, um invasor pode até mesmo gravar em arquivos arbitrários no servidor, permitindo-lhe modificar os dados ou o comportamento do aplicativo e, por fim, assumir o controle total do servidor.

Considere o seguinte exemplo, onde um aplicativo de compras exibe imagens dos itens à venda:

https://www.exemplo.com/item/getImage?filename=item.jpg

Nesta URL, o parâmetro `filename` é usado para especificar o nome do arquivo e o aplicativo retorna o conteúdo desse arquivo. Os arquivos de imagem são armazenados em disco no local `/var/www/images/`. Para retornar uma imagem, o aplicativo anexa o nome do arquivo solicitado a esse diretório base e usa uma API do sistema de arquivos para ler o conteúdo do arquivo. Em outras palavras, o aplicativo lê o seguinte caminho de arquivo: `/var/www/images/218.png`.

No entanto, se este aplicativo não implementar defesas contra ataques de travessia de diretório, um invasor pode explorar isso. Por exemplo, o invasor pode solicitar a seguinte URL para tentar recuperar o arquivo `/etc/passwd` do sistema de arquivos do servidor:

https://insecure-website.com/loadImage?filename=../../../etc/passwd

Isso faria com que o aplicativo lesse o seguinte caminho de arquivo: `/var/www/images/../../../etc/passwd`. A sequência `../` é válida em um caminho de arquivo e significa avançar um nível na estrutura de diretório. Assim, as três sequências consecutivas de `../` avançam da raiz do sistema de arquivos, permitindo ao invasor acessar o arquivo `/etc/passwd`.

No Windows, `../` e `..\` são sequências de travessia de diretório válidas, e um ataque equivalente para recuperar um arquivo de sistema operacional padrão seria:

https://www.exemplo.com/item/getImage?filename=..\..\..\windows\win.ini

### Detectando e contornando os obstáculos a este tipo de ataque

Para cada parâmetro fornecido pelo usuário sendo testado, determine se as sequências de travessia estão sendo bloqueadas pelo aplicativo ou se funcionam conforme o esperado. Um teste inicial que geralmente é confiável é enviar sequências de travessia de uma forma que não envolva voltar atrás do diretório inicial. Se for encontrada qualquer instância em que o envio de sequências de travessia sem passar acima do diretório inicial não afete o comportamento do aplicativo, o próximo teste é tentar atravessar o diretório inicial e acessar arquivos de outro lugar no servidor.

Representações simples codificadas por URL de sequências de travessia usando as seguintes codificações:

- Transformação Unicode em 16-bit:
    - Ponto — `%u002e`
    - Barra — `%u2215`
    - Barra invertida — `%u2216`

- Codificação dupla de URL:
    - Ponto — `%252e`
    - Barra — `%252f`
    - Barra invertida — `%255c`

- Transformação longa Unicode UTF-8:
    - Ponto — `%c0%2e`, `%e0%40%ae`, `%c0ae`, e assim por diante
    - Barra — `%c0%af`, `%e0%80%af`, `%c0%2f`, e assim por diante
    - Barra invertida — `%c0%5c`, `%c0%80%5c`, e assim por diante

Se o aplicativo está tentando limpar a entrada do usuário removendo sequências de travessia e não aplica este filtro recursivamente, pode ser possível contornar o filtro colocando uma sequência dentro de outra. Por exemplo:
	-  .....//
	- ..../
	- ..../
	- ....\


Tente colocar um byte nulo codificado por URL no final do nome do arquivo solicitado, seguido por um tipo de arquivo que o aplicativo aceita. Por exemplo:
	- ../../../etc/passwd%00.jpg


