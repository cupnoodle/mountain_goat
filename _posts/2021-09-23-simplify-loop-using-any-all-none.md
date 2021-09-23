---
layout: post
title: Simplify loop using any?, all? and none?
date: 2021-09-23 19:01 +0800
---


Ruby has some useful methods for enumerable (Array, Hash, etc),  this post will talk about usage of [any?](https://ruby-doc.org/core-3.0.2/Enumerable.html#method-i-any-3F), [all?](https://ruby-doc.org/core-3.0.2/Enumerable.html#method-i-all-3F), and [none?](https://ruby-doc.org/core-3.0.2/Enumerable.html#method-i-none-3F).



For examples used on this post, an **Order** model might have many **Payment**  model (customer can pay order using split payments).



## any?

To check if an order has any paid payments, a loop-based implementation might look like this :

```ruby
has_paid_payment = false

order.payments.each do |payment|
  if payment.status == "paid"
    # one of the payment is paid
    has_paid_payment = true
    break
  end
end

if has_paid_payment
  # ...
end
```



We can simplify the code above using **any?** like this :

```ruby
if order.payments.any?{ |payment| payment.status == 'paid'}
  # this will be executed if there is at least one paid payment
end
```



**any?** method can take in a block, and it will return true if the block ever returns a value that is not false or nil. (ie. true, or 123, or "abc")



If no block is supplied, it will perform a self check for the elements:

```ruby
order.payments.any?

# is equivalent to
order.payments.any? { |payment| payment }
```

<hr>


## all?

To check if all payments has been paid for an order, a loop-based implementation might look like this: 

```ruby
fully_paid = true

order.payments.each do |payment|
  if payment.status != "paid"
    # one of the payment is not paid, hence not fully paid
    fully_paid = false
    break
  end
end

if fully_paid
  # ...
end
```



We can simplify the code above using **all?** like this :

```ruby
if order.payments.all?{ |payment| payment.status == 'paid'}
  # this will be executed if all payments are paid
end
```



**all?** method can take in a block, and it will return true if the block **never** returns a value that is false or nil for all of the elements. 



If no block is supplied, it will perform a self check for the elements:

```ruby
order.payments.all?

# is equivalent to
order.payments.all? { |payment| payment }
```

<hr>

## none?

This is the opposite of **all?**. 

**none?** method accepts a block, and only returns true if the block **never returns true** for all elements.



```ruby
if order.payments.none? { |payment| payment.status == 'paid' }
  # this will be executed if there is no paid payment for the order
end
```



If no block is supplied, it will perform a self check for the elements:

```ruby
order.payments.none?

# is equivalent to
order.payments.none? { |payment| payment }
```


By utilizing any?, all? and none?, we can remove the usage of loop and make it more readable.


# Reference

[Ruby doc on enumeration](https://ruby-doc.org/core-3.0.2/Enumerable.html)


<script async data-uid="4776ba93ea" src="https://rubyyagi.ck.page/4776ba93ea/index.js"></script>

