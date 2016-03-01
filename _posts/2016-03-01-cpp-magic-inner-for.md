---
layout: post
section-type: post
title: C++ Magic Inner For
category: Cpp
tags: [ 'cpp' ]
---
Ever wonder what will happen if a switch meets a for ?\\
Magic ...

~~~ Cpp
int main() {
    int a = 2;
    int i;
    switch(a) {
        for(i=0;i<5;i++){
        case 1:
            std::cout << "foo" << std::endl;
            continue;
        case 2:
            std::cout << "bla" << std::endl;
            continue;
        default:
            std::cout << "bar" << std::endl;
        }
    }

    return 0;
}
~~~

The result is kinda surprising:\\
(Magic is not to be demonstrated :P )

~~~
bar
foo
foo
foo
foo
~~~

**glhf**
