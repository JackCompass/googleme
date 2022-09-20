# Postgres with typeorm and NestJs
Working with a database is essential when it comes to backend and there are plenty of choices available. But with any database you want to work, there are ORMs that are available for that database that help you to connect, query and perform all kinds of operation on it easily. TypeOrm is one such orm that allows you to connect with all different kind of databases (doesn't work well with mongo I think) and perform related queries to that database.

In this discussion we will see how we can configure Postgres Database with typeorm in NestJs and also take a deep dive into how we can perform curd operation via typeorm.

## Pre-requisite
1. Postgres (having it locally would be better)
2. Node
3. Editor or an IDE

## Working with the database
Assuming you have a basic nestJs project setup you can go ahead and run the following command to install the following packages.
```bash
npm i @nestjs/typeorm typeorm pg 
```
Once you have the necessary packages installed you are ready to connect your nestjs application with Postgres. The following code shows you how you can do it.
```js
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      entities: [],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```
If you want you can create an ormconfig.json file and specify all the configuration there but I like it this way. Above, there is a synchronize setting that should always be false in production. What it basically does is it tries to keep your database in sync with the code that you have written in your app. you can find more about it [here](https://typeorm.io/data-source-options#common-data-source-options).
Instead of mentioning entites manually you can pass another setting `autoLoadEntites: true` which will automatically load entites on the fly. But I like to have fine grained control over my application so I prefer this way.

Once you have reached up to this point, you can go ahead and create a class and annotate with `@Entity()` decorator to make it an entity which is basically symbolizes a relation in the database.
```js
import {Entity} from "@nestjs/typeorm";
@Entity()
export class User {}
```
Great, now we have our first Entity. We gotta tell typeorm about this entity, so all we gotta do is mention this class in the entites field in typeorm config. (Not if you have autoLoadEntites set to true).

Now we have everything setup we can discuss about the Columns, Relation mapping and the options that we generally use with postgres database.

To create a column you need to create a field in the entity class and annotate that field with the `@Column()` decorator.
```js
@Entity()
export class User {
    @Column()
    name: string;
}
```
this is a very basic example of how we can do it. but most of the time we have some constraints that we also have to follow for example we only want to allow string data for this field, we can do that by passing the options. Inside the column, we can pass an object like this
```typescript
@Column({type: string})
```
It ensures that the data is always going to be string and if not then it will result in error. Similarly, we have some more options that we use frequently
1. **default**: sets the default value in case if it is not provided.
2. **nullable**: set it to true if the field can have null values as well.
3. **type**: sets the type of data it will hold.
4. **length**: works with only certain kind of types. If type is string length could be 30.
5. **readonly**: This is used when you want to have a column that can only be written once. You can't update the value once set.
6. **update**: It's vice versa of readonly.
7. **primary**: to make a column primary key.
8. **unique**: column can have only unique values.
9. **array**: Indicates if the column is array. set it to true or pass a value to set its length.
10. **spatialFeatureType**: It is used geospatial queries. can set it to ("Geometry", "Point", "Polygon", etc)

In the above options there is one option that needs a bit of exploration, and it is `type`. We must know what are all the kinds of data that we can set in a column or a field can accept. The list goes very long, and you can find it [here](https://typeorm.io/entities#column-types-for-postgres) but here are the most used ones
- string, text,
- number, int, integer etc
- boolean
- float, double etc
- datetime, timestamp, time with time zone, time without time zone etc









