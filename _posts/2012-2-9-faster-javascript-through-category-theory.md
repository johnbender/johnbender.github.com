# Faster Javascript Through Category Theory

This post started out as a [gist](https://gist.github.com/7242a7434195565b4a9d) that I was using to work through what I've learned about category theory by applying it to something I already know: JavaScript. The surprising result was a clearly defined set of JavaScript functions and jQuery helpers that could be optimized to reduce execution time.

In the course of this post we'll define two categories: one for HTMLElements and the other for jQuery objects. We'll then construct a Functor that maps from the category of HTMLElements to the category of jQuery objects. At the end we'll see how jQuery plugin authors can help user's speed up their JavaScript.

### Asumptions/Requirements

Some things to know before getting started:

1. `this` is treated as an implicit parameter to jQuery helper functions, later referred to as **Jqry** morphisms.
2. Type guarantees are made by the closure compiler JSDoc type annotations and custom `@sig` type signature annotation. Don't worry if this is new to you -- it only adds a bit more rigor to the discussion.
3. My examples do not constitute proofs but I'm fairly confident that both categories and the functor satisfy their respective laws.<sup>1</sup>

### Categories

If you're perfectly happy taking the category definitions at face value, feel free to skip ahead to the next section.

A category consists of two things, a set of objects and a set of morphisms sometimes referred to as arrows that relate two of those objects. In layman's terms, a category is a consistent set of things (objects) and operations on those things (morphisms/arrows). The abstract nature of both is exactly what gives categories and category theory power, because you can define a wide range of systems in terms of a categories. A few examples, including the two defined later in the post, will help to build an intuition for the forms that objects and morphisms can take.

1. [Hask](http://en.wikibooks.org/wiki/Haskell/Category_theory#Hask.2C_the_Haskell_category) is the category where Haskell types are the objects and functions are the morphisms.
2. [Set](http://en.wikipedia.org/wiki/Category_of_sets) is the category where objects are sets and the morphism are, again, functions between those sets.

Additionally each and every category must satisfy three laws.

1. Identity: The morphisms in a category must include an identity morphism. In terms of the category Set or Hask and those defined here, the identity morphism is just a function that return it's first and only argument.
2. Composition: The composition of morphisms, which is itself a requirement, must be associative. For Hask, Set, and the following categories this is simply function composition (eg `.` for Hask) and we'll explicitly define a composition function for **Html** and **Jqry**. Function application is always associative<sup>2</sup> so for Hask, Set and the categories to follow this can be assumed.
3. Closed Under Composition: When composing two morphisms in a category the result must also be in the set of morphisms in the category. For Hask this means that the output of composing two functions must be another function that takes a Haskell type and returns a Haskell type which is the criteria for the function being in the set of Hask's morphisms.

These laws are important for understanding the conclusions drawn from the rest of the post and we'll cover how both the **Html** and **Jqry** categories satisfies each.

### Html

The Category **Html** is extremely simple and, if you're still interested in understanding categories more generally, it will help in building an intuition for them. The objects in **Html** are the HTMLElements that you're familiar with from JavaScript (eg, HTMLDivElement, HTMLAnchorElement). The morphisms are JavaScript functions that opperate on those objects, and _only_ those objects.

The next step is to make sure that **Html** satisfies the three category laws:

Identity:

<pre>
<span class="doc">/** @sig {HTMLElement} -&gt; {HTMLElement} */</span>
<span class="keyword">function</span> <span class="function-name">id</span>( <span class="js2-function-param">a</span> ) {
  <span class="keyword">return</span> a;
}
</pre>


Composition:

<pre>
<span class="doc">/** @typedef {function(HTMElement): HTMElement} */</span>
html.morphism;

<span class="doc">/** @sig {html.morphism}, {html.morphism} -&gt; {html.morphism} */</span>
<span class="keyword">function</span> <span class="function-name">compose</span>( <span class="js2-function-param">f</span>, <span class="js2-function-param">g</span> ){
  <span class="keyword">return</span> <span class="keyword">function</span>( <span class="js2-function-param">a</span> ) {
    <span class="keyword">return</span> f(g(a));
  };
}
</pre>

Sample use of `compose`:

<pre>
<span class="keyword">function</span> <span class="function-name">a</span>( <span class="js2-function-param">elem</span> ){
  elem.setAttribute( <span class="string">"foo"</span>, <span class="string">"bar"</span> );
  <span class="keyword">return</span> elem;
}

<span class="keyword">function</span> <span class="function-name">b</span>( <span class="js2-function-param">elem</span> ){
  elem.setAttribute( <span class="string">"baz"</span>, <span class="string">"bak"</span> );
  <span class="keyword">return</span> elem;
}

<span class="keyword">var</span> <span class="variable-name">elem</span> = document.getElementById( <span class="string">"example-anchor"</span> );
elem.getAttribute( <span class="string">"foo"</span> ); <span class="comment">// undefined
</span>elem.getAttribute( <span class="string">"baz"</span> ); <span class="comment">// undefined
</span>
elem = compose( a, b )( elem );
elem.getAttribute( <span class="string">"foo"</span> ); <span class="comment">// "bar"
</span>elem.getAttribute( <span class="string">"baz"</span> ); <span class="comment">// "bak"
</span></pre>


The identity function is trivial and, as noted before, function composition is always associative. We know that the functions in **Html** are closed under composition because of the type guarantees we've placed on them. That is, all the functions accept as their only argument HTMLElements and return only HTMLElements so there's no way to compose two of them that doesn't also have the same type signature. Having met the three requirements for a category with **Html** we can move on to the second, and more complex category **Jqry**.

### Jqry

**Jqry** is the category of `jQuery` objects and functions from `jQuery` objects to `jQuery` objects.<sup>3</sup> It's only slightly more complex than **Html** because the reader must accept `this` as an implicit parameter to the JavaScript functions that are the category's morphisms. Also, these morphisms must be defined on the `$.fn` object to guarantee the value of `this` is a `jQuery` object. Otherwise the objects are simply jQuery objects as you know them from day to day use, ie the result of something like `$("div")`, and the functions are JavaScript functions that retun jQuery objects.

Identity:

<pre>
<span class="doc">/** @this {jQuery}
    @return {jQuery} */</span>
<span class="js2-external-variable">$</span>.fn.<span class="function-name">id</span> = <span class="keyword">function</span>(){ <span class="keyword"><span class="js2-warning">return this</span></span> };
</pre>

Sample use of `$.fn.id`:

<pre>
<span class="keyword">var</span> <span class="variable-name">$elem</span> = $( <span class="string">"#example-anchor"</span> );

assert($elem.id() == $elem); <span class="comment">// true
</span></pre>


As you can see, the value of `this`, and therefore the constraint that the morphisms must be defined on `$.fn`, plays an important roll in the way that the functions behave in the **Jqry** category. If it's unclear why that is, remember that `this` is whatever object the method is invoked on using dot notation in JavaScript. Also recall that `$.fn == $.prototype`, meaning when you call `$("div").foo()` it finds `foo` on the `$.fn` by following the prototype chain.

Composition:

<pre>
<span class="doc">/** @typedef {function(this:jQuery): jQuery} */</span>
jqry.morphism;

<span class="doc">/** @sig {jqry.morphism}, {jqry.morphism} -&gt; {jqry.morphism} */</span>
<span class="js2-external-variable">$</span>.<span class="function-name">compose</span> = <span class="keyword">function</span>( <span class="js2-function-param">f</span>, <span class="js2-function-param">g</span> ){
  <span class="keyword">return</span> <span class="keyword">function</span>(){
    <span class="keyword"><span class="js2-warning">return f.apply(g.apply(this, arguments), arguments)</span></span>
  };
};
</pre>

Defining composition is a bit more complex for **Jqry** than it is for **Html** if only because forcing the value that `this` will represent in the composed functions takes more work than just passing in the values as parameters. The first function, `g`, is invoked by forwarding the arguments and explicitly defining its `this` value using the [apply](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function/apply) method common to all JavaScript functions. The value returned by applying `g` is a jQuery object, as required by membership in the category **Jqry**, which is used to explicitly define `this` in the application of `f`.

Sample use:

<pre>
<span class="js2-external-variable">$</span>.fn.<span class="function-name">a</span> = <span class="keyword">function</span>(){
  <span class="keyword">return</span> <span class="builtin">this</span>.map(<span class="keyword">function</span>( <span class="js2-function-param">elem</span> ){
    elem.setAttribute( <span class="string">"foo"</span>, <span class="string">"bar"</span> );
    <span class="keyword">return</span> elem;
  });
};

<span class="js2-external-variable">$</span>.fn.<span class="function-name">b</span> = <span class="keyword">function</span>(){
  <span class="keyword">return</span> <span class="builtin">this</span>.map(<span class="keyword">function</span>( <span class="js2-function-param">elem</span> ){
    elem.setAttribute( <span class="string">"baz"</span>, <span class="string">"bak"</span> );
    <span class="keyword">return</span> elem;
  });
};

<span class="keyword">var</span> <span class="variable-name"><span class="js2-warning">$elem</span></span> = $( <span class="string">"#example-anchor"</span> );
$elem.attr( <span class="string">"foo"</span> );         <span class="comment">// undefined
</span>$elem.attr( <span class="string">"baz"</span> );         <span class="comment">// undefined
</span><span class="js2-external-variable">$</span>.fn.aAndB = $.compose( $.fn.a, $.fn.b );

$elem.aAndB().attr( <span class="string">"foo"</span> ); <span class="comment">// "bar"
</span>$elem.attr( <span class="string">"baz"</span> );         <span class="comment">// "bak"
</span></pre>

Please direct your attention to the fact that you could substitute the `a` and `b` functions from the **Html** `compose` example here for the functions that are mapped using `this.map` (`$.fn.map`) over the set of elements in the jQuery object (represented by `this`). This will be important when defining the Functor from **Html** to **Jqry**.

Again, we know that the morphisms of **Jqry** are closed under composition because each accepts and returns only jQuery objects.

### $() ∉ Functors

Functors are a *single* morphism from one category to another. Another way to think about them is a an operation that can take either an object or a morphism from one category and translate it so that it exists or "works" in another category. This obvioulsy means that a given functor handles both morphisms and objects. Evaluating `$()` as a functor from **Html** to **Jqry**, shows that it translates the objects in the category **Html** (HTMLElements) properly to the objects in **Jqry** (instances of jQuery) but turns the morphisms (functions) into dom ready callbacks. So `$()` is not, strictly speaking, a category theoretic functor.

On the other hand it's quite easy to define a function that handles both objects functions properly. As I noted in a previos post and in an earlier in this post, `$.fn.map` provides an easy way to change functions in **Html** to **Jqry**. So, this new functor just needs to check if it's operating on an object or a function from the **Html** category and respond accordingly.

For the Haskell fans out there (and ciaranm in ##categorytheory) it's better to define a seperate function to handle the translation of morphisms from one category to another in the spirit of `fmap`, but since this is flexible ol' JavaScript, it's possible to use type unions in the closure compiler type constraints, and I think it makes the mapping from theory to Javascript a bit easier to understand, I've chosen to deal with both in a single function.

### Functor Html -> Jqry

<pre>
<span class="doc">/** @sig {HTMLElement | html.morphism} -&gt; {jQuery | jqry.morphism} */</span>
<span class="js2-external-variable">$</span>.<span class="function-name">Functor</span> = <span class="keyword">function</span>( <span class="js2-function-param">a</span> ){
  <span class="keyword">if</span>( <span class="keyword">typeof</span> a == <span class="string">"function"</span> ){
    <span class="keyword">return</span> <span class="keyword">function</span>() {
      <span class="keyword">return</span> <span class="builtin">this</span>.map(a);
    };
  } <span class="keyword">else</span> {
    <span class="keyword">return</span> $( a );
  }
};
</pre>

Sample use for objects:

<pre>
<span class="keyword">var</span> <span class="variable-name"><span class="js2-warning">elem</span></span> = document.getElementsById( <span class="string">"example"</span> );
elem.nodeName;                        <span class="comment">// "A"
</span>$.Functor( elem ).prop( <span class="string">"nodeName"</span> ); <span class="comment">// "A"
</span></pre>


Sample use for functions:

<pre>
<span class="js2-external-variable">$</span>.fn.myMorphism = $.Functor(<span class="keyword">function</span>( <span class="js2-function-param">a</span> ){
  a.setAttribute(<span class="string">"foo"</span>, <span class="string">"bar"</span> );
  <span class="keyword">return</span> a;
});

$( <span class="string">"#example"</span> ).attr( <span class="string">"foo"</span> );              <span class="comment">// undefined
</span>$( <span class="string">"#example"</span> ).myMorphism().attr( <span class="string">"foo"</span> ); <span class="comment">// "bar"
</span></pre>

In the first example it takes an HTMLElement, the objects onf **Html**, and translates it into a jQuery object by using the `$()` function. In the second example it takes a function from HTMLElements to HTMLElements, a morpishm in **Html**, and translates it to a function from jQuery objects to jQuery objects.

Having establish how the functor will operate it too has laws we must adhear to. First it must preserve identity:

<pre>
<span class="keyword">var</span> <span class="variable-name">a</span> = document.getElementById( <span class="string">"#example-anchor"</span> );

assert($.Functor(id(a)) == id($.Functor(a)));

a;                 <span class="comment">// &lt;a id="example-anchor"&gt;&lt;/a&gt;
</span>
id(a);             <span class="comment">// &lt;a id="example-anchor"&gt;&lt;/a&gt;
</span>$.Functor(id(a));  <span class="comment">// [&lt;a id="example-anchor"&gt;&lt;/a&gt;]
</span>
$.Functor(a);      <span class="comment">// [&lt;a id="example-anchor"&gt;&lt;/a&gt;]
</span>$.Functor(a).id(); <span class="comment">// [&lt;a id="example-anchor"&gt;&lt;/a&gt;]
</span></pre>

Whether the identity function is applied before or after the Functor the result should be the same. As you can see the return value of the two expressions is idenitcal. The second law states the the Functor must preserve composition. Assuming a and b from the html compose example:

<pre>
<span class="keyword">var</span> <span class="variable-name"><span class="js2-warning">$elem</span></span> = $( <span class="string">"#example-anchor"</span> );
assert($.Functor(compose(a, b)) == $.compose($.Functor(a), $.Functor(b)));

<span class="keyword">var</span> <span class="variable-name">aAndB</span> = compose(a, b);
<span class="js2-external-variable">$</span>.fn.fstAB = $.Functor(aAndB);                       <span class="comment">// function() { this.map(aAndB); }
</span>$elem.fstAB().attr(<span class="string">"foo"</span>);                           <span class="comment">// "bar"
</span>$elem.attr(<span class="string">"baz"</span>);                                   <span class="comment">// "bak"
</span>
$elem = $( <span class="string">"#second-example-anchor"</span> );
$.Functor(a);                                        <span class="comment">// function() { this.map(a) };
</span>$.Functor(b);                                        <span class="comment">// function() { this.map(b) };
</span><span class="js2-external-variable">$</span>.fn.sndAB = $.compose( $.Functor(a), $.Functor(b) );
$elem.sndAB().attr(<span class="string">"foo"</span>);                           <span class="comment">// "bar"
</span>$elem.attr(<span class="string">"baz"</span>);                                   <span class="comment">// "bak"
</span></pre>

This means that composing the functions and then applying the Functor should create a function that behaves identically to a function created by composing two functions that have had the Functor applied already. Essentially, for all input values the output will be the same. We'll see in a second why that distinction is important.

The assertion at the top will fail of course because the functions are not the same object. More importantly it would fail even if the functions were compared as strings, because they using different internal mechanisms. Ultimately though the functionality is the same and that's what we care about. In each case both `a` and `b` will be allowed to alter each element in the jQuery object's set and apply the attribute values `foo` and `baz` respectively.

While they have the same outcome, the underlying machinery is very different. `$.fn.fstAB` is _one_ iteration over the set of HTMLElements in the jQuery set and `$.fn.sndAB` is _two_. That is `$.fn.sndAB` is applying two seperate `$.fn` methods which is equivelant to `$("foo").a().b()`, where as `$.fn.fstAB` is calling `$.fn.map` only once and piping each element through two composed functions that operate on HTMLElements, `$("foo").map(aAndB)`.<sup>4</sup> They get the same results but `$.fn.sndAB` requires twice as many iterations over the jQuery set!

You might recognize the optimization as [loop fusion](http://en.wikipedia.org/wiki/Loop_fusion). By ensuring that the functor satisfies the requsite laws we've stumbled upon an interesting relationship. That is, any time we use n > 1 iterative `$.fn` methods that exist in **Jqry** it's possible to extract the pure HTMLElement-altering functions that _might_ underly them, compose those functions, and save n-1 full iterations in the process.

### Loop Fusing Hipster

It's clear that someone using `$("foo").a().b()` could, entirely indepdendent of this rigmarole, arrive at the same conclusion that the loops underlying `a` and `b` could be unified to speed up execution and subsequently point out they've been fusing loops since before it was cool. What's not clear is how to know in other cases when it's possible to extract the underlying functions from the `$.fn` methods for composition. Because of the Functor laws we've unambiguously defined a specific subset of functions -- the set of **Html** morphisms with `$.fn.map` applied -- that we _know_ can be fused and will produce the same result. Put differently:

As long as the *n* functions being mapped over the jQuery object's contents individually are all from HTMLElements to HTMLElements, we can compose them, map the resulting composed function over the jQuery object set *once*, and be asured the resulting jQuery object will be the same.

### Package it up

Now that we know which functions can be composed to save some execution time it would be nice to know how use this to improve jQuery plugins. One way is to get jQuery plugin authors provide the underlying DOM manipulation functions as attributes on the plugin methods. This would allow users to do the fusing when it makes sense. Lets use our two composition examples from the **Jqry** category to see what this might look like.

Plugin one:

<pre>
<span class="keyword">function</span> <span class="function-name">setFoo</span>( <span class="js2-function-param">elem</span> ){
  elem.setAttribute( <span class="string">"foo"</span>, <span class="string">"bar"</span> );
  <span class="keyword">return</span> elem;
}

<span class="js2-external-variable">$</span>.fn.<span class="function-name">a</span> = <span class="keyword">function</span>(){
  <span class="keyword">return</span> <span class="builtin">this</span>.map(setFoo);
};

<span class="js2-external-variable">$</span>.fn.a.pure = setFoo;
</pre>

Plugin two:

<pre>
<span class="keyword">function</span> <span class="function-name">setBaz</span>( <span class="js2-function-param">elem</span> ){
  elem.setAttribute( <span class="string">"baz"</span>, <span class="string">"bak"</span> );
  <span class="keyword">return</span> elem;
}

<span class="js2-external-variable">$</span>.fn.<span class="function-name">b</span> = <span class="keyword">function</span>(){
  <span class="keyword">return</span> <span class="builtin">this</span>.map(setBaz);
};

<span class="js2-external-variable">$</span>.fn.b.pure = setBaz;
</pre>


Now both the function from **Html** and the function from **Jqry** are available to the end user. Should they end up using them in serial it's trivial to fuse the two loops:

<pre>
<span class="comment">// two full iterations
</span>$(<span class="string">"foo"</span>).a().b();

<span class="comment">// one full iteration
</span>$(<span class="string">"foo"</span>).map(compose($.fn.a.pure, $.fn.b.pure));
</pre>

Obviously these two examples are tiny and could easily be consolidated by hand, but two plugins with more complex DOM manipulations would be much harder to classify as "fusable". By puting the onus on the plugin developer it gives the end user a better guarantee that it's safe.

### Did you make it?

And there you have it. After defining a mathematical framework in which to view DOM alterations with jQuery and JavaScript we identified an important set of characteristics to use for speeding up our client side code. Maybe this isn't enough to get you interested, but for me it's an afirmation that, even this far from the its ivory towers, math has a role to play.

### Footnotes

1. Thanks again to edwardk and xplat from #haskell-blah and ciaranm from ##categorytheory for reviewing my categories and providing feedback. I'd also like to thank [Tim Goh](http://twitter.com/keyist) for taking the time to enhance my terrible writting.
2. Function composition is always associative, even for impure functions. You'll have to google around a bit for a proof but this appears to be a [generally excepted truth](http://en.wikipedia.org/wiki/Function_composition).
3. If you're interested in digging a bit further into this topic **Jqry** is actually a monoidal category. Credit to both edwardk and xplat in #haskell-blah for pointing this out. As far as I can tell, the morphisms (`$.fn` methods) are muplication as they map from jQuery to jQuery and the unit element is `[]` since using it with `$()` as the unit morphism yields an empty jQuery object.
4. Haskellers will recognize this, and the ultimate conclusion that the rest of the post draws as a degenerate version of `(fmap a) . (fmap b) == fmap (a . b)`
