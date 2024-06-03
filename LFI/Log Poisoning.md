# Log Poisoning

No contexto da segurança de aplicações web, o log poisoning envolve a escrita de código malicioso em arquivos de log e, posteriormente, a inclusão desses arquivos de log vulneráveis na aplicação para executar o código malicioso. Esse ataque é *possível quando uma aplicação PHP tem permissões de leitura e execução em arquivos de log*.

Funções com `Execute` privilégios deverão estar vulneráveis a esses ataques:

| **Função**               | **Leia o conteúdo** | **Executar** | **URL remoto** |
|--------------------------|:-------------------:|:------------:|:--------------:|
| **PHP**                  |                     |              |                |
| `include()`/`include_once()` |         ✅         |      ✅       |       ✅        |
| `require()`/`require_once()` |         ✅         |      ✅       |       ❌        |
| **NodeJS**               |                     |              |                |
| `res.render()`           |         ✅           |      ✅       |       ❌        |
| **Java**                 |                     |              |                |
| `import`                 |         ✅           |      ✅       |       ✅        |
| **.NET**                 |                     |              |                |
| `include`                |         ✅           |      ✅       |       ✅        |

## Conceito de Log Poisoning

### Introdução ao Log Poisoning
   - O ataque envolve registrar código PHP malicioso em um campo controlado pelo usuário.
   - Esse código é então executado ao ser incluído por uma função PHP vulnerável, como `include()`, `require()`, `require_once()`, etc.
   - Para o ataque funcionar, a aplicação deve ter privilégios de leitura nos arquivos de log.

### Exemplo Prático
   - Vamos assumir que temos uma vulnerabilidade de local file inclusion (LFI) em uma aplicação.
   - Podemos envenenar um arquivo de log HTTP com código PHP malicioso ao definir um cabeçalho `User-Agent` personalizado.

## PHP Session Poisoning

### Exame do Arquivo de Sessão
   - A maioria das aplicações PHP utiliza cookies `PHPSESSID` para armazenar dados específicos do usuário.
   - Esses dados são salvos em arquivos de sessão, localizados em `/var/lib/php/sessions/` no Linux.
   - O nome do arquivo de sessão corresponde ao valor do cookie `PHPSESSID` com o prefixo `sess_`.
   - Por exemplo, se o cookie `PHPSESSID` estiver definido como `el4ukv0kqbv0irg6nkp4dncpk4`, então sua localização no disco será `/var/lib/php/sessions/sess_el4ukv0kqbv0irg6nkp4`.
   - A primeira coisa que precisamos fazer é examinar nosso arquivo de sessão PHPSESSID e ver se ele contém algum dado que possamos controlar e envenenar.
   - Se o valor do nosso cookie for `el4ukv0kqbv0irg6nkp4dncpk4`, então deve ser armazenado em `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd`.

### Manipulação do Arquivo de Sessão
   - Verificar o valor do cookie `PHPSESSID`.
   - Incluir o arquivo de sessão através da vulnerabilidade LFI para visualizar o conteúdo.
   - Modificar um valor controlado pelo usuário (por exemplo, `page`) no arquivo de sessão.
   - Escrever um web shell PHP no arquivo de sessão para execução de código remoto.

### Passo a Passo do PHP Session Poisoning
   1. Definir o valor do `PHPSESSID`.
   2. Acessar a URL vulnerável para incluir o arquivo de sessão e ver o conteúdo.
   3. Modificar o valor do parâmetro para um código PHP malicioso.
   4. Incluir novamente o arquivo de sessão e executar comandos.

### Nota
- Para executar outro comando, o arquivo da sessão deve ser envenenado novamente com o web shell, pois ele é sobrescrito em `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd` após nossa última inclusão. Idealmente, seria o web shell envenenado para gravar um web shell permanente no diretório da web ou enviaríamos um shell reverso para facilitar a interação.

## Server Log Poisoning

