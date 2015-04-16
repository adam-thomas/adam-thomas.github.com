---
layout: main
title: Filters, inner joins, distinct(), and you
---


### Filtering a QuerySet can increase its size

```
# Create our user, John Doe
user = factories.UserFactory.create(name='John Doe'))

# Create a discussion thread, and have John Doe post two comments to it
discussion = factories.DiscussionFactory.create()
factories.TextCommentFactory.create_batch(
    2,
    user=user,
    discussion=discussion,
)

# We have one user
print(User.objects.all())
[<User: John Doe>]

# And we can show that he's posted comments on our particular discussion
print(User.objects.filter(comments__discussion_id=discussion.pk))
[<User: John Doe>, <User: John Doe>]

# ...what?
```

This bug can be fixed by using `.distinct()` - as in `User.objects.filter(comments__discussion_id=discussion.pk).distinct()` - to remove the duplicate results, but understanding when and why to use and test `distinct()` is more difficult.

The ins and outs of how Django's `QuerySet`s work under the hood are complicated, and one area where they can behave strangely is when using `filter` commands on foreign-key relationships. As the example shows, it's actually possible to get _more_ items back from a queryset by filtering it, an operation that you'd think was only ever meant to remove elements.

For the uninitiated, a queryset's [`filter(**conditions)`](https://docs.djangoproject.com/en/1.8/ref/models/querysets/#filter) method accepts any number of conditions expressed as keyword arguments - such as the `comments__discussion_id=discussion.pk` above. The method returns another `QuerySet` which represents all the items from the queryset you were filtering that match the condition, and no other.

The syntax for conditions is [a bit complicated](https://docs.djangoproject.com/en/1.8/ref/models/querysets/#id4), so I won't go into it in depth here - the Django docs do a good job of that. The condition in the example above breaks down as:

```
comments
> a member of a User object called 'comments' (which in this case
> happens to be a class with a foreign key to User, and 'comments'
> is a 'ManyRelatedManager')

comments__discussion
> the 'discussion' object foreign-keyed to by any item in 'comments';
> the double-underscore '__' represents following that foreign key

comments__discussion_id
> the 'id' member of any 'discussion' foreign-keyed to by any item
> in 'comments'

comments_discussion_id=discussion.pk
> "the 'id' member of any 'discussion' foreign-keyed to by any item
> in a User's 'comments' must match the 'pk' of this 'discussion'
> object" - intuitively, "this user has posted to this discussion"
```

### How does `filter` work?

When a `QuerySet` is evaluated, such as when you try to list its contents, Django creates an SQL query and ships that off to the underlying database. The database runs the query and outputs the result, which will usually be a table of objects of a certain type. Django turns that into a list of the Python-friendly models we know and love (such as `User` in the example above) and we can perform operations on them.

`filter` creates what's called a `SELECT WHERE` statement. The `SELECT` is a standard data retrieval command in SQL - it asks for a number of fields from a table, and returns a table that has a row for each row in the source table and displays the chosen fields. (Django typically asks for all the fields from the model it's after.) `WHERE` staples a condition onto that, causing the `SELECT` to ignore any table rows that fail the condition. So far so good.

Foreign keys make it complicated. When you want to condition on properties of related models, SQL can't just start iterating over tables and tables and tables - that would be hideously inefficient. Instead, it combines the table you're `SELECT`ing `FROM` with the related tables using an operation called a join, typically an `INNER JOIN`. `INNER JOIN` returns a table where each row contains the fields from the first table and the fields from the table you're joining onto it, provided they match a condition. It's a table representing all the ways you can get from the first table to the second via relations (foreign keys in this case).

This is where it gets a bit complicated. An `INNER JOIN` might bolt each row from the source table onto each row from the destination table, if the relationships are dense enough. It's unlikely to do that, but it can still do more than you might think.

Imagine that this is our database from the running example, leaving out most of the fields:

```
 Table [User]        Table [Comment]      Table [Discussion]
id  name            id  body                  id  name
01  "John Doe"      01  "Hello world"         01  "Chat"
                    02  "How are you?"
```

Each `Comment` has a foreign key to the `User` who wrote it and the `Discussion` it belongs to.  The relations might look like this:

```
 Table [User]                Table [Comment]            Table [Discussion]
                belongs to                     belongs to
01  "John Doe" <----+------ 01  "Hello world" ------+-----> 01  "Chat"
                    |                               |
                    +------ 02  "How are you?" -----+
```

If you performed an `INNER JOIN` operation on `User` and `Comment`, the result would look like this:

```
Table [User JOIN Comment]
id  name         id  body
01  "John Doe"   01  "Hello world"
01  "John Doe"   02  "How are you?"
```

This table represents all the ways you can get from an item in `User` to an item in `Comment` by traversing relationships between tables.

When you `JOIN` on the `Discussion`s, the table becomes something like:

```
Table [User JOIN Comment JOIN Discussion]
id  name         id  body            id  name
01  "John Doe"   01  "Hello world"   01  "Chat"
01  "John Doe"   02  "How are you?"  01  "Chat"
```

This table represents all the ways you can get from a `User` to a `Discussion` by traversing relationships, which is necessary information for the filter query.

(Normally, SQL disambiguates fields by prepending the name of the table they appear in when it stores them - so the `User` table has fields `user_id`, `user_name`, etc - but I've left those off in the examples for the sake of brevity. Either way, don't worry about field name clashes in `JOIN`s, because SQL has that covered.)

