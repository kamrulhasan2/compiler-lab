# Compiler Design and Construction Lab: Lex Problems Explanation

এই নোটে `Nu.txt` ফাইলের corrected Lex/Flex lab problems গুলো বাংলায় step by step ব্যাখ্যা করা হলো। শেষে Viva-তে আসার মতো গুরুত্বপূর্ণ প্রশ্ন ও উত্তর দেওয়া আছে।

## Nu.txt Updated Correction Summary

`Nu.txt` ফাইলের code গুলো এখন নিচের correction অনুযায়ী update করা হয়েছে:

1. সব `printf"%s..."` syntax ঠিক করে `printf("%s...", yytext);` করা হয়েছে।
2. whitespace ignore করার ভুল pattern `[/t]+` বদলে `[ \t\n]+` করা হয়েছে।
3. প্রতিটি example-এ `#include <stdio.h>` এবং `int main()` সহ `return 0;` যোগ করা হয়েছে।
4. Negative integer/float-এর pattern-এ `-?` বাদ দিয়ে নির্দিষ্ট `-` ব্যবহার করা হয়েছে।
5. Real number-এর bracket mismatch ঠিক করা হয়েছে এবং decimal/scientific real number pattern ব্যবহার করা হয়েছে।
6. Identifier-এর invalid case digit দিয়ে শুরু হওয়া word হিসেবে ঠিক করা হয়েছে।
7. Float ও integer একসাথে চিনতে Ex-6-এ float rule আগে রাখা হয়েছে।
8. Digit চিনতে Ex-8-এ multiple digit আগে check করা হয়েছে, single digit আলাদা রাখা হয়েছে।
9. Operator চিনতে Ex-9-এ `*` এবং `==` যোগ করা হয়েছে, আর `<=`, `>=`, `==` আগে রাখা হয়েছে।
10. Ex-8 এবং Ex-9 এখন serial অনুযায়ী সাজানো হয়েছে।

## Lex Program-এর Basic Structure

প্রায় সব example একই structure অনুসরণ করেছে:

```lex
%{
/* C code বা header declaration */
%}
%%
pattern     { action }
%%
main()
{
    yylex();
}
```

### অংশগুলোর কাজ

1. `%{ ... %}`  
   এখানে C language-এর declaration, header file, comment ইত্যাদি লেখা যায়।

2. `%%` এর মাঝের অংশ  
   এখানে Lex-এর rule লেখা হয়। প্রতিটি rule-এর দুটি অংশ থাকে:
   - Pattern: কোন input match করবে
   - Action: match করলে কী কাজ হবে

3. `yytext`  
   Lex/Flex-এ `yytext` হলো matched string। যেমন input `if` হলে `yytext = "if"`।

4. `yylex()`  
   lexical analyzer চালু করে এবং input scan করতে থাকে।

5. `.|\n { ECHO; }`  
   যে character অন্য কোনো rule-এ match করে না, সেটি 그대로 print করে।

> গুরুত্বপূর্ণ correction: `Nu.txt`-এ এখন C syntax অনুযায়ী `printf("%s...", yytext);` ব্যবহার করা হয়েছে।

> whitespace ignore করার জন্য এখন `[ \t\n]+` ব্যবহার করা হয়েছে।

---

## Ex-1: Keyword Recognize করা

### Problem
Different keyword চিনতে হবে।

### Main Pattern

```lex
if |
else |
void |
float |
double |
int |
char |
while |
for |
do |
switch |
getch    { printf("%s: is a keyword\n", yytext); }
```

### Step by Step Explanation

1. Input থেকে token নেওয়া হয়।
2. যদি token `if`, `else`, `int`, `char` ইত্যাদির কোনো একটির সাথে match করে, তাহলে keyword হিসেবে print করবে।
3. যদি alphabetic word হয় কিন্তু keyword না হয়:

```lex
[a-zA-Z]+  { printf("%s: is not a keyword\n", yytext); }
```

4. অন্য character হলে `ECHO` করবে।

### Example

Input:

```text
int count if hello
```

Output idea:

```text
int: is a keyword
count: is not a keyword
if: is a keyword
hello: is not a keyword
```

### Note
Updated `Nu.txt`-এ non-keyword message এখন `is not a keyword` করা হয়েছে।

---

## Ex-2: Identifier Recognize করা

### Problem
Valid identifier চিনতে হবে।

### Identifier-এর সাধারণ rule

Compiler design-এ identifier সাধারণত:

- Letter দিয়ে শুরু হবে
- এরপর letter বা digit থাকতে পারে
- Digit দিয়ে শুরু হতে পারবে না

### Main Pattern

```lex
[a-zA-Z]([a-zA-Z]|[0-9])*
```

### Step by Step Explanation

