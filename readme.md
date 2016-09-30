# Rails Associations

In this lesson we'll talk about how tables in a relational database relate to each other and how to take advantage of those relationships in Rails apps.


## Objectives

* Describe how relational databases can be used to create relationships between resources.
* Create one to many model relationships in Rails.
* Create many to many model relationships in Rails.


1. Create one-to-many relationships in Rails
1. Modify migrations to add foreign keys to tables
1. Create model instances with associations

### List of data-types

Basic list:

* :binary
* :boolean
* :date
* :datetime
* :decimal
* :float
* :integer
* :primary_key
* :references
* :string
* :text
* :time
* :timestamp

See http://stackoverflow.com/questions/17918117/rails-4-datatypes


> Note: 90% of the time prefer *decimal* over *float* [2][2]

> Note: Prefer *text* over *string* if on postgresql (maybe).  Otherwise prefer *string* over *text* when your data is definitely always less than 255.  [3][3]

> Note: prefer datetime unless you have a specific reason to use one of the others.  ActiveRecord has extra tools for datetime


### Associations: Relationships Between Models

| Relationship Type | Abbreviation | Description | Example |
| :--- | :--- | :--- | :--- |
| One-to-One | 1:1 | An instance of one model is associated with one (and only one) instance of another model | One author can have one primary mailing address. |
| One-to-Many | 1:N | Parent model is associated with many children from another model | One author can have many books. |
| Many-to-Many | N:N | Two models that can both be associated with many of the other. | Libraries and books. One library can have many books, while one book can be in many libraries. |

### One-To-Many (1:N) Relationship

**Example:** One owner `has_many` pets and a pet `belongs_to` one owner (our `Pet` model will have a foreign key (FK) `owner_id`). The foreign key always goes on the table with the data that belongs to data from another table. In this example, a person **has_many** pets, and a pet **belongs_to** a person. The foreign key `person_id` goes on the `pets` table to indicate which person the pet belongs to.

![](https://raw.githubusercontent.com/sf-wdi-18/notes/master/lectures/week-07/day-1-intro-sql/dawn-simple-queries/images/primary_foreign_key.png)

**Always remember!** Whenever there is a `belongs_to` in the model, there should be a *FK in the matching migration!*

<img src="https://chryus.files.wordpress.com/2014/02/img_1839.jpg" style="max-width: 600px">

#### 2-steps to setting up relationships in Rails

Rails requires us to do two things to establish a relationship.  

1. database - create the foreign key
2. Rails models - tell Rails about the relationship

**First: Database** we need to add an `other_id` column in the database.  

This column belongs on the model that **belongs_to** the parent model.  When populated this will contain the id of the parent model.

This is a database change so it means we're going to write a migration (or edit one we're already writing).  We'll add something like the following to our migration:

```ruby
# we're editing an existing create_table migration to add this field - it HAS NOT BEEN committed to master yet
create_table :pets do |t|
  # You ONLY need to add ONE OF THESE THREE to your new migration
  t.integer :owner_id
  # OR...
  t.references :owner
  # OR...
  t.belongs_to :owner
end
```

<details><summary>What's the difference between `t.integer`, `t.references`, and `t.belongs_to`?</summary>

* `t.integer :owner_id` is technically accurate since the column name should be `owner_id` and database IDs are integers.
* `t.references :owner` is a bit more semantic and readable and has a few bonuses:

  1. It defines the name of the foreign key column (in this case, `owner_id`) for us.
  2. It adds a **foreign key constraint** which ensures **referential data integrity**[4][4]  in our Postgresql database.

**But wait, there's more...**

We can actually get even more semantic and _rail-sy_ and say:

`t.belongs_to :owner`

This will do the same thing as `t.references`, but it has the added benefit of being super semantic for anyone reading your migrations later on.
</details>


**Second: Rails Models** we have to establish the relationship in the rails models themselves.  That means adding code like:

```ruby
class Owner < ActiveRecord::Base
  has_many :pets  # note has_many uses plural form
end

class Pet < ActiveRecord::Base
  belongs_to :owner
end
```

Note: `belongs_to` uses the singular form of the class name (`:owner`), while `has_many` uses the pluralized form (`:pets`).

But if you think about it, this is exactly how you'd want to say this in plain English. For example, if we were just discussing the relationship between pets and owners, we'd say:

  - "One owner has many pets"
  - "A pet belongs to an owner"


