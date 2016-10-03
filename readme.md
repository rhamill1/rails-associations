<!--
Creator: Team
Last Edited by: Brianna
Location: SF
-->

![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)

# Active Record Associations

### Why is this important?
<!-- framing the "why" in big-picture/real world examples -->
*This workshop is important because:*

ActiveRecord manages the schemas, models, and the structure of our relational database so that we can do all of the CRUD functionality we're used to without learning SQL. Active Record lets us create associations between types of data, which better models the real world and which will be an important part of features like user login for complex sites.

### What are the objectives?
<!-- specific/measurable goal for students to achieve -->
*After this workshop, developers will be able to:*

* Describe how relational databases can be used to create relationships between resources.  
* Create one to-many-model relationships in Rails.  
* Create many to-many-model relationships in Rails.  

### Where should we be now?
<!-- call out the skills that are prerequisites -->
*Before this workshop, developers should already be able to:*

- Create models with Active Record.  
- Run migrations to add attributes to an existing model in Active Record.  


In this lesson we'll talk about how tables in a relational database relate to each other and how to take advantage of those relationships in Rails apps.


### A Brief Foray into SQL

> "In which we truly learn to appreciate ActiveRecord."


#### What is a Relational Database (RDB)?

Relational databases were invented in the 1970's as a way to structure data so that it can be queried by a "relational algebra." The basic idea of relational model, though, was to use collections of data, or "tables." Each table is organized like a spreadsheet with a row (also known as "record") for each data item and with attributes of those items arranged in columns.


**Authors Table**

| `id` | `first_name` | `last_name` | `year_of_birth` | `year_of_death` |
| :---  | :---  | :---  | :---  | :---  |
| 1 | Rudyard | Kipling | 1865 | 1936 |
| 2 | Lewis | Carroll | 1832 | 1892 |
| 3 | H.G.  | Wells |  1866 | 1946  |

**Books Table**

| `id` | `title` | `publication_year` | `isbn` | `author_id` |
| :---  | :---  | :---  | :---  | :---  |
| 1 | The Jungle Book | 1894 | 9788497896696 | 1 |
| 2 | Alice's Adventures in Wonderland | 1865 | 9781552465707 | 2 |
| 3 | Rikki-Tikki-Tavi | 1894 | 1484123689 | 1 |
| 4 | Through the Looking-Glass | 1871 | 9781489500182 | 2 |
| 5 | The Time Machine |  1895  | 9781423794417 | 3 |

**Primary Key:** The primary key of a relational table uniquely identifies each record in the table. This column is automatically assigned a btree index in postgres.

####What is SQL?

SQL, Structured Query Language, is a specialized language used to create, manipulate, and query tables in relational databases.

* Data **Definition** Language
  * Define and update database's structure
  * `CREATE`, `ALTER`, `RENAME`, `DROP`, `TRUNCATE`
  * Data Types
  * Constraints
