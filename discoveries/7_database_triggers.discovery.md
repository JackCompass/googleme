# Database triggers Or TypeORM subscribers

Let's establish a few things first I have worked only with the Postgres database so things might be a little different with other databases.

Before we begin, let me explain a little about triggers, In simple terms it is just a function that would run on a specific event on the database itself. The majority of the time it would be used to make some changes in the database itself.

A simple example would be to update a field such as updated_at in your relation every time the record gets updated.

## How to create a Trigger?

```sql
CREATE OR REPLACE FUNCTION update_person_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

- `update_person_updated-at` is the function name. You can use any name you want.
- Inside the `BEGIN` block, you will write the logic. `NEW` is the record itself that would have all the attributes present in the table.
- `RETURN` the keyword will return the modified entity.

You can write as simple or complex logic as you want. Now we have created the function we just need to inform when to run this function ( Setting up the trigger for this function )

```sql
CREATE TRIGGER person_updated_at_trigger
BEFORE UPDATE ON person
FOR EACH ROW
EXECUTE FUNCTION update_person_updated_at();
```

- `person_udpated_at_trigger` : Trigger name (No brainer)
- `BEFORE UPDATE ON` : It informs the trigger when to activate. You can run the trigger at multiple stages.

  1. BEFORE INSERT

  2. BEFORE UPDATE

  3. BEFORE DELETE

  4. AFTER INSERT

  5. AFTER UPDATE

  6. AFTER DELETE

There are a few more stages. You can look up the entire list on the Postgres docs.

- `EXECUTE FUNCTION` : Self Explanatory, which function to run? In our case, it's the function that we created above.

Simple enough RIGHT!!!

## How to create a subscriber?

Create a class and implement the methods of this class `EntitySubscriberInterface` and don't forget to add `@EventSubscriber` decorator to it and make sure to list this class in your datasource. Otherwise, typeORM would never know about this.

For the methods that you need to implement you can go to this link [`EntitySubscriberInterface`](https://typeorm.io/listeners-and-subscribers#event-object) Methods

## Conclusion

What should we use in our code?

Well I have used both and I didn't see any performance degradation with the subscribers (In large codebases triggers would work better), but triggers in the database itself are more robust. Also keeping in mind that the subscribers work at the application level, you can inject your project-specific logic into it.

So my opinion is if you don't need your application logic inside the trigger go with the Database trigger as it ensures data integrity as well. It also not going to allow any entry to bypass it (easily ).

Share your views on which one is better.
