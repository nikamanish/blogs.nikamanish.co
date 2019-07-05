---
layout: post
current: post
cover:  assets/images/posts/group-by-vs-index-by.jpg
navigation: True
title: Group by vs Index by
date: 2019-07-04 20:00:00
tags: [rails]
category: [rails]
class: post-template
subclass: 'post'
author: manish
---

Ruby on Rails provides us with a lot of `Array` methods. So many, that it is
sometimes confusing, as to what to use when.

One such confusing duo is `group_by` & `index_by`.

### Group by

It is useful when you want to group similar data together. Be it an `array` of
hashes or an `ActiveRecord` list.

Let's take a example.

We have a `Post` model and we want to fetch all of them, grouped by their
`category`. Something like this.

{% highlight ruby %}
{
  'rails' => [
    { id: 1, name: 'Group by vs Index by', category: 'rails' },
    { id: 3, name: 'Some awesome rails post', category: 'rails' }
  ],
  'react_js' => [
    { id: 2, name: 'Another awesome post, but for react', category: 'react_js' }
  ]
}
{% endhighlight %}

The traditional method would to use a `loop` and an empty `hash` to push the data into.

{% highlight ruby %}
def posts_by_category
  result = {}
  posts = Post.joins(:category).select('posts.*, categories.name as category')

  posts.each do |post|
    result[post.category] ||= []
    result[post.category] << post
  end

  result
end

{% endhighlight %}

`group_by` lets you do this in a single line.

{% highlight ruby %}
def posts_by_category
  posts = Post.joins(:category).select('posts.*, categories.name as category')

  posts.group_by(&:category)
end
{% endhighlight %}

### Index by

Now, if you want to convert the above list of posts into a hash, you can use
`index_by`

{% highlight ruby %}
def posts_by_id
  Post.all.index_by(&:id)
end
{% endhighlight %}

And you'll get the following result

{% highlight ruby %}
{
  1 => { id: 1, name: 'Group by vs Index by' },
  2 => { id: 2, name: 'Another awesome post, but for react' },
  3 => { id: 3, name: 'Some awesome rails post' }
}
{% endhighlight %}

### Conclusion

So, the main difference between `group_by` and `index_by` is that the former
groups the data together in an array, where the key-value relation is `one to many`.<br/>
A `category` can have many `posts`.

Whereas the latter provides you with a hash, where the key-value relation is `one to one`.<br/>
An `id` can belong to only one `post`

