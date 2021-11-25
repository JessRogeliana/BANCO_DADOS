CREATE DATABASE STEAM;

USE STEAM;

-- Criando um table pra importar os dados pertinentes da table steam
CREATE TABLE STEAM_DADOS (
APP_ID int,
NOME text,
DATA_LANCAMENTO date,
CATEGORIAS text,
GENEROS text,
AVALIACOES_POSITIVAS int,
AVALIACOES_NEGATIVAS int,
PRECO double
);

-- Criando um table pra importar os dados pertinentes da table steam_requirements_data
CREATE TABLE REQUISITOS(
APP_ID int,
MINIMO_NECESSARIO text,
REQUISITOS_MAC text,
REQUISITOS_LINUX text,
REQUISITOS_WINDOWS text
);

START TRANSACTION;

-- Salvando as alterações
COMMIT;

-- Desativando o salvamento automático
SET AUTOCOMMIT = 0;

-- Adicionando chave primária
ALTER TABLE `STEAM_DADOS` ADD PRIMARY KEY (APP_ID);

-- Adicionando chave estrangeira
ALTER TABLE `REQUISITOS` ADD foreign key (APP_ID) REFERENCES `STEAM_DADOS`(APP_ID);
ALTER TABLE `steam_description_data` ADD foreign key (steam_appid) REFERENCES `STEAM_DADOS`(APP_ID);
ALTER TABLE `steam_media_data` ADD foreign key (steam_appid) REFERENCES `STEAM_DADOS`(APP_ID);
ALTER TABLE `steam_support_info` ADD foreign key (steam_appid) REFERENCES `STEAM_DADOS`(APP_ID);
ALTER TABLE `steamspy_tag_data` ADD foreign key (appid) REFERENCES `STEAM_DADOS`(APP_ID);
ALTER TABLE `steam` ADD foreign key (appid) REFERENCES `STEAM_DADOS`(APP_ID);
ALTER TABLE `steam_requirements_data` ADD foreign key (steam_appid) REFERENCES `STEAM_DADOS`(APP_ID);

-- Aplicando filtros para responder às duvidas definidas

-- Filtro para exibir jogos com mais avaliações positivas, limitando o resultado a 5
SELECT APP_ID, NOME, AVALIACOES_POSITIVAS, data_lancamento FROM `STEAM_DADOS` ORDER BY `AVALIACOES_POSITIVAS` DESC LIMIT 5;
-- Filtro para exibir jogos com mais avaliações negativas, limitando o resultado a 5
SELECT APP_ID, NOME, AVALIACOES_NEGATIVAS FROM `STEAM_DADOS` ORDER BY `AVALIACOES_NEGATIVAS` DESC LIMIT 5;

-- Filtro para exibir jogos gratuitos com mais avaliações positivas, limitando o resultado a 5
SELECT APP_ID, NOME, AVALIACOES_POSITIVAS, PRECO FROM `STEAM_DADOS` WHERE `PRECO` = 0 ORDER BY `AVALIACOES_POSITIVAS` DESC LIMIT 5;
-- Filtro para exibir jogos pagos com mais avaliações negativas, limitando o resultado a 5
SELECT APP_ID, NOME, AVALIACOES_NEGATIVAS, PRECO FROM `STEAM_DADOS` WHERE PRECO <> 0 ORDER BY `AVALIACOES_NEGATIVAS` DESC LIMIT 5;


-- Filtro para exibir quantidade de lançamentos por mês
SELECT month(data_lancamento) as 'Mês_Lançamento',COUNT(*) as 'Quantidade_Jogos_Lançados' FROM steam_dados GROUP BY month(data_lancamento) ORDER BY COUNT(*) DESC;

-- Salvando as alterações
COMMIT

-- Aplicando filtros apenas para manipulação das tables

-- Filtro para selecionar jogos que possuem e não possuem filmes
select count(movies) as 'Total de jogos com filmes' from steam_media_data where movies not like '' group by 'Total de jogos com filmes';
select count(movies) as 'Total de jogos sem filmes' from steam_media_data where movies like '' group by 'Total de jogos sem filmes';

-- Filtro para selecionar quantos emails diferentes existem para suporte (não vazios) 
select SUPPORT_EMAIL, count(distinct support_email) as 'Emails para Suporte' from steam_support_info where support_email not like '' order by 'Emails para Suporte';

-- Filtro para selecionar os jogos mais bem avaliados exibindo a data de lançamento
select appid, name, release_date, positive_ratings from steam order by positive_ratings desc;

-- Filtro para exibir configurações de máquina de cada jogo
select nome, minimo_necessario, requisitos_mac, requisitos_linux, requisitos_windows from steam_dados Jogos inner join requisitos Requisitos using (APP_ID) order by Jogos.app_id;

-- Filtro para exibir qual jogo mais possui a tag action
select * from steam_dados d inner join steamspy_tag_data t on d.app_id = t.appid where action = (select MAX(action) from steamspy_tag_data) order by d.app_id;

-- Filtro para exibir qual jogo mais possui a tag puzzle
select * from steam_dados d inner join steamspy_tag_data t on d.app_id = t.appid where puzzle = (select MAX(puzzle) from steamspy_tag_data) order by d.app_id;

-- Filtro para exibir jogos que possuem descrição
SELECT appid,name, detailed_description from steam inner join steam_description_data on appid = steam_appid group by appid;

-- Filtro para exibir jogos que não possuem descrição
SELECT appid,name from steam where appid not in (select steam_appid from steam_description_data);

-- Filtro para exibir a quantidade de filmes para o gênero Indie
select genres, count(*) as quantidade from steam where genres like '%ndi%' order by quantidade;
