---
layout: post
title: 'Writing plug-ins for ReSharper: Part 2 of N'
categories: []
tags:
- ReSharper
status: publish
type: post
published: true
meta:
  _elasticsearch_indexed_on: '2010-07-20 11:11:00'
---
<p>Finally I’ve managed to get the second part of the post on plug-ins. Sorry for the delay to everyone that was waiting. Appreciate your patience.&nbsp; And now we’ll resume my holidays!</p> <p>In the <a href="http://devlicio.us/blogs/hadi_hariri/archive/2010/01/12/writing-plug-ins-for-resharper-part-1-of-undefined.aspx">previous part of this series</a>, we saw the basics of how to create a plug-in for <a href="http://www.jetbrains.com/resharper">ReSharper</a>, install it and run it. We created a context action that would allow us to mark a public method as virtual (where applicable). However, this was done as an explicit action by the user, as such, you didn’t get any kind of hint or suggestion to do this. What we want to do now is make this a hint, so that highlighting appears under methods that could be made virtual. In this part we are going to expand on the same plug-in and convert it into a <em>QuickFix</em>.</p> <h3>What is a <em>QuickFix?</em></h3> <p>Have you seen the little squiggly lines that appear in Visual Studio?</p> <p><a href="http://hhariri.files.wordpress.com/2010/11/118.png"><img style="border-bottom:0;border-left:0;display:inline;border-top:0;border-right:0;" title="1" border="0" alt="1" src="http://hhariri.files.wordpress.com/2010/11/1_thumb4.png" width="381" height="189"></a> </p> <p>They usually indicate a Suggestion (field can be made read-only), Warning (possible null reference) or Error. ReSharper analyzes and can detect potential issues in the code (similar to what static checker of Code Contracts does). These are known as Highlights and they are related to QuickFixes in that usually a highlight has an <strong>QuickFix</strong> associated to it, which invokes a context action. This is usually done by placing the cursor on top of the highlighting and press Alt+Enter</p> <p><a href="http://hhariri.files.wordpress.com/2010/11/215.png"><img style="border-bottom:0;border-left:0;display:inline;border-top:0;border-right:0;" title="2" border="0" alt="2" src="http://hhariri.files.wordpress.com/2010/11/2_thumb3.png" width="509" height="145"></a> </p> <p>&nbsp;</p> <h3>Highlighting Daemons</h3> <p>In the gutter of the Visual Studio editor (right-side), ReSharper displays a series of warnings, errors and hints, which indicate potential issues on a specific file. These issues are detected by background processes known as Daemons. Since what we are looking for is for ReSharper to warn us of existing methods that could be made virtual, what we need to do is somehow hook into these daemons.</p> <p><a href="http://hhariri.files.wordpress.com/2010/11/314.png"><img style="border-bottom:0;border-left:0;display:inline;border-top:0;border-right:0;" title="3" border="0" alt="3" src="http://hhariri.files.wordpress.com/2010/11/3_thumb3.png" width="66" height="249"></a> </p> <p>&nbsp;</p> <h3>Step by Step Guide</h3> <p>The Daemons in ReSharper use the Visitor pattern to use act on elements, be it code, files, etc. The first step is to implement an <strong>IDaemonStage </strong>interface, which hold metadata about our daemon stage at at the same time acts as a factory for the actual process we are implementing.</p> <div style="display:inline;float:none;margin:0;padding:0;" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:862f1f96-7fa7-42fd-b72d-f48172f17579" class="wlWriterSmartContent"> <div class="le-pavsc-container"> <div style="background-color:#000000;white-space:nowrap;overflow:auto;padding:2px 5px;"><span style="color:#ffffff;">[</span><span style="color:#ffc66d;">DaemonStage</span><span style="color:#ffffff;">(StagesBefore = </span><span style="color:#cc7832;">new</span><span style="color:#ffffff;">[]&nbsp; { </span><span style="color:#cc7832;">typeof</span><span style="color:#ffffff;">(</span><span style="color:#ffc66d;">LanguageSpecificDaemonStage</span><span style="color:#ffffff;">) })]</span><br>&nbsp;<span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">class</span><span style="color:#ffffff;"> </span><span style="color:#ffc66d;">MakeMethodVirtualDaemonStage</span><span style="color:#ffffff;">: </span><span style="color:#6897bb;">IDaemonStage</span><br>&nbsp;<span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#6897bb;">IDaemonStageProcess</span><span style="color:#ffffff;"> CreateProcess(</span><span style="color:#6897bb;">IDaemonProcess</span><span style="color:#ffffff;"> process, </span><span style="color:#6897bb;">DaemonProcessKind</span><span style="color:#ffffff;"> processKind)</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">return</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">new</span><span style="color:#ffffff;"> </span><span style="color:#ffc66d;">MakeMethodVirtualDaemonStageProcess</span><span style="color:#ffffff;">(process);</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#6897bb;">ErrorStripeRequest</span><span style="color:#ffffff;"> NeedsErrorStripe(</span><span style="color:#6897bb;">IProjectFile</span><span style="color:#ffffff;"> projectFile)</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">return</span><span style="color:#ffffff;"> </span><span style="color:#6897bb;">ErrorStripeRequest</span><span style="color:#ffffff;">.STRIPE_AND_ERRORS;</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br>&nbsp;<span style="color:#ffffff;">}</span></div></div></div> <p>&nbsp;</p> <p>There are two main methods to implement. The <strong>CreateProcess </strong>is what creates the actual process for us and the <strong>NeedsErrorStrip </strong>which indicates whether this daemon uses the gutter to display strips. The <strong>DaemonProcessKind</strong> parameter passed into the first method helps us discriminate on when this process should be executed, i.e. only during checking of visible (current) document, during solution wide analysis, etc.</p> <p>The next step is to implement the process via the <strong>IDaemonStageProcess </strong>interface:</p> <div style="display:inline;float:none;margin:0;padding:0;" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:41371a43-10bb-4def-b308-70fb9a66784d" class="wlWriterSmartContent"> <div class="le-pavsc-container"> <div style="background-color:#000000;white-space:nowrap;overflow:auto;padding:2px 5px;">&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">class</span><span style="color:#ffffff;"> </span><span style="color:#ffc66d;">MakeMethodVirtualDaemonStageProcess</span><span style="color:#ffffff;"> : </span><span style="color:#6897bb;">IDaemonStageProcess</span><br>&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">readonly</span><span style="color:#ffffff;"> </span><span style="color:#6897bb;">IDaemonProcess</span><span style="color:#ffffff;"> _process;</span>  <p><span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span> MakeMethodVirtualDaemonStageProcess(</span><span style="color:#6897bb;">IDaemonProcess</span><span style="color:#ffffff;"> process)</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">_process = process;</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">void</span><span style="color:#ffffff;"> Execute(</span><span style="color:#6897bb;">Action</span><span style="color:#ffffff;">&lt;</span><span style="color:#ffc66d;">DaemonStageResult</span><span style="color:#ffffff;">&gt; commiter)</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">if</span><span style="color:#ffffff;"> (_process.InterruptFlag)</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">return</span><span style="color:#ffffff;">;</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span></p> <p><span style="color:#ffffff;"></span><span style="color:#cc7832;">var</span><span style="color:#ffffff;"> file = _process.ProjectFile.GetPsiFile(</span><span style="color:#ffc66d;">CSharpLanguageService</span><span style="color:#ffffff;">.CSHARP) </span><span style="color:#cc7832;">as</span><span style="color:#ffffff;"> </span><span style="color:#6897bb;">ICSharpFile</span><span style="color:#ffffff;">;</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">if</span><span style="color:#ffffff;"> (file != </span><span style="color:#cc7832;">null</span><span style="color:#ffffff;">)</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">var</span><span style="color:#ffffff;"> highlights = </span><span style="color:#cc7832;">new</span><span style="color:#ffffff;"> </span><span style="color:#ffc66d;">List</span><span style="color:#ffffff;">&lt;</span><span style="color:#6897bb;">HighlightingInfo</span><span style="color:#ffffff;">&gt;();</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">var</span><span style="color:#ffffff;"> processor = </span><span style="color:#cc7832;">new</span><span style="color:#ffffff;"> </span><span style="color:#ffc66d;">RecursiveElementProcessor</span><span style="color:#ffffff;">&lt;</span><span style="color:#6897bb;">IMethodDeclaration</span><span style="color:#ffffff;">&gt;(declaration =&gt;</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">var</span><span style="color:#ffffff;"> accessRights = declaration.GetAccessRights();</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">if</span><span style="color:#ffffff;"> (accessRights == </span><span style="color:#6897bb;">AccessRights</span><span style="color:#ffffff;">.PUBLIC &amp;&amp; !declaration.IsStatic &amp;&amp; !declaration.IsVirtual &amp;&amp;</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">!declaration.IsOverride)</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">var</span><span style="color:#ffffff;"> docRange = declaration.GetNameDocumentRange();</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">highlights.Add(</span><span style="color:#cc7832;">new</span><span style="color:#ffffff;"> </span><span style="color:#6897bb;">HighlightingInfo</span><span style="color:#ffffff;">(docRange, </span><span style="color:#cc7832;">new</span><span style="color:#ffffff;"> </span><span style="color:#ffc66d;">MakeMethodVirtualSuggestion</span><span style="color:#ffffff;">(declaration)));</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">});</span><br><span style="color:#ffffff;"></span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">file.ProcessDescendants(processor);</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">commiter(</span><span style="color:#cc7832;">new</span><span style="color:#ffffff;"> </span><span style="color:#ffc66d;">DaemonStageResult</span><span style="color:#ffffff;">(highlights));</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><br>&nbsp; <span style="color:#ffffff;">}</span></p></div></div></div> <p>The main meat of this class is in the <strong>Execute </strong>method. We first check to make sure that we’ve not received an interruption (Interrupt Flag raised) due to some external action. Next step is to get access to the current file (remember that we are visiting the entire visible document, not just a specific method). Having the file, we can now create a <strong>RecusiveElementProcessor*</strong> to perform a tree walk of the AST and perform the specific action on each element. The action to perform is declared as the lambda expression. Since we’re interested in the method declaration, the type is <strong>IMethodDeclaration</strong> (there are many others). If we look at the expression, we can see that it’s pretty much the same as that of <a href="/blogengine/post/2010/01/12/Writing-plug-ins-for-ReSharper-Part-1-of-Undefined.aspx">Part 1</a>, the only difference is that we add the results to the highlighting variable.</p> <p>The <strong>HighlightingInfo </strong>class has a parameter which can be a Suggestion, Warning or Error, as explained previously. Since in our case we need a suggestion, we pass in the <strong>MakeMethodVirtualSuggestion</strong>:</p> <div style="display:inline;float:none;margin:0;padding:0;" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:781a1c96-7177-497a-94aa-8aca6403e14a" class="wlWriterSmartContent"> <div class="le-pavsc-container"> <div style="background-color:#000000;white-space:nowrap;overflow:auto;padding:2px 5px;"><span style="color:#ffffff;">[</span><span style="color:#ffc66d;">StaticSeverityHighlighting</span><span style="color:#ffffff;">(</span><span style="color:#6897bb;">Severity</span><span style="color:#ffffff;">.SUGGESTION)]</span><br>&nbsp;<span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">class</span><span style="color:#ffffff;"> </span><span style="color:#ffc66d;">MakeMethodVirtualSuggestion</span><span style="color:#ffffff;"> : </span><span style="color:#ffc66d;">CSharpHighlightingBase</span><span style="color:#ffffff;">, </span><span style="color:#6897bb;">IHighlighting</span><br>&nbsp;<span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#6897bb;">ICSharpTypeMemberDeclaration</span><span style="color:#ffffff;"> Declaration { </span><span style="color:#cc7832;">get</span><span style="color:#ffffff;">; </span><span style="color:#cc7832;">private</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">set</span><span style="color:#ffffff;">; }</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> MakeMethodVirtualSuggestion(</span><span style="color:#6897bb;">ICSharpTypeMemberDeclaration</span><span style="color:#ffffff;"> memberDeclaration)</span><br>&nbsp;&nbsp; �<br>� <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">Declaration = memberDeclaration;</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">string</span><span style="color:#ffffff;"> ToolTip</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">get</span><span style="color:#ffffff;"> { </span><span style="color:#cc7832;">return</span><span style="color:#ffffff;"> </span><span style="color:#a5c25c;">"Method could be marked as virtual"</span><span style="color:#ffffff;">; }</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">string</span><span style="color:#ffffff;"> ErrorStripeToolTip</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">get</span><span style="color:#ffffff;"> { </span><span style="color:#cc7832;">return</span><span style="color:#ffffff;"> ToolTip; }</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">override</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">bool</span><span style="color:#ffffff;"> IsValid()</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">return</span><span style="color:#ffffff;"> Declaration.IsValid();</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">int</span><span style="color:#ffffff;"> NavigationOffsetPatch</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">get</span><span style="color:#ffffff;"> { </span><span style="color:#cc7832;">return</span><span style="color:#ffffff;"> </span><span style="color:#6897bb;">0</span><span style="color:#ffffff;">; }</span><br>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br>&nbsp;<span style="color:#ffffff;">}</span></div></div></div> <p>This class is pretty simple. The main property to define is the <strong>ToolTip</strong>, which is the text that will show when we hover of the highlighting. The <strong>ErrorStripeToolTip</strong> is what’s displayed in the right-hand side gutter. Finally the Attribute <strong>StaticSeverityHighlighting </strong>is to indicate what type of tip it is (Warning, Error, etc.).</p> <p>&nbsp;</p> <p>[*Note: In this case, the operation we want to perform is very simple. If we want a more complex scenario where we need to do some processing before and after each element is visited or have a more fine-grained control, we can implement the <strong>IRecurisveElementProcessor</strong>. I’ll cover this in another post]. </p> <p>&nbsp;</p> <p>To recap, right now we would have everything place to display highlighting when a method that could be made virtual is encountered. The only remaining part is to now be able to apply a <strong>QuickFix</strong>. This is in many ways similar to the ContextAction we saw in <a href="/blogengine/post/2010/01/12/Writing-plug-ins-for-ReSharper-Part-1-of-Undefined.aspx">Part 1</a>:</p> <div style="display:inline;float:none;margin:0;padding:0;" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:9e844366-abb3-4116-887f-dfcbfb826cf0" class="wlWriterSmartContent"> <div class="le-pavsc-container"> <div style="background-color:#000000;white-space:nowrap;overflow:auto;padding:2px 5px;"><span style="color:#ffffff;">[</span><span style="color:#ffc66d;">QuickFix</span><span style="color:#ffffff;">]</span><br><span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">class</span><span style="color:#ffffff;"> </span><span style="color:#ffc66d;">MakeMethodVirtualQuickFix</span><span style="color:#ffffff;"> : </span><span style="color:#ffc66d;">BulbItemImpl</span><span style="color:#ffffff;">, </span><span style="color:#6897bb;">IQuickFix</span><br><span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">readonly</span><span style="color:#ffffff;"> </span><span style="color:#ffc66d;">MakeMethodVirtualSuggestion</span><span style="color:#ffffff;"> _highlighter;</span><br><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#808080;">// Takes as parameter the Highlighter the quickfix refers to</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> MakeMethodVirtualQuickFix(</span><span style="color:#ffc66d;">MakeMethodVirtualSuggestion</span><span style="color:#ffffff;"> highlighter)</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">_highlighter = highlighter;</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#808080;">// In the transaction we make the necessary changes to the code</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">protected</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">override</span><span style="color:#ffffff;"> </span><span style="color:#6897bb;">Action</span><span style="color:#ffffff;">&lt;</span><span style="color:#6897bb;">ITextControl</span><span style="color:#ffffff;">&gt; ExecuteTransaction(</span><span style="color:#6897bb;">ISolution</span><span style="color:#ffffff;"> solution, </span><span style="color:#6897bb;">IProgressIndicator</span><span style="color:#ffffff;"> progress)</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">_highlighter.Declaration.SetVirtual(</span><span style="color:#cc7832;">true</span><span style="color:#ffffff;">);</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">return</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">null</span><span style="color:#ffffff;">;</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#808080;">// Text that appears in the context menu</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">override</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">string</span><span style="color:#ffffff;"> Text</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">get</span><span style="color:#ffffff;"> { </span><span style="color:#cc7832;">return</span><span style="color:#ffffff;"> </span><span style="color:#a5c25c;">"Make Method Virtual"</span><span style="color:#ffffff;">; }</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#808080;">// Indicates when the option is available </span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">public</span><span style="color:#ffffff;"> </span><span style="color:#cc7832;">bool</span><span style="color:#ffffff;"> IsAvailable(</span><span style="color:#6897bb;">IUserDataHolder</span><span style="color:#ffffff;"> cache)</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">{</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;"></span><span style="color:#cc7832;">return</span><span style="color:#ffffff;"> _hi<br>ghlighter.IsValid();</span><br>&nbsp;&nbsp;&nbsp; <span style="color:#ffffff;">}</span><br><span style="color:#ffffff;">}</span></div></div></div> <p>The <strong>MakeMethodVirtualQuickFix </strong>needs to implement the <strong>IBulbItem </strong>and <strong>IQuickFix </strong>interfaces. For ease of implementation we can inherit from <strong>BulbItemImpl</strong>. The constructor should take as parameter always the actual highlighting that has given way to invoking the QuickFix, in our case the <strong>MakeMethodVirtualSuggestion</strong>. Similar to the ContextAction we implemented in <a href="/blogengine/post/2010/01/12/Writing-plug-ins-for-ReSharper-Part-1-of-Undefined.aspx">Part 1</a>, the actual fix itself is pretty trivial. All we need to do is make the method virtual. How do we get access to the method? The easiest way is via the Declaration property of the highlighting passed in (this is a property we added before). The only thing left is to call the <strong>SetVirtual </strong>method on it. Since we are in the <strong>ExecuteTransaction </strong>method, ReSharper makes sure that any change made is executed as a whole.</p> <p>The rest of the properties are trivial. Text returns the text of the QuickFix (what appears in the menu), and <strong>IsAvailable</strong> indicates when the <strong>QuickFix</strong> is available, which in our case is whenever the highlighting is valid.</p> <h3>&nbsp;</h3> <h3>The End Result</h3> <p>Once we compile the plug-in and place it in the corresponding Plugins folder under ReSharper\Bin, we’re done. Here’s the end result:</p> <p><a href="http://hhariri.files.wordpress.com/2010/11/414.png"><img style="border-bottom:0;border-left:0;display:inline;border-top:0;border-right:0;" title="4" border="0" alt="4" src="http://hhariri.files.wordpress.com/2010/11/4_thumb3.png" width="384" height="119"></a> </p> <p>and invoking Alt+Enter on the highlighting gives us:</p> <p>&nbsp;</p> <p><a href="http://hhariri.files.wordpress.com/2010/11/514.png"><img style="border-bottom:0;border-left:0;display:inline;border-top:0;border-right:0;" title="5" border="0" alt="5" src="http://hhariri.files.wordpress.com/2010/11/5_thumb3.png" width="313" height="145"></a> </p> <p>&nbsp;</p> <h3>Summary</h3> <p>Extending ReSharper to create highlightings and quick fixes is pretty simple once you understand how all the pieces fall into place. Most of the code will usually be the same and what will vary will be the actual element processing to be performed and the corresponding QuickFix. As mentioned previously (in the Note), for complex scenarios, we can have more control over the tree walk and that’s something we’ll examine in a future post.</p> <p>I’ve placed the code up on my <a href="http://github.com/hhariri">github account</a> so feel free to download it, play with it and ping me if you have any comments or questions. The code is updated to work with ReSharper 5.1</p> <p>[Thanks to <a href="http://howard.vanrooijen.co.uk/">Howard</a> for his valuable input on some issues I first encountered]</p>