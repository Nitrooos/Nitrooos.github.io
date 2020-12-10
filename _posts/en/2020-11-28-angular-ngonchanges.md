---
title: Angular's change detection fails
date: 2020-11-28 20:00:00.000000000 +02:00
identifier: angular-ngonchanges
categories:
- en
- frontend
tags:
- angular
- javascript
excerpt:
  Why sometimes Angular's change detection doesn't work as expected? How it's
  implemented? See the answer in this post!
---
Recently I met a situation when value change of @Input() property wasn't
catched in the **ngOnChanges** lifecycle hook of Angular component. That's because,
the *value* has changed, but the *reference* not. And that's how Angular
detects changes - by using strict equality comparator (===) and thus comparing
the references, not values. How did it "break" the detection mechanism?

## The example

Let's say we have a @Input() property "items" defined in a component as an
array of objects of type Items. Also, we try to detect changes of this property
in ngOnChanges lifecycle hook:

{% highlight typescript %}
@Input() items: Items[];

ngOnChanges(changes: SimpleChanges) {
  if (changes.items) {
    console.log("'items' property has changed!");
  }
}
{% endhighlight %}

And this code didn't detect any changes in the "items" array, regardless of the
fact I clearly saw the values inside it (and even the length of array!) changed 
many times - I logged "items" on some click events, after finishing some async 
tasks etc.

The problem was noticed in the upper-level component, in the way how the
elements were added/removed/modified in the "items" array. Let's see:

{% highlight typescript %}
const newItem = { ...some definition };
items.push(newItem);
{% endhighlight %}

In this example, the "items" is still the same array, meaning the same 
reference, so modyfing it in any way **won't trigger change detection**. The 
same effect could be observed by using any mutating method of *Array*, like:

{% highlight typescript %}
.fill()
.pop()
.reverse()
.shift()
.splice()
.sort()
.unshift()
{% endhighlight %}

The same effect occurs also when we're dealing with objects - adding/removing
keys or modifying the values of keys doesn't change the object's reference!

## Solution

Two possible solutions exist:

* creating a new object each time you want to modify it
* (not recommended) implementing a custom change detection in **ngDoCheck** hook

I used the first option, so rewrote all places where "items" were mutated. How?

### Create each time a new object

{% highlight typescript %}
// items.push(newItem)
items = items.concat([newItem])
items = [...items, newItem]

// items.pop()
items = items.slice(0, items.length - 1)

// items.splice(5, 10)
items = items.filter((_, index) => index < 5 || index >= 5 + 10)
{% endhighlight %}

or used some tricky methods forcing JS to create a shallow copy (it was 
sufficient in my case) of array like below:

{% highlight typescript %}
// they are equivalent
const copyArray = array => [...array]
const copyArray2 = array => array.slice()
{% endhighlight %}

and then ensuring a copy is created when we sort or reverse an array:

{% highlight typescript %}
items = copyArray(items.reverse())
items = copyArray2(items.sort())
{% endhighlight %}

### Custom change detection in ngDoCheck

An alternative approach is to define custom change detection mechanism, best in
the **ngDoCheck** hook, however it's not recommended. Why? This hook (if 
defined) is executed very often and can impact the performance of the 
application. If you ever really want to use it, then make sure the code in
**ngDoCheck** is very lightweight.

That's how the solution to the problem would look like using **ngDoCheck**:

{% highlight typescript %}
oldItems: Items[];

ngDoCheck() {
  let changeDetected = false;
  if (this.items.length !== this.oldItems.length) {
    changeDetected = true;
  } else {
    changeDetected = this.items.some((item, index) => item !== this.oldItems[index]);
  }

  if (changeDetected) {
    console.log("'items' property has changed!");
    this.oldItems = copyArray(this.items);
  }
}
{% endhighlight %}

Please notice it requires us to store the "old" value of checked variable -
we need to remember it in order to make any manual checks (*this.oldItems* in
the code above). In the example, I implemented a simple "shallow" check, which
detects change when the number of elements in array differs or any of the 
elements hasn't the same reference as before the check. And even there, I had
to use function (*copyArray*) to make a copy of array to have the current and 
old values indepedently stored in memory.

As always, I forward you to official 
[documentation](https://angular.io/guide/lifecycle-hooks#defining-custom-change-detection){:target="_blank"}
to see more examples and detailed explanation of this hook.

## Summary

We need to remember about how Angular detects changes - it uses the simplest,
reference strict equality check, which has sometimes unexpected consequences.
But having them in mind, we can simply deal with such cases, like detecting
changes in mutated arrays or objects.
