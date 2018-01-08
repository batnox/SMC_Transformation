#Henil Shah: 200208366


Installing, Compiling and running the code:

THE CODE DOES NOT WORK ON CUDA EXPRESSION CALLS FOR MATRIXMUL.CU IF THE PATH TO HEADER FILES OF THE TEST CASES GIVEN WITH THE PROBLEM STATEMENT (FILES INSIDE COMMON/INC)
THE SCRIPT DOES NOT INCLUDE THE PATH BECAUSE IT MAY CHANGE FOR THE VBOX THE SCRIPT CODES ARE BEING TESTED ON

The file "smc.cpp" should be placed in the foler ~/llvm/llvm/tools/clang/tools/extra/smc/
Place CMakeLists.txt in ~/llvm/llvm/tools/clang/tools/extra/smc
In the CMakeLists.txt file in the folder ~/llvm/llvm/tools/clang/tools
Go to ~/llvm/build-release
Run: ninja smc
(The code uses test files given with the problem description and requires headerfile given with those testfiles)
Run:
bin/smc ./test_files/matrixAdd_org.cu -- --cuda-device-only -I/usr/local/cuda/include -I/path to header files with the given test-files

Output:
The output file will be generated in the folder test_files with the name <inputfile>_smc.cu

Overall structure of the code:
The code has AST frontend action class where the clang works.
1) It has a preproccesing handler which will get triggered for each #include statement. We store end position for each statement in the same variable and that way we will get last #include statement's end position when the last call returns. Here we add the #include "smc.h"

2)The AST Matchr has three matchers for function call, function declaration and CUDA Call expression.

3) The function declaration handler is for Kernel function declaration and defintion. 
It will check if the function has CUDA attribute, it will add SMC_Begin, SMC_End and additional parameters.
It will go to the body of the function and looks through declaration which are initialized by an expression.
It checks the expressions for being blockIdx.x and blockIDx.y and modifies as required (more details in the report)

4)The CUDA call expression handler adds SMC_Init() right before the call and adds additional arguments at the end of the call

5)Variable declaraion handler looks for the variable wiht name grid aand  dim3 __SMC_orgGridDim(...)
