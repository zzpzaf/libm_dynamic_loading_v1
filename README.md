# <p style="color:blue;"> C++ Dynamic loading of the libm.so/lybm.dylib shared library</p>

### <p style="background-color:darkblue; color:white">Dive not that deep, but just enough to get the grasp!</p>
---

This is a simple project, consisting of just 1 file - the main.cpp.
Its purpose is to provide a simple example of how to dynamically load a shared library and use one of its functions.
Actually, it loads the math C library libm.so - for Linux, libm.dylib - for macOS, and uses its sqrt function.

- The source code contains quite explainable comments. 
- The **makefile** is also included.


The code:

```
//libm_dynamic_loading_v1

/* 
Dynamic Loading of a shared library and using one of its functions
Targer library: "libm.so" (or libm.dylib for macOS)
Target library's function: "sqrt"
*/


#include <iostream>
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>
#include <string>



int main(int argc, char **argv)
{


    // Here we define the name of the shared library 
    std::string soname;
    soname = "libm"; 
    // For Linux-es the extension is 'so' For macOS-es is 'dylib'
    std::string libext = "so";
    #ifdef __APPLE__
        libext = "dylib";
    #endif 
    std::string libfullname = soname + "." + libext;

    // Here we define the function symbol/name
    std::string func;
    func = "sqrt";


    // Here we declare just a pointer (a handler) of our target library  
    void * plibobj;

    // If we are going to access a function we have to know its type and the type of its parameters
    // Here we declare the pointer of our target function - it is of type double *
    // The function in our case should accept an argument also of type double  
    double (*fsqr)(double nn) ;



    // Load the library object in a new-allocated memory (*handle)
    // Since we dont care about other symbols of the library, here, we use the RTL_LAZY mode-flag.
    // RTL_LAZY generally instructs the dlopen to resolve symbols only if the code references them, and upon code execution.
    // If the symbol is never referenced, then it is never resolved. Note that RTL_LAZY is OK for function references. 
    // However, for references to variables, better to use the RTL_NOW to resolve them immediatly upon library loading
    plibobj = dlopen(libfullname.c_str(), RTLD_LAZY); 
    // If there is an error, output it and exit
    if (!plibobj) {
        std::cerr << "Error loading the library " + libfullname + " - " << dlerror() << "\n";
        // fprintf(stderr, "%s\n",error);
        exit(EXIT_FAILURE);
    }

    //Clear any error
    dlerror();    

  

    // Here we get the pointer of our target function, it is just a pointer to an undefined object
    void * psqr =  dlsym(plibobj, func.c_str()) ;
    
    // Again, if there is an error accessing the symbol, output it and exit
    if (psqr == NULL) {
        std::cerr << "Error accessing the symbol:" + func + dlerror() << "\n";
        exit(EXIT_FAILURE);
    } 

    // In C++, a void pointer (here the *psqr returned as result of dlsym) has to be converted to a pointer of the appropriate type before it can be used.
    // Here we convert the pointer returned from dlsym, to the type of our target function
    // Actually we cast it to the declared type of the pointer of our target function.
    // For casting we use the reinterpret_cast<>. It does not check if the pointer type and data pointed by the pointer is same or not.
    // The reinterpret_cast converts any pointer type to any other pointer type. And guarantees that when we again reinterpret_cast the 
    // casted pointer back to the original type, we get the original value. 
    fsqr = reinterpret_cast<decltype(fsqr)>(psqr);
    

    // Get a number from user-input
    double n; 
    std::cout << "Give me a number to find its squre root: ";
    std::cin >> n;

    // Calculate the square root result    
    double r = (*fsqr)(n);

    // Output the input number, and the result
    printf("Square root of %.2f is: %.5f\n", n, r);

    //Unload the library - free memory
    dlclose(plibobj);   
    exit(EXIT_SUCCESS);
    
}

```