* Data **Manipulation** Language
  * CRUD data within the database
  * `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `ORDER BY`    
  * `UPSERT` (attempts an UPDATE, or on failure, INSERT)
  * Queries
  * `JOIN`s
  * Aggregation: `GROUP BY`, `SUM`, `AVG`, `MIN`
* Data **Control** Language (beyond our scope)
  * `GRANT` access to parts of the table


<details><summary>Click for a simple SQL example that you can run in `psql`</summary>

  ```sql
  CREATE TABLE people (
    id serial primary key,
    name TEXT,
    age INTEGER
  );

  CREATE TABLE pets (
    id SERIAL primary key,
    name TEXT,
    age INTEGER,
    breed TEXT,
    people_id INTEGER
  );

  INSERT INTO people ( name, age)
        VALUES ('Zed', 37);

  INSERT INTO people ( name, age)
      VALUES ('Bobby', 7);

  SELECT * FROM people;

  INSERT INTO pets (name, breed, age, people_id)
        VALUES ( 'Fluffy', 'Unicorn', 1000, 1);

  INSERT INTO pets (name, breed, age, people_id)
        VALUES ('Rocko', 'Dog', 4, 2);

  INSERT INTO pets (name, breed, age, people_id)
       VALUES ('Gigantor', 'Robot', 25, 1);

  INSERT INTO pets (name, breed, age, people_id)
       VALUES ('Goldy', 'Fish', 1, 2);

  SELECT * FROM pets;

  SELECT * FROM people
       LEFT JOIN pets
       ON people.id = pets.people_id;
  ```
</details>

#### Why Are Joins Important?

Each table in a relational database is considered a relation. All the data is naturally related by a single set of attributes defined for the table. However, in order to be a relational database, we need to be able to make queries between different relations or tables of data.

JOINS are our means of implementing queries that join together data from multiple tables and show results.

<details><summary>click for image</summary>

![Joins](https://raw.githubusercontent.com/SF-WDI-LABS/shared_modules/master/04-ruby-rails/intro-sql/27/assets/sqljoins.jpg)
</details>


### Keys

* Primary Key: The primary key of a relational table uniquely identifies each record in the table. This column is automatically assigned a btree index in postgres.

* Foreign Key: a foreign key is a field (or collection of fields) in one table that uniquely identifies a row of another table. **In other words, a foreign key is a column or a combination of columns that is used to establish and enforce a link between the data in two tables.**

### Associations: Relationships Between Models

| Relationship Type | Abbreviation | Description | Example |
| :--- | :--- | :--- | :--- |
| One-to-One | 1:1 | An instance of one model is associated with one (and only one) instance of another model | One author can have one primary mailing address. |
| One-to-Many | 1:N | Parent model is associated with many children from another model | One author can have many books. |
| Many-to-Many | N:N | Two models that can both be associated with many of the other. | Libraries and books. One library can have many books, while one book can be in many libraries. |



### One-To-Many (1:N) Relationship

**Example:** One owner `has_many` pets and a pet `belongs_to` one owner (our `Pet` model will have a foreign key (FK) `owner_id`). The foreign key always goes on the table with the data that belongs to data from another table. In this example, a person **has_many** pets, and a pet **belongs_to** a person. The foreign key `person_id` goes on the `pets` table to indicate which person the pet belongs to.

![](https://raw.githubusercontent.com/sf-wdi-18/notes/master/lectures/week-07/day-1-intro-sql/dawn-simple-queries/images/primary_foreign_key.png)

**Always remember!** Most relational databases can only store one field, not an array of values. We need to store the information about the relationship on the side where just one value can keep all the necessary facts.  Whenever there is a `belongs_to` in the model, there should be a *FK in the matching migration!*  

<img src="https://chryus.files.wordpress.com/2014/02/img_1839.jpg" width="50%">

#### Two steps to set up model relationships in Rails

Rails requires us to do two things to establish a relationship.  

1. Database - create the foreign key
2. Models - tell Rails about the relationship so it makes convenient methods

**First: Database** we need to add an `other_id` column in the database.  

This column will be on the model that **belongs_to** the parent model.  

This is a database change, so it means we're going to write a migration (or edit one we're already writing).  We'll add something like the following to our migration:

```ruby
# we're editing an existing create_table migration to add this field - it HAS NOT BEEN committed to master yet
create_table :pets do |t|
  # You ONLY need to add ONE OF THESE THREE to your new migration
  t.integer :owner_id
  # OR...
  t.references :owner
  # OR...
  t.belongs_to :owner  # BEST!
end
```

<details><summary>What's the difference between `t.integer`, `t.references`, and `t.belongs_to`?</summary>

* `t.integer :owner_id` is technically accurate since the column name should be `owner_id` and database IDs are integers.
* `t.references :owner` is a bit more semantic and readable and has a few bonuses:

  1. It defines the name of the foreign key column (in this case, `owner_id`) for us.
  2. It adds a **foreign key constraint** which ensures **referential data integrity**[4][4]  in our Postgresql database.

* `t.belongs_to :owner` is even more semantic and Railsy. It does the same thing as `t.references`, but it has the added benefit of being super semantic for anyone reading your migrations later on.
</details>


**Second: Models** we have to establish the relationship in the Rails models themselves.  That means adding code like:

```ruby
class Owner < ActiveRecord::Base
  has_many :pets  # note has_many uses plural form
end

class Pet < ActiveRecord::Base
  belongs_to :owner
