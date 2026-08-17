# Chapter 1: Introduction to Python and The Way of the Program

## What is Python?

Python is a high-level, interpreted programming language known for its simplicity and readability. It was created by Guido van Rossum and first released in 1991. Python supports multiple programming paradigms, including procedural, object-oriented, and functional programming. It is widely used in various domains such as web development, data analysis, artificial intelligence, scientific computing, and data engineering.

### Year Created
Python was created in the late 1980s, and its first official version, Python 0.9.0, was released in February 1991. Python 2.0 was released in 2000, and Python 3.0, which is not backward-compatible with Python 2, was released in 2008.

**Current stable Python version:** Python 3.13.2

### Important Resources
- [W3School Python tutorial](https://www.w3schools.com/python/) (beginner friendly)
- [Python official documentation](https://docs.python.org/)
- [Programiz Python tutorial](https://www.programiz.com/python-programming)
- [Pythontutorial.net](https://www.pythontutorial.net/)

## The Way of the Program

The goal of learning Python is to teach you to think like a computer scientist. This way of thinking combines some of the best features of mathematics, engineering, and natural science.

Like mathematicians, computer scientists use formal languages to denote ideas (specifically computations). Like engineers, they design things, assembling components into systems and evaluating tradeoffs among alternatives. Like scientists, they observe the behavior of complex systems, form hypotheses, and test predictions.

The single most important skill for a computer scientist is **problem solving**. Problem solving means the ability to formulate problems, think creatively about solutions, and express a solution clearly and accurately. As it turns out, the process of learning to program is an excellent opportunity to practice problem-solving skills.

## 1.1 What is a Program?

A program is a sequence of instructions that specifies how to perform a computation. The computation might be something mathematical, such as solving a system of equations or finding the roots of a polynomial, but it can also be a symbolic computation, such as searching and replacing text in a document or something graphical, like processing an image or playing a video.

The details look different in different languages, but a few basic instructions appear in just about every language:

- **input:** Get data from the keyboard, a file, the network, or some other device
- **output:** Display data on the screen, save it in a file, send it over the network, etc.
- **math:** Perform basic mathematical operations like addition and multiplication
- **conditional execution:** Check for certain conditions and run the appropriate code
- **repetition:** Perform some action repeatedly, usually with some variation

Believe it or not, that's pretty much all there is to it. Every program you've ever used, no matter how complicated, is made up of instructions that look pretty much like these.

You can think of programming as the process of breaking a large, complex task into smaller and smaller subtasks until the subtasks are simple enough to be performed with one of these basic instructions.

## 1.2 Running Python

One of the challenges of getting started with Python is that you might have to install Python and related software on your computer. If you are familiar with your operating system, and especially if you are comfortable with the command-line interface, you will have no trouble installing Python. But for beginners, it can be painful to learn about system administration and programming at the same time.

To avoid that problem, it's recommended that you start out running Python in a browser. Later, when you are comfortable with Python, you can install Python on your computer.

There are a number of web pages you can use to run Python. Some popular options include:
- [Replit](https://replit.com/)
- [Python.org's online console](https://www.python.org/shell/)
- [Trinket](https://trinket.io/python)

### Python Versions

There are two major versions of Python: Python 2 and Python 3. They are very similar, so if you learn one, it is easy to switch to the other. However, Python 2 reached end-of-life on January 1, 2020, so all new projects should use Python 3.

The Python interpreter is a program that reads and executes Python code. When you start the interpreter, you should see output like this:

```
Python 3.13.2 (default, Jan 15 2025, 14:20:21)
[GCC 11.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

The `>>>` is a prompt that indicates that the interpreter is ready for you to enter code. If you type a line of code and hit Enter, the interpreter displays the result:

```python
>>> 1 + 1
2
```

## 1.3 The First Program

Traditionally, the first program you write in a new language is called "Hello, World!" because all it does is display the words "Hello, World!". In Python, it looks like this:

```python
>>> print('Hello, World!')
Hello, World!
```

This is an example of a print statement, although it doesn't actually print anything on paper. It displays a result on the screen. The quotation marks in the program mark the beginning and end of the text to be displayed; they don't appear in the result.

The parentheses indicate that `print` is a function. We'll learn more about functions in later chapters.

## 1.4 Arithmetic Operators

After "Hello, World!", the next step is arithmetic. Python provides operators, which are special symbols that represent computations like addition and multiplication.

The operators `+`, `-`, and `*` perform addition, subtraction, and multiplication:

```python
>>> 40 + 2
42
>>> 43 - 1
42
>>> 6 * 7
42
```

The operator `/` performs division:

```python
>>> 84 / 2
42.0
```

Note that the result is `42.0` instead of `42` because division in Python 3 always returns a floating-point number.

The operator `**` performs exponentiation (raising a number to a power):

```python
>>> 6**2 + 6
42
```

**Warning:** In some other languages, `^` is used for exponentiation, but in Python it is a bitwise operator called XOR:

```python
>>> 6 ^ 2
4
```

## 1.5 Values and Types

A value is one of the basic things a program works with, like a letter or a number. Some values we have seen so far are `2`, `42.0`, and `'Hello, World!'`.

These values belong to different types:
- `2` is an **integer**
- `42.0` is a **floating-point number**
- `'Hello, World!'` is a **string**

If you are not sure what type a value has, the interpreter can tell you using the `type()` function:

```python
>>> type(2)
<class 'int'>
>>> type(42.0)
<class 'float'>
>>> type('Hello, World!')
<class 'str'>
```

In these results, the word "class" is used in the sense of a category; a type is a category of values.

- Integers belong to the type `int`
- Strings belong to `str`
- Floating-point numbers belong to `float`

### String vs Numbers

Values like `'2'` and `'42.0'` look like numbers, but they are in quotation marks, so they are strings:

```python
>>> type('2')
<class 'str'>
>>> type('42.0')
<class 'str'>
```

## Python Basics

### Identifiers

Identifiers are names used to identify variables, functions, classes, modules, or other objects. Rules for naming identifiers in Python:

- Identifiers can be a combination of letters (a-z, A-Z), digits (0-9), and underscores (_)
- Identifiers cannot start with a digit
- Identifiers are case-sensitive (`myVar` and `myvar` are different)
- Reserved keywords cannot be used as identifiers

**Valid identifiers:**
```python
my_variable
_private_var
variable1
MyClass
```

**Invalid identifiers:**
```python
1variable    # Cannot start with digit
my-variable  # Hyphen not allowed
class        # Reserved keyword
```

### Keywords

Keywords are reserved words in Python that have special meanings and cannot be used as identifiers. Some of the keywords in Python include:

`if`, `else`, `elif`, `for`, `while`, `break`, `continue`, `def`, `return`, `lambda`, `class`, `import`, `from`, `try`, `except`, `finally`, `and`, `or`, `not`, `True`, `False`, `None`

You can see all keywords by running:
```python
import keyword
print(keyword.kwlist)
```

### PEP 8 Rules

PEP 8 is the official style guide for Python code. It provides conventions for writing readable and consistent code. Some key PEP 8 rules include:

- **Indentation:** Use 4 spaces per indentation level
- **Line Length:** Limit all lines to a maximum of 79 characters (72 for docstrings/comments)
- **Imports:** Imports should usually be on separate lines
- **Whitespace:** Avoid extraneous whitespace in various situations
- **Naming Conventions:**
  - Variables: `my_variable`
  - Functions: `my_function`
  - Classes: `MyClass`
  - Constants: `MY_CONSTANT`

## 1.6 Formal and Natural Languages

Natural languages are the languages people speak, such as English, Spanish, and French. They were not designed by people (although people try to impose some order on them); they evolved naturally.

Formal languages are languages that are designed by people for specific applications. For example, the notation that mathematicians use is a formal language that is particularly good at denoting relationships among numbers and symbols. Chemists use a formal language to represent the chemical structure of molecules. And most importantly:

**Programming languages are formal languages that have been designed to express computations.**

### Key Differences

Although formal and natural languages have many features in common—tokens, structure, and syntax—there are some differences:

- **Ambiguity:** Natural languages are full of ambiguity, which people deal with by using contextual clues. Formal languages are designed to be nearly or completely unambiguous.

- **Redundancy:** Natural languages employ lots of redundancy to reduce misunderstandings. Formal languages are less redundant and more concise.

- **Literalness:** Natural languages are full of idiom and metaphor. Formal languages mean exactly what they say.

## 1.7 Debugging

Programmers make mistakes. For whimsical reasons, programming errors are called **bugs** and the process of tracking them down is called **debugging**.

Programming, and especially debugging, sometimes brings out strong emotions. If you are struggling with a difficult bug, you might feel angry, despondent, or embarrassed.

### Debugging Tips

- Think of the computer as an employee with certain strengths (speed and precision) and weaknesses (lack of empathy and inability to grasp the big picture)
- Find ways to take advantage of the strengths and mitigate the weaknesses
- Use your emotions to engage with the problem, without letting reactions interfere with your ability to work effectively
- Learning to debug is frustrating, but it's a valuable skill useful for many activities beyond programming

## Exercises

### Exercise 1.1
Experiment with the "Hello, world!" program and try to make mistakes on purpose:

1. In a print statement, what happens if you leave out one of the parentheses, or both?
2. If you are trying to print a string, what happens if you leave out one of the quotation marks, or both?
3. You can use a minus sign to make a negative number like `-2`. What happens if you put a plus sign before a number? What about `2++2`?
4. In math notation, leading zeros are ok, as in `09`. What happens if you try this in Python? What about `011`?
5. What happens if you have two values with no operator between them?

### Exercise 1.2
Start the Python interpreter and use it as a calculator:

1. How many seconds are there in 42 minutes 42 seconds?
2. How many miles are there in 10 kilometers? (Hint: there are 1.61 kilometers in a mile)
3. If you run a 10 kilometer race in 42 minutes 42 seconds, what is your average pace (time per mile in minutes and seconds)? What is your average speed in miles per hour?

## Glossary

- **Problem solving:** The process of formulating a problem, finding a solution, and expressing it
- **High-level language:** A programming language like Python that is designed to be easy for humans to read and write
- **Interpreter:** A program that reads another program and executes it
- **Program:** A set of instructions that specifies a computation
- **Value:** One of the basic units of data, like a number or string, that a program manipulates
- **Type:** A category of values (int, float, str)
- **Bug:** An error in a program
- **Debugging:** The process of finding and correcting bugs
- **Identifier:** Names used to identify variables, functions, classes, or other objects
- **Keyword:** Reserved words in Python with special meanings
- **PEP 8:** The official style guide for Python code