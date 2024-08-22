## Tipos de Dados

O C difere as variáveis através da tipagem que o programador informou, então se
definirmos uma variável como `int` e atribuirmos o valor **10**, significa que
a representação desse valor a nível de máquina fará sentido em algum uso futuro
dessa informações, o nome dado a esse tipo de tratamento é "tipagem estática",
todos os dados que serão informados pelo usuário devem ser informados previamente
ao compilador para que o mesmo crie uma região de memória grande o suficiente para
o armazenamento desses dados.

Um código (00\_data\_types.c) simples, que pode ser destrinchado e melhor compreendido
com as ferramentas do GNU/Linux:

```c
#include <stdio.h>

int main(void) {
	int foo = 10;

	printf("%d\n", foo);

	return 0;
}
```

O que esse código faz é basicamente armazenar um valor em uma variável chamada `foo` e
depois imprime na tela com o auxílio da função `printf`.

Para compilar esse código basta utilizar o `gcc` e alguns de seus parâmetros para facilitar
a análise do programa gerado:

```bash
gcc -Wall -Wextra -ggdb 00_data_types.c -o bin
```

Em ordem, o primeiro e segundo parâmetro do GCC serve para habilitar alguns avisos caso haja
algum problema na codificação, então somos avisados de antemão, o terceiro parâmetro habilita
as informações de _debugging_ e aqui iremos utilizar o GDB (GNU Debugger) para fazer a
análise desse código, supostamente bobo. Por último, o parâmetro `-o` recebe como argumento
o nome do binário que será gerado, em nosso caso o nome é "bin".

Agora que o binário foi gerado e está pronto para a análise, vamos utilizar o gdb:

```bash
gdb ./bin
```

Caso tudo ocorra bem, no momento você estará vendo o shell do gdb, há muitos comandos mas vamos
nos atentar em apenas algumas unidades de comandos :).

Para que você não se assuste tanto com o primeiro comando que faremos para ver como o código
foi gerado pelo compilador, ative o conjunto de instrução (_instruction set_) utilizada pela
família Intel x86: `set disassembly-flavor intel`.
Como o nosso programa tem apenas uma função, que é a função principal e obrigatória de todo
código em C, vamos "disassemblar" a mesma para que vejamos o código em assembly, isso pode
ser feito através do comando `disas main`.

O log gerado pelo disassembly será esse:

```gdb
Dump of assembler code for function main:
	0x0000000000001149 <+0>:     endbr64 
	0x000000000000114d <+4>:     push   rbp
	0x000000000000114e <+5>:     mov    rbp,rsp
	0x0000000000001151 <+8>:     sub    rsp,0x10
	0x0000000000001155 <+12>:    mov    DWORD PTR [rbp-0x4],0xa
	0x000000000000115c <+19>:    mov    eax,DWORD PTR [rbp-0x4]
	0x000000000000115f <+22>:    mov    esi,eax
	0x0000000000001161 <+24>:    lea    rax,[rip+0xe9c]        # 0x2004
	0x0000000000001168 <+31>:    mov    rdi,rax
	0x000000000000116b <+34>:    mov    eax,0x0
	0x0000000000001170 <+39>:    call   0x1050 <printf@plt>
	0x0000000000001175 <+44>:    mov    eax,0x0
	0x000000000000117a <+49>:    leave  
	0x000000000000117b <+50>:    ret    
End of assembler dump.
```

Sim, uma sopa de letrinhas, esse é parte do código em assembly gerado pelo compilador para
que nosso código em C seja executado. O que faremos com essa informação? Bom, tente
identificar onde está a nossa variável `foo`.

Como o foco desse artigo não é explicar sobre assembly e sim sobre C, seguiremos a análise do
nosso binário através do gdb e a dúvida sobre onde está a nossa variável nesse código vai
ficar te incomodando até que você procure sobre (ou não).

