# semana17modelagem

Aula 1 

1-Comunicação empática com o Marketing
Para evitar que a resposta pareça um "não" preguiçoso da TI, use a técnica da parceria de negócios. Em vez de focar no que não dá para fazer, foque no impacto real e na alternativa viável.

Evite o "Tech Speak": Não fale sobre índices clusterizados, queries nested ou joins ineficientes. Eles não precisam saber disso.
Use analogias do mundo real: "Imagine que nossa tabela de compras é uma biblioteca com milhões de livros. Se o marketing me pedir para ler a biblioteca inteira todo dia às 9h para achar quem comprou ontem, a biblioteca vai parar de atender os clientes que querem comprar livros novos."
Apresente a solução (o "Sim, mas..."): "Eu entendo perfeitamente que vocês precisam desses dados diariamente para as campanhas. Para garantir que o site continue rápido para os clientes, em vez de rodar essa busca no banco principal em tempo real, podemos agendar a entrega automática desse relatório na madrugada. O que acham?"

2-Mantendo a calma em problemas complexos
Milhões de registros geram pressão. Para não entrar em desespero ao ver uma consulta travando o banco de dados, adote práticas de higiene mental e técnica:

Divida e conquiste (Slicing): Não tente resolver os milhões de linhas de uma vez. Crie a consulta limitando para os dados de apenas 1 hora ou 1 dia de um único cliente. Se funcionar perfeitamente no pequeno, você expande.
Ambiente de testes (Staging): Nunca teste soluções pesadas direto no banco de produção. Saber que você está mexendo em uma cópia segura reduz a ansiedade em 90%.
Pausas programadas: Se travar em um erro por mais de 30 minutos, levante, tome uma água ou mude de tarefa por 5 minutos. O cérebro continua processando o problema em segundo plano.

3-Incluindo outras equipes na solução
Ninguém cresce sozinho em uma empresa de e-commerce. Eu traria os seguintes aliados para o jogo:

Engenharia de Dados / Infraestrutura: Eles podem criar uma Replica de Leitura (um espelho do banco de dados). Assim, o marketing pode rodar relatórios pesados à vontade sem afetar as compras do site.
Time de UX/Product Design (se aplicável): Se o marketing quiser esses dados em um painel (dashboard), o time de produto pode ajudar a desenhar uma ferramenta interna de autoatendimento (como Metabase ou PowerBI).
Benefícios: Você divide a carga técnica, evita retrabalho, aprende novas arquiteturas e cria um relacionamento forte entre departamentos que geralmente não se falam.

4-Alinhando prazos e expectativas com o gestor
Se a liderança pedir esse relatório "para ontem" em um banco de dados gigante sem estrutura, a honestidade radical — acompanhada de dados — é o melhor caminho.

Abordagem baseada em dados: Mostre o cenário atual. "Gestor, se eu rodar essa query hoje do jeito que está, o tempo de resposta do site vai aumentar em X%, o que pode derrubar o carrinho de compras. Precisamos de Y dias para criar um índice ou uma tabela agregada."
Entregas em fatias (MVP): Ofereça o mínimo viável imediatamente. "Para não travar o marketing, eu posso extrair os dados manualmente apenas da última semana hoje à noite. Enquanto eles usam isso, eu trabalho na automação definitiva para a próxima semana. Pode ser?"
Foque no negócio: Mostre que seu objetivo é proteger a receita da empresa, e não apenas evitar o trabalho técnico.

Aula 2 

1-Valor total das vendas realizadas até hoje
SELECT SUM(valor_total) AS valor_total_vendas
FROM compras;

2-Quantidade de clientes únicos que fizeram compras
SELECT COUNT(DISTINCT cliente_id) AS total_clientes_unicos
FROM compras;

3-Valor médio gasto por compra
SELECT AVG(valor_total) AS valor_medio_por_compra
FROM compras;

4-Quantidade de compras feitas em um determinado mês (Ex: Setembro de 2024)
SELECT COUNT(*) AS total_compras_setembro
FROM compras
WHERE data_compra BETWEEN '2024-09-01' AND '2024-09-30';

Aula 3

1-Relacionamento completo entre clientes e compras (INNER JOIN)
SELECT c.nome, c.email, co.valor_total
FROM clientes AS c

2-Exibindo todos os clientes, mesmo os que não compraram (LEFT JOIN)
SELECT c.nome, c.email, co.valor_total
FROM clientes AS c
LEFT JOIN compras AS co ON c.cliente_id = co.cliente_id;

3-Exibindo todas as compras, mesmo sem clientes registrados (RIGHT JOIN)
SELECT c.nome, c.email, co.compra_id, co.valor_total
FROM clientes AS c
RIGHT JOIN compras AS co ON c.cliente_id = co.cliente_id;

resposta das pergundas 

1-O INNER JOIN garante isso porque ele funciona como uma interseção matemática. Ele filtra e retorna apenas as linhas que possuem correspondência exata em ambas as tabelas (ou seja, onde o cliente_id existe tanto na tabela clientes quanto na tabela compras).
2- O LEFT JOIN preserva todos os registros da tabela à esquerda do comando (neste caso, clientes), independentemente de haver uma linha correspondente na tabela da direita (compras). Quando não há compras, o MySQL preenche automaticamente os campos vazios com NULL.
3-O RIGHT JOIN faz o oposto do anterior: ele prioriza e força a exibição de todos os registros da tabela à direita do comando (neste caso, compras). Mesmo que um cliente_id na tabela de compras não encontre um correspondente na tabela de clientes (devido a uma exclusão ou erro), a compra continuará aparecendo no relatório.

Aula 4 

1-Subconsulta em SELECT – Qual o total de compras por cliente?
SELECT 
    c.nome,
    (SELECT SUM(co.valor_total) 
     FROM compras AS co 
     WHERE co.cliente_id = c.cliente_id) AS total_gasto_por_cliente
FROM clientes AS c;

2-Subconsulta em WHERE – Quais clientes gastaram acima da média?
SELECT DISTINCT c.nome, c.email
FROM clientes AS c
INNER JOIN compras AS co ON c.cliente_id = co.cliente_id
WHERE co.valor_total > (SELECT AVG(valor_total) FROM compras);

3-Subconsulta em FROM – Relatório com total de compras por cliente acima da média geral
SELECT resumo.nome, resumo.total_acumulado
FROM (
    SELECT c.nome, SUM(co.valor_total) AS total_acumulado
    FROM clientes AS c
    INNER JOIN compras AS co ON c.cliente_id = co.cliente_id
    GROUP BY c.cliente_id, c.nome
) AS resumo
WHERE resumo.total_acumulado > (SELECT AVG(valor_total) FROM compras);
