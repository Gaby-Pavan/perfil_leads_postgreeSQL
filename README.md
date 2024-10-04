# sql_acompanhamento_vendas

criação de dashboard para acompanhamento de perfil de leads de uma página de venda de carros usados, utilizando PostgreeSQL e Excel

## Base de Dados:
Base de dados fictícia utilizando a ferramenta PgAdmin com as seguintes tabelas:

![Tabelas](![Diagrama img](https://github.com/user-attachments/assets/6f29da10-0491-4665-8500-27352b3a032d))

## Queries 
Para o primeiro gráfico do Dashboard, foi avaliado o gênero dos leads a partir da seguinte query:
```
select
	case
		when ibge.gender = 'male' then 'homens'
		when ibge.gender = 'female' then 'mulheres'
		end as "gênero",
	count(*) as "leads (#)"
	
from sales.customers as cus
left join temp_tables.ibge_genders as ibge
	on lower(cus.first_name) = lower(ibge.first_name)
group by ibge.gender
```
Para o segundo gráfico foi avaliado o status profissional dos leads a partir da seguinte query:
```
select
	case
		when professional_status = 'freelancer' then 'freelancer'
		when professional_status = 'retired' then 'aposentado'
		when professional_status = 'clt' then 'clt'
		when professional_status = 'self_employed' then 'autônomo(a)'
		when professional_status = 'other' then 'outro'
		when professional_status = 'civil_servent' then 'funcionário(a) público(a)'
		when professional_status = 'student' then 'estudante'
		end as "status_profissional",
	(count(*)::float)/(select count(*) from sales.customers) as "leads(%)"
from sales.customers
group by status_profissional
order by "leads(%)"
```
A query 3 avalia a faixa etária dos leads:
```
select 
	case
		when datediff('years', birth_date, current_date) < 20 then '0-20'
		when datediff('years',birth_date, current_date) < 40 then '20-40'
		when datediff('years', birth_date, current_date) < 60 then '40-60'
		when datediff('years',birth_date, current_date) < 80 then '60-80'
		else '80+' end as "faixa etária",
	count(*)::float/(select count(*) from sales.customers) as "leads(%)"
	
from sales.customers
group by "faixa etária"
order by "leads(%)"
```
A query 4 avalia a faixa salarial dos leads:
```
select 
	case
		when income < 5000 then '0-5000'
		when income < 10000 then '5000-10000'
		when income < 15000 then '10000-15000'
		when income < 20000 then '15000-20000'
		else '20000 +' end as "faixa salarial",
	count(*)::float/(select count(*) from sales.customers) as "leads(%)",
	case
		when income < 5000 then 1
		when income < 10000 then 2
		when income < 15000 then 3
		when income < 20000 then 4
		else 5 end as "ordem"
	
from sales.customers
group by "faixa salarial", "ordem"
order by "ordem"
```
Foi adotado como regra de negócio que veículos novos tem até 2 anos e semi-novos acima de 2 anos, a partir disso podemos avaliar a classificação dos veículos visitados com a query 5:
```
with
	classificacao_veiculos as (
		select 
			fun.visit_page_date,
			pro.model_year,
			extract('year' from visit_page_date) - pro.model_year::int as idade_veiculo,
			case
				when (extract('year' from visit_page_date) - pro.model_year::int)<= 2 then 'novo'
				else 'seminovo' end as "classificação do veículo"
	
	from sales.funnel as fun
	left join sales.products as pro
		on fun.product_id = pro.product_id
	)

select 
	"classificação do veículo",
	count(*) as "veículos visitados (#)"
from classificacao_veiculos
group by "classificação do veículo"
```
A fim de entender qual a idade dos veículos visitados, foi elaborada a query 6:
```
with
	faixa_de_idade_veiculo as (
		select 
			fun.visit_page_date,
			pro.model_year,
			extract('year' from visit_page_date) - pro.model_year::int as idade_veiculo,
			case
				when (extract('year' from visit_page_date) - pro.model_year::int)<= 2 then 'até 2 anos'
				when (extract('year' from visit_page_date) - pro.model_year::int)<= 4 then 'de 2 a 4 anos'
				when (extract('year' from visit_page_date) - pro.model_year::int)<= 6 then 'de 4 a 6 anos'
				when (extract('year' from visit_page_date) - pro.model_year::int)<= 8 then 'de 6 a 8 anos'
				when (extract('year' from visit_page_date) - pro.model_year::int)<= 10 then 'de 8 a 10 anos'
				else 'acima de 10 anos' end as "idade do veículo",
			case
				when (extract('year' from visit_page_date) - pro.model_year::int)<= 2 then 1
				when (extract('year' from visit_page_date) - pro.model_year::int)<= 4 then 2
				when (extract('year' from visit_page_date) - pro.model_year::int)<= 6 then 3
				when (extract('year' from visit_page_date) - pro.model_year::int)<= 8 then 4
				when (extract('year' from visit_page_date) - pro.model_year::int)<= 10 then 5
				else 6 end as "ordem"
	from sales.funnel as fun
	left join sales.products as pro
		on fun.product_id = pro.product_id
	)

select 
	"idade do veículo",
	count(*)::float/ (select count(*) from sales.funnel) as "veículos visitados (#)",
	ordem
from faixa_de_idade_veiculo
group by "idade do veículo", "ordem"
order by ordem
```
Para completar o dashboards, foi avaliado os veículos mais visitados por marca com a query 7:
```
select
	pro.brand,
	pro.model,
	count(*) as "visitas(#)"
from sales.funnel as fun
left join sales.products as pro
	on fun.product_id = pro.product_id
group by pro.brand, pro.model
order by pro.brand, pro.model, "visitas(#)"
```
Depois de desenvolvida as consultas em SQL, as tabelas resultantes das queries foram passadas para o Excel e cada uma gerou um gráfico. O resultado final pode ser
visualizado abaixo:

![dashboard](![image](https://github.com/user-attachments/assets/3dbb142b-bc92-4318-bfe2-074845310879)





