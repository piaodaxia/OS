#In memory detection of Windows API call hooking technique

本文发表于Computer, Communications, and Control Technology (I4CT), 2015 International Conference。

###摘要
API call hooking 是一种恶意代码研究者使用的挖掘恶意代码调用的API的方法。在分析、分类或者检测恶意代码时，API调用经常代表恶意代码的行为。文中提供分析目前Windows API call hooking技术，各检测技术一般可以在内于存里。在运行时，此可以使恶意代码感觉到API call hooking技术的存在，并改变其恶意代码的行为。

###文中提出的三种用户模式中的API call hooking
-	Import Address Table Hook : 该技术吧PE文件结构中的IAT改成任意数值，在IAT当中有其文件需要调用的函数的内存地址（即指针），通过改变其地址在运行软件是可以执行任意代码。
-	Debugger Hook : 该技术在调试软件时，通过运行debugger进行hooking，debugger可以通过设置API entry point的breakpoint进行hooking（如IA32中的INT3）。 当当前指令指针指向指令时，处理器导致调试异常的BP可以覆盖API的entry point。 该异常将被调试器捕获，发生异常的地址用于识别软件访问任何API。
-	Inline Hook ：给技术修改API的entry point，此时把entry point改成重定向执行代码（即Detour Function）。由于一旦修改entry point不能执行原有的代码，因此再重定向另外一个执行代码（即Trampoline Function），此代码的作用是重定向原有的代码。

###结论
该技术不能可预测的改变内存的一部分，也不能修改其软件的运行环境，而在很难平凡检测的情况下，一定要可以重定向从使用非凡方法的API的执行代码。如果一定要修改内存领域，则一定要保持API原有的功能。
