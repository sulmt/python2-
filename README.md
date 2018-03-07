老是遇到编码问题，快折腾死了，找到一篇不错的，转载一下

edu.codepub.com/2009/1029/17037.php
这个问题在python3.0里已经解决了。

这有篇很好的文章，可以明白这个问题:
为什么会报错“UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)”？本文就来研究一下这个问题。字符串在Python内部的表示是unicode编码，因此，在做编码转换时，通常需要以unicode作为中间编码，即先将其他编码的字符串解码（decode）成unicode，再从unicode编码（encode）成另一种编码。
decode的作用是将其他编码的字符串转换成unicode编码，如str1.decode('gb2312')，表示将gb2312编码的字符串str1转换成unicode编码。
encode的作用是将unicode编码转换成其他编码的字符串，如str2.encode('gb2312')，表示将unicode编码的字符串str2转换成gb2312编码。
因此，转码的时候一定要先搞明白，字符串str是什么编码，然后decode成unicode，然后再encode成其他编码
代码中字符串的默认编码与代码文件本身的编码一致。
如：s='中文'
如果是在utf8的文件中，该字符串就是utf8编码，如果是在gb2312的文件中，则其编码为gb2312。这种情况下，要进行编码转换，都需 要先用decode方法将其转换成unicode编码，再使用encode方法将其转换成其他编码。通常，在没有指定特定的编码方式时，都是使用的系统默 认编码创建的代码文件。
如果字符串是这样定义：s=u'中文'
则该字符串的编码就被指定为unicode了，即python的内部编码，而与代码文件本身的编码无关。因此，对于这种情况做编码转换，只需要直接使用encode方法将其转换成指定编码即可。
如果一个字符串已经是unicode了，再进行解码则将出错，因此通常要对其编码方式是否为unicode进行判断：
isinstance(s, unicode) #用来判断是否为unicode
用非unicode编码形式的str来encode会报错
如何获得系统的默认编码？
#!/usr/bin/env python #coding=utf-8 import sys print sys.getdefaultencoding()
该段程序在英文WindowsXP上输出为：ascii
在某些IDE中，字符串的输出总是出现乱码，甚至错误，其实是由于IDE的结果输出控制台自身不能显示字符串的编码，而不是程序本身的问题。
如在UliPad中运行如下代码：
s=u"中文"print s
会提示：UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)。这是因为UliPad在英文WindowsXP上的控制台信息输出窗口是按照ascii编码输出的（英文系统的默认编码是 ascii），而上面代码中的字符串是Unicode编码的，所以输出时产生了错误。
将最后一句改为：print s.encode('gb2312')
则能正确输出“中文”两个字。
若最后一句改为：print s.encode('utf8')
则输出：\xe4\xb8\xad\xe6\x96\x87，这是控制台信息输出窗口按照ascii编码输出utf8编码的字符串的结果。
unicode(str,'gb2312')与str.decode('gb2312')是一样的，都是将gb2312编码的str转为unicode编码
使用str.__class__可以查看str的编码形式
>>>>>groups.google.com/group/python-cn/browse_thread/thread/be4e4e0d4c3272dd -----python是个容易出现编码问题的语言。所以，我按照我的理解写下下面这些文字。 =首先，要了解几个概念。=  *字节：计算机数据的表示。8位二进制。可以表示无符号整数：0-255。下文，用“字节流”表示“字节”组成的串。 *字符：英文字符“abc”，或者中文字符“你我他”。字符本身不知道如何在计算机中保存。下文中，会避免使用“字符串”这个词，而用“文本”来表  示“字符”组成的串。 *编码（动词）：按照某种规则（这个规则称为：编码（名词））将“文本”转换为“字节流”。（在python中：unicode变成str） *解码（动词）：将“字节流”按照某种规则转换成“文本”。（在python中：str变成unicode） **实际上，任何东西在计算机中表示，都需要编码。例如，视频要编码然后保存在文件中，播放的时候需要解码才能观看。 unicode：unicode定义了，一个“字符”和一个“数字”的对应，但是并没有规定这个“数字”在计算机中怎么保存。（就像在C中，一个整数既  可以是int，也可以是short。unicode没有规定用int还是用short来表示一个“字符”） utf8：unicode实现。它使用unicode定义的“字符”“数字”映射，进而规定了，如何在计算机中保存这个数字。其它的utf16等都是 unicode实现。 gbk：类似utf8这样的“编码”。但是它没有使用unicode定义的“字符”“数字”映射，而是使用了另一套的映射方法。而且，它还定义了如何在  计算机中保存。 
=python中的encode，decode方法= 首先，要知道encode是 unicode转换成str。decode是str转换成unicode。  下文中，u代表unicode类型的变量，s代表str类型的变量。 u.encode('...')基本上总是能成功的，只要你填写了正确的编码。就像任何文件都可以压缩成zip文件。 s.decode('...')经常是会出错的，因为str是什么“编码”取决于上下文，当你解码的时候需要确保s是用什么编码的。就像，打开zip文  件的时候，你要确保它确实是zip文件，而不仅仅是伪造了扩展名的zip文件。 u.decode(),s.encode()不建议使用，s.encode相当于s.decode().encode()首先用默认编码（一般是 ascii）转换成unicode在进行encode。 
=关于#coding=utf8= 当你在py文件的第一行中，写了这句话，并确实按照这个编码保存了文本的话，那么这句话有以下几个功能。 1.使得词法分析器能正常运作，对于注释中的中文不报错了。 2.对于u"中文"这样literal string能知道两个引号中的内容是utf8编码的，然后能正确转换成unicode  3."中文"对于这样的literal string你会知道，这中间的内容是utf8编码，然后就可以正确转换成其它编码或unicode了。 
没有写完，先码那么多字，以后再来补充，这里不是wiki，太麻烦了。
>>>>>>>>>>  =Python编码和Windows控制台= 我发现，很多初学者出错的地方都在print语句，这牵涉到控制台的输出。我不了解linux，所以只说控制台的。  首先，Windows的控制台确实是unicode（utf16_le编码）的，或者更准确的说使用字符为单位输出文本的。  但是，程序的执行是可以被重定向到文件的，而文件的单位是“字节”。  所以，对于C运行时的函数printf之类的，输出必须有一个编码，把文本转换成字节。可能是为了兼容95，98，  没有使用unicode的编码，而是mbcs（不是gbk之类的）。 windows的mbcs，也就是ansi，它会在不同语言的windows中使用不同的编码，在中文的windows中就是gb系列的编码。  这造成了同一个文本，在不同语言的windows中是不兼容的。  现在我们知道了，如果你要在windows的控制台中输出文本，它的编码一定要是“mbcs”。  对于python的unicode变量，使用print输出的话，会使用sys.getfilesystemencoding()返回的编码，把它变成str。  如果是一个utf8编码str变量，那么就需要 print s.decode('utf8').encode('mbcs') 最后，对于str变量，file文件读取的内容，urllib得到的网络上的内容，都是以“字节”形式的。  它们如果确实是一段“文本”，比如你想print出来看看。那么你必须知道它们的编码。然后decode成unicode。  如何知道它们的编码： 1.事先约定。（比如这个文本文件就是你自己用utf8编码保存的） 2.协议。（python文件第一行的#coding=utf8，html中的<meta>等） 3.猜。 >>>>>。 
