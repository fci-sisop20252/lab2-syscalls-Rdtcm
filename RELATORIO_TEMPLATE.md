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
- Linhas: _____ (esperado: 25)
- Caracteres: _____
- Chamadas read(): _____
- Tempo: _____ segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |                 |           |
| 64          |                 |           |
| 256         |                 |           |
| 1024        |                 |           |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
[Sua análise aqui]
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
[Sua análise aqui]
```

**3. Qual é a relação entre syscalls e performance?**

```
[Sua análise aqui]
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: _____
- Operações: _____
- Tempo: _____ segundos
- Throughput: _____ KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [ ] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
[Sua análise aqui]
```

**2. Que flags são essenciais no open() do destino?**

```
[Sua análise aqui]
```

**3. O número de reads e writes é igual? Por quê?**

```
[Sua análise aqui]
```

**4. Como você saberia se o disco ficou cheio?**

```
[Sua análise aqui]
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
[Sua análise aqui]
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
[Sua análise aqui]
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
[Sua análise aqui]
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
[Sua análise aqui]
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** _____

**Por que você acha que foi mais rápido?**

```
[Sua análise aqui]
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
