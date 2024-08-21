## Tipos de Dados

O C difere as variáveis através da tipagem que o programador informou, então se
definirmos uma variável como `int` e atribuirmos o valor **10**, significa que
a representação desse valor a nível de máquina fará sentido em algum uso futuro
dessa informações, o nome dado a esse tipo de tratamento é "tipagem estática",
todos os dados que serão informados pelo usuário devem ser informados previamente
ao compilador para que o mesmo crie uma região de memória grande o suficiente para
o armazenamento desses dados.

Um código (00\_data\_types.c) simples que pode ser destrinchado e melhor compreendido
com as ferramentas do GNU/Linux:

```c
#include <stdio.h>

int main(void) {
	int foo = 10;

	printf("%d\n", foo);

	return 0;
}
```

Oque esse código faz é basicamente armazenar um valor em uma variável chamada `foo` e
depois imprime na tela com o auxílio da função `printf`.

Para compilar esse código basta utilizar o `gcc` e alguns de seus parâmetros para facilitar
a análise do programa gerado:

```bash
gcc -Wall -Wextra -ggdb 00_data_types.c -o bin
```

Em ordem, o primeiro e segundo parâmetro do GCC serve para habilitar alguns avisos caso haja
algum problema na codificação, então somos avisados de antemão, o terceiro parâmetro habilita
as informações de _debugging_ e aqui iremos utilizar o GDB (GNU Debugger) para fazer a
análise desse código supostamente bobo. Por último o parâmetro `-o` recebe como argumento
o nome do binário que será gerado, em nosso caso o nome é "bin".

Agora que o binário foi gerado e está pronto para análise, vamos utilizar o gdb:

```bash
gdb ./bin
```

Caso tudo ocorra bem, no momento você está vendo o shell do gdb, há muitos comandos mas vamos
nos atentar em apenas algumas unidades de comandos :).

Para que você tome menos susto com o primeiro comando que faremos para ver como o código
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
que nosso código em C seja executado. Oque faremos com essa informação? Bom, tente
identificar onde está a nossa variável `foo`.

Como o foco desse artigo não é explicar sobre assembly e sim sobre C, seguiremos a análise do
nosso binário através do gdb e a dúvida sobre onde está a nossa variável nesse código vai
ficar te incomodando até que você procure sobre (ou não).

Todo _debugger_ necessita de pontos de parada, sem eles não faria sentido a análise, se
executarmos o nosso código sem ponto de parada não há tempo para fazer uma "validação
minuciosa" dos dados, então aqui vai o segundo comando, o `break`, marque um _breakpoint_
na função main assim: `break main`.
Agora podemos "rodar" o código através do comando `run`, para encontrar a nossa variável
basta executar o comando `info locals` e nesse momento pós-início do programa não haverá
nada em `foo`, ou melhor haverá o valor 0.
Isso significa que estamos na etapa anterior a atribuição do valor 10 e para continuar o
programa andaremos de passo em passo com o comando `step`, executando o `info locals`
novamente é possível ver que a variável possui o valor desejado.
Como foi dito inicialmente, o compilador precisa saber o tipo da variável para habilitar uma
região de memória que suporte o tipo do dado que será informado, logo toda variável possui
um endereço e para ver esse endereço pelo gdb basta usar o operador "&", que assim como no
C serve para pegar o endereço de uma variável, para ver o valor execute o comando
`print &foo`, o endereço que foi mostrado aqui (o valor pode mudar) é o `0x7fffffffde9c`.
