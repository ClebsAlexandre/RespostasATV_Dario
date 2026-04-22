# RespostasATV_Dario

📊 PERGUNTAS SQL - APP DE INGESTÃO DE ÁGUA
DATA: 22/04/2026

Este documento contém 20 perguntas estratégicas para análise de dados do aplicativo, acompanhadas de seus respectivos resultados e as consultas SQL utilizadas para extraí-los.

👤 Usuários
1. Quantos usuários estão cadastrados no sistema?

Resposta: 5 Usuários


SELECT COUNT(*) AS total_usuarios 
FROM usuarios;

2. Qual o peso médio dos usuários?

Resposta: 73 Kg


SELECT AVG(peso) AS peso_medio
FROM usuarios;

3. Qual usuário possui a maior meta diária?

Resposta: Carlos Lima, 3150 ml

SELECT u.nome, m.meta_ml 
FROM usuarios u 
JOIN metas_diarias m ON u.id = m.usuario_id 
ORDER BY m.meta_ml DESC 
LIMIT 1;

4. Qual usuário possui a menor meta diária?

Resposta: Ana Paula, 2000 ml


SELECT u.nome, m.meta_ml 
FROM usuarios u 
JOIN metas_diarias m ON u.id = m.usuario_id 
ORDER BY m.meta_ml ASC 
LIMIT 1;

💧 Consumo de água
5. Qual o total de água ingerido por cada usuário?

Resposta: 12300 ml (exemplo do maior)


SELECT u.nome, SUM(i.quantidade_ml) AS total_ingerido 
FROM usuarios u 
JOIN ingestao_agua i ON u.id = i.usuario_id 
GROUP BY u.nome 
ORDER BY total_ingerido DESC;

6. Qual o total de água ingerido em um dia específico?

Resposta: 10250 ml


SELECT SUM(quantidade_ml) AS total_do_dia 
FROM ingestao_agua 
WHERE data_hora::date = '2026-04-19';

7. Qual o usuário que mais bebeu água no período?

Resposta: Carlos Lima, 12300 ml


SELECT u.nome, SUM(i.quantidade_ml) AS total_periodo 
FROM usuarios u 
JOIN ingestao_agua i ON u.id = i.usuario_id 
GROUP BY u.nome 
ORDER BY total_periodo DESC 
LIMIT 1;

8. Qual o usuário que menos bebeu água no período?

Resposta: Carlos Lima, 12300 ml (Nota: em seus dados de teste, os valores se repetiram)


SELECT u.nome, SUM(i.quantidade_ml) AS total_periodo 
FROM usuarios u 
JOIN ingestao_agua i ON u.id = i.usuario_id 
GROUP BY u.nome 
ORDER BY total_periodo ASC 
LIMIT 1;

9. Qual o consumo médio diário por usuário?

Resposta: 2050.00 ml


SELECT nome, ROUND(AVG(total_do_dia), 2) AS media_diaria_ml 
FROM ( 
  SELECT u.nome, i.data_hora::date AS dia, SUM(i.quantidade_ml) AS total_do_dia 
  FROM usuarios u 
  JOIN ingestao_agua i ON u.id = i.usuario_id 
  GROUP BY u.nome, i.data_hora::date 
) AS consulta_interna 
GROUP BY nome;

10. Qual o consumo médio por ingestão (copo)?

Resposta: 293 ml


SELECT u.nome, ROUND(AVG(i.quantidade_ml), 0) AS media_por_copo_ml 
FROM usuarios u 
JOIN ingestao_agua i ON u.id = i.usuario_id 
GROUP BY u.nome;

📅 Análise por dia
11. Quanto cada usuário bebeu por dia?

Resposta: 2050.00 ml (média nos testes)


SELECT u.nome, i.data_hora::date AS dia, SUM(i.quantidade_ml) AS total_no_dia 
FROM usuarios u 
JOIN ingestao_agua i ON u.id = i.usuario_id 
GROUP BY u.nome, i.data_hora::date 
ORDER BY dia DESC;

12. Qual foi o dia com maior consumo total de água?

Resposta: 2026-04-18, 10250 ml


SELECT data_hora::date AS dia, SUM(quantidade_ml) AS total_geral 
FROM ingestao_agua 
GROUP BY dia 
ORDER BY total_geral DESC 
LIMIT 1;

13. Qual foi o dia com menor consumo total?

Resposta: 2026-04-18, 10250 ml (Nota: refletindo os dados de teste)


SELECT data_hora::date AS dia, SUM(quantidade_ml) AS total_geral 
FROM ingestao_agua 
GROUP BY dia 
ORDER BY total_geral ASC 
LIMIT 1;

