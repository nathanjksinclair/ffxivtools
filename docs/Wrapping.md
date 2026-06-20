markdown# Wrapping a C# Project with Python in VS Code

This project demonstrates how to compile a C# program into a **Class Library (.dll)** and natively call its code from Python using the **`pythonnet`** library inside Visual Studio Code.

---

## Prerequisites

Ensure you have the following installed on your machine:
* [VS Code](https://visualstudio.com)
* [.NET SDK](https://microsoft.com)
* [Python 3](https://python.org)

### Recommended VS Code Extensions
* **Python** (by Microsoft)
* **C# Dev Kit** (by Microsoft)

---

## Step-by-Step Implementation

### Step 1: Set Up the Project Workspace
Open your terminal, create a root directory for your project, and open it in VS Code:
```bash
mkdir csharp_python_wrapper
cd csharp_python_wrapper
code .
```

### Step 2: Create the C# Library
1. Open the integrated terminal in VS Code (`Ctrl+\`` or `Cmd+\``).
2. Scaffold a new C# class library project:
   ```bash
   dotnet new classlib -n MyCSharpLib -o csharp_src
   ```
3. Open `csharp_src/Class1.cs` and replace its contents with this demo code:
   ```csharp
   using System;

   namespace MyCSharpLib
   {
       public class WrapperDemo
       {
           public string SayHello(string name)
           {
               return $"Hello {name} from the C# codebase!";
           }

           public int AddNumbers(int a, int b)
           {
               return a + b;
           }
       }
   }
   ```
4. Build the project to compile your code into a `.dll` file:
   ```bash
   dotnet build csharp_src/MyCSharpLib.csproj
   ```
   *Note: Your compiled binary will be located around: `csharp_src/bin/Debug/net8.0/MyCSharpLib.dll` (depending on your .NET version).*

### Step 3: Set Up the Python Environment
1. In the project root, create a Python virtual environment:
   ```bash
   python -m venv .venv
   ```
2. Activate the virtual environment:
   * **Windows (Cmd/PowerShell):** `.venv\Scripts\activate`
   * **macOS/Linux:** `source .venv/bin/activate`
3. Install the required Python.NET dependency:
   ```bash
   pip install pythonnet
   ```

### Step 4: Write the Python Wrapper
Create a file named `main.py` in your project root directory and add the following code:

```python
import os
import sys
import clr

# 1. Define the absolute path to your compiled C# DLL
dll_path = os.path.abspath(
    "./csharp_src/bin/Debug/net8.0/MyCSharpLib.dll"
)

# Verify the file exists to catch path errors early
if not os.path.exists(dll_path):
    raise FileNotFoundError(f"Could not find the C# DLL at {dll_path}")

# 2. Add the directory containing the DLL to the system path
sys_dir = os.path.dirname(dll_path)
sys.path.append(sys_dir)

# 3. Load the assembly reference using clr
clr.AddReference("MyCSharpLib")

# 4. Import your C# namespace and class natively
from MyCSharpLib import WrapperDemo

def main():
    # Instantiate the C# class
    csharp_instance = WrapperDemo()
    
    # Execute C# methods seamlessly from Python
    greeting = csharp_instance.SayHello("Python Developer")
    math_result = csharp_instance.AddNumbers(35, 7)
    
    print(greeting)
    print(f"Math result from C#: {math_result}")

if __name__ == "__main__":
    main()
```

### Step 5: Run the Project
Execute the Python script inside your activated terminal environment:
```bash
python main.py
```

**Expected Output:**
```text
Hello Python Developer from the C# codebase!
Math result from C#: 42
```

---

## Development Notes

### Recompiling After Changes
Because Python imports the compiled `.dll` binary directly, any changes you make to your C# source files (`.cs`) will **not** automatically update in Python. 

Whenever you alter your C# codebase, you must rebuild the library:
```bash
dotnet build csharp_src/MyCSharpLib.csproj
```

## Advanced Configuration & Features

### 1. Automating Builds in VS Code (`tasks.json`)

To stop running `dotnet build` manually every time you modify your C# code, you can configure VS Code to rebuild the C# library automatically whenever you run your Python script.

Create a folder named `.vscode` in your project root, and add a file named `tasks.json` inside it with the following configuration:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build C# Library",
            "type": "shell",
            "command": "dotnet build ${workspaceFolder}/csharp_src/MyCSharpLib.csproj",
            "group": "build",
            "problemMatcher": "$msCompile"
        }
    ]
}
```

Next, create a file named `launch.json` inside the same `.vscode` folder to link this build task to your Python debugger:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File with C# Build",
            "type": "debugpy",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "preLaunchTask": "Build C# Library"
        }
    ]
}
```
*Now, whenever you press **F5** while viewing `main.py`, VS Code will build your C# project before launching the Python script.*

