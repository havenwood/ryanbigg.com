--- 
wordpress_id: 201
layout: post
title: Stop Being So Self-Centered!
wordpress_url: http://frozenplague.net/?p=201
---
I've seen so many examples of people being self-centered in their code, always calling their methods on <span class='term'>self</span> when they don't need to!

self is a handy reference to the current object (or class) depending on how the method is defined. In a class that inherits from ActiveRecord for example, calling self before defining a method will define that method on that class, rather than the objects of that class. Once inside of the method, you are free to go about as you will; Ruby will know what you mean when you call other methods inside that method.

A prime example would be this self-centered piece of code:

<pre lang='rails'>
class Forum < ActiveRecord::Base
  acts_as_tree
  
  def to_s
    self.title
  end
  
  def self.find_all_without_parent
    self.find_all_by_parent_id(nil)
  end
  
  def root
    self.parent.nil? ? self : self.parent
  end
  
end
</pre>

These are a few methods with both good and bad examples of using self. 

We'll start with <span class='term'>#to_s</span>. This method is defined by typing <span class='term'>def to_s</span>, which tells ruby we want to define a method called "to_s" on all objects of the Forum class. Inside the method however, we have a bad-case of self-centering, by calling self where it's not needed. Ruby already knows that we're operating with an object derived from the forum class, and calling self will only return the same object! So why do we do it here? We can remove the superfluous self call, and the method will still work.

Our next method is <span class='term'>self.find_all_without_parent</span>. The <span class='term'>self.</span> prefix to the method tells Ruby that we want to define this method on the Forum class itself, rather than a derived object from the class. Inside the method again we have a bad case of self-centering! Ruby already knows we're operating with the Forum class, and calling self will only return the Forum class once more. Again we can remove the superfluous self call and the method will still work.

In both of these examples, you could've called self on self and then the method multiple times: <span class='term'>self.self.title</span> or even <span class='term'>self.self.self.self.self.title</span>, but that's just going a bit overboard!

In the last method is two incorrect usages of self, and one correct use of self. The method is <span class='term'>#root</span>. The method defines a one-lined if statement by calling a method (<span class='term'>self.parent.nil?</span>) which will return either true or false. The next character is a space followed by a ?, which indicates to ruby anything after this is what we want executed when <span class='term'>self.parent.nil?</span> returns true. Here we just call <span class='term'>self</span>, and this is the good use of self. The <span class='term'>#root</span> method is attempting to find the root element for the tree heirachy of the forums, if the forum object we're calling root does not have a parent, we want it to return itself as it is the highest level of the forum structure. The next character in the code is a colon (:), which tells Ruby what we want to do if <span class='term'>self.parent.nil?</span> returns false. Here we've called <span class='term'>self.parent</span>, which again is a superflous call to the self object. We only need to call parent because Ruby already knows which context we are referring to! The final superfluous self call is the <span class='term'>self.parent.nil?</span> from where our one-lined if statement originated.

I hope you've enjoyed this little tidbit, and please stop being so self-centered.

UPDATE: <a href='http://luckysneaks.com/blog'>RSL</a> has posted an easier way than <span class='term'>parent.nil? ? self : parent</span> as the first comment. The code is now simply:

<pre lang='rails'>
def root
  parent || self
end
</pre>

UPDATE #2: The root method was still broken as I realised this morning as I tampered with the tests. The method is supposed to get the highest forum in a string of forums. This works perfectly for 2-level-deep forums, but not for 3-level-deep. The third-level element would've returned the second-level element rather than the first. The revised code is now:

<pre lang='rails'>
  def root
    parent.nil? ? self : (parent.root == parent ? parent : parent.root)
  end
</pre>
