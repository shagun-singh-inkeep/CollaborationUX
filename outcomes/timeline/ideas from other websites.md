**mintlify**

<img src="/outcomes/timeline/pasted-20260708-201616.png" />

different colors for different people/agents

**notion**

<img src="/outcomes/timeline/pasted-20260708-201832.png" />

like how you can see a full screen view and can click into the exact part

**google docs**

<img src="/outcomes/timeline/pasted-20260708-202151.png" />

**ink and switch**

<img src="/outcomes/timeline/pasted-20260709-144500.png" />

comments might conflict with the side bar diff



**claude design**



<img src="/outcomes/timeline/pasted-20260709-144557.png" />

<img src="/outcomes/timeline/pasted-20260709-152233.png" />

<img src="/outcomes/timeline/pasted-20260709-152420.png" />







Making it truly identical would mean rendering the diff *through the actual editor*, and that's a much larger build for two reasons. First, the editor isn't a simple "render this text" component — it's bound to the live document's Y.js state, the provider pool, and the shared document context, so showing a read-only historical version means standing up a separate editor instance decoupled from all of that machinery (which several of the codebase's safety rules specifically guard against). Second, highlighting what changed inside that render can't be done with a text diff — it requires computing the difference at the ProseMirror node level and painting it as editor "decorations," which is a known-hard problem in its own right and has to correctly handle every one of OK's custom node types (links, embeds, mirrors, task lists, etc.). Each of those is a real project on its own, with meaningful correctness risk, versus the static approach that





<img src="/outcomes/timeline/pasted-20260710-151352.png" />

like the faint lines in the side to see who changes what

gitblame [[outcomes/timeline/SPEC-git-blame]]
