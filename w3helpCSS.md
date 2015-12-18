!important 规则1

根据 CSS2.1 规范中的描述，'!important' 可以提高样式的优先级，它对样式优先级的影响是巨大的。 注意，'!important' 规则在 IE7 以前的版本中是被支持不完善。因此，经常被用作 CSS hack2。

为了平衡开发者设置的样式和浏览器用户设定的样式，默认开发者的样式优于浏览器用户设置的样式；另外，声明了 '!important' 的样式优于普通声明。

注：

    关于 '!important' 的详细信息，请参考 CSS2.1 规范 6.4.2 !important rules 中的内容；
    见：
        W3Help - RY8003: 各浏览器对 CSS 错误解析规则的差异及 CSS hack
        W3Help - RA8003: IE6 IE7(Q) IE8(Q) 不完全支持 !important 规则
        CSDN - 【分享】CSS Hack的基本原理、常用CSS hack及使用原则

CSS 层叠
顺序( Cascading order )

为了找到元素/特性组合的值，用户端必须应用如下的排列顺序：

    对于目标媒介类型 (media type)，找到存有疑问的元素和属性的所有声明。如果相关联的选择器匹配存有疑问的元素，并且目标媒介匹配包含声明并且和样式表的路径在所有链接上达到一致的 @media 规则中所有的媒介列表。
    根据 CSS 样式的来源和重要性(是否含 !important )，给出了优先级的升序排列：
        用户端声明( UA declarations )
        一般用户声明( user normal declarations )
        一般作者声明( author normal declarations )
        加了 '!important' 的作者声明( author important declarations )
        加了 '!important' 的用户声明( user important declarations )
    拥有相同重要性和来源的规则，按照 CSS specificity 来排序。此处，需要注意一下层叠顺序和选择器的特殊性1的关系。 选择器的特殊性是在相同来源，相同重要性的规则之间判定最终哪个规则会起作用。 比如， 同是开发者自己定义的样式，并且，没有使用 "!important" 规则，这样的样式才可以计算特殊性。
    最后，根据先后次序来排列：如果两条规则具有相同的权重，相同的来源和相同的选择器特殊性，则后出现的规则超越先出现的规则。 引入的样式表( @import )中的规则被认为出现在样式表本身的所有规则之前。

例：

用户自定义样式，userContent.css：

p{
    background-color:black;
    color:red !important;
}

按照上节中的做法，给浏览器设置用户样式。

测试页面，test.html:

<style>
    p {
        color: green !important;
    }
</style>
<p style="background-color : white; color : blue;"> hello!! </p>

可自行测试结果。

注：

    选择器的特殊性，即下节的 Selector's specificity

