# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 9 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
O printf funciona de forma diferente do write.
Enquanto o write faz uma chamada de sistema direta sempre que é usado, o printf escreve os dados em um buffer interno, que só é enviado ao sistema, via write por baixo dos panos, quando o buffer está cheio ou quando encontra uma quebra de linha (\n), se estiver escrevendo no terminal.

Por isso, embora printf possa reduzir o número de syscalls em alguns casos, ele também pode gerar várias syscalls se for usado repetidamente com quebras de linha, ao contrário de write, que é mais previsível e controlado.
```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
O metodo write e mais previsivel por que ele faz uma unica chamada de sistema a cada vez que e chamado, diferente do printf que pode variar o numero de syscalls por conta de varios fatores, como o tipo de buffer, presenca de (\n) e etc.
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 128

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
Foi utilizado o File Descriptor 3, pois os descritores 0, 1 e 2 já estavam ocupados por padrão. Como esses tres sao automaticamente abertos pelo kernel ao iniciar um processo, o sistema atribuiu o próximo descritor livre, que é o 3, ao abrir o arquivo.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
Sei que o arquivo foi lido completamente por que o retorno da funcao read foi exatamente o tamanho definido para o meu buffer. Alem disso, nao houve retorno com valores menores que zero que indicaria erro. Portanto ocorreu a leitura corretamente ate aquele ponto do arquivo.
```

**3. Por que verificar retorno de cada syscall?**

```
E importante verificar o retorno de cada syscall para garantir que a operacao foi realizada com sucesso. As chamdas de sistema podem falhar por varios motivos e verificar o retorno permite tratar erros de forma correta.
```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1300
- Chamadas read(): 21
- Tempo: 0.000082 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          | 82              | 0.000191  |
| 64          | 21              | 0.000082  |
| 256         | 6               | 0.000069  |
| 1024        | 2               | 0.000069  |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto maior o tamanho do buffer, menos syscalls sao feitas. Consequentemente, resulta em menos tarefas para o kernel assumir o controle da CPU e realizar a tarefa (transicao entre modo usuario e modo kernel). Por este motivo, nota-se que o tempo diminui, justamente pelos motivos supracitados.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
Nao,pois a penultima chamada read() pode ser menor que BUFFER_SIZE bytes. Isso por que nao necessriamente o input tera um numero mutiplo de BUFFER_SIZE bytes de tamanho. Em seguida ocorre a ultima chamada read() com 0 bytes indicando EOF.
```

**3. Qual é a relação entre syscalls e performance?**

```
Quanto menor o numero de syscalls, mais performance temos. Isso ocorre porque transitar de modo usuario para modo kernel muitas vezes aumenta o overhead do programa.
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000190 segundos
- Throughput: 5620.39 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [ x ] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Precisamos verificar a fim de verificar se foi escrito todos os dados lidos. A funcao write nem sempre escreve todos os dados por alguns motivos como interrupcoes ou limites de buffer. Por isso, a verificaco e importante pois ela garante a integridade dos dados.
```

**2. Que flags são essenciais no open() do destino?**

```
as flags `O_CREAT` e `O_WRONLY`. Isso porque o arqquivo deve ser criado e deve ter permissao de escrita.
```

**3. O número de reads e writes é igual? Por quê?**

```
O número de bytes lidos e escritos tende a ser o mesmo, pois cada bloco lido é escrito no destino. No entanto, o número de chamadas read() e write() pode ser diferente, já que write() nem sempre escreve todos os bytes de uma só vez, podendo exigir múltiplas chamadas para completar a escrita de um único bloco lido.
```

**4. Como você saberia se o disco ficou cheio?**

```
Na chamada do write() se o disco se encontrar cheio ele retorna -1 e o erro ENOSPC que indica justamente que o disco esta cheio.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
Caso os arquivos nao sejam devidamente fechados, o numero de file descriptors pode ser excedido ao limite. Alem disso, fechar os arquivos garante que os ddos sejm escritos e libera recursos do sistema.
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
As syscalls so chamadas que fazem um programa sair do modo usuario e entrar em modo kernel, para que o sistem operacional possa executar operacoes com privilegio elevado
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
File descriptors são numeros inteiros que representam recursos abertos pelo sistema operacional, como arquivos, sockets ou pipes. Eles permitem que o programa faça operações de leitura, escrita e controle sobre esses recursos de forma eficiente e padronizada, funcionando como “handles” para gerenciar o acesso ao sistema de arquivos e outros dispositivos. Portanto, ele e importante pois permite que SO identifique e gerencie recursos, e eles sao a comunicacao entre programa e kernel pois sem os fds o programa nao teria acesso de escrita ou leitura em arquivos por exemplo.

```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
O tamanho do buffer é diretamente proporcional a performance do programa. Pois com buffers grande temos menos syscalls (menos trocas entre modo usuario e modo kernel) e menos overhead por consequencia do numero menor de syscalls. No entanto, buffers grandes exigem mais memoria e por isso sao limitados, por isso o interessante é equilibrar performance com uso de recursos.
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** O meu.

**Por que você acha que foi mais rápido?**

```
Acredito que pelo fato do meu program ser mais enxuto, diferentemente do cp que possui varias flags e verificcoes. O meu apenas le e escreve dados em arquivos.
```

---

## 📤 Entrega
Certifique-se de ter:
- [ ] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [ ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
