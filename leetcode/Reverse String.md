Reverse String
===

Write a function that takes a string as input and returns the string reversed.

Example:
Given s = "hello", return "olleh".

# Answer

* one

```java
public class Solution {
    public String reverseString(String s) {
        if (s == null){
            return null;
        }
        char[] chars = s.toCharArray();
        int l = chars.length - 1;
        int i = 0;
        char first = 0;
        while (i < l){
            first = chars[i];
            chars[i] = chars[l];
            chars[l] = first;
            i++;
            l--;
        }
        return new String(chars);
    }
}
```

* two

```java
public class Solution {
    public String reverseString(String s) {
        if (s == null){
            return null;
        }
        char[] chars = s.toCharArray();
        int l = chars.length - 1;
        int i = 0;
        char first = 0;
        while (i < l){
            first = chars[i];
            chars[i] = chars[l];
            chars[l] = first;
            i++;
            l--;
        }
        return new String(chars);
    }
}
```