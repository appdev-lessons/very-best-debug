# Very Best Debug

In this project, you will get more practice with GUIs by debugging a Very Best venue tracking app.

Here is your target: [very-best-debug.matchthetarget.com](https://very-best-debug.matchthetarget.com/)

This project includes automated tests, so click on this button to get started:

LTI{Load Very Best Debug assignment}(https://grades.firstdraft.com/launch)[S9ymPy6WCsn18gLbByVbZQ7k]{vfdtzJb5bLYqYwuqgeRKpc5d}(5)[Very Best Debug Project]

Then, fork the repository and create your codespace.

## Workflow

1. The goal is to make the code work by correcting all of the bugs that are contained within
1. Checkout the target to see how your code should function
1. Run `rake sample_data` to populate your database
1. Start with visiting the route `/users`
1. You will see an error message, **Read The Error Message**
1. When you understand the error message, work to figure out a solution to fix the error so the Route/Controller/Action/View functions like the target.
1. Remember that the error will guide you to the bug and our skills as developers are determined by how many bugs we understand how to fix
1. Once you have `/users` working correctly, check the routes file and get each working the same as the target
1. Run `rake grade` once all the routes are working to see what else still does not match the target

## Hints

### The POST verb

Any create or update actions should use `post()` routes, and the `<form>` actions associated with them should have a `method="post"`.

### Foreign key queries

There is one tricky bug that we should discuss and solve together.

Assuming a series of `/venue/ID` show page bugs have been dealt with, if we refresh `/venues/1`, then we eventually get the error:

```
undefined method `username' for nil:NilClass
```

And we see the highlighted code in the `app/views/venue_templates/details.html.erb` view template is coming where we try to call `comment.commenter.username`:

```erb
  <% @the_venue.comments.each do |comment| %>
  <tr>
    <td>
      <%= comment.commenter.username %>
    </td>
```

![](/assets/better-errors-venue-comments-bug.png)

From our experience, we know that the `comment.commenter` is somehow `nil`. Above this, we see that we are calling `.comments` on our `@the_venue` object in order to get a list of comments for looping over and displaying: `@the_venue.comments.each do |comment|`.

Because the `.each` call didn't throw an error, we can assume that the `.comments` method is returning an array-like object `ActiveRecord::Relation` of the comments associated with that venue ID. But inside of this loop, the `comment`, or the `.commenter` method called on it, is `nil`.

Let's check in our better errors page built-in REPL. We can start with just `>> @the_venue`, which should return our instance of `Venue`. Then we can try `>> @the_venue.comments`, which should return an array-like object full of comments that match the venue. Good. 

But where is this `.comments` method from? It's defined in the model as one of our association accessors! Any method that we call on a class from our database is defined in the class file. Let's investigate:

First, in order to get from a `Venue` to a `Comment`, we can look at the `Comment` model to remind us of the names of the columns:

```ruby
# app/models/comment.rb

# == Schema Information
#
# Table name: comments
#
#  id         :integer          not null, primary key
#  body       :string
#  created_at :datetime         not null
#  updated_at :datetime         not null
#  author_id  :integer
#  venue_id   :integer
#
```

In the `# == Schema Information` we see there is a foreign key column called `venue_id`. So we want to find all of the comments where ID in this column matches my current venue of interest. In the `Venue` model, we find:

```ruby
# app/models/venue.rb

# == Schema Information
#
# Table name: venues
#
#  id           :integer          not null, primary key
#  address      :string
#  name         :string
#  neighborhood :string
#  created_at   :datetime         not null
#  updated_at   :datetime         not null
#

# ...
  def comments
    my_id = self.id
    matching_comments = Comment.where({ :venue_id => my_id })
    return matching_comments
  end
# ...
```

The `comments` method is an association accessor method. It first says, get the current venue ID (our `@the_venue` instance variable) using the `self.id` call. It then searches the `venue_id` column of the `Comment` model (which accesses the `comments` table) for my current ID, and then it returns all of these matching rows.

So it looks like in our view template the `comments` method should be doing the right thing. And we saw this was the case in our better errors page REPL when we typed `>> @the_venue.comments`. It returned an array-like object full of comments that match the venue.

That brings us to the block variable `comment` in our `.each` loop. At the better errors page REPL, type `>> comment`. Indeed a single `Comment` object is return, which is a row from the table `comments` in our database.

Now at the REPL try: `>> comment.commenter`. It gives us `=> nil`! 

How could we replace our code and manually traverse from the comment to the username associated with that comment? If we look at our `Comment` instance, we can see there is a foreign key column called `author_id` with a value of "96". This is the key that we can use in `User` to look up which user authored a given comment:

```ruby
>> comment
=> #<Comment id: 2730, author_id: 96, body: "There was never a genius without a tincture of mad...", venue_id: 1, created_at: "2019-03-19 03:56:09.000000000 +0000", updated_at: "2019-10-08 10:25:00.000000000 +0000">
>> comment.commenter
=> nil
>>
```

To write this out: `User.where({ :id => 96 })`. Then we can `.at(0)` and `.username` on that to get the name we want. At the REPL: `>> User.where({ :id => 96 }).at(0).username` should return `=> "jolie"`: 

```ruby
>> comment
=> #<Comment id: 2730, author_id: 96, body: "There was never a genius without a tincture of mad...", venue_id: 1, created_at: "2019-03-19 03:56:09.000000000 +0000", updated_at: "2019-10-08 10:25:00.000000000 +0000">
>> comment.commenter
=> nil
>> User.where({ :id => 96 })
=> #<ActiveRecord::Relation [#<User id: 96, username: "jolie", created_at: "2018-12-19 04:59:05.000000000 +0000", updated_at: "2019-10-08 10:25:00.000000000 +0000">]>
>> User.where({ :id => 96 }).at(0)
=> #<User id: 96, username: "jolie", created_at: "2018-12-19 04:59:05.000000000 +0000", updated_at: "2019-10-08 10:25:00.000000000 +0000">
>> User.where({ :id => 96 }).at(0).username
=> "jolie"
>>
```

So this does work. We can traverse manually to get the username, and `comment.commenter` is not failing and returning `nil` because there is no username. This is something you need to start to consider. There may be entries that *do* have `nil` values in a given column. Some bugs can be caused by invalid data and not problems in our code.

However, here we saw the bug was not caused by invalid data. That means we need to go look in the `Comment` model at the `.commenter` method:

```ruby
# app/models/comment.rb

# == Schema Information
#
# Table name: comments
#
#  id         :integer          not null, primary key
#  body       :string
#  created_at :datetime         not null
#  updated_at :datetime         not null
#  author_id  :integer
#  venue_id   :integer
#

# ...
  def commenter
    my_id = self.id

    matching_users = User.where({ :id => my_id })

    the_user = matching_users.at(0)
    
    return the_user
  end
# ...
```

Aha! We are taking the `self.id`! But what we want is the `author_id` foreign key column *not* the primary key of the given comment. So change the line to `my_id = self.author_id`, and refresh `/venues/1` once more in our browser. It works!

---
