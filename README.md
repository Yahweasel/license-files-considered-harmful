**TL;DR:** LICENSE files are unmaintainable and often out of date, leading to inadvertent license violations. Their presence does more harm than good.

I choose not to include license text in the LICENSE files of repositories, and would even prefer not to have LICENSE files at all. This isn't because I'm unconcerned about licenses; quite the opposite. I believe that putting license information in a LICENSE file, while convenient, actually does more harm than good, for one critical reason: it splits the license text—which is a *legal contract*—from the content that it licenses.

Instead, I store license text in the source files themselves, as traditional license comments—not in LICENSE files. If you agree with my concerns after reading this, I recommend adopting a similar approach, and I give some advice on doing so at the end of this document.

(When I remember, I also mark the license correctly in the repository metadata. For libraries, I often use [SPDX license headers](https://spdx.org/licenses/), though I'm not entirely consistent with this.)

To explain why I think LICENSE files are more harmful than helpful, I'll walk through the problem and use an example of a project that attempts to “do it right,” but ultimately fails.


## The Problem

Almost all open-source licenses require attribution in binaries that link against them. Yet, for all the attention we give to licenses, open-source developers are notoriously bad at following this requirement. Let me be clear: if you include code that is, for example, licensed under MIT, you *must* include the license and copyright notice with that code—whether in source or binary form. Most other open-source licenses have similar documentation requirements.

If you don't believe me, check the “About” section of the browser you're using right now. You'll find a “Licensing Info” or “Open Source Software” link, listing all the software they use and their respective licenses. This isn't just a nice gesture; it’s a legal obligation.

Ideally, this would be handled by a smart linker. To a small extent it can be, for languages that are distributed in textual form, particularly in TypeScript/JavaScript, where comments can be preserved by minifiers. But this is rarely done correctly and doesn't apply to languages like C.

The unfortunate truth is that the only way to know what attribution text you need is to go through all the source code you use and compile its license information yourself.

LICENSE files *could* make this job easier.

They *could*—if LICENSE files weren't always wrong.


## An Object Lesson: FFmpeg

I love FFmpeg. This is not a criticism of FFmpeg, but its license file (called `COPYING` in the FFmpeg repository) illustrates the problem perfectly.

Let's say you're writing software that uses FFmpeg's libraries. FFmpeg is licensed under LGPL, which—surprisingly—is one of the few licenses that doesn't require attribution per se, as long as you provide access to the source code. You don't need to bundle the source with your distribution, and for practical reasons, most people don't.

However, not all files in FFmpeg are licensed under LGPL. For example, `libavcodec/jfdctfst.c` is under the JPEG license, which requires that “the accompanying documentation must state that this software is based in part on the work of the Independent JPEG Group.” Fortunately, the LICENSE.md file in FFmpeg makes this clear. It explains that most of FFmpeg is under LGPL, but some parts, like `libavcodec/jfdctfst.c`, fall under different licenses, and it even tells you how to comply with those requirements. Problem solved, right?

Not quite.

Let's say that the part of FFmpeg I'm using is the `arnndn` filter. When I check the LICENSE.md file, there's no mention of `af_arnndn.c`. Everything looks fine, so I don't add any special attribution.

And now, I’ve violated the license of the authors of that file. `af_arnndn.c` is licensed under the 2-cluse BSD license, which requires that redistributions in binary form reproduce the copyright notice and license text. You could argue that providing the source code under LGPL satisfies this requirement, but that's a stretch. Unless you're bundling the source code directly with the binaries, you're probably not in compliance.

So, what went wrong? You assumed that the FFmpeg maintainers had done the work you were avoiding: meticulously cataloging the licensing requirements of every single file. But they didn’t, and FFmpeg's license file is incorrect and misleading because it hasn't been maintained.

Why isn't it maintained? Because, who would want to maintain that file? No one. Programmers want to program, not fight attribution requirements.

This issue is particularly visible in complex projects like FFmpeg, but most LICENSE files are poorly maintained, even in simpler projects. The best LICENSE file is one that simply directs you to find the relevant licenses in the source files themselves.


## Why Is This Better?

From the perspective of the user of a software library, putting license text only in the source files is more work. Reading a LICENSE file is easier. But, it’s usually inaccurate, and all the ease of use in the world won’t account for being wrong.

The reason why including licenses only in the source files is better is that it avoids the problem of drift. LICENSE files often become out of date because programmers forget to update them when they modify code or add dependencies. Of course, simple modifications do not require any change, but transplanting code—a valid and even commendable act in the open-source community—does, as does adding dependencies. The LICENSE file is documentation, not code, and requires careful human scrutiny to keep it up to date. We cannot expect programmers to maintain it, and yet we do.

By keeping license information only in the same file as the licensed code, we greatly increase the chances that it remains accurate. Separating the two invites error.


## LICENSE Files Considered Harmful (Redux)

The mere presence of a LICENSE file implies clarity and certainty that often doesn’t exist. Trying to follow the license while assuming the LICENSE file is accurate inevitably leads to license violations when it so frequently is not.

I wish I had a better solution than “sift through everything and find all the licenses.” I don’t. But, creating a LICENSE file that obscures that complexity with a lie is not a solution.


## How to Make Your Licensing Clear

Since some people expect a LICENSE file, I usually include one, but only to explain that the actual license text is in the source files. Here’s a template for a LICENSE.md file:

```
<short name of license used by most of the source code>.

The license text for this code is in the source files.
[LICENSE files do more harm than good](https://yahweasel.github.io/license-files-considered-harmful/).

This file exists because some people expect it to be here.
```

At the top of each source file, I include a standard license header. For short licenses like MIT, this means the entire license. For longer licenses, it’s just a notification and a pointer to the full license text.


## Managing License Hell

Managing your license attribution obligations is complicated. An automated solution would be ideal, but I know of none that are satisfactory. [SPDX](https://spdx.org/licenses/) is a good start, as it at least provides a basis for automation to find license information, but I think it slightly misses the point; the SPDX guidelines encourage you to include only the license short name in the file and to include the license text in a LICENSE file, but that makes code transplantation just as dangerous with SPDX as it is without. The way I use SPDX is to include the SPDX metadata *and* the full license text.

Tools exist to manage compliance, such as [FOSSology](https://www.fossology.org/). Personally, I find them to be more intrusive than just `grep`ing source files, but I imagine that for sufficiently large, complex projects, these tools are extremely useful. In my opinion, the right way to write code that these tools will understand is to follow my advice: put the license text in the source files, not a LICENSE file, so that FOSSology will definitely find it.

For me, managing licenses depends on the language.

In languages like C/C++, where code is not distributed in text form, I can offer no better solution than to go through each file meticulously to check for license information—`grep` is your friend here. It’s tedious, but this isn’t something you’re doing out of virtue—it’s a legal requirement.

For languages like TypeScript/JavaScript, which are distributed in text form, it is sufficient to include the license comments themselves. To that end, I use `/*!` for license comments and ensure that my minifiers preserve common license patterns such as `/*!` and `/* @license`. A fair amount of software in this space is smart enough to mark license headers this way, though certainly not all.

Unfortunately, the TypeScript/JavaScript/Node community is particularly terrible with licensing. It’s not uncommon to see projects that declare themselves to be under some license requiring attribution, but not to include the actual license text at all. Technically speaking, this means that to use such software, it’s the *user’s* job to add the license. While this situation suggests the author likely doesn’t intend to enforce these license terms, it’s generally a sign of poor license literacy.

Perhaps the most important advice is just to become more license literate. Most open-source licenses are very short, and written in fairly readable English. If you understand what the most common licenses are and what they require, keeping track of your obligations isn’t too bad.
