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
O write necessariamente gera apenas uma syscall para cada vez que ele é chamado. Também precisamos definir qual código chamamos, no caso 1, e qual é o tamanho do texto que queremos escrever. O printf faz tudo isso automaticamente, não precisamos colocar o código nem o tamanho do texto. Além disso, o printf tem opções de formatação usando o símbolo de porcentagem, mas não necessariamente gera uma syscall para cada chamada do printf.
```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
O write é mais previsível, pois ele escreverá apenas o que você pediu a ele, do tamanho que você especificou, com exatamente uma syscall.
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
O file descriptor utilizado foi o 3. Os números 0, 1 e 2 não foram utilizados pois eles correspondem respectivamente ao stdin, stdout, stderr. O próximo número livre após o 2 é, logicamente, o 3, que foi utilizado.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
Conseguimos saber se o arquivo foi lido completamente se o número de bytes lidos for menor que o tamanho do buffer.
```

**3. Por que verificar retorno de cada syscall?**

```
Precisamos verficar o retorno de cada syscall para ter certeza de que ela foi bem sucedida.
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
| 16          |       82        |  0.000197 |
| 64          |       21        | 0.000081  |
| 256         |       6         | 0.000083  |
| 1024        |        2        | 0.000051  |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto maior o tamanho do buffer, menor o número de syscalls que precisamos fazer, porque conseguimos armazenar mais informação em apenas uma chamada do read, tendo que chamar menos vezes até atingirmos o final da file.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
Não necessariamente. O tamano do read depende de quantos caracteres estão disponíveis para serem lidos. Se chegarmos no final da file, leremos apenas os caracteres restantes, o que pode ser menor que o tamanho do buffer.
```

**3. Qual é a relação entre syscalls e performance?**

```
Geralmente, quanto menor o número de syscalls, melhor a performace.
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000204 segundos
- Throughput: 6529.56 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Para ter certeza que não estamos copiando mais ou menos bytes do que deveríamos.
```

**2. Que flags são essenciais no open() do destino?**

```
O_WRONLY | O_CREAT | O_TRUNC e 0644. O 0644 nos dá as seguintes permissões: rw-r--r--.
```

**3. O número de reads e writes é igual? Por quê?**

```
Tratando-se apenas das operações de copia/cola, o número de reads e writes deve ser igual, já que fazemos uma escrita para cada vez que lemos.
```

**4. Como você saberia se o disco ficou cheio?**

```
Erros na operação de write como menos bytes escritos do que esperado, erros como -1, etc.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
Mantemos memória inútil e impossibilitamos outros programas de acessar aqueles arquivos, já que o kernel vai manter o arquivo como locked, por ter um fd aberto.
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
Os files descriptors ajudam o kernel a impedir operações inválidas como um programa tentando ler de uma file enquanto ele esté sendo escrita ou lida por outro.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Ao aumentar o tamanho do buffer, aumentamos a memória usada no processo, mas diminuimos o número de syscalls necessárias, trocamos memória por um tempo de execução mais rápido.
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** Minha

**Por que você acha que foi mais rápido?**

```
Provavelmente o cp faz mais verficações para evitar erros em outros cenários onde meu programa teria erros.
```

---

## 📤 Entrega
Certifique-se de ter:
- [X] Todos os códigos com TODOs completados
- [X] Traces salvos em `traces/`
- [X] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
