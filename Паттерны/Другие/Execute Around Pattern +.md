#otherpatterns
Encapsulating Pre and Post Execution Steps. This pattern is particularly useful for resource management, such as handling files, databases, or network connections

```cs
using System;


public class FileWriter
{
    public FileWriter(string filename) { }
    public void Write(string text) { }
}


public delegate void FileWriterAction(FileWriter writer);

public class SimpleFileWriter
{
    public SimpleFileWriter(string filename, FileWriterAction action)
    {
        var writer = new FileWriter(filename);
        Console.WriteLine("executing the action");
        action(writer);
        Console.WriteLine("closing the file");
    }    
}


public class ShotEffect
{
    public void Test()
    {

        /*
        A class needs to be provided for writing text strings to files. 
        To make it easy for the user, the service class opens and closes the file automatically. 
        The user only has to specify what is written into which file.
        */
        FileWriterAction helloDelegate = (w) => w.Write("Hello World");
        new SimpleFileWriter("some.txt", helloDelegate);
    }
}
```

