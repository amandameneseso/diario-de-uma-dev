Title: ✧ 0.1 + 0.2 = 0.3? Como a matemática do computador funciona
Date: 2025-11-03
Category: Computação
Tags: matemática, computação
Author: Amanda Meneses

### **A conta que não fecha**

Se você já se aventurou na programação, talvez tenha encontrado um resultado que desafia a lógica. Digite `0.1 + 0.2` no console do JavaScript ou Python e, em vez do esperado `0.3`, você verá algo como `0.30000000000000004`. Como uma máquina capaz de cálculos complexos pode errar uma conta tão básica?

```jsx
console.log(0.1 + 0.2); // 0.30000000000000004
```

Essa aparente falha não é um bug, mas uma consequência fundamental de como os computadores representam números. Este artigo vai revelar os segredos por trás dessa matemática contraintuitiva, mostrando por que a precisão no mundo digital é uma ilusão cuidadosamente gerenciada.

### **A ilusão da precisão decimal**

Nós, humanos, usamos o **sistema decimal (base 10)**. Já os computadores só entendem o **sistema binário (base 2)** — números feitos apenas de 0s e 1s.

Isso cria um problema: **nem todos os números decimais podem ser representados exatamente em binário**.

Por exemplo:

| Número decimal | Binário aproximado (IEEE 754) |
| --- | --- |
| 0.5 | 0.1 |
| 0.25 | 0.01 |
| 0.1 | 0.0001100110011001100110011... (dízima infinita) |
| 0.2 | 0.0011001100110011001100110... (dízima infinita) |

Como o computador tem uma memória finita, ele não pode armazenar uma sequência infinita de dígitos. Em vez disso, ele "arredonda" o número, guardando uma aproximação extremamente próxima, mas não exata. Assim, quando o computador tenta armazenar `0.1`, ele **corta** a dízima e guarda uma **aproximação**. Em 64 bits (precisão dupla), `0.1` é armazenado como:

```
0.1000000000000000055511151231257827021181583404541015625
```

e `0.2` como:

```
0.200000000000000011102230246251565404236316680908203125
```

Quando somamos os dois, o resultado é **0.3000000000000000444...**, que aparece arredondado como `0.30000000000000004`. A soma `0.1 + 0.2` é, na verdade, a soma de duas aproximações, e a pequena imprecisão de cada uma se manifesta no resultado final.

Isso acontece **em todas as linguagens modernas** (Python, JavaScript, C, Java etc.), porque todas seguem o mesmo padrão: **IEEE 754**.

Esse padrão define que um número real (de ponto flutuante) é dividido em **três partes**:

1. **Sinal** → indica se o número é positivo ou negativo
2. **Expoente** → indica o tamanho (escala) do número
3. **Mantissa** → guarda os dígitos significativos

Mais bits significam **maior precisão**, mas **nenhum número decimal infinito pode ser representado exatamente**.

### **Erro não é bug: É uma consequência matemática**

Os pequenos desvios que vemos nos cálculos não são falhas inesperadas no código; são características inerentes e previsíveis da computação.

Existem dois tipos principais de erros conceituais que surgem dessa limitação:

- **Erro de arredondamento:** Ocorre devido à limitação de bits para representar números, como vimos no exemplo `0.1 + 0.2`. O computador é forçado a escolher o valor representável mais próximo.
- **Erro de truncamento:** Acontece quando aproximamos um processo infinito com um número finito de passos. Por exemplo, o cálculo do seno de um ângulo (`sin(x)`) pode ser feito usando uma série matemática infinita (a série de Taylor). Como não dá pra calcular infinitos termos, escolhemos um número finito (por exemplo, 5 termos). Essa **interrupção** gera um pequeno **erro de truncamento**. Quanto mais termos da série são usados, mais preciso é o resultado. No entanto, usar um número excessivo de termos pode levar a um `OverflowError` se os valores intermediários se tornarem grandes demais para serem representados.

