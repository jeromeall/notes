# Blocks as awesomesauce

Collections of data are everywhere. 

Here we have a classroom of students. 

Outside we have bus routes with bus schedules.

At Amoeba Records, we have rows upon rows of music by various artists, that serve as a hipster safari for the music lovers whom peruse its treasures.

If you are a landlord, you want to manage a collection of apartment units inside your building.

We've been playing quite a bit with `Array` and `Hash`. Mastering these two extremely powerful ruby objects is essential to being an effective ruby programmer.

However, to understand how to implement `Array` and `Hash` like Jedis, we really have to ***grok*** how blocks work.

**Blocks, procs, and lambdas are ruby's most powerful and most misunderstood awesomesauce.**

## Goals

### Grok block semantics really well
### Grok how `Array.each` and other iterators work under the hood

## A deep look at `Array.each`

###Syntax and rules

```
a = [1, 2, 3, 4]

# syntax with braces
a.each { |a| puts a }

# syntax with do/end
a.each do |a|
	a *= 5
	puts a
end

# useless syntax without an argument
a.each { puts "kinda useless" }

```

* A block may be a set of ruby statements and expressions between braces or a do/end pair.
	* **Explain why is a hash literal not a block even though it looks a little like a block?**

* May start with an argument list between vertical bars.

* **It may appear only immediately after a method invocation.**

	* **You can't write a free floating block**
	
* **The optional block argument must be the last in the list when passing it to a function. Ruby checks for an associated block. If a block is present, it is converted to an object of class `Proc` and assigned to the block argument. If no block is present, the argument is set to `nil`.**

```
def example(&block)
	puts block.inspect
end

# call example and pass in a block
example { "a block" }

# call example and don't pass in a block
example

```

* Be **very careful** when not using parentheses and passing in blocks!
	* Order of operations!
		* Braces have high precedence
		* do/end has low precedence	

Let's use `Array.map` for this example:
	
```
# low precedence
[3] pry(main)> puts arr.map do |element|
[3] pry(main)*   element * 2
[3] pry(main)* end
#<Enumerator:0x007fa619a0e0a8>
=> nil

# high precedence
[4] pry(main)> puts arr.map { |element| element * 2 }
2
4
6
=> nil

# getting back high precedence, with parens
[8] pry(main)> puts(arr.map do |element|
[8] pry(main)*     element * 2
[8] pry(main)* end)
2
4
6
=> nil

```

#### You do exercise (5 minutes)
Given the hash:

```
musicians_with_genres = {
	id_osbourne: {
		first_name: "Ozzy",
		last_name: "Osbourne",
		genre: "Heavy metal"
	},
	id_morrison: {
		first_name: "Jim",
		last_name: "Morrison",
		genre: "Psychedelic rock"
	}
}
```

Write two examples with iterators other than `Array.map` and `Array.each` that demonstrate the problem and unintended consequences of low precedence vs. high precedence when using do/end instead of braces when passing blocks to the iterators.

### What's really happening when we use `Array.each`?

We've been having a lot of fun with iterators... ruby methods like `Array.each`, `Array.map`, `Hash.each_with_index`, and others. Let's dig deeper and understand how they work under the covers, by examining `Array.each`.

Let's take another look at one of the previous examples:

```
a = [1, 2, 3, 4]
for i in 0..a.size
	puts "#{a[i]}"
end
```

It works. What are some problems with this?

* Tight coupling between the iteration operation and the retrieval operation
* Having to use a meaningless index element
* Readability

If we had a more complicated array... for example a chessboard represented by a two-dimensional array, we'd now have to write a much more complicated loop!

If we had a hash of hashes, things would get ugly quickly. We'd have to possibly extract the keys of the top level hash, then the keys of the child hashes, etc. We'd have to keep track of them. Ugly, upon ugly, upon ugly.

Iterators to the rescue!

```
a = [1, 2, 3, 4]
a.each { |element| puts element }
```

* Much more beautiful
* It reads well
* The "looping" is decoupled from the retrieval
* The output is decoupled from the looping
	* Changing this to do something else is absolutely trivial
	
#### The method `Array.each` is an iterator

An iterator is a method that invokes a block of code repeatedly (one or more times).

Let's write our own iterator.

```
def fun_iterator &block
	yield
end

fun_iterator { puts "repeat this code" }
```

Let's add another `yield` to our iterator.

```
def fun_iterator &block
	yield
	yield
end

fun_iterator { puts "repeat this code" }
```

So what does `yield` do?

`yield` invokes the block passed into the method as the method's block argument, denoted by `&block`, much like you would invoke a method.

Let's add the ability to have the block take an argument.

```
def fun_iterator &block
	current_year = 2014
	yield current_year
end

fun_iterator { |year| puts "It's #{year}!" }
```

Of course, this isn't so interesting. Let's implement an iterator that is capable of outputting the last ten years.

``` 
def decade &block
	(Time.now.year - 10..Time.now.year).each { |year| yield year }
end

decade { |year| puts "The year is #{year}" }
```

#### To understand how Array.each works, let's implement it!

We will do it on our `SortableArray` class from yesterday.

Before we do so, let me in on a secret:

The only true built in ruby loop constructs are `while` and `until`.

`for..in` is actually syntactic sugar for the `each` method, if it exists on the class that's being used.

So, let's take a quick peek at that old example again:

```
a = [1, 2, 3, 4]
for i in 0..a.size
	puts "#{a[i]}"
end
```

This translates roughly to:
```
[1, 2, 3, 4].each do |value|
	puts value
end
```

In other words `for..in` is *not* a loop, it's an alias for `.each`.

Using one built in loop, a temporary variable, `Array.size`, `yield`, and no other iterators, your mission is to implement `SortableArray.each`. `SortableArray.each` needs to return the original array untouched.

```
class SortableArray < Array
	def each &block
		# counter for retrieving array elements
		i = 0
		# use a built in loop
		until i == self.size
			yield self[i]
			i += 1
		end
		self
	end
end
```