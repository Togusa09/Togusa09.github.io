---
layout: post
title: Converting the dotnet console to support double buffering
categories: dotnet
---

Dot net console apps are useful for rapidly creating and testing concepts, and it is even possible to fairly rapidly produce a tetris clone or text based RPG with a rudimentary UI. However the console has flickering issues when you try redrawing sections of the screen too frequently, due to content being drawn straight to the screen. This is a known problem with 2D/3D applications, and is solved through a technique called double buffering, where the image is drawn to a second buffer, then the display is then swapped to displaying this second buffer. My investigation here is to find a robust and easy to use way to provide this kind of buffering to the dot net console.

For this investigation, I'm using dot net core,  as I assumed that there was greater chance of the console working in a more flexible way.

From the Console [documentation](https://docs.microsoft.com/en-us/dotnet/api/system.console?view=netcore-2.0), the console allows it's output streamwriter to be overwriten by setting its 'Out' property to a new streamwriter. This can be used to output the console to a file or other destination stream.

I thought it may be possible to override this with a new class that allows me to buffer all the data sent to console.Write/WriteLine, then send it on to the actual console stream in one go.

Unfortunately the console implementation is static, and depends on a lot of internal methods and classes for its setup ([Console Source code](https://github.com/dotnet/corefx/blob/master/src/System.Console/src/System/Console.cs) ). These can of course be retrieved via reflection, but at some point it's no longer worth the effort, and becomes easier to just copy and paste the existing code, then alter it to your needs.

As we can see from the `CreateOutputWriter` method, the default stream writer is set to autoflush, which causes it to write to the stream automatically after text is sent to the writer. We want to try and replacy this with a streamwriter that we can manually flush. The default StreamWriter is also wrapped in a SyncTextWriter, another internal class that is a [wrapper class](https://github.com/dotnet/corefx/blob/6dd451f51451a7d0ceea6104b51bd17005e9a0e6/src/System.Console/src/System/IO/SyncTextWriter.cs) providing a lock on all the methods to protect against concurrent access. For this investigation, I decided to just leave it out, as my use case is still very simple.

Get hold of the consoles output stream. Console Pal is an abstraction around platform specific console creation. This looks like it's a holdover from mono and isn't public, so may be removed in future.

```
static void Main(string[] args)
{
    // Get output stream from internal console pal class. A lott of the console methods just call this class.
    //var consolePalType = typeof(Console).Assembly.GetType("System.ConsolePal");
    //MethodInfo info = consolePalType.GetMethod(
    //    "OpenStandardOutput",
    //    BindingFlags.NonPublic | BindingFlags.Public | BindingFlags.Static | BindingFlags.FlattenHierarchy);
    //var inputStream = (Stream)info.Invoke(null, new object[] { });

    // Get output stream from console
    var inputStream = Console.OpenStandardOutput();

    var writer = new StreamWriter(inputStream);
    
    writer.Write("TEST MESSAGE");
    writer.Flush();
}```

This allows writing with a controlled flush, but doesn't yet provide access to any of the more sophisticated features of the console, which this method of just replacing the stream starts falling apart: All the lower level Console functions are done through Interop, so things like changing colour can't be queued into the stream buffer to execute during the flush.

My original goal was to try and find a way to leverage the existing console methods to create a double buffered output. While it appears that this would be possible for a pure stream of text, but would not support more advanced features like changing text colour and moving the write position as they are not send via the stream.

I win continue investigating implementing a buffered console in future, either by reimplementing the buffer functionality in a new class with buffering support, or by creating a wrapper class that buffers commnds and content before sending them to the actual console on flush.