1. প্রথম character অবশ্যই `a-z` বা `A-Z` হতে হবে।
2. এরপর zero or more letter/digit থাকতে পারে।
3. তাই `abc`, `a1`, `sum10` valid identifier।
4. কিন্তু `1abc`, `9x` invalid, কারণ digit দিয়ে শুরু।

### Example

```text
sum1 1sum total
```

Output idea:

```text
sum1: is an identifier
1sum: is not an identifier
total: is an identifier
```

### Better Rule

```lex
[a-zA-Z][a-zA-Z0-9]*       { printf("%s: is an identifier\n", yytext); }
[0-9][a-zA-Z0-9]*          { printf("%s: is not an identifier\n", yytext); }
```

---

## Ex-3: Real Number Recognize করা

### Problem
Real number চিনতে হবে।

### Real Number কী?

Real number বলতে সাধারণত fractional বা decimal number বোঝায়। যেমন:

```text
3.14
-2.5
0.75
10.0
```

Scientific notation-ও real number হতে পারে:

```text
1e5
2.5E-3
```

### Previous Code Issue

আগের pattern:

```lex
-?(([0-9]+)|([0-9]+([eE][-+]?[0-9]+)?)
```

এখানে bracket mismatch আছে এবং decimal point নেই। তাই এটি real number properly চিনবে না।

### Better Pattern

```lex
-?((([0-9]+\.[0-9]*)|(\.[0-9]+))([eE][-+]?[0-9]+)?|([0-9]+[eE][-+]?[0-9]+))
```

### Step by Step Explanation

1. `-?` মানে optional minus sign।
2. `[0-9]+` মানে এক বা একাধিক digit।
3. `\.` মানে decimal point।
4. decimal অংশে `3.14`, `.75`, `10.` এর মতো value match করতে পারে।
5. `([eE][-+]?[0-9]+)?` থাকলে decimal-এর পরে scientific notation support করবে।
6. `[0-9]+[eE][-+]?[0-9]+` থাকলে `1e5`, `2E-3` এর মতো exponent-only real value support করবে।

### Example

```text
3.14 -5.6 1e5 10 abc
```

Output idea:

```text
3.14: is a real number
-5.6: is a real number
1e5: is a real number
10: not matched as real number
abc: is not a real number
```

---

## Ex-4: Integer Recognize করা

### Problem
Positive ও negative integer চিনতে হবে।

### Main Patterns

```lex
[0-9]+       { printf("%s: is a positive integer\n", yytext); }
-[0-9]+      { printf("%s: is a negative integer\n", yytext); }
```

### Step by Step Explanation

1. `[0-9]+` positive integer match করে।
2. `-[0-9]+` minus sign সহ negative integer match করে।
3. negative integer-এর জন্য pattern হলো:

```lex
-[0-9]+
```

কারণ `-?` দিলে positive number-ও match করতে পারে। যদিও Lex longest/first rule অনুসারে আগে positive rule থাকলে positive number প্রথম rule-এ match হবে।

### Better Version

```lex
[0-9]+       { printf("%s: is a positive integer\n", yytext); }
-[0-9]+      { printf("%s: is a negative integer\n", yytext); }
```

### Example

```text
25 -10 abc
```

```text
25: is a positive integer
-10: is a negative integer
abc: is not an integer
```

---

## Ex-5: Float Recognize করা

### Problem
Positive ও negative float চিনতে হবে।

### Main Patterns

```lex
[0-9]*\.[0-9]+
-[0-9]*\.[0-9]+
```

### Step by Step Explanation

1. `[0-9]*` মানে decimal point-এর আগে zero or more digit থাকতে পারে।
2. `\.` decimal point match করে।
3. `[0-9]+` decimal point-এর পরে এক বা একাধিক digit থাকতে হবে।
4. negative float-এর জন্য শুরুতে `-` ব্যবহার করা হয়।

### Previous Issue

Negative float-এর জন্য better pattern:

```lex
-[0-9]*\.[0-9]+
```

কারণ `-?` দিলে positive float-ও match করতে পারে।

### Example

```text
3.14 -0.5 .75 10
```

```text
3.14: is a positive float
-0.5: is a negative float
.75: is a positive float
10: not a float
```

---

## Ex-6: Positive/Negative Integer and Float Recognize করা

### Problem
Integer ও float দুটোই চিনতে হবে, positive/negative আলাদা করে।

### Rule Ordering Important

Lex-এ rule order গুরুত্বপূর্ণ। Float rule আগে রাখা ভালো, কারণ `12.5` input হলে আগে `12` integer হিসেবে match হয়ে যেতে পারে যদি integer rule আগে থাকে।

### Better Rule Order

