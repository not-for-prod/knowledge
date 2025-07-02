insert:
```sql
insert into balance (account_id, client_id, currency, active, frozen, income, expense)
select *
from unnest($1::uuid[], $2::uuid[], $3::text[], $4::numeric[], $5::numeric[], $6::numeric[],
            $7::numeric[]) as t(account_id, client_id, currency, active, frozen, income, expense);
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
upsert:
```sql
insert into balance (account_id, client_id, currency, active, frozen, income, expense)
select *
from unnest($1::uuid[], $2::uuid[], $3::text[], $4::numeric[], $5::numeric[], $6::numeric[],
            $7::numeric[]) as t(account_id, client_id, currency, active, frozen, income, expense)
on conflict (account_id)
    do update set
                  active = excluded.active,
                  frozen = excluded.frozen,
                  income = excluded.income,
                  expense = excluded.expense;
```