Todo _debugger_ necessita de pontos de parada, sem eles não faria sentido a análise. Se
executarmos o nosso código sem ponto de parada não há tempo para fazer uma "validação
minuciosa" dos dados, então aqui vai o segundo comando, o `break`, marque um _breakpoint_
na função main assim: `break main`.
Agora podemos "rodar" o código através do comando `run`, para encontrar a nossa variável
basta executar o comando `info locals` e nesse momento pós-início do programa, não haverá
nada em `foo`, ou melhor haverá o valor 0.
Isso significa que estamos na etapa anterior a atribuição do valor 10 e para continuar o
programa andaremos de passo em passo com o comando `step`, executando o `info locals`
novamente é possível ver que a variável possui o valor desejado.
Como foi dito inicialmente, o compilador precisa saber o tipo da variável para habilitar uma
região de memória que suporte o tipo do dado que será informado, logo toda variável possui
um endereço e para ver esse endereço pelo gdb basta usar o operador "&", que assim como no
C serve para pegar o endereço de uma variável, para ver o valor, execute o comando
`print &foo`, o endereço que foi mostrado aqui (o valor pode mudar) é o `0x7fffffffde9c`.

No GNU/Linux temos a filosofia de que "tudo é arquivo", se tudo é arquivo os dados desse
programa devem estar em algum lugar e mapeados de alguma forma, na árvore de estrutura do
Linux temos a pasta '/proc', ela serve para mapear as informações (estrutura de dados) para
o kernel operante, então em cada um subdiretório temos informações do processo, basta termos
o PID desse processo para verificar as informações deles, no GNU/Linux temos o comando
`pidof`, que obtém o PID do programa que é passado como parâmetro, em nosso caso geramos o
binário "bin" através do GCC, então ao executarmos o comando `pidof bin` obteremos o número
do PID, no momento obtive o valor **21072** e com um pouco de Bash no auxílio pegaremos o mapa
de endereços do nosso programa com o comando

```bash
$ more /proc/$(pidof bin)/maps

555555554000-555555555000 r--p 00000000 08:02 152731                     /home/sic/DevThings/sic_2024/misc/notes/src/bin
555555555000-555555556000 r-xp 00001000 08:02 152731                     /home/sic/DevThings/sic_2024/misc/notes/src/bin
555555556000-555555557000 r--p 00002000 08:02 152731                     /home/sic/DevThings/sic_2024/misc/notes/src/bin
555555557000-555555558000 r--p 00002000 08:02 152731                     /home/sic/DevThings/sic_2024/misc/notes/src/bin
555555558000-555555559000 rw-p 00003000 08:02 152731                     /home/sic/DevThings/sic_2024/misc/notes/src/bin
555555559000-55555557a000 rw-p 00000000 00:00 0                          [heap]
7ffff7d7a000-7ffff7d7d000 rw-p 00000000 00:00 0 
7ffff7d7d000-7ffff7da5000 r--p 00000000 08:02 4591173                    /usr/lib/x86_64-linux-gnu/libc.so.6
7ffff7da5000-7ffff7f3a000 r-xp 00028000 08:02 4591173                    /usr/lib/x86_64-linux-gnu/libc.so.6
7ffff7f3a000-7ffff7f92000 r--p 001bd000 08:02 4591173                    /usr/lib/x86_64-linux-gnu/libc.so.6
7ffff7f92000-7ffff7f93000 ---p 00215000 08:02 4591173                    /usr/lib/x86_64-linux-gnu/libc.so.6
7ffff7f93000-7ffff7f97000 r--p 00215000 08:02 4591173                    /usr/lib/x86_64-linux-gnu/libc.so.6
7ffff7f97000-7ffff7f99000 rw-p 00219000 08:02 4591173                    /usr/lib/x86_64-linux-gnu/libc.so.6
7ffff7f99000-7ffff7fa6000 rw-p 00000000 00:00 0 
7ffff7fbb000-7ffff7fbd000 rw-p 00000000 00:00 0 
7ffff7fbd000-7ffff7fc1000 r--p 00000000 00:00 0                          [vvar]
7ffff7fc1000-7ffff7fc3000 r-xp 00000000 00:00 0                          [vdso]
7ffff7fc3000-7ffff7fc5000 r--p 00000000 08:02 4591151                    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7ffff7fc5000-7ffff7fef000 r-xp 00002000 08:02 4591151                    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7ffff7fef000-7ffff7ffa000 r--p 0002c000 08:02 4591151                    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7ffff7ffb000-7ffff7ffd000 r--p 00037000 08:02 4591151                    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7ffff7ffd000-7ffff7fff000 rw-p 00039000 08:02 4591151                    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7ffffffde000-7ffffffff000 rw-p 00000000 00:00 0                          [stack]
ffffffffff600000-ffffffffff601000 --xp 00000000 00:00 0                  [vsyscall]
```
