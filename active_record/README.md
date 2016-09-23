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

####users
| id | name | email |
|----|------|-------|
| 1 | Fred | fred@somedomain.com |
| 2 | Mary | mary@somedomain.com |
| 3 | Jamie | jamie@somedomain.com |

####profiles
| id | user_id | avatar | mobile | suburb | state | country |
|----|---------|--------|--------|------|------|---------|
| 1 | 1 | http://... | 0141 111 111 | Brunswick | Lower Saxony | Germany |
| 2 | 2 | http://... | 0142 222 222 | Sydney | NSW | Australia |
| 3 | 3 | http://... | 0143 333 333 | Campo Belo | Sao Paulo | Brazil |

###Eager loading
Avoiding N+1 using includes
`@users = User.includes :profile`
Above code will eager load the profile association so that:
`@users.first.profile`
will not execute another SQL query

###Joins
When executing a where clause based on the association, user inner join

```User.joins(:profile).where(profiles: {pcode: 2000})```

| id | user_id | avatar | mobile | suburb | state | country |
|----|---------|--------|--------|------|------|---------|
| 1 | 1 | http://... | 0141 111 111 | Brunswick | Lower Saxony | Germany |
|*2* | *2* | *http://...* | *0142 222 222* | *Sydney* | *NSW* | *Australia* |
| 3 | 3 | http://... | 0143 333 333 | Campo Belo | Sao Paulo | Brazil |

Returns all users who are in the '2000' post code
Note profiles is plural, as this is a reference to the table, not the
association.