### **O limite da visão: O Épsilon da Máquina**

Se a precisão do computador é finita, qual é o seu limite? A resposta está em um conceito chamado **Épsilon da Máquina (ε)**. De forma simples, ele é "o menor número que, somado a 1, o computador consegue diferenciar de 1".

A implicação disso é profunda: embora matematicamente existam infinitos números entre `1` e `1 + ϵ`, para o computador esse intervalo é um "ponto cego". Ele não consegue representar nenhum valor ali. Para todos os efeitos, qualquer número nesse intervalo é simplesmente `1`. Esse 'ponto cego' existe por causa da limitação de bits na **Mantissa** que discutimos. Uma vez que o número é tão pequeno que sua adição não alteraria o último dígito disponível na mantissa, para o computador, nada mudou.

Em Python, podemos descobri-lo assim:

```python
import sys
print(sys.float_info.epsilon)
# 2.220446049250313e-16
```

Ou seja, o computador **não consegue perceber diferenças menores que ε**. Isso significa que, se somarmos 1 + 2.220446049250313 x 10^(-16), o computador ainda percebe a diferença. Mas se somarmos um número menor que isso, ele acha que é o mesmo número (1).

```python
1 + 1e-16 == 1  # False
1 + 1e-17 == 1  # True
```

**Nota:** Se estivermos usando uma máquina com 64 bits, temos que ϵ ≈ 2,22×10^(−16). O formato de ponto de flutuação de precisão dupla ocupa 64 bits da memória do computador e é muito mais preciso que o formato de precisão simples. Esse formato é geralmente chamado de FP64 e é usado para representar valores que exigem um intervalo maior ou um cálculo mais preciso. Embora a precisão dupla permita maior exatidão, ela também exige mais recursos computacionais, armazenamento de memória e transferência de dados.

### **Ordem no caos: Como controlamos a imperfeição**

Antes do padrão IEEE 754, cada fabricante de computador representava números de forma diferente. Isso significava que **a mesma conta poderia dar resultados diferentes em máquinas distintas**.

O IEEE 754 resolveu isso padronizando:

- A forma de armazenar números de ponto flutuante;
- As regras de arredondamento;
- O comportamento em situações especiais.

Enquanto os erros de precisão são silenciosos e inerentes, existem outros erros, chamados de erros de execução, que são catastróficos e podem travar um programa. Para gerenciar esses eventos abruptos, os programadores usam ferramentas de controle como `try…except`. Os erros mais comuns são:

- **Overflow:** O resultado de um cálculo é maior que o número máximo que o sistema pode representar.
- **Underflow:** O resultado é tão próximo de zero que o sistema o aproxima para zero.
- **Divisão por zero:** Uma operação matematicamente indefinida.
- **Número inválido (NaN - Not a Number):** Ocorre como resultado de operações impossíveis, como `0/0` ou a raiz quadrada de um número negativo.

Vejamos um exemplo clássico: tentar dividir um número por zero, uma operação matematicamente impossível. Para lidar com essas situações, os programadores usam estratégias como os blocos `try...except` em Python:

```python
x = 10
y = 0

try:
  resultado = x / y
except ZeroDivisionError:
  print('Infelizmente ocorreu um erro. Não é possível realizar uma divisão por zero.')

# O programa continua a execução normalmente.
```

Em vez de travar, o programa executa o código dentro do bloco `except`, exibe a mensagem de erro de forma controlada e continua sua execução, demonstrando a robustez dessa abordagem.

**Concluindo…**

A precisão nos computadores é uma abstração poderosa, mas fundamentalmente uma aproximação do mundo infinito dos números reais. Cada cálculo, desde uma simples soma até simulações complexas, opera dentro desses limites.

Felizmente, graças à **norma IEEE 754**, podemos confiar que `0.1 + 0.2` vai “errar” exatamente da mesma forma em qualquer computador do mundo!