### Goldiloader
---
https://github.com/salsify/goldiloader

```sh
blogs = Blogs.limit(5).to_a
# SELECT * FROM blogs LIMIT 5
blogs.each { |blog| blog.posts.to_a }
SELECT * FROM posts WHERE blog_id = 1
SELECT * FROM posts WHERE blog_id = 2
SELECT * FROM posts WHERE blog_id = 3
SELECT * FROM posts WHERE blog_id = 4
SELECT * FROM posts WHERE blog_id = 5

blogs = Blogs.limit(5).to_a
# SELECT * FROM blogs LIMIT 5
blogs.each { |blog| blog.posts.to_a }
# SELEC * FROM posts WHERE blog_id IN (1,2,3,4,5)

gem 'goldiloader'
bundle
gem install goldiloader
```
```ruby
blogs = Blogs.limit(5).to_a
# SELECT * FROM blogs LIMIT 5
blogs.each do |blog|
  if blog.post.exists?
    puts blog.posts
  else
    pust 'No posts'
  end
end
# SELECT 1 AS one FROM posts WHERE blog_id = 1 LIMIT 1
# SELECT * FROM posts WHERE blog_id IN (1,2,3,4,5)

class Blog < ActiveRecord::Base
  has_many :posts, fully_load: true
end

class Blog < ActiveRecord::Base
  has_many :posts
  has_one :most_recent_post, -> { order(published_at: desc) }, class_name: 'Post'
end
```
```sh
SELECT * FROM posts WHERE blog_id = 1 ORDER BY published_at DESC LIMIT 1
SELECT * FROM posts WHERE blog_id IN(1,2,3,4,5) ORDER BY published_at DESC
```

```ruby
class Blog < ActiveRecord::Base
  has_many :posts
  has_one :most_recent_post, -> { order(published_at: desc) }, class_name: 'Post'
  has_many :recent_post, -> { order(published_at: desc).limit(5) }, class_name: 'Post'
end
```
```sql
CREATE VIEW most_recent_post_references AS
SELECT blogs.id AS blog_id, p.id as post_id
FROM blogs, LATERAL (
  SELECT posts.id
  FROM posts
  WHERE posts.blog_id = blogs.id
  ORDER BY published_at DESC
  LIMIT 1
) p
CREATE VIEW recent_post_references AS
SELECT blogs.id AS blog_id, p.id as post_id, p.published_at AS post_published_at
FROM blogs, LATERAL (
  SELECT posts.id, posts.published_at
  FROM posts
  WHERE posts.blog_id = blogs.id
  ORDER BY published_at DESC
  LIMIT 5
) p
```
```ruby
class Blog < ActiveRecord::Base
  has_many :posts
  has_one :most_recent_post_reference
  has_one :most_recent_post, throught: :most_recent_post_reference, source: :post
  has_many :recent_post_references, -> { order(post_published_at: desc) }
  has_many :recent_posts, throught: :recent_post_reference, source: :post
end
class MostRecentPostReference < ActiveRecord::Base
  belongs_to :post
  belongs_to :blog
end
class RecentPostReference < ActiveRecord::Base
  belongs_to :post
  belogns_to :blog
end

class Blog < ActiveRecord::Base
  has_many :posts, auto_include: false
  has_many :posts, -> { autoinclude(false) }
end






```


```ruby
class Blog < ActiveRecord::Base
  has_many :posts
end
class Post < ActiveRecord::Base
  belongs_to :blog
end

Blog.order(:name).auto_include(false)

class Blog < ActiveRecord::Base
  has_many :posts, -> { auto_include(false) }
end



```

