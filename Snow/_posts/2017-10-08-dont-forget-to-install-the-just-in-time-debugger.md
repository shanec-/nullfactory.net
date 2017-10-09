---
layout: post
title: Don't Forget to Install the Just-In-Time Debugger
category: Visual Studio, Miscellaneous
---

Today I learned that you can have an installation of Visual Studio 2017 without the JIT debugger - Yup, you really can.  This meant that I could not use the technique of attaching a debugger at runtime using the  `System.Diagnostics.Debugger.Launch();` code snippet. 

When the application executes the launch code, nothing happens - no error nor exception. The method even returns `true` indicating a a successful launch.

<!--excerpt-->

After equal parts of scouring the web and pulling my hair out, I eventually stumbled upon the following option in the Visual Studio setup:

![Install Debugger](/images/posts/JITDebugger/10_installdebugger.png)

The combination of features selected had somehow unchecked the Just-In-Time debugger. Heck, I didn't even know you could ***not*** install it. I hope this helps someone else out there.

![Choose Debugger](/images/posts/JITDebugger/20_choosejitdebugger.png)

## References

- [Debugger.Launch Method (System.Diagnostics)](https://msdn.microsoft.com/en-us/library/system.diagnostics.debugger.launch(v=vs.110).aspx)
- [The wonders of Debugger.Launch() | Mark's Devblog](http://defragdev.com/blog/?p=668)
- [Debugger.Launch() not displaying JIT debugger selection popup on Windows 8/8.1 – Scattered Notes](https://blogs.msdn.microsoft.com/mapo/2013/11/07/debugger-launch-not-displaying-jit-debugger-selection-popup-on-windows-88-1/)
- [c# - Debugger.Launch not working - Stack Overflow](https://stackoverflow.com/questions/12655965/debugger-launch-not-working)
- [Mysterious VS JIT debugger invocation failure](https://social.msdn.microsoft.com/Forums/vstudio/en-US/9a539c40-1869-44b8-9aed-9f9d7d402d7c/mysterious-vs-jit-debugger-invocation-failure?forum=vsdebug)