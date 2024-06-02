## Log Poisoning

No contexto da segurança de aplicações web, o log poisoning envolve a escrita de código malicioso em arquivos de log e, posteriormente, a inclusão desses arquivos de log vulneráveis na aplicação para executar o código malicioso. Esse ataque é *possível quando uma aplicação PHP tem permissões de leitura e execução em arquivos de log*. 

### Conceito de Log Poisoning

1. **Introdução ao Log Poisoning**:
   - O ataque envolve registrar código PHP malicioso em um campo controlado pelo usuário.
   - Esse código é então executado ao ser incluído por uma função PHP vulnerável, como `include()`, `require()`, `require_once()`, etc...
   - Para o ataque funcionar, a aplicação deve ter privilégios de leitura nos arquivos de log.

2. **Exemplo Prático**:
   - Vamos assumir que temos uma vulnerabilidade de local file inclusion (LFI) em uma aplicação.
   - Podemos envenenar um arquivo de log HTTP com código PHP malicioso ao definir um cabeçalho `User-Agent` personalizado.

### PHP Session Poisoning

1. **Exame do Arquivo de Sessão**:
   - A maioria das aplicações PHP utiliza cookies `PHPSESSID` para armazenar dados específicos do usuário.
   - Esses dados são salvos em arquivos de sessão, localizados em `/var/lib/php/sessions/` no Linux.
   - O nome do arquivo de sessão corresponde ao valor do cookie `PHPSESSID` com o prefixo `sess_`.

2. **Manipulação do Arquivo de Sessão**:
   - Verificar o valor do cookie `PHPSESSID`.
   - Incluir o arquivo de sessão através da vulnerabilidade LFI para visualizar o conteúdo.
   - Modificar um valor controlado pelo usuário (por exemplo, `page`) no arquivo de sessão.
   - Escrever um web shell PHP no arquivo de sessão para execução de código remoto.

3. **Passo a Passo do PHP Session Poisoning**:
   - Definir o valor do `PHPSESSID`.
   - Acessar a URL vulnerável para incluir o arquivo de sessão e ver o conteúdo.
   - Modificar o valor do parâmetro `language` para um código PHP malicioso.
   - Incluir novamente o arquivo de sessão e executar comandos.

### Server Log Poisoning

1. **Manipulação de Arquivos de Log**:
   - Os servidores web, como Apache e Nginx, mantêm logs que registram várias informações sobre as solicitações.
   - Logs do Apache geralmente estão em `/var/log/apache2/` e do Nginx em `/var/log/nginx/`.
   - Podemos controlar o cabeçalho `User-Agent` para injetar código PHP nos logs.

2. **Passo a Passo do Server Log Poisoning**:
   - Interceptar a solicitação HTTP e modificar o cabeçalho `User-Agent`.
   - Incluir o log através da vulnerabilidade LFI para verificar o código injetado.
   - Enviar uma solicitação com código PHP malicioso no cabeçalho `User-Agent`.
   - Executar comandos ao incluir o arquivo de log injetado.

### Técnicas Avançadas

1. **Logs de Outros Serviços**:
   - Além de logs de servidores web, outros serviços também mantêm logs que podem ser envenenados, como:
     - `/var/log/sshd.log` para serviços SSH.
     - `/var/log/mail` para serviços de e-mail.
     - `/var/log/vsftpd.log` para FTP.
   - Envenenar esses logs e incluir através de LFI pode resultar na execução do código PHP.

2. **Exemplo de Ataque via User-Agent**:
   - Usar ferramentas como Burp Suite ou `curl` para modificar o `User-Agent`.
   - Injetar um web shell PHP básico no log:
     ```bash
     curl -s "http://test.com/index.php" -A "<?php system($_GET['cmd']); ?>"
     ```
   - Incluir o log através da LFI para executar comandos como `?cmd=id`.

### Conclusão

O envenenamento de logs e sessões são técnicas avançadas de ataque que exploram vulnerabilidades de inclusão de arquivos locais (LFI). Com essas técnicas, um atacante pode injetar código PHP malicioso em arquivos de log ou sessão e executá-lo para comprometer a segurança do servidor. É crucial que administradores de sistemas e desenvolvedores implementem práticas de segurança robustas para mitigar essas vulnerabilidades, como a limitação de permissões de leitura e execução, sanitização de entradas e configuração adequada dos servidores.
