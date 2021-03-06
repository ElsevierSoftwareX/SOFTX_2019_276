+++
title = "Setting up a project"
weight = 20
type="page"
creatordisplayname = "Ruman Gerst"
creatoremail = "ruman.gerst@leibniz-hki.de"
lastmodifierdisplayname = "Ruman Gerst"
lastmodifieremail = "ruman.gerst@leibniz-hki.de"
+++

# Automated project creation

We provide scripts for Linux and Windows that automate creating a valid MISA++ project.
The resulting project folders can be directly compiled.

You can find the scripts [on our GitHub page](https://github.com/applied-systems-biology/misaxx-utils/tree/master/project-management).

## Linux

1. Copy the `misaxx-create-project.sh` script into a folder
2. Open a terminal and make the script executable via `chmod +x misaxx-create-project.sh`
2. Run `misaxx-create-project.sh` and follow the instructions

## Windows

1. Copy the `misaxx-create-project.ps1` script into a folder
2. Right click the script and click "Run in Powershell"
3. Follow the instructions

# Manual project creation

Following guide shows how to create a project manually.

## Creating the folder structure

Each module is designed to be compiled into a shared library (.dll/.so), which requires a strict separation between the API and implementation. Create following folder structure:

```
./CMakeLists.txt
./include/<module-name>/
./src/<module-name>/
```

`<module-name>` is the name of the CMake target, for example `misaxx-ome`.
The `include` directory contains the public API of the module and will be installed into the global include directory. The `src` directory contains implementation-only files and will not be installed.

## Creating the CMakeLists.txt

{{% notice note %}}
The following CMakeLists.txt file makes use of helper functions that are automatically imported on
making use of the MISA++ Core package.
You are of course free to setup your project without those functions.
{{% /notice %}}

Fill the `CMakeLists.txt` file with following information:

```cmake
cmake_minimum_required(VERSION 3.11) # Or higher if required
project(<module-name> VERSION <module-version> DESCRIPTION "<Short description>")

find_package(misaxx-core REQUIRED)
# Add additional packages if necessary

add_library(<module-name> <Files ...>)

# Add additional link targets if necessary
target_link_libraries(<module-name> misaxx::misaxx-core)

# MISA++ helper script (automatically included by Core Library)
set(MISAXX_LIBRARY <module-name>)
set(MISAXX_LIBRARY_NAMESPACE <module-namespace>::)
set(MISAXX_API_NAME <module_name>)
set(MISAXX_API_INCLUDE_PATH <module-name>)
set(MISAXX_API_NAMESPACE <module_name>)
misaxx_with_default_module_info()
misaxx_with_default_api()

# Only if it's a worker module:
misaxx_with_default_executable()
```

Please note that `<module-name>` and `<module_name>` might be different. `<module_name>` must be a name that is valid in C++.

## For worker modules: Create the module interface and module dispatcher

If you want to create a module that performs processing of data, create the following files:

```
./include/<module-name>/module_interface.h
./include/<module-name>/module.h
./src/<module-name>/module_interface.cpp
./src/<module-name>/module.cpp
```

Please no not forget to add them to the `CMakeLists.txt`

### module_interface.h

```cpp
#include <misaxx/core/misa_module_interface.h>

namespace <module_name> {
    struct module_interface : public misaxx::misa_module_interface {
        void setup() override;
    }
}
```

## module.h

```cpp
#include <misaxx/core/misa_module.h>

namespace <module_name> {
    struct module : public misaxx::misa_module<module_interface> {
        using misaxx::misa_module<module_interface>::misa_module;

        void create_blueprints(blueprint_list &t_blueprints, parameter_list &t_parameters) override;
        void build(const blueprint_builder &t_builder) override;
    }
}
```

# Initial configuration

Create a building directory and run `cmake ..` once. This will automatically generate additional code files such as the module info.

```bash
mkdir build
cd build
cmake ..
```

Add following files to your CMake target:

```
./include/<module-name>/module_info.h
./src/<module-name>/module_info.cpp
```
