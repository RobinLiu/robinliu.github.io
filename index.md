---
layout: post
title: A classic C++ query problem from C++ Primer
permalink: /cppquery/
comments: True
---
Following are copy-pasted from C++ Primer 5rd edition.

##Design of the Query Program

A good way to start the design of a program is to list the program’s operations. Knowing what operations we need can help us see what data structures we’ll need. Starting from requirements, the tasks our program must do include the following:

* When it reads the input, the program must remember the line(s) in which each word appears. Hence, the program will need to read the input a line at a time and break up the lines from the input file into its separate words
* When it generates output,
	- The program must be able to fetch the line numbers associated with a given word
	- The line numbers must appear in ascending order with no duplicates
	- The program must be able to print the text appearing in the input file at a given line number.

These requirements can be met quite neatly by using various library facilities:  

* We’ll use a vector\<string> to store a copy of the entire input file. Each line in the input file will be an element in this vector. When we want to print a line, we can fetch the line using its line number as the index. 
* We’ll use an istringstream (§ 8.3, p. 321) to break each line into words.
* We’ll use a set to hold the line numbers on which each word in the input appears. Using a set guarantees that each line will appear only once and that the line numbers will be stored in ascending order.
* We’ll use a map to associate each word with the set of line numbers on which the word appears. Using a map will let us fetch the set for any given word.

For reasons we’ll explain shortly, our solution will also use shared_ptrs.

###Data Structures

Although we could write our program using vector, set, and map directly, it will be more useful if we define a more abstract solution. We’ll start by designing a class to hold the input file in a way that makes querying the file easy. This class, which we’ll name TextQuery, will hold a vector and a map. The vector will hold the text of the input file; the map will associate each word in that file to the set of line numbers on which that word appears. This class will have a constructor that reads a given input file and an operation to perform the queries.

The work of the query operation is pretty simple: It will look inside its map to see whether the given word is present. The hard part in designing this function is deciding what the query function should return. Once we know that a word was found, we need to know how often it occurred, the line numbers on which it occurred, and the corresponding text for each of those line numbers.

The easiest way to return all those data is to define a second class, which we’ll name QueryResult, to hold the results of a query. This class will have a print function to print the results in a QueryResult.

###Sharing Data between Classes	
Our QueryResult class is intended to represent the results of a query. Those results include the set of line numbers associated with the given word and the corresponding lines of text from the input file. These data are stored in objects of type TextQuery.

Because the data that a QueryResult needs are stored in a TextQuery object, we have to decide how to access them. We could copy the set of line numbers, but that might be an expensive operation. Moreover, we certainly wouldn’t want to copy the vector, because that would entail copying the entire file in order to print (what will usually be) a small subset of the file.

We could avoid making copies by returning iterators (or pointers) into the TextQuery object. However, this approach opens up a pitfall: What happens if the TextQuery object is destroyed before a corresponding QueryResult? In that case, the QueryResult would refer to data in an object that no longer exists.

This last observation about synchronizing the lifetime of a QueryResult with the TextQuery object whose results it represents suggests a solution to our design problem. Given that these two classes conceptually “share” data, we’ll use shared_ptrs (§ 12.1.1, p. 450) to reflect that sharing in our data structures.

### Code

``` C++
#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <string>
#include <set>
#include <map>
#include <unistd.h>

using namespace std;

string make_plural(size_t ctr, const string &word, const string &ending)
{
    return (ctr > 1) ? word + ending : word;
}

class QueryResult;
class TextQuery {
public:
    using line_no = vector<string>::size_type;  // check the usage of keyword "using"
    TextQuery(ifstream&);
    QueryResult query(const string& sought) const;
private:
    shared_ptr<vector<string>> file;  // Check how the pointor is initialized;
    map<string, shared_ptr<set<line_no>>> wm; // Check how the pointor is initialized;
};

TextQuery::TextQuery(ifstream& is):file(new vector<string>)
{
    string text;
    while (getline(is, text)) {
        file->push_back(text);
        line_no n = file->size() - 1;
        istringstream line(text);
        string word;
        while (line >> word) {
            cout << " " << word <<" " <<endl;
            auto &lines = wm[word]; // Check the return type of the map;
            if (!lines)
                lines.reset(new set<line_no>);  // new ptr for value of the map;
            lines->insert(n);
        }
    }
}


class QueryResult {
    friend ostream& print(ostream&, const QueryResult&); // check the scope of friend function of a class;
public:
    using line_no = vector<string>::size_type;
    QueryResult(string s, shared_ptr<set<line_no>> p, shared_ptr<vector<string>> f):
    sought(s), lines(p), file(f) {}
private:
    string sought;
    shared_ptr<set<line_no>> lines;
    shared_ptr<vector<string>> file;
};


QueryResult TextQuery::query(const string &sought) const
{
    static shared_ptr<set<line_no>> nodata(new set<line_no>);
    auto loc = wm.find(sought);
    if (loc == wm.end())
        return QueryResult(sought, nodata, file);
    else
        return QueryResult(sought, loc->second, file);  // loc->second is a share_ptr of class TextQuery;
}

ostream &print(ostream & os, const QueryResult &qr)
{
    os << qr.sought << " occurs " << qr.lines->size() <<" "
    <<make_plural(qr.lines->size(), "time", "s")<<endl;
    
    for (auto num: *qr.lines ) {
        os << "\t(line " << num + 1 << ") "
        << *(qr.file->begin() + num) << endl;
    }
    
    return os;
}


void runQueries(ifstream &infile)
{
    TextQuery tq(infile);
    while (true) {
        cout << "enter word to look for, or q to quit: ";
        string s;
        if( !(cin >> s) || s == "q") break;
        
        print(cout, tq.query(s)) << endl;
    }
}

int main(int argc, const char * argv[]) {
    ifstream f("main.cpp");
    runQueries(f);
    
    return 0;
}

```
