---
title:  "Find and Replace Characters in C++"
date:   2018-07-12
categories: Algorithm
tags: [Algorithm, C++]
---

# The Issue

Let's say you have a string and somehow it contains a random character that you want to discard and change to something else. For example, let's say you got a string with question marks `?` in the places where exclamation marks `!` should be at, like "Hello? Your encoding is terrible?".

You may want the code that looks something like this:
```c
using namespace std;

int main(int argc, char* argv[])
{
  string str = "Hello? Your encoding is terrible?";
  cout << "The original string is : " << str << endl;

  if (findAndReplaceStr(str, "/", " "))
  {
      cout <<
          "Find-and-replace done! Output : " <<
          str << endl;
  }
  else
  {
      cout << "Nothing found." << endl;
  }
}  
```

How are you going to approach this type of problem? Of course, you need to first find the target string to replace, while iterating over the original string as few as possible. I would first approach like below:

```c
using namespace std;

bool findAndReplaceStr(string& rInput,
        const string& from,
        const string& to)
{
  string::size_type pos = 0;
  while ((pos = rInput.find(from)) != string::npos)
  {
    rInput.replace(rInput.begin() + pos,
                   rInput.begin() + pos + from.size(),
                   to);
  }
}
```

It's a very simple, naive solution, which seems to work fine. At least this compiles and works.

However, if you take a closer look at this algorithm, the time complexity is O(N<sup>2</sup>).

Here's why: Let's say you have a string `aaaaa`, which you want to change to `bbbbb`. Inside our while-loop, we have std::string::find() and std::string::replace() function, which execute the main job of find-and-replacing. Here, the both functions have time complexities of O(N), while these must be called as many times as the number of replacements needed, which is equal to N in the worst case. That makes the entire time complexity O(N<sup>2</sup>).  

After some research, I found an overloaded version of string::find(); you can specify an offset as an argument. For example, if you want to look for `e`, but after the 7th index in `Abercrombie`, you can save time by not looking up the first 7 characters at all.
That being discovered, I could update the offset every time I have a replacement, so that I don't have to look up the characters that I already visited. The final solution looks like this:
```c
using namespace std;

bool findAndReplaceStr(string& rInput,
        const string& from,
        const string& to)
{
    string::size_type pos = 0;
    string::size_type offset = 0;
    bool bResult(false);

    while((pos = rInput.find(from, offset)) != string::npos)
    {
        cout << "Found : " << from << "\n"
            << "Replacing it with : "
            << to << "\n" << endl;

        rInput.replace(rInput.begin() + pos,
                rInput.begin() + pos + from.size(),
                to);

        offset = pos + to.size();

        bResult = true;
    }

    return bResult;
}
```

The time complexity now is O(N), which is much more desirable.
After building and running this, the result would be like this:
```console
user@ubuntu:~$ g++ findAndReplaceStr.cpp -o findAndReplaceStr.out -g
user@ubuntu:~$ ./findAndReplaceStr.out
The original string is : Hello? Your encoding is terrible?
Found : ?
Replacing it with : !

Found : ?
Replacing it with : !

Find-and-replace done! Output : Hello! Your encoding is terrible!
```
