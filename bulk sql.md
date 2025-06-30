insert:
```sql
insert into aboba (id, topic, key, headers, body)  
select *  
from unnest($1::uuid[],  
            $2::text[],  
            $3::text[],  
            $4::jsonb[],  
            $5::jsonb[]  
     ) as t(id, topic, key, headers, body);
```
update:
```sql
update balance  
set active  = t.active,  
    frozen  = t.frozen,  
    income  = t.income,  
    expense = t.expense  
from unnest($1::uuid[], $2::numeric[], $3::numeric[], $4::numeric[],  
            $5::numeric[]) as t(id, active, frozen, income, expense)  
where account_id = t.id
```