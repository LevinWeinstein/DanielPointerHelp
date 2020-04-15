# What you Did Wrong: Passing by Reference vs Passing by Value

Let's say that I pass an integer into a function.
Then, I change the value inside the function.
```c++
void doSomething(int item){
  cout << "In function, after incrementing: " << ++item << endl;
}

int main(int argc, char **argv){
  int item = 4;
  doSomething(item);
  cout << "Outside of function: " << item << endl;
}
```
Our output would be the following:
```
In function, after incrementing: 5
Outside of function: 4
```
Why is this? Because we passed the item in by _value_, not by _reference_.

You've learned this already, __kind of__. You've learned it insofar as you understand that you have to do it when you want to change the value of an _integer_:

```c++
// Passing in by pointer, c style
void doSomething(int *item)
{
  // increments the value in the outer scope
  (*item)++;
}

//Passing in by reference, fancy c++ style.
//Is harder to to explain on a low level, don't worry about this.
void doSomething(int &item)
{
  // increments the value in the outer scope.
  item++;
}
```

But what about when you pass in an _array_ as a parameter?
Well, an N-length array of type \<T\> is simply N Ts in a row,
all stored behind the pointer passed into the function.

When I write a function like either of these:
```c++
void doSomething(int n[]);
// or
void doSomething(int *n);
```
I am passing a *reference* to n, sure.
But what I am passing in as a *value* is the *memory address* of the start of the array.

Since I have a *reference* to each **element** in the array, I can change any element in the array, and the element will also change in the _outer scope_.

But if I "change the array itself", aka _change the memory-address number passed in by **value**_, what will happen?

Answer: It __won't__ change in the global scope! That is because the _pointer memory address itself is passed by value_.

```c++
//Example:
// Example program
#include <iostream>
#include <string>

using namespace  std;

void printArray(int n[], int size)
{
    for (int i = 0; i < size; i++)
    {
        cout << n[i] << (size - 1 - i ? ' ' : '\n');
    }
}

void changeArray(int n[])
{
    int m[] = {3, 6, 9};
    n = m;
    printArray(n, 3);
}
int main()
{
    int n[] = {1, 2, 3};
    printArray(n, 3);
    changeArray(n);
    printArray(n, 3);  
}

/**
 * Output:
 * 1 2 3
 * 3 6 9
 * 1 2 3
 */
```

This became a problem in your program. You resized your array inside your -innermost- function. This function was called by another function, which in turn was called by main.

_Both of these functions passed the array pointer by value_.

Yes, this does mean that the integers _within_ the array were passed by reference.
But the array itself? (Meaning the integer value of the memory address?) __by value__.

This is what you had:
```c++
int main();
// called
void addWord(string word, wordItem words[], int *length);
// which in turn called
void resizeArray(wordItem words[]);
```
The array wasn't resizing. So it would go out of bounds for a while, but eventually, shortly after it hit 100 items, it would cause a segmentation fault.

The solution: Instead of passing the _array_ (or _pointer_, same thing) into the function, pass a _pointer to that pointer_.

Note: this must be done in both functions in order to make it to main.

```c++

void resizeArray(wordItem **words)
{
  // ... ... ... ... 
  for (int i = 0; i < max_length / 2; i++)
  {
    newArray[i] = (*words)[i];
  }
  delete[] *words;
  *words = newArray;
}

void addWord(string word, wordItem **_words, int *length)
{
  // ... ... ... ...
  if (*length >= max_length)
  {
    resizeArray(_words);
  }

  wordItem *words = *_words;
  // ... ... ... ...
}
```

Result: (including your debug messages)
```
... ... ...
Length is 7275
max length is 12800
Probability of next 10 words from rank 100
---------------------------------------
0.0016 - moment
0.0015 - once
0.0015 - than
0.0015 - face
0.0015 - place
0.0015 - house
0.0015 - might
0.0015 - much
0.0015 - school
0.0015 - seemed
```

# Takeaways

## What is a pointer as a value
  * A pointer is an integer: it specifies the memory location of the start of the information in the heap.
  * Let's say you have an wordItem \*n.
    * The pointer value is an integer (for example: 0xfaf779eb32)
    * The _type_ of the pointer specifies _how many bytes to read at a time_
     * for structs, it also specfies the relative locations of _each of the children_, so that they may be accessed easily.
  * an "array" parameter to a function is exactly the same as a pointer parameter to a function.

## How to change the pointer's value 
  * To change the value of the pointer (or, "set the array equal to another array"), fromwithin a function, you must pass a _pointer to this pointer_, since the pointer itself was passed by value otherwise.

