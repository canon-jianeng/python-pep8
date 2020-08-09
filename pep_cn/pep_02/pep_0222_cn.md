
PEP: 222
Title: Web Library Enhancements
Version: $Revision$
Last-Modified: $Date$
Author: A.M. Kuchling <amk@amk.ca>
Status: Deferred
Type: Standards Track
Content-Type: text/x-rst
Created: 18-Aug-2000
Python-Version: 2.1
Post-History: 22-Dec-2000


Abstract
========

该 PEP 为 Python 标准库中的 CGI 开发工具提出了一组增强功能.
增强功能可能是新功能，用于cookie支持等任务的新模块，或删除过时的代码.

最初的目的是对 Python 2.1 进行改进. 然而, Python 社区似乎没什么兴趣, 缺乏时间,
所以这个 PEP 已经推迟到将来的 Python 版本中.


Open Issues
===========

本节列出了已建议的更改, 但尚未做出明确的决定.
在本 PEP 的最终版本中, 此部分应该为空, 因为所有更改都应该归类为已接受或已拒绝.

cgi.py: 我们不应该被告知要创建我们自己的子类, 以便我们可以处理文件上传.
实际上, 我还没有找到时间做正确的事情, 所以我最终将 cgi.py 的临时文件读入最好的另一个文件.
我们的一些遗留代码实际上将其读入第二个临时文件, 然后进入最终目标!
即使我们这样做, 也意味着用 ``__init__`` 调用和相关的开销创建另一个对象.

cgi.py: 目前, 忽略没有 "=" 的查询数据. 即使设置了``keep_blank_values``,
像``...？value =＆...``之类的查询也会返回空白值, 但像``...？value＆...``这样的查询会完全丢失.
如果通过``FieldStorage``接口提供这样的数据, 或者作为值为 "None" 的条目,
或者在单独的列表中, 这将是很好的.

Utility function: 从2元组列表中构建查询字符串

与字典相关的实用程序类:
``NoKeyErrors`` (返回一个空字符串, 从不返回``KeyError``),
``PartialStringSubstitution`` (返回原始键字符串, 从不返回``KeyError``)



New Modules
===========

本节列出了有关应该添加到 Python 标准库的整个新包或模块的详细信息.

* fcgi.py : 一个新模块, 增加了对 FastCGI 协议的支持.
  不过, Robin Dunn 的代码需要移植到 Windows.


Major Changes to Existing Modules
=================================

本节列出了现有模块的主要更改的详细信息, 无论是在实现中还是在接口中.
因此, 本节中的更改会带来更大程度的风险, 无论是引入错误还是后向不兼容.

不推荐使用 cgi.py模块.
(XXX 尚未选择新模块或包名: 'web'? 'cgilib'?)


Minor Changes to Existing Modules
=================================

本节列出了对现有模块的微小更改的详细信息.
这些更改应该具有相对较小的实现, 并且几乎没有引入与先前版本的不兼容性的风险.


Rejected Changes
================

本节中列出的更改是针对 Python 2.1 提出的, 但被拒绝为不合适.
对于每个被拒绝的变更, 给出了一个理由, 说明为什么变更被认为是不合适的.

* HTML 生成模块不是此 PEP 的一部分. 存在几个这样的模块,
  从 HTMLgen 的纯编程接口到 ASP 启发的简单模板到 DTML 的复杂模板.
  没有迹象表明要在标准库中加入哪个模板模块, 这可能意味着不应该选择任何模块.

* cgi.py: 允许查询数据和 POST 数据的组合.
  这似乎根本不是标准, 因此是可疑的做法.


Proposed Interface
==================

XXX open issues: 命名约定 (studlycaps 或下划线?);
需要查看 ``cgi.parse*()`` 函数, 看看它们是否也可以简化.

解析函数: 从 cgi.py 中继承大多数 ``parse*`` 函数

::

    # The Response class borrows most of its methods from Zope's
    # HTTPResponse class.

    class Response:
        """
        Attributes:
        status: HTTP status code to return
        headers: dictionary of response headers
        body: string containing the body of the HTTP response
        """

        def __init__(self, status=200, headers={}, body=""):
            pass

        def setStatus(self, status, reason=None):
            "Set the numeric HTTP response code"
            pass

        def setHeader(self, name, value):
            "Set an HTTP header"
            pass

        def setBody(self, body):
            "Set the body of the response"
            pass

        def setCookie(self, name, value,
                      path = '/',
                      comment = None,
                      domain = None,
                      max-age = None,
                      expires = None,
                      secure = 0
                      ):
            "Set a cookie"
            pass

        def expireCookie(self, name):
            "Remove a cookie from the user"
            pass

        def redirect(self, url):
            "Redirect the browser to another URL"
            pass

        def __str__(self):
            "Convert entire response to a string"
            pass

        def dump(self):
            "Return a string representation useful for debugging"
            pass

        # XXX methods for specific classes of error:serverError,
        # badRequest, etc.?


    class Request:

        """
        Attributes:

        XXX should these be dictionaries, or dictionary-like objects?
        .headers : dictionary containing HTTP headers
        .cookies : dictionary of cookies
        .fields  : data from the form
        .env     : environment dictionary
        """

        def __init__(self, environ=os.environ, stdin=sys.stdin,
                     keep_blank_values=1, strict_parsing=0):
            """Initialize the request object, using the provided environment
            and standard input."""
            pass

        # Should people just use the dictionaries directly?
        def getHeader(self, name, default=None):
            pass

        def getCookie(self, name, default=None):
            pass

        def getField(self, name, default=None):
            "Return field's value as a string (even if it's an uploaded file)"
            pass

        def getUploadedFile(self, name):
            """Returns a file object that can be read to obtain the contents
            of an uploaded file.  XXX should this report an error if the
            field isn't actually an uploaded file?  Or should it wrap
            a StringIO around simple fields for consistency?
            """

        def getURL(self, n=0, query_string=0):
            """Return the URL of the current request, chopping off 'n' path
            components from the right.  Eg. if the URL is
            "http://foo.com/bar/baz/quux", n=2 would return
            "http://foo.com/bar".  Does not include the query string (if
            any)
            """

        def getBaseURL(self, n=0):
            """Return the base URL of the current request, adding 'n' path
            components to the end to recreate more of the whole URL.

            Eg. if the request URL is
            "http://foo.com/q/bar/baz/qux", n=0 would return
            "http://foo.com/", and n=2 "http://foo.com/q/bar".

            Returned URL does not include the query string, if any.
            """

        def dump(self):
            "String representation suitable for debugging output"
            pass

        # Possibilities?  I don't know if these are worth doing in the
        # basic objects.
        def getBrowser(self):
            "Returns Mozilla/IE/Lynx/Opera/whatever"

        def isSecure(self):
            "Return true if this is an SSLified request"


    # Module-level function
    def wrapper(func, logfile=sys.stderr):
        """
        Calls the function 'func', passing it the arguments
        (request, response, logfile).  Exceptions are trapped and
        sent to the file 'logfile'.
        """
        # This wrapper will detect if it's being called from the command-line,
        # and if so, it will run in a debugging mode; name=value pairs
        # can be entered on standard input to set field values.
        # (XXX how to do file uploads in this syntax?)


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
