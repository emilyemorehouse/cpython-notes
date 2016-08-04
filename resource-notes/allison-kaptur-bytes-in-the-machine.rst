============================================================
Byterun: A (C)Python interpreter in Python
============================================================

`Allison Kaptur - Byterun: A (C)Python interpreter in Python <https://www.youtube.com/watch?v=HVUTjQzESeo>`_

**PyCon 2015 talk:**
Have you ever wondered how the CPython interpreter works? Do you know where to
find a 1,500 line switch statement in CPython? I'll talk about the structure of
the interpreter that we all use every day by explaining how Ned Batchelder and I
chased down a mysterious bug in Byterun, a Python interpreter written in Python.
We'll also see visualizations of the VM as it executes your code.

Slides can be found at: `https://speakerdeck.com/pycon2015 <https://speakerdeck.com/pycon2015>`_ and  `https://github.com/PyCon/2015-slides <https://github.com/PyCon/2015-slides>`_


The Python Interpreter: the last step in the process of running a Python program

Four steps:

#. Lexing
#. Parsing
#. Compiling
#. Interpreting

Lexing and parsing take a Python file and convert it to AST, that the compiler
knows what to do with.
Compiling takes the AST and turns it into bytecode that the interpreter knows
how to handle.
Interpreting takes the bytecode and executes the code; takes compiled code objects and makes them run.

Most interpreted languages, including Python, includes a compilation step. The
compiler doesn't control much and the heavy-lifting is all handled by the
interpreter.

The Python interpreter is a virtual machine, which is software that emulates a
real computer. This particular machine is stack machine (in contrast with a
register machine).

Bytecode is often called an intermediate or internal representation of Python
code. Analagous to Assembly for C.

Lexing, parsing and compiling only happens once for a given code object that
can then be reused later, instead of having to repeat the process.


-------------------------
Starting from the bottom:
-------------------------
A minimal VM with only three functions:

#. LOAD_VALUE
#. ADD_TWO_VALUES
#. PRINT_ANSWER

The data stack provides a way to track the state of the compiler very easily.
Having a set of well-formed instructions gives us a framework in which we can
begin to add a lot of complexity to our interpreter.

Functions themselves do not take arguments, the arguments are automatically
taken from the data stack.::

	what_to_execute = {
		"instructions":
			[
				("LOAD_VALUE", 0),
				("LOAD_VALUE", 1),
				("ADD_TWO_VALUES", None),
				("PRINT_ANSWER", None)
			],
		"numbers": [7, 5]
	}

	class Interpreter(object):
		def __init__(self):
			self.stack = []

		def value_loader(self, number):
			self.stack.append(number)

		def answer_printer(self):
			answer = self.stack.pop()
			print(answer)

		def two_value_adder(self):
			first_num = self.stack.pop()
			second_num = self.stack.pop()
			total = first_num + second_num
			self.stack.append(total)

	def run_code(self, what_to_execute):
		instrs = what_to_execute["instructions"]
		numbers = what_to_execute["numbers"]
		for each_step in instrs:
			instruction, argument = each_step
			if intruction == "LOAD_VALUE":
				number = numbers[argument]
				self.value_loader(number)
			elif instruction == "ADD_TWO_VALUES":
				self.two_value_adder()
			elif instruction == "PRINT_ANSWER":
				self.answer_printer()


	interpreter = Interpreter()
	interpreter.run_code(what_to_execute)


---------
Exploring
---------
Finding our byte code:::

	>>> def mod(a, b):
	... 	ans = a % b
	... 	return ans
	>>> mod.func_code.co_code  # functionname.codeobject.bytecode
	>>> [ord(b) for b in mod.func_code.co_code]

Python reveals a lot of internal implementation details in the REPL, so we can
easily explore and poke around.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^
dis, a bytecode disassembler
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When you have a set of instructions that are intended for a machine and you
want to make it readable for humans.::

	>>> import dis
	>>> dis.dis(mod)

^^^^^^^^^^^^^^^^^^^^^^^^^^^
Zooming out: the call stack
^^^^^^^^^^^^^^^^^^^^^^^^^^^
The call stack is made of frames. You have one frame on the call stack for each
level of your code. This is seen in traceback errors.

Each frame has it's own data stack. There is one frame per function call (think
about with recursion - each has to be tracked).


-------------------------------------
Basic Pieces for a Python Interpreter
-------------------------------------

* A collection of frames
* Data stacks on frames
* A way to run frames

In our toy interpreter, we had a simple if statement to select the function to
run. CPython is implemented in the same way.

**TODO: Check out CEval.c, where the 1500 line switch statement exists.**

If Python were implemented with only one data stack instead of one data stack
per frame, the only feature we would lose is generators.



**Resource note:** `http://tech.blog.aknin.name/category/my-projects/pythons-innards <http://tech.blog.aknin.name/category/my-projects/pythons-innards>`_ by @aknin

**Resource note:** `http://eli.thegreenplace.net <http://eli.thegreenplace.net>`_ by Eli Bendersky



**First pass: 2016-07-29, ~2 hours**
