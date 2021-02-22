# ActiveRecord Review Discussion

## ActiveRecord

Before we learned about ActiveRecord, we were able to call on a class method such as `Pet.all` which would return a collection of all the pet instances. That method would look like this:

```ruby
  class Pet
    # ...

    def self.all
      ALL
    end

    # ...
  end
```

When our class inherits from `ActiveRecord::Base`, we get the `all` method (and many more methods) for free.

Discuss with your table the steps involved in ActiveRecord's implementation of the class method `all`. Read the SQL logger output. How is SQL used? What's the return value of this method?

## Domain Modeling

With your table discuss how you would model out the relationships between three models: `Voter`, `Vote`, and `Candidate`. On which table do the foreign keys belong?

## AR Query Methods

Take a look at the following `Voter` class:

```ruby
class Voter < ActiveRecord::Base
  has_many :votes
  has_many :candidates, through: :votes
end
```

By providing the macros `has_many :votes` and `has_many :candidates, through: :votes`, ActiveRecord gives the `Voter` class the instance methods `Voter#votes` and `Voter#candidates`. These methods will fire some SQL, grab some rows from the database, and return the appropriate Ruby instances.

Your task is to run the following commands and read the SQL logger outputs. If you feel very comfortable or passionate about SQL, you can even try and guess what SQL commands will run under the hood when these methods are triggered:

```ruby
voter = Voter.create

candidate = Candidate.create

voter.votes

voter.candidates

vote = Vote.create(voter_id: voter.id, candidate_id: candidate.id)

vote = voter.votes.create(candidate: candidate) # alternative approach to above

vote.voter


```

Here is what I got when I ran the commands using a different model domain with a many-to-many relationship

```ruby

# Guest -< Reservation >- Room
# Guest
# has_many :reservations
# has_many :rooms, through: :reservations

# Reservation
# belongs_to :room
# belongs_to :guest

# Room
# has_many :reservations
# has_many :guests, through: :reservations

guest = Guest.create

# INSERT INTO "guests" ("created_at", "updated_at") VALUES (?, ?)  [["created_at", "2021-02-22 17:06:42.150934"], ["updated_at", "2021-02-22 17:06:42.150934"]]

room = Room.create

# INSERT INTO "rooms" ("created_at", "updated_at") VALUES (?, ?)  [["created_at", "2021-02-22 16:41:39.299276"], ["updated_at", "2021-02-22 16:41:39.299276"]]

guest.reservations

# SELECT "reservations".* FROM "reservations" WHERE "reservations"."guest_id" = ? LIMIT ?  [["guest_id", 101], ["LIMIT", 11]]

guest.rooms

# SELECT "rooms".* FROM "rooms" INNER JOIN "reservations" ON "rooms"."id" = "reservations"."room_id" WHERE "reservations"."guest_id" = ? LIMIT ?  [["guest_id", 101], ["LIMIT", 11]]

reservation = Reservation.create(guest_id: guest.id, room_id: room.id)

# INSERT INTO "reservations" ("guest_id", "room_id", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["guest_id", 101], ["room_id", 701], ["created_at", "2021-02-22 16:42:25.748106"], ["updated_at", "2021-02-22 16:42:25.748106"]]

reservation_two = guest.reservations.create(room: room) # alternative approach to above

# INSERT INTO "reservations" ("guest_id", "room_id", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["guest_id", 101], ["room_id", 701], ["created_at", "2021-02-22 16:42:46.766964"], ["updated_at", "2021-02-22 16:42:46.766964"]]

reservation_two.guest

# No SQL command

```
