# First thoughts

The prompt displayed before having access to the challenge itself is quite interesting. We are given a program, in hex opcode, a load address and a Signature, with the explanation that the program must be signed.

![image](https://user-images.githubusercontent.com/17447180/224371893-f3f01ef3-8c4f-4711-85e6-c70a4a6b8d90.png)

_Load address_ : 
8000

_Program text_ : 
3540088000450545054505450545054505450f433041

_Signature_ : 
8605e027f42368ea6bba9de66409f6a8ddedcd49614a4648281c47a7b4ad252f5639069b17ba8ff104d371e2d8a625b038f0750667364087e7987e40ea81510f

Clearly the first thing I wanna do here is to check what this program is doing, using the assemble/disassemble tool... However let's start with a quick overview of the main function.