end
```

Note: `belongs_to` uses the singular form of the class name (`:owner`), while `has_many` uses the pluralized form (`:pets`).

If you think about it, this is exactly how you'd want to say this in plain English. For example, if we were just discussing the relationship between pets and owners, we'd say:

  - "One owner has many pets"
  - "A pet belongs to an owner"


This makes Rails aware of the relationship. Active Record will make it easy for us to do things in the console or in our code that make use of this relationship, like writing `an_owner.pets` and `one_pet.owner`.

### Wading in Deeper: Using our Associations

First things first, we need to create our models, run our migrations, do all things necessary to set up our database.

Let's run:

```console
rails db:create
rails db:migrate
```

Now, let's jump into our rails console by typing `rails c` at a command prompt, and check out how these new associations can help us define relationships between models:

```ruby
Pet.count
Owner.count
theo = Pet.create(name: "Theo")
lassie = Pet.create(name: "Lassie")
ilias = Owner.create(name: "Ilias")
ilias.pets
theo.owner
ilias.pets << theo # Makes "Theo" one of Ilias's pets
ilias.pets << lassie # Makes "Lassie" another one of Ilias's pets
ilias.pets.size
ilias.pets.map(&:name)
ilias.pets.each {|x| puts "My pet is named #{x.name}!"}
theo.owner

# What's going to be returned when we do this?
theo.owner.name
```

Remember: We just saw that in Rails, we can associate two model **instances** together using the `<<` operator.

---

#### Wait!!!! What if I forget to add a foreign key before I first run `rails db:migrate`?

If you run `rails db:migrate` before adding a foreign key to the table's migration, it's ok. There's no need to panic. You can always fix this by creating a new migration.  This will be a change migration rather than creating a new table.

```console
rails generate migration AddOwnerReferenceToPets owner:references
```
OR
```console
rails generate migration AddOwnerReferenceToPets owner:belongs_to
```

...and then verify that the migration looks something like the following:

```ruby
class AddOwnerReferenceToPets < ActiveRecord::Migration
  def change
    add_reference :pets, :owner, index: true, foreign_key: true
  end
