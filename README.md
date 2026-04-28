
``` SQL
drop schema if exists pandemic;

create schema pandemic;

use pandemic;

select count(*) from infectious_cases; 

drop table if exists infectious_cases_norm;
drop table if exists countries;

create table countries (
  id     int auto_increment primary key,
  entity varchar(100) not null,
  code   varchar(20),
  unique key uq_entity (entity)
);

insert into countries (entity, code)
select distinct entity, code
from infectious_cases;

create table infectious_cases_norm as
select c.id as country_id, ic.*
from infectious_cases ic
join countries c on c.entity = ic.entity;

alter table infectious_cases_norm
  drop column entity,
  drop column code,
  add column id int auto_increment primary key first,
  modify country_id int not null,
  add constraint fk_country
      foreign key (country_id) references countries(id);


select count(id) from infectious_cases_norm;

describe infectious_cases_norm;

select
	c.entity,
    c.code,
    avg(ic.Number_rabies) as avg_rabies,
    min(ic.Number_rabies) as min_rabies,
    max(ic.Number_rabies) as max_rabies,
    sum(ic.Number_rabies) as sum_rabies
from infectious_cases_norm as ic
join countries as c on c.id = ic.country_id
where ic.Number_rabies is not null and ic.Number_rabies != ''
group by c.id, c.entity, c.code
order by avg_rabies desc
limit 10;


select 
    Year,
    makedate(Year, 1) as year_start,
    curdate() as today,
    timestampdiff(year, makedate(Year, 1), curdate()) as years_diff
from infectious_cases_norm;


drop function if exists year_diff_from_today;

delimiter //
create function year_diff_from_today(year_value int)
returns int
deterministic
begin
    return timestampdiff(year, makedate(year_value, 1), curdate());
end //
delimiter ;


select year_diff_from_today(1990);

select distinct Year, year_diff_from_today(Year)
from infectious_cases
order by Year desc;

```
