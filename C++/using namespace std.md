#Why you should stop writing `using namespace std` in most cases?
To answer the question, we first need to have a solid understanding of what they are and how to use them. After discussing that, it should be clear that `using namespace std` in the global scope is erroneous.
##What are C++ namespaces and how to use them?
Primarily it is a name mangling mechanism. At least this is what is observable. But does that mean it's useless? Let's figure that out.
##Performance
So, first of all let's settle down performance issues, which are probably the first questions of the beginners. **Namespaces don't have impact on runtime performance whatsoever**. C++ namespaces are purely a compile time mechanism which helps compiler differentiate between the same names in the different libraries (some people call it modules, some packages, but they all are the same). In the resulting object code (also called assembly, binary), namespaces disappear. So, to sum up, from performance point of view, namespaces doesn't do anything.
##Code size
Code size might not be that bad issue, but some people don't want to write `std::` all the time. Also, it arguably makes code harder to read for some people. It is impossible to deny this disadvantage.
##Name collision
If you came from C background, you probably are familiar with this. Name collision is an error which is caused by the same names (in case of functions, it is the same signature. In case of template functions, it is the same function signature after substitution and return type. In case of template structs/classes, it is the same arguments passed to template). Name collision makes choosing good names much harder, because you will always need to worry about preventing the name collision by using some macros. In C++, namespaces resolve this issue (actually they diminish it to the point that it is no longer a problem). 
##Overload resolution control
This is one of the reasons when you want to actually import a name from a namespace. Just do it in a very narrow scope and only on very few situations, one of which I will cover next.
I will call it swap expression. So, basically when you write `using namespace std`, names from `std` will get imported into current scope. What will happen is if there is a situation, for example, swap, compiler first will try to find `swap()` function in the current scope. So, if you want to have different behavior of swap, you can write your own in the current scope (or you can import it) and compiler will happily invoke it when you call swap.

    using std::swap;
    swap(first, second);
    
But, if there is no `swap()` function declared in the current scope, it will invoke standard one, `std::swap()`. Isn't this great? Namespaces provide sort of a fallback mechanism which simplifies metaprogramming.

#Conclusion
If you made here reading everything above, congratulations! I believe that article gave you solid understanding on how to use namespaces and what impact they have on your code. Keep `using ...` expressions small in count and narrow in scope, but don't fret from it. Also, stop writing `using namespace std` in global scope.
