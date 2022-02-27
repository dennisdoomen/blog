---

title: "Comparing objects with disparate members in Fluent Assertions"
excerpt: "Some examples on how to use the new WithMapping API to compare objects with differently named properties or fields"

tags:
- fluent-assertions
---

One of the most powerful features of Fluent Assertions is its ability to do a deep comparison of two object graphs. There are tons of options to tell the library how to do that, but out of the box, it is smart enough to understand anonymous types, records and how to compare collections. But up to now, it did not know how to compare two objects with misnamed properties or fields. At least, not until version 6.5. 

Imagine you want to assert that an `OrderDto` is equivalent to an `Order` entity using BeEquivalentTo. This is a pretty common thing to do in FA. But what if the first has an `OrderName` property and the second a `Name` property? Assuming that those two properties are supposed to contain the same value, you can now use the new `WithMapping` option to tell FA that anytime it finds the `Name` property on the expectation, it should compare it to the `OrderName` property of the subject-under-test. 

	// Using names with the expectation member name first. Then the subject's member name.
	orderDto.Should().BeEquivalentTo(order, options => options
	    .WithMapping("Name", "OrderName"));
	
	// Using expressions, but again, with expectation first, subject last.
	orderDto.Should().BeEquivalentTo(order, options => options
	    .WithMapping<OrderDto>(e => e.Name, s => s.OrderName));
	
Another option is to map two deeply nested members to each other. In that case, your path must start at the root:
	
	// Using dotted property paths 
	rootSubject.Should().BeEquivalentTo(rootExpectation, options => options
	    .WithMapping("Parent.Collection[].Member", "Parent.Collection[].Member"));
	
	// Using expressions
	rootSubject.Should().BeEquivalentTo(rootExpectation, options => options
	    .WithMapping<SubjectType>(e => e.Parent.Collection[0].Member, s => s.Parent.Collection[0].Member));

Note that specific collection indices in string-based paths are not allowed, hence the generic `[]` notation. Within expressions however, you must use an index to make it a valid property path, but it’ll be ignored. So both the examples will end up doing the same. Also, such nested paths must have the same parent. So mapping properties or fields at different levels is not (yet) supported.

In the previous example, the `Order` and `OrderDto` were the root types themselves. Now what if those types appear somewhere as a nested type in an object graph? Then you can use a set of different overloads.

	// Using names
	orderDto.Should().BeEquivalentTo(order, options => options
	    .WithMapping<Order, OrderDto>("Name", "OrderName"));
	
	// Using expressions
	orderDto.Should().BeEquivalentTo(order, options => options
	    .WithMapping<Order, OrderDto>(e => e.Name, s => s.OrderName));

And note that all of the above also works with fields or a combination of a field and a property.

So what do you think? Is this a feature that will make your life easier? Let me know by commenting below. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.