```lex
[0-9]*\.[0-9]+       { printf("%s: is a positive float\n", yytext); }
-[0-9]*\.[0-9]+      { printf("%s: is a negative float\n", yytext); }
[0-9]+               { printf("%s: is a positive integer\n", yytext); }
-[0-9]+              { printf("%s: is a negative integer\n", yytext); }
```

### Step by Step Explanation

1. Decimal point থাকলে float হিসেবে ধরা হবে।
2. Minus sign থাকলে negative।
3. Decimal point না থাকলে integer।
4. Letter হলে number নয়।

### Example

```text
10 -20 3.5 -7.8 abc
```

```text
10: is a positive integer
-20: is a negative integer
3.5: is a positive float
-7.8: is a negative float
abc: is not a number
```

---

## Ex-7: Punctuation Symbol Recognize করা

### Problem
Different punctuation symbol চিনতে হবে।

### Main Patterns

```lex
";" |
"," |
"%" |
"-" |
":" |
"?" |
"!"     { printf("%s: is a punctuation\n", yytext); }
```

### Step by Step Explanation

1. Input character যদি listed punctuation symbol হয়, তাহলে punctuation print করবে।
2. Word হলে punctuation নয়।
3. অন্য character হলে `ECHO`।

### Example

```text
; , ? hello
```

```text
;: is a punctuation
,: is a punctuation
?: is a punctuation
hello: is not a punctuation
```

---

## Ex-8: Digit Recognize করা

### Problem
Single digit চিনতে হবে।

### Main Patterns

```lex
[0-9][0-9]+ { printf("%s: is not a digit\n", yytext); }
[0-9]       { printf("%s: is a digit\n", yytext); }
```

### Important Issue

Lex longest match rule অনুসরণ করে। তাই input `123` হলে `[0-9][0-9]+` বেশি length match করবে এবং `123: is not a digit` print করবে।

Single digit হলো `0` থেকে `9` পর্যন্ত একটি character।

### Example

```text
5 123 a
```

```text
5: is a digit
123: is not a digit
a: is not a digit
```

---

## Ex-9: Operator Recognize করা

### Problem
Different types of operator চিনতে হবে।

### Operator Types

1. Arithmetic operator: `+`, `-`, `*`, `/`, `%`
2. Relational operator: `>`, `<`, `<=`, `>=`, `=`, `==`
3. Logical operator: `and`, `or`, `not`

### Step by Step Explanation

1. Input `+`, `-`, `/`, `%` হলে arithmetic operator।
2. Input `<`, `>`, `<=`, `>=`, `=`, `==` হলে relational operator।
3. Input `and`, `or`, `not` হলে logical operator।
4. Word কিন্তু operator না হলে `not an operator`।

### Note
Updated `Nu.txt`-এ arithmetic operator list-এ `*` যোগ করা হয়েছে।

### Better Pattern

```lex
"+"|"-"|"*"|"/"|"%"       { printf("%s: is an arithmetic operator\n", yytext); }
">="|"<="|"=="|">"|"<"|"=" { printf("%s: is a relational operator\n", yytext); }
and|or|not                { printf("%s: is a logical operator\n", yytext); }
```

### Example

```text
+ <= and hello
```

```text
+: is an arithmetic operator
<=: is a relational operator
and: is a logical operator
hello: is not an operator
```

---

## Ex-10: Verb Recognize করা

### Problem
English verb চিনতে হবে।

### Main Patterns

```lex
is |
are |
am |
was |
were |
be |
being |
been |
do |
does |
will |
would |
can |
could |
has |
had |
have |
go      { printf("%s: is a verb\n", yytext); }
```

### Step by Step Explanation

1. Input word যদি listed verb-এর মধ্যে থাকে, verb হিসেবে print করবে।
2. অন্য alphabetic word হলে verb নয়।
3. অন্য character হলে `ECHO`।

### Example

```text
is go eat book
```

```text
is: is a verb
go: is a verb
eat: is not a verb
book: is not a verb
```

---

# Previous Common Mistakes Fixed in Nu.txt

1. `printf"%s..."` ভুল syntax ছিল। Correct:

```c
printf("%s: message\n", yytext);
```

2. `[/t]+` whitespace নয়। Correct:

```lex
[ \t\n]+
```

3. Negative number-এর জন্য `-?` না দিয়ে শুধু `-` ব্যবহার করা হয়েছে:

```lex
-[0-9]+
```

4. Float/integer একসাথে চিনলে float rule আগে রাখা উচিত।

5. Real number-এর pattern-এ decimal point থাকা উচিত।

6. Identifier digit দিয়ে শুরু হতে পারে না।

7. Lex longest match করে; same length হলে আগে লেখা rule priority পায়।

---

# Viva Questions and Answers

## 1. Lex কী?

