# bitcom+ proposal  

This is a proposal for an extension to the [Bitcom](https://bitcom.planaria.network/#/) protocol to provide more flexibility and utility. 
 
## Problem   

Bitcom provides a basic but powerful standard for viewing and using Op_return data as commands that are broadcast to the "Bitcoin computer", a decentralized network of servers and services. However, currently there are limitations regarding protocol development.  
  
If we compare Bitcom to functions, we have the ability for required positional arguments and optional positional arguments. It's left to the interpreter of the protocol to figure out what is meant if there are optional arguments. 
 
However, what if we want 25 possible arguments and to choose any 3 particular arguments? This is impossible by using push data. Instead the solution people have found is the use JSON in a single push data and use whichever fields they like. There are a few problems with this approach:  
1. It is not as easy to query. Suppose you wanted to query for a specific field in your JSON in Bitbus? You can use a $text search but it is not very elegant. What if your JSON gets larger than 256 characters and is stored in Bitfs?  
2. What if you want to have an argument that is not a string, number or bool, such as a picture or a pdf file? You could encode the file as a string in your JSON (doesn't sound like a good idea) but you then wouldn't be able to easily access the file using Bitfs.  
  
## Solution 
Since Bitcom draws heavily from the Unix shell, we can simply extend the analogy. I propose 2 new extensions to the bitcom standard. 

The first is single letter options which are identified with a single dash and must go right after the protocol prefix:  
```
[protocol prefix] 
-abc 
[rest of the protocol]
``` 
These can be seen as shorthand for bool values. This sets a,b,c to true.  
 
Second is named arguments. This would use a double dash and the push data afterwards would be the argument: 
```
[protocol prefix] 
[positional arguments] 
--username 
Jeffery
```  
 
(it would probably be better to use camelcase instead of the standard dash for multi-word argument names as this makes it more robust in Javascript where JSON is interpreted as objects rather than maps with string keys)

### Proposed modification to BOB  
To fully harness these features, a small but powerful modification to BOB is proposed. Each cell that contains an argument or option now has an 'args' property in addition to 'cell' and 'i': 
```
{
  cell:[ 
  /* cell objects *not including* options and named arguments */
  ],
  i:1,
  args:{
    a:1,
    b:1,
    c:1,
    username:{
      b: [b64 encoded],
      s: "Jeffery",
      ii: 4,
      i: 3
    }
  }
}
``` 

This makes transactions with arguments exceptionally easy to query: 
```
var query = {
  q:{
    find:{
      "out.tape.cell.args.b":1,
      "out.tape.cell.args.username.s:'Jeffrey'
    }
  }
}
```

Note that while I propose *not including* arguments in the "cell" array, you can reconstruct the original opreturn using the order of options and the 'ii' property of named arguments. 
 
Please let me know what you think!

      
    
    
