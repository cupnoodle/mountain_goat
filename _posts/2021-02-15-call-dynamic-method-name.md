---
layout: post
title: How to call methods dynamically using string of method name
date: 2021-02-15 22:40 +0800
---

I was working on a feature for my Rails app, which allows merchant to set fulfillment times on each day like this :

![fulfillment_time](https://rubyyagi.s3.amazonaws.com/20-call-dynamic-method-name/fulfillment_time.png)



The main feature is that once an order is placed, the customer can't cancel the placed order after the fulfillment time starts on that day (or on tomorrow).



The fulfillment start times are saved in different columns like this : 

```ruby
create_table "fulfillments", force: :cascade do |t|
  t.time "monday_start_time", default: "2000-01-01 00:00:00", null: false
  t.time "tuesday_start_time", default: "2000-01-01 00:00:00", null: false
  t.time "wednesday_start_time", default: "2000-01-01 00:00:00", null: false
  t.time "thursday_start_time", default: "2000-01-01 00:00:00", null: false
  t.time "friday_start_time", default: "2000-01-01 00:00:00", null: false
  t.time "saturday_start_time", default: "2000-01-01 00:00:00", null: false
  t.time "sunday_start_time", default: "2000-01-01 00:00:00", null: false
end
```



To check if the fulfillment time has started, my app will get the current day of week using `DateTime.current.strftime('%A').downcase` . If today is wednesday, then it will return `wednesday` .



The question now is that **how do I access value of different column based on different day of week**? Say I want to get the value of `thursday_start_time` if today is **thursday** ; or the value of `sunday_start_time` is today is **sunday** .



Fortunately, Ruby's metaprogramming feature allows us to call methods dynamically by just passing the method name into **public_send(method_name)**  or **send(method_name)** .



Say we have a class like this : 

```ruby
class Duck
  def make_noise
    puts "quack"
  end
  
  private

  def jump
    puts "can't jump"
  end
end
```



We can call the **make_noise** method by calling `Duck.new.public_send("make_noise")`,  this is equivalent to calling Duck.new.make_noise .



For the **jump** method, we would need to use **send** instead, as it is a private method, `Duck.new.send("jump")`



For the fulfillment start time, we can then pass in the day dynamically like this :

```ruby
# this will return 'monday' if today is monday, or 'tuesday' if today is tuesday etc..
day_of_week = DateTime.current.strftime('%A').downcase

# if today is monday, it will call 'monday_start_time', etc..
start_time = fulfillment.public_send("#{day_of_week}_start_time")
```


Sweet!



Alternatively, you can also pass in symbol instead of string : **public_send(:make_noise)** .



## Passing parameters to public_send / send

If you would like to send parameters to the method, you can pass them after the method name :

```ruby
class Animal
  def make_noise(noise, times)
    1..times.each do 
      puts noise
    end
  end
end

Animal.new.public_send('make_noise', 'quack', 3)
# quack
# quack
# quack
```

<script async data-uid="d862c2871b" src="https://rubyyagi.ck.page/d862c2871b/index.js"></script>


## Reference

Ruby Doc's [public_send documentation](https://ruby-doc.org/core-3.0.0/Object.html#method-i-public_send)

Ruby Doc's [send documentation](https://ruby-doc.org/core-3.0.0/Object.html#method-i-send)

