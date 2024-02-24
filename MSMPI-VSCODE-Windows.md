Running MSMPI with VSCode

1. Install [Visual Studio Code](https://code.visualstudio.com/download)

2. Install MinGW64
- Download [x86_64-posix-sjlj](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/8.1.0/threads-posix/sjlj/x86_64-8.1.0-release-posix-sjlj-rt_v6-rev0.7z)
- Unzip to C:/MinGW64
- add C:/MinGW64/mingw64/bin to System Environment Variables
    - While here you can check if you have other versions of MinGW64 installed and remove the paths to prevent potential issues.
- test gcc from commandline with
```gcc --version```
- check the version in the output
```(x86_64-posix-sjlj-rev0, Built by MinGW-W64 project) 8.1.0```

3. Install MSMPI Runtime and SDK
- Download [Microsoft MPI v10.1.2](https://www.microsoft.com/en-us/download/details.aspx?id=100593)
- Install msmpisetup.exe to directory C:/MSMPI/
- Install msmpisdk.msi to directory C:/MSMPI/SDK
- verify that the install set the correct environment variables by running ```set msmpi``` in the command line, verify that you see the following
    ```
    MSMPI_BENCHMARKS=C:\MSMPI\Benchmarks\
    MSMPI_BIN=C:\MSMPI\Bin\
    MSMPI_INC=C:\MSMPI\SDK\Include\
    MSMPI_LIB32=C:\MSMPI\SDK\Lib\x86\
    MSMPI_LIB64=C:\MSMPI\SDK\Lib\x64\
    ```
- verify msmpi install with ```mspiexec``` in command line.

4. Create Sample Code
- create mpi.c
- paste code and save
```c
#include <stdio.h>
#include <math.h>
#include <mpi.h>

int main(int argc, char *argv[]){
    int my_rank, numprocs;
    MPI_Init(&argc,&argv);

    // get number of processors
    MPI_Comm_size(MPI_COMM_WORLD,&numprocs);

    // get rank of each processor
    MPI_Comm_rank(MPI_COMM_WORLD,&my_rank);

    // get the name of processor
    int  namelen;
    char processor_name[MPI_MAX_PROCESSOR_NAME];
    MPI_Get_processor_name(processor_name,&namelen);
    
    printf("Hello World! Process %d of %d on %s\n",my_rank,numprocs,processor_name);
    MPI_Finalize();
}
```

5. Configure VSCode
- install C++ Extension Plugin,
  - Search for "ms-vscode.cpptools-extension-pack" in extensions or,
  - download from (https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack)
- Fix intellisense
  - hover over ```#include <mpi.h>```  in mpi.c and click quick fix, or press ctrl-shift-p and search for "C/C++: Edit Configurations (UI)"
  - add "${MSMPI_INC}" to the include path 
  - check that this saved by opening /.vscode/c_cpp_properties.json and looking for ${MSMPI_INC}
  - Check that this works by right clicking <mpi.h> in the mpi.c and clicking go to definition. You should see the content of mpi.h

6. Create/Configure Build task
- Click Terminal > Configure Default Build Task
- Select the MinGW64 from c:/MinGW64/ from the list.
- This will create /.vscode/tasks.json
- in the args array, in between the ```"${file}",``` and ``` "-o", ``` arguments, add the following the following arguments
```json
    "-I",
    "${MSMPI_INC}",
    "-L",
    "${MSMPI_LIB64}",
    "-lmsmpi",
```

7. Build and Execute mpi.c
- while mpi.c is open. Click Terminal > Run Build Task...
- you should see a new mpi.exe file
- in command line run .\mpi.exe