14. Quantos registros de ingestão existem por dia?

Resposta: 35 registros


SELECT data_hora::date AS dia, COUNT(*) AS total_de_registros 
FROM ingestao_agua 
GROUP BY dia 
ORDER BY dia DESC;

🎯 Metas
15. Qual usuário atingiu a meta diária em cada dia?

Resposta: Ana Paula


SELECT u.nome, i.data_hora::date AS dia, SUM(i.quantidade_ml) AS total_bebido, m.meta_ml, 
CASE WHEN SUM(i.quantidade_ml) >= m.meta_ml THEN 'Sim' ELSE 'Não' END AS atingiu_meta 
FROM usuarios u 
JOIN ingestao_agua i ON u.id = i.usuario_id 
JOIN metas_diarias m ON u.id = m.usuario_id AND i.data_hora::date = m.data 
GROUP BY u.nome, i.data_hora::date, m.meta_ml 
ORDER BY dia DESC;

16. Quantos dias cada usuário bateu a meta?

Resposta: 6 dias


SELECT u.nome, COUNT(*) FILTER (WHERE total_dia >= m.meta_ml) AS dias_com_meta_batida 
FROM ( 
  SELECT usuario_id, data_hora::date AS dia, SUM(quantidade_ml) AS total_dia 
  FROM ingestao_agua 
  GROUP BY usuario_id, dia 
) AS resumo_diario 
JOIN usuarios u ON u.id = resumo_diario.usuario_id 
JOIN metas_diarias m ON m.usuario_id = u.id AND m.data = resumo_diario.dia 
WHERE total_dia >= m.meta_ml 
GROUP BY u.nome 
ORDER BY dias_com_meta_batida DESC;

17. Qual a porcentagem da meta atingida por dia por usuário?

Resposta:

Ana Paula

15/04/2026: 102.50%

16/04/2026: 95.00%

17/04/2026: 88.20%

João Silva

20/04/2026: 73.21%

18/04/2026: 73.21%

15/04/2026: 60.15%

Carlos Lima

20/04/2026: 65.08%

18/04/2026: 65.08%

17/04/2026: 40.00%


SELECT u.nome, i.data_hora::date AS dia, 
ROUND((SUM(i.quantidade_ml)::numeric / m.meta_ml) * 100, 2) AS porcentagem 
FROM usuarios u 
JOIN ingestao_agua i ON u.id = i.usuario_id 
JOIN metas_diarias m ON u.id = m.usuario_id AND i.data_hora::date = m.data 
GROUP BY u.nome, i.data_hora::date, m.meta_ml 
ORDER BY u.nome, dia DESC;

18. Qual usuário tem melhor desempenho (mais dias com meta atingida)?

Resposta: Ana Paula


WITH progresso_diario AS ( 
  SELECT u.nome, i.data_hora::date AS dia, SUM(i.quantidade_ml) >= m.meta_ml AS bateu 
  FROM usuarios u 
  JOIN ingestao_agua i ON u.id = i.usuario_id 
  JOIN metas_diarias m ON u.id = m.usuario_id AND i.data_hora::date = m.data 
  GROUP BY u.nome, i.data_hora::date, m.meta_ml 
) 
SELECT nome, COUNT(*) AS dias_vitoriosos 
FROM progresso_diario 
WHERE bateu = true 
GROUP BY nome 
ORDER BY dias_vitoriosos DESC 
LIMIT 1;

🧠 Inteligência / Produto
19. Em quais horários ocorre maior consumo de água?

Resposta: * Período da Manhã: 10h e 12h.

Período da Tarde: 14h, 16h e 18h.

Período da Noite: 20h.


SELECT EXTRACT(HOUR FROM data_hora) AS hora, COUNT(*) AS frequencia 
FROM ingestao_agua 
GROUP BY hora 
ORDER BY frequencia DESC;

20. Qual o intervalo médio entre ingestões por usuário?

Resposta: * Carlos Lima: 02:00:00 (2 horas)

João Silva: 02:00:00 (2 horas)

Ana Paula: 02:00:00 (2 horas)

Pedro Santos: 02:00:00 (2 horas)

Maria Souza: 02:00:00 (2 horas)


SELECT u.nome, AVG(intervalo) AS media_horas 
FROM ( 
  SELECT usuario_id, data_hora - LAG(data_hora) OVER (PARTITION BY usuario_id, data_hora::date ORDER BY data_hora) AS intervalo 
  FROM ingestao_agua 
) AS diffs 
JOIN usuarios u ON u.id = diffs.usuario_id 
GROUP BY u.nome;
