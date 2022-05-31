# 11911EAU020-ATV1

Descrição do projeto.

Esse projeto consiste na criação e configuração de arquivos necessários para a partida e compilação de um sistema 
**Cortex-M**. Além disso, o programa também é capaz de piscar um LED integrado ao kit de desenvolvimento 
**STM32F411 Blackpill**. 

Para isso, será necessário a criação de 4 arquivos: main.c, Makefile, startup.c e stm32f411-rom.ld. A nomeação fica a
critério de cada desenvolvedor, mantendo apenas o Makefile.

Abaixo o detalhamento de cada um desses arquivos.

# main.c #

Definição do endereço do registrador **RCCC_AHB1ENR** para alterar o estado do *clock* do **GPIOC** (porta onde o LED
será ligado), além da alteração nesse registrador com o valor necessário para habilitar a porta.

Configuração do pino **PC13** para operação do LED, modificando os valores dos registradores do periférico **GPIOC**.
Alteração do estado do pino de saída através do registrador **GPIOx_BSRR - GPIO port bit set/reset register** de forma
**atômica**.

# Makefile #

O utilitário **make** é usado nesse projeto para automatizar o processo de compilação. 

Nesse arquivo encontramos as instruções da compilação dos arquivos do projeto para a geração dos arquivos .o, .elf e
.bin.

Basicamente o arquivo Makefile é formado por regras compostas de *targets*, *prerequisites* e *recipe*, de seguinte
formato:

```
target: prerequisites
  recipe
```

Essa regra informa que se o *target* estiver desatualizado, ou seja, se ele não existe ou se é mais antigo que qualquer
um dos *prerequisites*, a *recipe* será executada.

Nesse projeto utilizamos somente dois comandos:

```
make
```
Executa todas as regras. Note que se o *target* estiver atualizado a regra não será executada.

```
make clean
```
Remove os arquivos .o e .elf do projeto.

# startup.c #

Esse arquivo realiza diversas funções executadas da função **main()**. Sendo elas:

Definições de constantes para o início, tamanho e fim da memória SRAM do processador STM32F411 e uma constante para o
início da **Stack**.

Definição dos vetores de interrupção que se inicia com o endereço do ponteiro para o **Stack Pointer**, seguido por 15
vetores de endereços para as rotinas de tratamento das **System Exceptions**. A implementação desse vetor foi feita
utilizando um array de uint32_t, tipo de variável inteira de 4 bytes.

Definição do protótipo da função **main()**.

Definição do primeiro vetor de interrupção, o **Reset Handler**, atribuído de copiar o conteúdo da seção *.data* da
memória FLASH para a SRAM, inicializar a seção *.bss* em zero e chamar a função **main()**. A rotina **reset_handler()**
implementada para esse vetor de interrupção realiza essas tarefas.

Definição de uma nova seção para armazenar o vetor de interrupção para posteriormente, na **linkedição**, informar a 
região da memória FLASH que esse vetor deve ir.

Definição de mais vetores de interrupção e uma rotina chamada **default_handler()** para esses vetores. Essa rotina é 
atribuída por padrão aos vetores de interrupção não implementados através do atributo **alias**. Além disso, o atributo 
**weak** é utilizado para posteriormente a rotina padrão desses vetores de interrupção serem sobrescritas durante o 
processo de linking por uma função com o mesmo nome sem o atributo **weak**, ou seja, substituindo a função 
**default_handler()** sem que haja a necessidade de alterar o arquivo **startup.c**. É importante observar que vetores
de interrupção sem rotinas foram inicializados em zero para não precisar especificar o tamanho que o array vai ocupar na
memória.

Definição de variáveis externas que carregam o início e fim de cada seção. Essas variáveis são usadas na função
**reset_handler()** e são criadas durante o processo de **linkedição**.

# stm32f411-rom.ld #

Esse arquivo tem por objetivo juntar o código objeto (relocável) em um local e gerar um único arquivo executável, ou
seja, realiza a **linkedição**.

O principal objetivo do linker script é descrever como as seções dos arquivos objeto de entrada devem ser mapeados e
controlar o layout da memória no arquivo de saída.

Assim, é feita a descrição das regiões de memória disponíveis no dispositivo e a alocação das seções de acordo com as
especificações feitas no arquivo.

Por fim, é feita a exportação dos endereços de memória das seções criadas nas variáveis usadas em **startup.c**.