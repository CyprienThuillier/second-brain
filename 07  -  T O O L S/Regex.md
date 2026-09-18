### Definition

Regex are patterns of text that you define to search documents and match exactly what you're looking for.

### Charsets

**Syntax :** ```[<CHAR(S)>]```

**Examples :**

```[abc]``` will match ```a```, ```b``` and ```c```, but also ```cba``` and ```ca```.
```[abc]zz``` will match ```azz```, ```bzz``` and ```czz```.

You can also use a ```-``` dash to define ranges :
```[a-c]zz``` is the same as above.

And then you can combine ranges together :
```[a-cx-z]zz``` will match ```azz```, ```bzz```, ```czz```, ```xzz```, ```yzz``` and ```zzz```.

Most notably, this can be used to match any alphabetical character :
```[a-zA-Z]``` will match any single letter (lowercase or uppercase)

You can use numbers too :
```file[1-3]``` will match ```file1```, ```file2``` and ```file3```.

Then, there is a way to exclude characters from a charset with the ```^``` hat symbol and include everything else :
```[^k]ing``` will match ```ring```, ```sing```, ```$ing```, but not ```king```.

Of course, you can exclude charsets, not just single characters : 
```[^a-c]at``` will match ```fat```, ```hat```, but not ```cat```.

### Wildcards

**Syntax :**

- ```.<CHAR(S)>``` for any char
- ```<CHAR>?``` for optional chars
- ```\.``` for using the char ```.```

**Examples :**

```.at``` will match any singe char with at (```aat```, ```1at```, ```%at```, ...)
```cats?``` will match ```cat``` and ```cats```
```\.a``` will match ```.a```

*The following section is PCRE regex only*
### Metacharacters and repetitions

**Syntax :** 

- ```\d``` matches a digit
- ```\D``` matches a non-digit
- ```\w``` matches an alphanumeric char and ```_```
- ```\W``` matches a non-alphanumeric char
- ```\s``` matches a whitespace char (spaces, tabs and line breaks)
- ```\S``` matches everything else

- ```<CHAR>{n}``` matches the ```CHAR``` ```n``` times
- ```<CHAR>*``` matches the ```CHAR``` 0 or more times
- ```<CHAR>+``` matches the ```CHAR``` 1 or more times

**Examples :**

```cats{4}``` will match ```catssss```
```cats*``` will match ```cat```, ```cats```, ```catss```, ...
```cats+``` will match the same excepted ```cat```

### Starts with / ends with, groups, and  either / or

**Syntax :** 
- ```^``` starts with
- ```$``` ends with
- ```(...)``` that define a group between the parentheses
- ```|``` is "or"

**Example :**

```^abc``` will match every line that starts by ```abc```
```xyz$``` will match every line that ends by ```xyz```
```(no){5}``` will match ```nonononono```
```during the (day|night)``` will match ```during the day``` and ```during the night```
