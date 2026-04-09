# Modelando um Dashboard de E-commerce com Power BI Utilizando Fórmulas DAX

**Bootcamp:** Klabin - Excel e Power BI Dashboards 2026  
**Módulo:** 7 - Desafio de Projeto 2  
**Arquivos Finais:** `modelo-powerbi-dashboard-ecommerce-desafio.pbix`
`modelo-powerbi-dashboard-ecommerce-desafio.png`

---

## 📌 Objetivo do Projeto
O principal objetivo deste projeto é construir um modelo de dados otimizado utilizando o esquema em estrela (**Star Schema**). A partir de uma única tabela plana contendo dados financeiros (`financials_origem`), foram derivadas tabelas fato e dimensão utilizando recursos de agrupamento, colunas condicionais no Power Query e modelagem de calendário com funções DAX no Power BI.

---

## 🏗️ Estrutura do Modelo (Star Schema)
O banco de dados foi modelado para separar as métricas quantitativas das informações descritivas, facilitando o uso em dashboards e otimizando o desempenho das consultas.

### Tabela Backup (Origem)
* **`financials_origem`**: Tabela base carregada a partir do Financial Sample. Mantida no modo oculto no modelo para servir como backup.

### Tabela Fato
* **`F_Vendas`**: Registra os eventos transacionais de vendas.
    * *Colunas Principais:* SK_ID, ID_Produto, Product, Units Sold, Sales Price, Discount Band, Segment, Country, Salers, Profit, Date.

### Tabelas Dimensão
* **`D_Produtos`**: Agrupamento de informações estatísticas sobre os produtos.
    * *Colunas:* ID_produto, Product, Média de Unidades Vendidas, Médias do valor de vendas, Mediana do valor de vendas, Valor máximo de Venda, Valor mínimo de Venda.
* **`D_Produtos_Detalhes`**: Detalhamento sobre os preços de manufatura e venda dos produtos.
    * *Colunas:* ID_produtos, Discount Band, Sale Price, Units Sold, Manufacturing Price.
* **`D_Descontos`**: Informações relacionadas às faixas de desconto aplicadas.
    * *Colunas:* ID_produto, Discount, Discount Band.
* **`D_Detalhes`**: Tabela para armazenar demais detalhes e métricas adicionais das transações.
* **`D_Calendário`**: Tabela de tempo, gerada via DAX, para amparar as análises temporais.

---

## ⚙️ Etapas de Transformação e Power Query
A construção do diagrama envolveu as seguintes etapas de manipulação de dados:
1.  **Duplicação:** Criação de cópias da tabela `financials_origem` para que, a partir da seleção de colunas específicas, fossem compostas as visões das novas tabelas fato e dimensão.
2.  **Agrupamento de Dados (Group By):** Ferramenta utilizada para criar tabelas sumarizadas, como a `D_Produtos`, calculando agregações como contagem (soma), máximos, mínimos, médias e medianas.
3.  **Coluna Condicional:** Utilizada para construir índices (ex: Índice de Produtos), mapeando números aos nomes dos produtos para gerar chaves de relacionamento (ID_Produto).
4.  **Reorganização:** Renomeação e ordenação de colunas geradas pelo Power Query para adequar a nomenclatura do esquema em estrela.

---

## 🧮 Funções DAX Utilizadas
A dimensão de calendário foi construída inteiramente utilizando funções DAX, garantindo um eixo temporal dinâmico e consistente para o Dashboard. 

* **Tabela Principal:**
    * `D_Calendário = CALENDARAUTO(12)`: Cria uma tabela base com datas extraídas automaticamente do modelo. 
    * **💡 Nota sobre a formatação da coluna Date:** Inicialmente, a função `CALENDARAUTO` retorna as datas acompanhadas de um horário em branco/zerado (comportamento padrão do tipo *DateTime*). Posteriormente, essa coluna foi formatada para o tipo de dados "Data" com o formato `DD/MM/AAAA`. Essa é uma etapa fundamental de modelagem porque:
        1. **Melhora a performance:** Armazenar apenas a data consome menos memória do que armazenar data e hora.
        2. **Evita quebras de relacionamento:** Garante que a ligação de `1 para muitos` com a tabela `F_Vendas` ocorra perfeitamente, sem que resquícios de horas ocultas causem divergências nas chaves.
        3. **Visualização:** Torna a exibição mais limpa e amigável para o usuário final ao utilizar filtros e segmentadores de dados.

* **Colunas Calculadas:**
    * `Year = YEAR('D_Calendário'[Date])`: Extrai o ano da data.
    * `Month Number = MONTH('D_Calendário'[Date])`: Extrai o número do mês.
    * `Week Number = WEEKNUM('D_Calendário'[Date])`: Retorna o número da semana.
    * `Day of the week = WEEKDAY('D_Calendário'[Date])`: Retorna o dia da semana em formato numérico.
    * `Day of the week 2 = FORMAT('D_Calendário'[Date], "DDDD")`: Retorna o nome do dia da semana por extenso.
    * `Month Name = FORMAT('D_Calendário'[Date], "MMMM")`: Retorna o nome do mês por extenso.

---

## 🔗 Relacionamentos 
A modelagem segue as diretrizes de um Star Schema, onde as dimensões filtram a tabela fato `F_Vendas`. Os seguintes relacionamentos foram estabelecidos no modelo e estão ativos:

* `D_Descontos (Índice)` **1 --- 1** `F_Vendas (SK_ID)`
* `D_Detalhes (Índice)` **1 --- 1** `F_Vendas (SK_ID)`
* `D_Produtos_Detalhes (Índice)` **1 --- 1** `F_Vendas (SK_ID)`
* `D_Calendário (Date)` **1 --- \*** `F_Vendas (Date)`
* `D_Produtos (Product)` **1 --- \*** `F_Vendas (Product)`

---

### 👨‍💻 Autor
Desenvolvido por **Adriel Bartolomeu Machado**
