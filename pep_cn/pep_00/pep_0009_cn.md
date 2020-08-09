
PEP: 9
Title: Sample Plaintext PEP Template
Version: $Revision$
Last-Modified: $Date$
Author: Barry Warsaw <barry@python.org>
Status: Withdrawn
Type: Process
Content-Type: text/x-rst
Created: 14-Aug-2001
Post-History:
Resolution: https://mail.python.org/mailman/private/peps/2016-January/001165.html

::
  Update

      截至2016年1月5日, PEP 9 正式弃用并由 PEP 12 取代
      所有 PEP 现在应使用 PEP 12 描述的 reStructuredText 格式, 并且将不再接受明文 PEP

  Abstract

      此 PEP 提供了用于创建您自己的纯文本 PEP 的样板或样本模板
      结合 PEP 1 [1] 中的内容指南, 这可以使您轻松地将自己的 PEP 符合下面列出的格式

      注意: 如果您通过网络阅读此 PEP, 您应首先获取此 PEP 的明文来源, 以完成以下步骤
      不要使用 HTML 文件作为您的模板!
    
      如果要获取此(或任何) PEP 的来源, 请查看 HTML 页面的顶部, 然后单击 "Last-Modified" 行的日期和时间
      它是 Python 仓库中源文本的链接

      如果您希望在 PEP 中使用轻量级标记, 请参阅 PEP 12, "示例 reStructuredText PEP 模板" [2]

  Rationale

      PEP 提交的形式多种多样, 并非所有形式都遵循下面列出的格式指南
      将此模板与 PEP 1 中的内容指南结合使用, 以确保您的 PEP 提交不会因为表单而自动被拒绝

  How to Use This Template

      要使用此模板, 您必须首先确定您的 PEP 是否将成为 Informational 或 Standards Track PEP
      大多数 PEP 都是 Standards Track, 因为它们为 Python 语言或标准库提出了一个新功能
      如有疑问, 请阅读 PEP 1 了解详情或联系 PEP 编辑 <peps@python.org>

      一旦您确定了您的 PEP 类型, 请按照以下说明操作

      - 复制此文件 (.txt 文件, 而不是 HTML), 并执行以下编辑

      - 将 "PEP: 9" 标题替换为 "PEP: XXX", 因为您尚未进行 PEP 编号分配

      - 将标题更改为您的 PEP 标题

      - 保留 Version 和 Last-Modified 标题头;
        当我们将您的 PEP 检查到 Python 的 Subversion 仓库时, 我们会处理这些问题
        这些标题由关键字 ("修订版" 和 "日期" 包含在 "$" - 符号中) 组成, 这些关键字由存储库自动扩展
        请不要编辑扩展日期或修订文本

      - 更改作者标题以包含您的姓名, 以及您的电子邮件地址
        请务必仔细遵循格式: 您的名字必须首先出现, 并且不得包含在括号中
        您的电子邮件地址可能会显示在第二位 (或者可以省略), 如果显示, 则必须显示在尖括号中
        模糊您的电子邮件地址是可以的

      - 如果有用于讨论新功能的邮件列表, 请在 Author 标题后面添加一个 Discussions-To 标题
        如果要使用的邮件列表是 python-list@python.org 或 python-dev@python.org,
        或者应该直接向您发送讨论, 则不应添加 Discussions-To 标题头
        大多数 Informational PEPs 没有 Discussions-To 标题

      - 将状态标题更改为 "Draft"

      - 对于 Standards Track PEP, 将 Type 标题更改为 "Standards Track"

      - 对于 Informational PEPs, 将 Type 标题更改为 "Informational"

      - 对于 Standards Track PEP, 如果您的功能取决于其他当前正在开发的 PEP 的接受程度,
        请在 Type 标题头后面添加 Requires 标题头, 该值应该是您所依赖的 PEP 的 PEP 编号
        如果在最终 PEP 中描述了相关功能, 请不要添加此标题头

      - 将 Created 标题更改为今天的日期, 请务必仔细遵循格式: 它必须采用dd-mmm-yyyy格式,
        其中 mmm 是月份缩写-3个英文字母, 例如 Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec 之一

      - 对于 Standards Track PEP, 在 Created 标题之后, 添加一个 Python-Version 标题头,
        并将值设置为下一个计划版本的 Python, 即您的新功能有望首次出现的版本, 不要使用 alpha 或 beta 版本
        因此, 如果 Python 的最后一个版本是 2.2 alpha 1, 并且您希望将新功能添加到 Python 2.2 中, 请将标题设置为:

        Python-Version: 2.2
      
      - 暂时留下 Post-History; 每次将 PEP 发布到 python-list@python.org 或 python-dev@python.org 时, 都会在此标题中添加日期
        例如, 如果您在2001年8月14日和2001年9月3日将您的PEP发布到列表中, 则 Post-History 标题将如下所示:
      
        Post-History: 14-Aug-2001, 03-Sept-2001

        您必须手动添加新日期并将其签入, 如果您没有签入权限, 请将更改发送到 PEP 编辑器

      - 如果您的 PEP 废弃了早期的 PEP, 请添加替换标题头
        此标题头的值是新 PEP 要替换的 PEP 的编号
        如果较旧的 PEP 处于 "final" 形式, 即仅接受, 最终或拒绝, 则仅添加此标题
        如果您提交的是竞争性想法, 那么您不会更换旧的开放式 PEP

      - 