The condition we're using wants the discussion `id` to equal `1`, which both of those rows match. SQL scoops out the user-related parts (because those are the fields Django has asked for) and returns them as the result of the query.

This means we get back:

```
id  name
01  "John Doe"
01  "John Doe"
```


### To `distinct()` or not to `distinct()`?

It seems that the cause of our unwanted duplicate results is having duplicates in the `JOIN`ed tables, which in turn is caused by the existence of multiple ways to get from the queryset you're filtering to the related object that's being conditioned on.

For instance, in the example above, there are two paths from `User` to `Discussion`:

```
 Table [User]                Table [Comment]            Table [Discussion]
01  "John Doe" -----+-----> 01  "Hello world" ------+-----> 01  "Chat"
                    |                               |
                    +-----> 02  "How are you?" -----+
```

There are two paths from `User` to `Comment` as well, so a query like `User.objects.filter(comments__id__in=[1,2])` will also produce duplicate results:

```
 Table [User]                Table [Comment]
01  "John Doe" -----+-----> 01  "Hello world"
                    |
                    +-----> 02  "How are you?"
```

Both of the comments have an `id` in `[1, 2]` and will be included in the `INNER JOIN`. This means that that query needs a `distinct()`.

On the other hand, if you're querying all the comments in the first discussion:

```
 Table [Comment]            Table [Discussion]
01  "Hello world" ------+-----> 01  "Chat"
                        |
02  "How are you?" -----+
```

The `INNER JOIN` produces two rows, as with the `User` queries, but the model whose queryset you're filtering - `Comment` - has two distinct values in those rows. There's no need for a `distinct()`, because there's only one way of getting from either `Comment` to that particular discussion.


### More examples

For more worked examples, let's look at some real code. This is a (trimmed) snippet from `managers.py` in `incuna-groups`:

```
class GroupQuerySet(models.QuerySet):
    """A queryset for Groups that can retrieve related objects."""
    def discussions(self):
        """All the discussions on these groups."""
        return Discussion.objects.filter(group__in=self)

    def comments(self):
        """All the comments posted in these groups."""
        return BaseComment.objects.filter(discussion__group__in=self)

    def users(self):
        """All the users who have ever posted in these groups."""
        return User.objects.filter(comments__in=self.comments()).distinct()
```

The overall structure of the models is that each `Group` has a number of `Discussion`s, and each `Discussion` contains one or more `BaseComment`s. Each `BaseComment` is posted by a `User`. The foreign keys look like this:

```
User <---- BaseComment ----> Discussion ----> Group
```

That is, a `BaseComment` has a foreign key to both a `User` and a `Discussion`, and a `Discussion` has a key to a `Group`.

Let's look at that snippet again, method by method, and go over why each queryset filter does and doesn't use `distinct()`.

```
def discussions(self):
    """All the discussions on these groups."""
    return Discussion.objects.filter(group__in=self)
```

This filter checks whether a `Discussion`'s `Group` is part of the queryset of `Group`s being filtered. There's only one way to get from a `Discussion` to a member of the `Group` queryset, because each `Discussion` only has one `Group`, so the filter condition can't match the same pair together in multiple ways. This means we don't need a `distinct()`.

```
def comments(self):
    """All the comments posted in these groups."""
    return BaseComment.objects.filter(discussion__group__in=self)
```

This one checks whether a `BaseComment` was posted to a `Discussion` that was part of a `Group` in the current queryset. It's essentially an extension of the previous method. It's more complicated, but there's still only one way to get from any `BaseComment` to the queryset of `Group`s; the comment only has one `Discussion`, and the `Discussion` only has one `Group` as before. This means we still don't need a `distinct()`.

```
def users(self):
    """All the users who have ever posted in these groups."""
    return User.objects.filter(comments__in=self.comments()).distinct()
```

Ah, here it gets more awkward. This time we're checking that a `User` has posted a `BaseComment` that's within the queryset of comments we generated using the previous method. The catch here is that `BaseComment` has a foreign key to `User` and not the other way around, meaning we're following a one-to-many relationship for the first time. Django is quite happy to do this for us, but it requires a `distinct()`.

We know that `self.comments()` doesn't generate duplicates, as we've already examined that method. However, `comments__in=self.comments()` can definitely be satisfied in multiple ways for a single `User`. Since a `User` could have any number of comments, we can deduce that it could have any number of comments _in the queryset returned by `self.comments()`_. This means the `INNER JOIN` table can contain multiple rows for the same `User`, and that in turn means that the filter can produce duplicate `User` entries in the resulting queryset. This means we need a `distinct()`.


### Testing distinctness

We can also use this knowledge to write tests. Verifying that your returned queryset doesn't contain duplicate entries is a good idea, just in case somebody accidentally removes the `distinct()`.

Creating a good test case for duplication involves making some objects in the database that will cause the filter condition to be satisfied in multiple ways. In the running example I'm using, that involves a `User` with two `BaseComment`s:

```
def test_users_distinct(self):
    """Assert that Group.objects.users() contains no duplicates."""
    user = factories.UserFactory.create()
    factories.TextCommentFactory.create_batch(2, user=user)

    self.assertCountEqual([user], models.Group.objects.users())
```

This test passes with the `distinct()` in place, and fails without.


### TL;DR

* SQL is complicated.
* A `filter(...)` on a queryset can produce duplicate results if there are multiple ways for the same item (in the queryset being filtered) to satisfy its conditions.
* To avoid this, add `.distinct()` after your filter.
* Testing your filter to ensure it has no duplicates involves finding some test data that'll cause the condition to pass in multiple ways.
