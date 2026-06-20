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