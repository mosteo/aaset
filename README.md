# AASET

`aaset` (Automatic Ada Symbolic Execution Tool) is a tool for the automatic
symbolic execution of Ada source code.

Follow this link for the [Spanish version](README_es.md) of this document.

## Operation

Through a simple web form, one must specify a module and upload the source code
packaged in a `.tar` file.

Once the source code is uploaded, all the procedures included in the
specification file with the same name as the module indicated in the form will
be analyzed. For example, if the module indicated is "test", all the procedures
included in the `test.ads`` package will be analyzed.

Automatically, a new package will be created with source code that declares all
the necessary variables to perform the calls to the found procedures, setting
them as symbolic by using the KLEE libraries. Finally, once the variables have
been declared and made symbolic, calls to all the procedures will be made,
passing them the necessary variables as arguments. In this way, the application
would be executed symbolically.

Once generated using `gnat-llvm`, the bytecodes of the generated source code,
plus a bytecode that includes the bytecodes of each of the packages included in
the tar (this is necessary to link it with the main bytecode, in order to mark
as internal all the subprograms within the provided source code), the
application is executed symbolically through KLEE, generating the test cases as
well as some errors in the execution.

After the symbolic execution is concluded, the converter tool will be executed,
which will take care of executing the procedures declared in the module in a
concrete manner, using the test cases generated in the symbolic execution. The
aaset tool will automatically create a new package within the converter tool
that declares all the necessary variables for the call to all the procedures
declared in the specification of the indicated module (in the same manner as
the source code to perform the symbolic execution). Then, it will read, in the
same order in which they were declared, an integer representation of each
variable as execution arguments, obtaining such representation from the test
cases, and converting that integer representation into an object of the type
corresponding to each variable. When all the variables are assigned to the
values of the test case to be analyzed, calls will be made to all the
procedures declared in the specification of the module, executing the
application in a concrete manner, with the aim of searching for runtime errors
for each of the test cases generated by KLEE during the symbolic execution.

For each error found in the symbolic or concrete execution in each test case, a
`.txt` file will be generated with the information of the test case and the
errors in question. Finally, all these `.txt` files will be converted into a
report in JSON format with all the information to be presented to the user.
Likewise, a JSON will also be returned with the compilation errors in case the
source code for performing the symbolic execution cannot be generated, either
due to errors in the provided source code or due to an error in the generation
of the new source code. The `'Content-Type'` header of the response is marked as
`application/json` so, depending on the browser, the display will be different,
possibly requiring the installation of a plugin for the 'friendly' presentation
of the JSON report.

## Prerequisites

For the execution of the tool, it is necessary to have installed:

[gcc-11.3.0](https://gcc.gnu.org/install/index.html)

[llvm-13.0.0](https://llvm.org/)

[gnat-llvm](https://github.com/AdaCore/gnat-llvm)

[KLEE](https://klee.github.io/build-llvm11/)

## Installation

`aaset` uses `web2py` as the Framework for web server development. `web2py` is
included as a submodule of this repository, so it will be necessary to
initialize it when cloning the repository:
```
git clone --recurse-submodules https://github.com/mgarcp13/aaset.git
```
By default, the `web2py` module initializes to the content of the remote
repository, which does not include the application that implements the web
server. This application is packaged inside the web-application directory. This
directory contains a subdirectory called `aaset` and a file `w2p` which is a
package of the application to be installed through the web2py admin options.
With the `aaset` subdirectory, you can install the application simply by copying
it to the `web2py/applications` directory:
```
cp -r ./web-application/aaset ./web2py/applications
```

## Use

### Web Client

Start the `web2py` application:
```
python3 web2py/web2py.py
```
Set an administration password, server port, and IP:

![Captura de pantalla 2022-07-23 142023](https://user-images.githubusercontent.com/9430650/180604618-d9cbb982-be56-48d8-acf3-8e29e0210aca.png)

From a browser, connect to the URL [127.0.0.1:8000/aaset/main/load](127.0.0.1:8000/aaset/main/load)

![Captura de pantalla 2022-07-23 141358](https://user-images.githubusercontent.com/9430650/180604488-b325831b-6ec1-4612-929d-ad8f4f05640c.png)

Provide the module name and upload a tar with the source code. Once the report
is generated, the JSON report will be displayed in a table format, in addition
to text information with a brief summary of the execution.

![reporte_caso1](https://user-images.githubusercontent.com/9430650/185758140-24c32369-31aa-40f6-98b8-c5d6db2b01f9.png)

### REST API

To execute the tool through the REST API, it is necessary to start flask, which
is the framework with which the web server has been developed. The first step
is to indicate to flask which web service should be set by modifying the
FLASK_APP environment variable to indicate the python file that implements the
service, in this case, web_service.py included in the root directory of the
project:

```
export FLASK_APP=web_service.py
```

Once the web service to be executed is indicated, you need to start `flask`.
This is done with the command, from the root directory of the tool:

```
flask run
```

At the very start of flask, the URL where the server is listening to handle
requests is indicated:

![run_flask](https://user-images.githubusercontent.com/9430650/185758275-e2225f45-14a6-497e-b1f3-076028e68a2d.png)

Now, any client who wishes can obtain the report by sending a form via POST
with the same data as for the web execution, obtaining the report in raw JSON.
For instance, to execute the tool using curl, the command would be:

```
curl -X POST http://127.0.0.1:5000/report -F "file=@/tmp/sample.tar" -F "module=q_math"
```

However, for a more user-friendly display of the JSON, the use of the jq tool
is recommended:

```
curl -X POST http://127.0.0.1:5000/report -F "file=@/tmp/sample.tar" -F "module=q_math" | jq
```

Another possibility is to use the Postman tool:

![postman](https://user-images.githubusercontent.com/9430650/185758397-7ee2a715-f776-4cd8-bbda-b728a1f8c4a3.png)
