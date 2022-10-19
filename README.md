## Challenge BI - Financeiro

Praticar o desenvolvimento de relatórios gerencias utilizando o Power BI

| :placard: Vitrine.Dev |     |
| -------------  | --- |
| :sparkles: Nome        | **Semana 3 e 4 - Financeiro**
| :label: Tecnologias | Power BI, SQL (tecnologias utilizadas)
| :rocket: URL         | https://bit.ly/dashboarddefinanças
| :fire: Desafio     | https://www.alura.com.br/challenges/bi

<!-- Inserir imagem com a #vitrinedev ao final do link -->

**Overview financeiro**
![image](https://user-images.githubusercontent.com/62486279/135931730-dd62ae27-431b-448b-911f-5783f904973d.png#vitrinedev)

**Análise de cenário**
![image](https://user-images.githubusercontent.com/62486279/135931070-ef61471b-d337-4a3a-a871-a3e1348a7da4.png#vitrinedev)

## Detalhes do projeto

Nesse terceiro e ultimo desafio da **Alura** através do Challenge BI (https://www.alura.com.br/challenges/bi) foi proposto a criação de dois (2) dashboard para a área financeira da **Alura Store** (empresa ficticia), sendo uma delas um **overview financeiro** para acompanhamento das métricas abaixo:

- Receita 
- Custos
- Despesas
- Lucro

e um outro para **análise de cenário**

E como nos outros projetos, todo o acompanhemento foi feito via Trello:

![image](https://user-images.githubusercontent.com/62486279/135931836-584355c2-29fc-440a-b0bf-15e9e7ff2a55.png)

Link: https://trello.com/b/8gb2Kmzn/challenge-bi-semana-3-e-4

## Como o report foi criado?

## 1) ETL e Relacionamento entre tabelas

Nesse projeto foi disponibilizado bases em SQL:

![image](https://user-images.githubusercontent.com/62486279/135931933-c2b1a81a-c17e-45e9-96a7-49e66ad81194.png)

Onde foi necessario fazer a restauração utilizando o MySQL conforme passo a passo abaixo: 

https://www.alura.com.br/artigos/restaurar-backup-banco-de-dados-mysql

E para criar a conexão de dados junto ao Power BI, criamaos as seguintes Query 

**Tabela - Notas fiscais, pedidos e produtos**

~~~SQL
SELECT 
  a.id_nota,a.numero_nota,a.id_pedido,a.id_vendedor,a.valor_venda,a.frete,a.imposto,
  b.id_produto,b.quantidade,b.data_compra,
  (c.preco/100) AS preco_tratado,(c.custos/100) as custo_tratado,(c.custos/100)*b.quantidade AS custos_total

FROM 
  notas_fiscais A left join pedidos B on a.id_pedido = b.id_pedido
                  left join produtos C on b.id_produto = c.id_produto;
~~~

![image](https://user-images.githubusercontent.com/62486279/135932511-df50f06c-2bb1-4385-a29d-e9f753807cef.png)

**Tabela - Notas fiscais, pedidos e produtos**
~~~SQL
SELECT 
  id_produto,categoria_produto,preco/100 as preco_tratado,custos/100 as custos_tratado 
FROM 
  produtos;
~~~

![image](https://user-images.githubusercontent.com/62486279/135934723-34da45c1-55e1-40c1-9d63-b84e487fe7ca.png)

**Tabela - Vendedores**
~~~SQL
SELECT 
  id_vendedor,nome_vendedor 
FROM 
  vendedores;
~~~

![image](https://user-images.githubusercontent.com/62486279/135934885-256f94eb-cd94-44dc-902f-4edc38db3a33.png)

Alem de realizar a tratativa necessaria na Query, pois os valores estavam armezado por 100 e por conta dos filtros necessarios, criamos três (3) tabelas auxiliares:

![image](https://user-images.githubusercontent.com/62486279/135935061-6c0e90f1-0942-4e4e-ad5a-e8dd7f928c9f.png)

~~~
let
    Fonte = Date.StartOfYear (List.Min(F_notas_fiscais_pedidos[data_compra])),
    DataFinal = Date.EndOfYear (List.Max(F_notas_fiscais_pedidos[data_compra])),
    Personalizar1 = List.Dates (Fonte,Number.From(DataFinal-Fonte)+1, #duration (1,0,0,0)),
    #"Convertido para Tabela" = Table.FromList(Personalizar1, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Convertido para Tabela",{{"Column1", type date}}),
    #"Colunas Renomeadas" = Table.RenameColumns(#"Tipo Alterado",{{"Column1", "Data"}}),
    #"Ano Inserido" = Table.AddColumn(#"Colunas Renomeadas", "Ano", each Date.Year([Data]), Int64.Type),
    #"Nome do Mês Inserido" = Table.AddColumn(#"Ano Inserido", "Nome do Mês", each Date.MonthName([Data]), type text),
    #"Colocar Cada Palavra Em Maiúscula" = Table.TransformColumns(#"Nome do Mês Inserido",{{"Nome do Mês", Text.Proper, type text}})
in
    #"Colocar Cada Palavra Em Maiúscula"
~~~

E relacionando as tabelas entre sim da seguinte forma:

![image](https://user-images.githubusercontent.com/62486279/135935234-8b429627-797d-40ed-ae15-1f67f7f6ea05.png)

## 2) Calculos 

Medida   | Dax | Comentário
-------- | ---------- | ----------
Custos | SUM(F_notas_fiscais_pedidos[custos_total]) | Soma dos custos
Despesas | Var Imposto = <br/> SUM(F_notas_fiscais_pedidos[imposto]) <br/> Var Frete = <br/> SUM(F_notas_fiscais_pedidos[frete_tratado]) <br/> Return Imposto + Frete <br/> | Soma das despesas 
Frete | SUM(F_notas_fiscais_pedidos[frete_tratado]) | Soma do frete 
Impostos | SUM(F_notas_fiscais_pedidos[imposto]) | Soma dos impostos
Lucro | Var Receita = <br/> [Receita] <br/> Var Despesas = <br/> [Despesas]+[Custos] <br/> Return <br/> Receita-Despesas | Lucro da operação
Receita | SUM(F_notas_fiscais_pedidos[valor_venda_tratado]) | Soma de receita 
Todas Despesas | VAR Custos = <br/> [Custos] <br/> Frete = <br/> [Frete] <br/> Var Imposto = <br/> [Impostos] <br/> Return <br/> Custos + Frete + Imposto | Soma de todas as despesas (Custo, frete e impostos)
Top Lucro | CALCULATE(<br/> [Lucro], <br/> TOPN( 5, <br/> ALLSELECTED(D_produtos[categoria_produto]), <br/> [Lucro]), <br/> VALUES(D_produtos[categoria_produto])) | Top cinco dos produtos com mais lucro 
Top Vendedores | CALCULATE(<br/> [Receita],<br/> TOPN(5,<br/> ALLSELECTED(D_vendedores[nome_vendedor]), <br/> [Receita]), <br/> VALUES(D_vendedores[nome_vendedor])) |  Top cinco dos vendedores com mais venda 
Ultima Atualização | "Ultima atualização em: " & SELECTEDVALUE('Ultima Atualiação'[Data e Hora]) | Buscar o valor da ultima atualização
Receita_Cenario | SUM(F_notas_fiscais_pedidos[valor_venda_tratado]) * (1 + Cenario_Receita[Cenario_Receita Valor]) | Receita conforme % do cenario 
Custos_Cenario | SUM(F_notas_fiscais_pedidos[custos_total]) * (1+Cenario_Custo[Cenario_Custo Valor]) | Custos conforme % do cenario
Despesas_Cenario | Var Imposto = <br/> SUM(F_notas_fiscais_pedidos[imposto]) <br/> Var Frete = <br/> SUM(F_notas_fiscais_pedidos[frete_tratado]) <br/>Return (Imposto + Frete) * (1 + Cenario_Despesa[Cenario_Despesa Valor]) | Despesas conforme % do cenario
Lucro_Cenario | Var Receita = <br/> [Receita_Cenario] <br/> Var Despesas = <br/> [Despesas_Cenario]+[Custos_Cenario] <br/> Return <br/> Receita-Despesas | Lucro conforme % do cenario
Despesas + Custo Cenario | [Custos_Cenario] + [Despesas_Cenario] | Soma das despesas e custo conforme % do cenario

## Materiais de apoio 

**Livro:**

https://databinteligencia.com.br/produtos/dominando-o-power-bi/

**Videos:**

https://www.youtube.com/watch?v=6LkCEsWSySY
