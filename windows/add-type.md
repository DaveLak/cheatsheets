### Compile and run cs code in memory
```
$sourceCode = @"
    using System.IO;
    public class ClassName
    {
        public static void Test()
        {
            Directory.CreateDirectory("newDir");
        }
    }
"@

Add-Type -TypeDefinition $sourceCode  
[ClassName]::Test()  
```

