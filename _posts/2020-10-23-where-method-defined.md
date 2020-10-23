---
layout: post
title: Check where a method is defined in Ruby
---

I was working on a pull request for [Sinatra-ActiveRecord](https://github.com/sinatra-activerecord/sinatra-activerecord/pull/103) gem recently, and I was not sure where does the "set" method is defined, as it is not shown in the gem code.



![Set method](https://rubyyagi.s3.amazonaws.com/10-method-defined/set_method.png)





I want to know where the "set" method belongs to, and luckily Ruby has a "[method](https://ruby-doc.org/core-2.2.2/Method.html)" method for each object , which gives you information of the methods of an object.



## Method object

You can use "method" method on every class, for example : 

```ruby
class Human
  def greet
    "Howdy human"
  end
end

matz = Human.new

puts matz.method('greet')
# #<Method: Human#greet() ex.rb:2>
```

 



We can pass in the method name into the 'method' method, then it will return an method object for the name specified (which here is the 'greet' method).



You can execute the method object using 'call' method like this : 

```ruby
class Human
  def greet
    "Howdy human"
  end
end

matz = Human.new

puts matz.method('greet').call
# "Howdy human"
```

 





## Check the owner class of the method

We can check which class owns the method by using 'owner' like this :

```ruby
puts matz.method('greet').owner
# Human
```



For the "set" method I want to check earlier, 

```ruby
module ActiveRecordExtension
  def database_file=(path)
    path = File.join(root, path) if Pathname(path).relative? and root
    spec = YAML.load(ERB.new(File.read(path)).result) || {}

    set :database, spec
    puts method('set').owner
    #<Class:Sinatra::Base>
  end
end
```

 

turns out the "set" method was owned by Sinatra::Base class! Now I can go check the Sinatra gem source code on Github to find out where this 'set' method is defined, or... use the **source_location** method!



## Check where a method is defined

**source_location** will tell us the exact file name and line number that the method is defined.



From the previous example : 

```ruby
module ActiveRecordExtension
  def database_file=(path)
    path = File.join(root, path) if Pathname(path).relative? and root
    spec = YAML.load(ERB.new(File.read(path)).result) || {}

    set :database, spec
    puts method('set').source_location
    # /Users/soulchild/.rbenv/versions/2.7.1/lib/ruby/gems/2.7.0/gems/sinatra-2.1.0/lib/sinatra/base.rb , 1262
  end
end
```

 

As the 'set' method comes from the Sinatra gem, it will output the path to the base.rb file from the gem, and the line number.



If you are using a Mac, you can open Finder, press **Command + Shift + G**  and paste in the path above to navigate to it quickly, then you can open the file there.



Sure enough, the 'set' method is on line 1262 of base.rb file from the Sinatra gem, thanks **source_location**!

![set source code](https://rubyyagi.s3.amazonaws.com/10-method-defined/set_source.png)





I find **source_location** and **owner** methods very handy when I want to trace where a method is defined, especially when working in a large codebase, hope this is useful for you too.


<script async data-uid="d862c2871b" src="https://rubyyagi.ck.page/d862c2871b/index.js"></script>







