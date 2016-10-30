# C++ promise/A+ library with Javascript styles.

## Usage
   promise-cpp is header only and easy to use, just #include "promise.hpp" is enough.
   With promise-cpp, you can resolve or reject any type of data without writing complex template code.

## Example
```cpp
#include <stdio.h>
#include <string>
#include "promise.hpp"

using namespace promise;

#define output_func_name() do{ printf("in function %s, line %d\n", __func__, __LINE__); } while(0)

void test1() {
    output_func_name();
}

int test2() {
    output_func_name();
    return 5;
}

void test3(int n) {
    output_func_name();
    printf("n = %d\n", n);
}

Defer run(Defer &next){

    return newPromise([](Defer d){
        output_func_name();
        d.resolve(3, 5, 6);
    }).then([](const int &a, int b, int c) {
        printf("%d %d %d\n", a, b, c);
        output_func_name();
    }).then([](){
        output_func_name();
    }).then([&next](){
        output_func_name();
        next = newPromise([](Defer d) {
            output_func_name();
        });
        //Will call next.resole() or next.reject() later
        return next;
    }).then([](int n, char c) {
        output_func_name();
        printf("n = %d, c = %c\n", (int)n, c);
    }).fail([](char n){
        output_func_name();
        printf("n = %d\n", (int)n);
    }).fail([](short n) {
        output_func_name();
        printf("n = %d\n", (int)n);
    }).fail([](int &n) {
        output_func_name();
        printf("n = %d\n", (int)n);
    }).fail([](const std::string &str) {
        output_func_name();
        printf("str = %s\n", str.c_str());
    }).fail([](uint64_t n) {
        output_func_name();
        printf("n = %d\n", (int)n);
    }).then(test1)
    .then(test2)
    .then(test3)
    .always([]() {
        output_func_name();
    });
}

int main(int argc, char **argv) {
    Defer next;

    run(next);
    printf("======  after call run ======\n");

    next.resolve(123, 'a');
    //next.reject('c');
    //next.reject(std::string("xhchen"));
    //next.reject(45);

    return 0;
}
```

## Class Defer
Defer object is the promise object itself.

### Defer newPromise(FUNC func);
Create a new Defer object with a user-defined function.
The user-defined functions, used as parameters by newPromise, must have a parameter Defer d. 
for example --
```cpp
return newPromise([](Defer d){
})
```

### Defer::resolve(const RET_ARG... &ret_arg);
Resolve the promise object with arguments, where you can put any number of ret_arg with any type.
for example --
```cpp
return newPromise([](Defer d){
    //d.resolve();
    //d.resolve(3, '2', std::string("abcd"));
    d.resolve(9567);
})
```

### Defer::reject(const RET_ARG &ret_arg);
Reject the promise object with arguments, where you can put any number of ret_arg with any type.
for example --
```cpp
return newPromise([](Defer d){
    //d.reject();
    //d.reject(std::string("oh, no!"));
    d.reject(-1, std::string("oh, no!"))
})
```

### Defer::then(FUNC_ON_RESOLVED on_resolved, FUNC_ON_REJECTED on_rejected)
Return the chaining promise object, where on_resolved is the function to be called when 
previous promise object calls function resolve, on_rejected is the function to be called
when previous promise object calls function reject.
for example --
```cpp
return newPromise([](Defer d){
    d.resolve(9567, 'A');
}).then(
    /* function on_resolved */ [](int n, char ch){
        printf("%d %c\n", n, ch);   //will print 9567 here
    },
    /* function on_rejected */ [](){
         
    });
```

### Defer::then(FUNC_ON_RESOLVED on_resolved)
Return the chaining promise object, where on_resolved is the function to be called when 
previous promise object calls function resolve.
for example --
```cpp
return newPromise([](Defer d){
    d.resolve(9567);
}).then([](int n){
    printf("%d\n", n);  b //will print 9567 here
});
```

### Defer::fail(FUNC_ON_REJECTED on_rejected)
Return the chaining promise object, where on_rejected is the function to be called when
previous promise object calls function reject.
for example --
```cpp
return newPromise([](Defer d){
    d.reject(-1, std::string("oh, no!"));
}).fail([](int err, string &str){
    printf("%d, %s\n", err, str.c_str());   //will print "-1, oh, no!" here
});
```

### Defer::always(FUNC_ON_ALWAYS on_always)
Return the chaining promise object, where on_always is the function to be called whenever
the previous promise object is be resolved or rejected.
for example --
```cpp
return newPromise([](Defer d){
    d.reject(std::string("oh, no!"));
}).always([](){
    printf("in always\n");   //will print "in always" here
});
```

### about exceptions
To throw any object in the callback functions above, including on_resolved, on_rejected, on_always, 
will same as d.reject(the_throwed_object) and returns immediately.
for example --
```cpp
return newPromise([](Defer d){
    throw std::string("oh, no!");
}).fail([](string &str){
    printf("%s\n", str.c_str());   //will print "oh, no!" here
});
```
For the performance, we suggest to use function reject instead of throw.

### about the chaining parameter
Any type of parameter can be used when call resolve, reject or throw, except that the plain string or array.
To use plain string or array as chaining parameters, we may wrap it into an object.
```cpp
newPromise([](Defer d){
    // d.resolve("ok"); may cause a compiling error, use the following code instead.
    d.resolve(std::string("ok"));
})
```

### copy the promise object
To copy the promise object is allowed and effective, please do that when you need.
```cpp
Defer d = newPromise([](Defer d){});
Defer d1 = d;  //It's safe and effective
```