Lex হলো lexical analyzer generator। এটি regular expression pattern থেকে scanner তৈরি করে, যা input program কে token-এ ভাগ করে।

## 2. Lexical analyzer-এর কাজ কী?

Lexical analyzer source code scan করে keyword, identifier, operator, number, punctuation ইত্যাদি token identify করে।

## 3. Token কী?

Token হলো program-এর meaningful unit। যেমন `int`, `sum`, `=`, `10`, `;`।

## 4. Lexeme কী?

Lexeme হলো actual matched string। যেমন input `int x;` হলে `int` একটি lexeme।

## 5. Pattern কী?

Pattern হলো regular expression, যার মাধ্যমে token match করা হয়। যেমন identifier-এর pattern: `[a-zA-Z][a-zA-Z0-9]*`।

## 6. `yytext` কী?

`yytext` হলো Lex/Flex-এর built-in variable, যেখানে current matched lexeme থাকে।

## 7. `yylex()` কী করে?

`yylex()` lexical analyzer চালু করে এবং input scan করে matching rule অনুযায়ী action execute করে।

## 8. Lex file-এর তিনটি section কী কী?

1. Definition section: `%{ ... %}`
2. Rules section: `%% ... %%`
3. User subroutine section: C function বা `main()`

## 9. Lex-এ rule matching priority কীভাবে কাজ করে?

Lex প্রথমে longest match নেয়। যদি একাধিক rule same length match করে, তাহলে আগে লেখা rule execute হয়।

## 10. Identifier digit দিয়ে শুরু করতে পারে না কেন?

কারণ compiler সাধারণত digit দিয়ে শুরু হওয়া lexeme-কে number হিসেবে ধরতে পারে। তাই identifier letter বা underscore দিয়ে শুরু হয়।

## 11. Keyword এবং identifier-এর পার্থক্য কী?

Keyword language-এর reserved word, যেমন `if`, `else`, `int`। Identifier programmer-defined name, যেমন `sum`, `count`।

## 12. Whitespace ignore করতে কোন pattern ব্যবহার করা হয়?

```lex
[ \t\n]+    ;
```

এখানে action empty, তাই whitespace ignore হয়।

## 13. `ECHO` কী?

`ECHO` unmatched বা matched text output-এ 그대로 print করে। এটি Lex-এর built-in macro।

## 14. Integer-এর regular expression কী?

Positive integer:

```lex
[0-9]+
```

Negative integer:

```lex
-[0-9]+
```

## 15. Float-এর regular expression কী?

```lex
[0-9]*\.[0-9]+
```

Negative float:

```lex
-[0-9]*\.[0-9]+
```

## 16. Real number এবং integer-এর পার্থক্য কী?

Integer-এ decimal point থাকে না, যেমন `10`। Real/float number-এ decimal point থাকতে পারে, যেমন `10.5`।

## 17. Logical operator কী?

Logical operator condition combine বা negate করে। যেমন `and`, `or`, `not`।

## 18. Relational operator কী?

Relational operator দুই value compare করে। যেমন `<`, `>`, `<=`, `>=`, `=`, `==`।

## 19. Arithmetic operator কী?

Arithmetic operator mathematical operation করে। যেমন `+`, `-`, `*`, `/`, `%`।

## 20. Lex এবং Yacc-এর পার্থক্য কী?

Lex lexical analyzer তৈরি করে, যা token generate করে। Yacc parser তৈরি করে, যা grammar অনুযায়ী syntax analysis করে।

## 21. Regular expression কেন ব্যবহার করা হয়?

Token-এর pattern describe করার জন্য regular expression ব্যবহার করা হয়।

## 22. `[0-9]+` এবং `[0-9]` এর পার্থক্য কী?

`[0-9]` শুধু একটি digit match করে। `[0-9]+` এক বা একাধিক digit match করে।

## 23. `*` এবং `+` operator-এর পার্থক্য কী regular expression-এ?

`*` মানে zero or more occurrence। `+` মানে one or more occurrence।

## 24. `?` operator-এর অর্থ কী?

`?` মানে optional, অর্থাৎ zero or one occurrence।

## 25. `\.` কেন ব্যবহার করা হয়?

Regular expression-এ `.` যেকোনো character বোঝায়। তাই actual dot match করতে `\.` ব্যবহার করা হয়।

---

# Short Exam-Friendly Correct Template

নিচের template ব্যবহার করে প্রায় সব problem solve করা যায়:

```lex
%{
#include <stdio.h>
%}

%%
[ \t\n]+    ;
pattern     { printf("%s: message\n", yytext); }
.           { ECHO; }
%%

int main()
{
    yylex();
    return 0;
}
```

এই template-এ শুধু `pattern` এবং `message` problem অনুযায়ী বদলালেই হবে।