This makes rails aware of the relationship and ActiveRecord will make it easy for us to do things in the console or in our code that make use of this relationship.



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
fido = Pet.create(name: "Fido")
lassie = Pet.create(name: "Lassie")
nathan = Owner.create(name: "nathan")
nathan.pets
fido.owner
nathan.pets << fido # Makes "fido" one of my pets
nathan.pets << lassie # Makes "lassie" another one of my pets
nathan.pets.size
nathan.pets.map(&:name)
nathan.pets.each {|x| puts "My pet is named #{x.name}!"}
fido.owner

# What's going to be returned when we do this?
fido.owner.name
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

Head over to the [One-To-Many Challenges](one_to_many_challenges.md) and work together in pairs.

-----

## Many-To-Many (N:N) with 'through'

**Example:** A student `has_many` courses and a course `has_many` students. Thinking back to our SQL discussions, recall that we used a *join* table to create this kind of association.

A *join* table has two different foreign keys, one for each model it is associating. In the example below, 3 students have been associated with 4 different courses:

| student_id | course_id |
| ---------- | --------- |
| 1          | 1         |
| 1          | 2         |
| 1          | 3         |
| 2          | 1         |
| 2          | 4         |
| 3          | 2         |
| 3          | 3         |

### Set Up

To create N:N relationships in Rails, we use this pattern: `has_many :related_model, through: :join_table_name`

1. In the terminal, create three models:

  ```
  rails g model Student name:string
  rails g model Course name:string
  rails g model Enrollment
  ```

  `Enrollment` is the model for our *join* table. When naming your join table, you can either come up with a name that makes semantic sense (like "Enrollment"), or you can combine the names of the associated models (e.g. "StudentCourse").

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


3. Open up the models in your text-editor, and edit them so they include the proper associations:

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

1. In the terminal, run `rails db:migrate` to create the new tables.

2. Enter the rails console (`rails c`) to create and associate data!

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


## Stretch Challenge: Self-Referencing Associations

Lots of real-world apps create associations between items that are the same type of resource.  Read (or reread) <a href="http://guides.rubyonrails.org/association_basics.html#self-joins" >the "self joins" section of the Associations Basics Rails Guide</a>, and try to create a self-referencing association in your `practice_associations` app. (Classic use cases are friends and following, where both related resources would be users.)

## Migration Workflow

Getting your models and tables synced up is a bit tricky. Pay close attention to the following workflow, especially the rake tasks.

```
# create a new rails app
rails new my_app -d postgresql
cd my_app

# create the database
rails db:create

# REPEAT THESE TASKS FOR EVERY CHANGE TO YOUR DATABASE
# <<< BEGIN WORKFLOW LOOP >>>

# -- IF YOU NEED A NEW MODEL --
# auto-generate a new model (AND automatically creates a new migration)
rails g model Pet name:string
rails g model Owner name:string

# --- OTHERWISE ---

# if you only need to change fields in an *existing* model,
# you can just generate a new migration
rails g migration AddAgeToOwner age:integer

# never try to create a migration file yourself through the file system! it's really hard to get the name right!

# -- EITHER WAY --
### whether we're creating a new model or updating an existing one, we can manually edit our models and migrations in our text editor.
# update associations in model files --> this affects model interface
# update foreign keys in migrations --> this affects database tables

# generate schema for database tables
rails db:migrate

# <<< END LOOP >>>

# finally, we need some data to play with
# for now, we'll seed it manually, from the rails console...
rails c
> Pet.create(name: "Wowzer")
> Pet.create(name: "Rufus")

# but later we will run a seed task
rails db:seed
```

## Helpful Hints

When you're **creating associations** in Rails ActiveRecord (or most any ORM, for that matter):

  * Define the relationships in your models (the blueprint for your objects)
    * Don't forget to define all sides of the relationship (e.g. `has_many` and `belongs_to`)
  * Remember to put the foreign key for a relationship in your migration
    * If you're not sure which side of the relationship has the foreign key, just use this simple rule: the model with `belongs_to` must include a foreign key.

## Less Common Associations

These are for your references and are not used nearly as often as `has_many` and `has_many through`.

  * <a href="http://guides.rubyonrails.org/association_basics.html#the-has-one-association">has_one</a>
  * <a href="http://guides.rubyonrails.org/association_basics.html#the-has-one-through-association">has_one through</a>
  * <a href="http://guides.rubyonrails.org/association_basics.html#has-and-belongs-to-many-association-reference">has_and_belongs_to_many</a>

## Useful Docs

* <a href="http://guides.rubyonrails.org/association_basics.html">Associations Rails Guide</a>
