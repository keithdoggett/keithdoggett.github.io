---
layout: post
title: "Inserting Records in Rails using a Relation"
date: 2022-12-29 11:00:00 -0400
categories: rails ruby arel
---
Say you have a Rails app where you have 2 tables and a join table between them. If you want to insert records into the join table, generally you'll create tuples of `table_a_id` and `table_b_id`, but for large inserts this can be inefficient, especially when table_a might only have a few records in the tuple and table_b has many. Luckily with the help of ARel, we can insert records from a subquery (ActiveRecord Relation).

## Setup

In this example, we have 2 main models `Part` and `Widget`. Our factory has a bunch of parts to make widgets with, but we're very good at using what's available, so we design our widgets that they can be made of many different parts that are similar in specs. For that reason we have a `WidgetPart` table that lets us know which parts can be included in a widget.

Here's the migrations and models:

```ruby
# create_parts.rb
class CreateParts < ActiveRecord::Migration[7.0]
  def change
    create_table :parts do |t|
      t.integer :part_type
      t.float :height
      t.float :width
      t.float :depth
      t.float :weight
      t.string :material
      t.timestamps
    end
  end
end
```

```ruby
# create_widgets.rb
class CreateWidgets < ActiveRecord::Migration[7.0]
  def change
    create_table :widgets do |t|
      t.string :name
      t.timestamps
    end
  end
end
```

```ruby
# create_widget_parts.rb
class CreateWidgetParts < ActiveRecord::Migration[7.0]
  def change
    create_table :widget_parts do |t|
      t.references :widget
      t.references :part
      t.timestamps
    end

    add_index :widget_parts, %i[part_id widget_id], unique: true
  end
end
```

```ruby
class Part < ApplicationRecord
  has_many :widget_parts
  has_many :widgets, through: :widget_parts

  PART_TYPES = {
    nut: 0,
    bolt: 1,
    gear: 2,
    spring: 3,
    belt: 4,
    pulley: 5
  }.freeze

  enum part_type: PART_TYPES
end
```

```ruby
class Widget < ApplicationRecord
  has_many :widget_parts
  has_many :parts, through: :widget_parts
end
```

```ruby
class WidgetPart < ApplicationRecord
  belongs_to :widget
  belongs_to :part
end
```

## Insert All Implementation

A simple way to insert parts into a new widget would be something like this which utilizes the `#insert_all` method.

```ruby
pump = Widget.create(name: "booster pump")
pump_parts = Part.where(part_type: [:nut, :bolt], material: ["stainless steel", "aluminum"], width: 0..5, height: 0..10, depth: 0..2, weight: 0..30)
pump_parts = pump_parts.or(Part.where(part_type: [:gear], material: ["stainless steel", "aluminum"], weight: 0..200))
pump_parts = pump_parts.or(Part.where(part_type: [:belt], weight: 0..500))

tuples = pump_parts.map do |pump_part|
  {part_id: pump_part.id, widget_id: pump.id}
end
WidgetPart.insert_all(tuples)
```

This is a fine approach, but can be inefficient as the list of available parts grows. Now we'll look at how to handle this as a subquery and avoid loading all of the `pump_parts` into the application.

## ARel Implementation

ARel includes an `InsertManager` class that handles this exact case we want and can help speed up this bulk insertion from a subquery. Our goal is to generate sql like this:

```sql
INSERT INTO widget_parts (part_id, widget_id, created_at, updated_at)
SELECT parts.id as part_id, 1 as widget_id, NOW() as created_at, NOW() as updated_at
FROM parts
WHERE ("parts"."material" IN ('stainless steel', 'aluminum') AND ("parts"."part_type" IN (0, 1) AND "parts"."width" BETWEEN 0.0 AND 5.0 AND "parts"."height" BETWEEN 0.0 AND 10.0 AND "parts"."depth" BETWEEN 0.0 AND 2.0 AND "parts"."weight" BETWEEN 0.0 AND 30.0 OR "parts"."part_type" = 2 AND "parts"."weight" BETWEEN 0.0 AND 200.0) OR "parts"."part_type" = 4 AND "parts"."weight" BETWEEN 0.0 AND 500.0)
```

Where the `WHERE` condition is what we use to build our `pump_parts` relation above. Here's how we can leverage ARel to build this query.

```ruby
# pump_parts and widget code above

wp_tbl = WidgetPart.arel_table
im = Arel::Nodes::InsertStatement.new
im.relation = wp_tbl
im.columns = [wp_tbl[:part_id], wp_tbl[:widget_id], wp_tbl[:created_at], wp_tbl[:updated_at]]

# select just the fields we need
select_stmt = 'parts.id as part_id, ? as widget_id, NOW() as created_at, NOW() as updated_at'
select_stmt = ActiveRecord::Base.sanitize_sql_array([select_stmt, pump.id])
im.select = Arel.sql(pump_parts.select(select_stmt).to_sql)

ActiveRecord::Base.connection.execute(im.to_sql)
```

Pretty simple! As you can see `im.relation` defines which table we're inserting into, `im.columns` defines which columns we'll insert into and `im.select` deifnes the subquery we're using.

Using this technique, it's easy to create data in a join table from an `ActiveRecord::Relation` and avoid loading unnecessary data into the application.

## Benchmarks

To demonstrate the increased efficiency of using subqueries for insertions instead of pulling data into the app, I added 40,000 eligible parts for our pump and compared insert speeds using both methods.

```
                user     system      total        real
Insert All:    0.857074   0.008193   0.865267   (1.954667)
ARel:          0.003582   0.000000   0.003582   (0.319263)
```

Even at this relatively small scale you can see the performance improvement.