end
```

Make sure you then update the models with the appropriate `has_many` and `belongs_to` relationships.  

## Challenges, Part 1: 1:N

Head over to the [One-To-Many Challenges](one_to_many_challenges.md).

-----

## Many-To-Many (N:N) with 'through'

**Example:** A student `has_many` courses and a course `has_many` students.  Here's what each table might look like:

Courses

| id |  abbreviation | semester |   
| :-- | :-----------   | :-------- |  
| 1  |  MATH103      |   F16    |  
| 2  |  ENGL200      |   F16    |   
| 3  |  CHEM104      |   F16    |  
| 4  |  COMP200      |   F16    | 
| 5  |  BSKT101      |   F16    |


Students

| id |  first_name  | last_name    |      
| :-- | :-----------  | :---------    |     
| 1  |  Eliza       |  Doolittle   |      
| 2  |  Eliza       |  Thornberry  |    
| 3  |  Eliza       |  Bennet      |    


**Check for understanding**: Where should we add the foreign key? 


#### Join Tables

We'll need a *join table* to create this kind of association.

A *join* table has two different foreign keys, one for each model it is associating. (It can also have other fields.) In the example below, 3 students have been associated with 4 different courses:

| id |  student_id | course_id | grade |   
| :-- | :----------- | :--------- | :----- |  
| 1  |  1          | 1         |   A   |  
| 2  |  1          | 2         |   B   |  
| 3  |  1          | 3         |   B   |   
| 4  |  2          | 1         |   A   |  
| 5  |  2          | 4         |   C   |  
| 6  |  3          | 2         |   A   |  
| 7  |  3          | 3         |   B   |  



### Set Up

To create N:N relationships in Rails, we use this pattern: `has_many :related_model, through: :join_table_name`.  Here's the relevant section of the Rails [Active Record Associations](http://guides.rubyonrails.org/association_basics.html#the-has-many-association) Guide.

1. In the Terminal, create three models:

  ```
  rails g model Student name:string
  rails g model Course name:string
  rails g model Enrollment
  ```

  `Enrollment` is the model for our join table. When naming your join table, you can either come up with a name that makes semantic sense (like "Enrollment"), or you can combine the names of the associated models (e.g. "CoursesStudent").

2. Add the foreign keys to the enrollments migration:

  ```ruby
  #
  # db/migrate/20150804040426_create_enrollments.rb
  #
  class CreateEnrollments < ActiveRecord::Migration
    def change
      create_table :enrollments do |t|
        t.timestamps

        # define foreign keys for associated models
        t.belongs_to :student
        t.belongs_to :course
      end
    end
  end
  ```

    <details><summary>Could we have done this from the command-line?</summary>
    `rails g model Enrollment course:belongs_to student:belongs_to`</details>


3. Open up the models in your text editor, and edit them so they include the proper associations:

  ```ruby
  #
  # app/models/course.rb
  #
  class Course < ActiveRecord::Base
    has_many :enrollments, dependent: :destroy
    has_many :students, through: :enrollments
  end
  ```

  ```ruby
  #
  # app/models/student.rb
  #
  class Student < ActiveRecord::Base
    has_many :enrollments, dependent: :destroy
    has_many :courses, through: :enrollments
  end
  ```

  ```ruby
  #
  # app/models/enrollment.rb
  #
  class Enrollment < ActiveRecord::Base
    belongs_to :course
    belongs_to :student
  end
  ```



### Using Your Associations

1. In the Terminal, run `rails db:migrate` to create the new tables.

2. Enter the Rails console (`rails c`) to create and associate data!

  ```ruby
  # create some students
  sally = Student.create(name: "Sally")
  fred = Student.create(name: "Fred")
  alice = Student.create(name: "Alice")

  # create some courses
  algebra = Course.create(name: "Algebra")
  english = Course.create(name: "English")
  french = Course.create(name: "French")
  science = Course.create(name: "Science")

  # associate our model instances
  sally.courses << algebra
  # ^ same as:
  # sally.courses.push(algebra)
  sally.courses << french

  fred.courses << science
  fred.courses << english
  fred.courses << french

  # here's a little trick: use an array to associate multiple courses with a student in just one line of code
  alice.courses << [english, algebra]
  ```

  **Note:** Because we've used `through`, we can create our associations in the same way we do for a 1:N association (`<<`).

3. Still in the Rails console, test your data to make sure your associations worked:

  ```ruby
  sally.courses.map { |course| course.name }
  # => ["Algebra", "French"]

  fred.courses.map(&:name)  # short-hand!
  # => ["Science", "English", "French"]

  alice.courses.map(&:name)
  # => ["English", "Algebra"]
  ```

## Challenges, Part 2: Many-To-Many

Head over to the [Many-To-Many Challenges](many_to_many_challenges.md) and work together in pairs.


## Note: Self-Referencing Associations

Lots of real-world apps create associations between items that are the same type of resource.  Read (or reread) <a href="http://guides.rubyonrails.org/association_basics.html#self-joins" >the "self joins" section of the Associations Basics Rails Guide</a>, and try to create a self-referencing association in your `practice_associations` app. (Classic use cases are friends and following, where both related resources would be users.)

## Helpful Hints

When you're **creating associations** in Rails Active Record (or most any ORM, for that matter):

  * Define the relationships in your models (the blueprint for your objects)
    * Don't forget to define all sides of the relationship (e.g. `has_many` and `belongs_to`)
  * Remember to put the foreign key for a relationship in your migration
    * If you're not sure which side of the relationship has the foreign key, just use this simple rule: the model with `belongs_to` must include a foreign key.

## Less Common Associations

These are for your references but are not used nearly as often as `has_many` and `has_many through`.

  * <a href="http://guides.rubyonrails.org/association_basics.html#the-has-one-association">has_one</a>
  * <a href="http://guides.rubyonrails.org/association_basics.html#the-has-one-through-association">has_one through</a>
  * <a href="http://guides.rubyonrails.org/association_basics.html#has-and-belongs-to-many-association-reference">has_and_belongs_to_many</a>

## Useful Docs

* <a href="http://guides.rubyonrails.org/association_basics.html">Associations Rails Guide</a>