计算选择器的特殊性( Selector's specificity )

正如上面提到的，根据层叠顺序，优先级相同的样式，如何判断哪一条声明会起作用，取决于对其选择器特殊性的计算值。

例如，都是作者样式，并且没有使用 '!important' 规则：
```HTML
<!DOCTYPE HTML>
<style type="text/css">
    div {
        width: 100px;
        height: 100px;
    }
    #c1 #c2 div.con {
        background-color: yellow;
    }
    div {
        background-color: black;
    }
    #c2 div {
        background-color: blue;
    }
    #c2 #content {
        background-color: red;
    }
</style>
<div id="c1">
    <div id="c2">
        <div id="content" class="con"></div>
    </div>
</div>
```
如上代码中，多个样式中的 'background-color' 同时作用于 content，那么最后，到底 content 应该会是什么颜色呢？

这就应该求助于选择器特殊性的计算规则了。

特殊性的值可以看作是一个由四个数组成的一个组合，用 a，b，c，d 来表示它的四个位置。 依次比较 a，b，c，d 这个四个数比较其特殊性的大小。比如，a 值相同，那么 b 值大的组合特殊性会较大，以此类推。 注意，W3C 中并不是把它作为一个 4 位数来看待的。

a，b，c，d 值的确定规则：

    如果 HTML 标签的 'style' 属性中该样式存在，则记 a 为 1；
    数一下选择器中 ID 选择器的个数作为 b 的值。比如，以上样式中包含 '#c1' 和 '#c2' 的选择器；
    其他属性以及伪类（pseudo-classes）的总数量是 c 的值。比如，上面例子中的 '.con'，':hover' 等；
    元素名和伪元素的数量是 d 的值；比如上面例子中的 ‘div’。

现在，应用上面的规则，计算例子中各个样式的特殊性的值，结果为：
```HTML
<!DOCTYPE HTML>
<style type="text/css">
    div {
        width: 100px;
        height: 100px;
    }
    #c1 #c2 div.con {     /* a=0 b=2 c=1 d=1 -> specificity = 0,2,1,1 */
        background-color: yellow;
    }
    div {                 /* a=0 b=0 c=0 d=1 -> specificity = 0,0,0,1 */
        background-color: black;
    }
    #c2 div {             /* a=0 b=1 c=0 d=1 -> specificity = 0,1,0,1 */
        background-color: blue;
    }
    #c2 #content {        /* a=0 b=2 c=0 d=0 -> specificity = 0,2,0,0 */
        background-color: red;
    }
</style>
<div id="c1">
    <div id="c2">
        <div id="content" class="con"></div>
    </div>
</div>
```
可见，'#c1 #c2 div.con" 的特殊性( [0,2,1,1] )最高，是背景色应该是黄色。
```HTML
W3C 官方给出的例子：

*             {}  /* a=0 b=0 c=0 d=0 -> specificity = 0,0,0,0 */
li            {}  /* a=0 b=0 c=0 d=1 -> specificity = 0,0,0,1 */
li:first-line {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
ul li         {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
ul ol+li      {}  /* a=0 b=0 c=0 d=3 -> specificity = 0,0,0,3 */
h1 + *[rel=up]{}  /* a=0 b=0 c=1 d=1 -> specificity = 0,0,1,1 */
ul ol li.red  {}  /* a=0 b=0 c=1 d=3 -> specificity = 0,0,1,3 */
li.red.level  {}  /* a=0 b=0 c=2 d=1 -> specificity = 0,0,2,1 */
#x34y         {}  /* a=0 b=1 c=0 d=0 -> specificity = 0,1,0,0 */
style=""          /* a=1 b=0 c=0 d=0 -> specificity = 1,0,0,0 */
------------------------------------------------------------------------
<HEAD>
<STYLE type="text/css">
 #x97z { color: red }
</STYLE>
</HEAD>
<BODY>
<P ID=x97z style="color: green">
</BODY>
```
如上代码中，P 中字体的颜色应该为绿色。
非 CSS 显示的优先级

用户端应该优先考虑那些不是来自样式表的显示属性。如果是这样，这些非CSS呈现提示必须被转换到相应的CSS规则， 且其特殊性为零( 既，使其特殊性最低 )，并认为它的引入处于样式表的最顶端。它们可能被后续的样式规则所覆盖，

在 HTML 中，任何不属于以下列表中的属性都需要以显示属性对待：abbr, accept-charset, accept, accesskey, action, alt, archive, axis, charset, checked, cite, class, classid, code, codebase, codetype, colspan, coords, data, datetime, declare, defer, dir, disabled, enctype, for, headers, href, hreflang, http-equiv, id, ismap, label, lang, language, longdesc, maxlength, media, method, multiple, name, nohref, object, onblur, onchange, onclick, ondblclick, onfocus, onkeydown, onkeypress, onkeyup, onload, onload, onmousedown, onmousemove, onmouseout, onmouseover, onmouseup, onreset, onselect, onsubmit, onunload, onunload, profile, prompt, readonly, rel, rev, rowspan, scheme, scope, selected, shape, span, src, standby, start, style, summary, title, type (except on LI, OL and UL elements), usemap, value, valuetype, version.

例如，对于 align1 属性，各浏览器应当将它转换成 CSS 中的浮动。