### Manipulação de Arquivos de Log
   - Os servidores web, como Apache e Nginx, mantêm logs que registram várias informações sobre as solicitações.
   - Logs do Apache geralmente estão em `/var/log/apache2/access.log` e `error.log` e do Nginx em `/var/log/nginx/access.log` e `error.log`.
   - Podemos controlar o cabeçalho `User-Agent` para injetar código PHP nos logs.
   - Uma vez envenenado, precisamos incluir os logs através da vulnerabilidade LFI e, para isso, precisamos ter acesso de leitura aos logs. Nginx os logs são legíveis por usuários com poucos privilégios por padrão (por exemplo, www-data), enquanto os logs do Apache só podem ser lidos por usuários com altos privilégios (por exemplo, root / adm groups). Entretanto, em servidores mais antigos ou mal configurados Apache, esses logs podem ser lidos por usuários com poucos privilégios.

### Passo a Passo do Server Log Poisoning
   1. Interceptar a solicitação HTTP e modificar o cabeçalho `User-Agent`.
   2. Incluir o log através da vulnerabilidade LFI para verificar o código injetado.
   3. Enviar uma solicitação com código PHP malicioso no cabeçalho `User-Agent`.
   4. Executar comandos ao incluir  o arquivo de log injetado.   

**Nota:** Como todas as solicitações ao servidor são registradas, pos envenenar qualquer solicitação à aplicação web, e não necessariamente a LFI.

### Outras Técnicas
**Dica:** `User-Agent` cabeçalho também é mostrado nos arquivos de processo no `/proc/` diretório Linux. Portanto, podemos tentar incluir os arquivos `/proc/self/environ` ou `/proc/self/fd/N`(onde N é um PID geralmente entre 0-50) e poderemos realizar o mesmo ataque nesses arquivos. Isso pode ser útil caso não tenhamos acesso de leitura aos logs do servidor; no entanto, esses arquivos também podem ser lidos por usuários privilegiados.

**Logs de Outros Serviços**:
   - Além de logs de servidores web, outros serviços também mantêm logs que podem ser envenenados, como:
   - `/var/log/sshd.log` para serviços SSH.
   - `/var/log/mail` para serviços de e-mail.
   - `/var/log/vsftpd.log` para FTP.
   - Envenenar esses logs e incluir através de LFI pode resultar na execução do código PHP.

**Exemplo de Ataque via User-Agent**:
   - Usar ferramentas como Burp Suite ou `curl` para modificar o `User-Agent`.
   - Injetar um web shell PHP básico no log:
     ```bash
     curl -s "http://test.com/index.php" -A "<?php system($_GET['cmd']); ?>"
     ```
   - Incluir o log através da LFI para executar comandos como `?cmd=id`.

### Observações
Devemos primeiro tentar ler esses logs através do LFI e, se tivermos acesso a eles, podemos tentar envenená-los com formas mencionadas logo acima. **Por exemplo, se os serviços `ssh` ou `ftp` forem expostos a nós e pudermos ler seus logs por meio de LFI, podemos tentar fazer login neles e definir o nome de usuário para o código PHP e, ao incluir seus logs, o código PHP será executado.** O mesmo se aplica aos `mail` serviços, pois podemos enviar um email contendo o código PHP, e ao incluir o log, o código PHP será executado. `Podemos generalizar essa técnica para quaisquer logs que registrem um parâmetro que controlamos e que possamos ler através da vulnerabilidade LFI.`

### Conclusão

O envenenamento de logs e sessões são técnicas avançadas de ataque que exploram vulnerabilidades de inclusão de arquivos locais (LFI). Com essas técnicas, um atacante pode injetar código PHP malicioso em arquivos de log ou sessão e executá-lo para comprometer a segurança do servidor. É crucial que administradores de sistemas e desenvolvedores implementem práticas de segurança robustas para mitigar essas vulnerabilidades, como a limitação de permissões de leitura e execução, sanitização de entradas e configuração adequada dos servidores.