---

### 2. Handling Complex Data Types

`pythonnet` automatically handles basic types like `int`, `string`, and `bool`. For more complex structures, use the following mapping strategies:

#### C# Implementation (`csharp_src/Class1.cs`)
Add these methods to your `WrapperDemo` class to see how data structures transfer:

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;

public class WrapperDemo
{
    // 1. Working with Lists/Arrays
    public List<string> ProcessList(List<string> inputs)
    {
        for (int i = 0; i < inputs.Count; i++)
        {
            inputs[i] = inputs[i].ToUpper();
        }
        return inputs;
    }

    // 2. Working with Dictionaries
    public Dictionary<string, int> GetScores()
    {
        return new Dictionary<string, int> { { "Alice", 95 }, { "Bob", 88 } };
    }

    // 3. Working with Async/Tasks
    public async Task<string> FetchDataAsync()
    {
        await Task.Delay(1000); // Simulate network work
        return "Async data fetched successfully";
    }
}
```

#### Python Implementation (`main.py`)
Import the native `System` namespaces from .NET within Python to handle collections and get the results of asynchronous Tasks:

```python
# Import native .NET Collection types
from System.Collections.Generic import List as NetList
from MyCSharpLib import WrapperDemo

instance = WrapperDemo()

# 1. Lists: Convert Python list to .NET Generic List
py_list = ["apple", "banana"]
net_list = NetList[str]()
for item in py_list:
    net_list.Add(item)

upper_net_list = instance.ProcessList(net_list)
print(list(upper_net_list))  # Output: ['APPLE', 'BANANA']

# 2. Dictionaries: Python handles C# Dictionaries seamlessly
scores = instance.GetScores()
print(scores["Alice"])  # Output: 95
print(dict(scores))     # Output: {'Alice': 95, 'Bob': 88}

# 3. Async Tasks: Use GetAwaiter().GetResult() to handle C# Tasks synchronously
async_task = instance.FetchDataAsync()
result = async_task.GetAwaiter().GetResult()
print(result)  # Output: Async data fetched successfully
```

---

### 3. Managing Third-Party NuGet Packages

If your C# project depends on external libraries (like `Newtonsoft.Json`), they will be restored and built into your workspace. However, you must make sure Python knows where to find them if they do not automatically load.

1. Add your package to the C# project terminal:
   ```bash
   dotnet add csharp_src/MyCSharpLib.csproj package Newtonsoft.Json
   ```
2. When you build the project via `dotnet build`, .NET generates a `.deps.json` file. `pythonnet` reads this file to locate dependencies, provided they are in the same folder as your main DLL. 
3. If a NuGet dependency fails to load with a `FileNotFoundException` in Python, add the dependency `.dll` path explicitly to your system path inside `main.py` before adding the reference:
   ```python
   # Example if a package dependency is isolated
   sys.path.append(os.path.abspath("./csharp_src/bin/Debug/net8.0/"))
   clr.AddReference("Newtonsoft.Json")
   ```