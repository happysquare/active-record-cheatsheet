Active Record querying
======================

Associations
------------

##has_one associations

```
class User < ActiveRecord::Base
  has_one :profile
end
```
```
class Profile < ActiveRecord::Base
  belongs_to :user
end
```

###Eager loading
Avoiding N+1 using includes
`@users = User.includes :profile`
Above code will eager load the profile association so that:
`@users.first.profile`
will not execute another SQL query

###Joins
When executing a where clause based on the association, user inner join

```User.joins(:profile).where(profiles: {pcode: 2000})```

Returns all users who are in the '2000' post code
Note profiles is plural, as this is a reference to the table, no the
association.

