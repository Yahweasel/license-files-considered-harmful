# LICENSE Files Considered Harmful

tl;dr: LICENSE files are unmaintainable, and when they're out of date or incorrect, they mislead people into violating licenses. Their presence does more harm than good.

I do not put license text in the LICENSE files of repositories, and indeed would prefer not to have LICENSE files at all. The reason is not that I'm particularly blaise about licenses; in fact, the reason is exactly the opposite. It is my opinion that putting license information in a LICENSE file, while obviously convenient, is doing much more harm than good.

My license text is stored in the licensed files, in plain old fashioned license comments. I mark the license correctly on repositories when I remember to. For libraries I often use [SPDX license headers](https://spdx.org/licenses/), though I'll admit I'm not completely consistent with that. If, after reading this, you agree with my concerns about LICENSE files, I recommend you adopt the same behavior, and include some advice at the end of this rant.

To describe why I think LICENSE files do more harm than good, I'll describe the problem and use an example of a project that tries to do it right, but fails.


## The problem

Almost all open source licenses require attribution in binaries that link against them. For all of our obsession with licenses, open source programmers are bizarrely terrible at following this requirement. To be abundantly clear: if you include any code that is under, for instance, the MIT license, you must include along with that code, in source or binary form, the license and its copyright list. And, most other open-source licenses have similar documentation requirements.

If you don't realize this or don't believe me, look in the “about” window of the browser you're using right now. There will be a “licensing info” or “open source software” link with an extremely exhaustive list of all the software they use, and their licenses.

Ideally, this would be something that could be handled with linking. To a very limited extent, in the world of TypeScript/JavaScript, it is, as comments can usually be made in such a way that minifiers won't remove them, and that suffices. But, this is rarely done correctly in TypeScript or JavaScript, and doesn't apply to, for example, C.

The unfortunate reality is that the only correct way to know what license text you need is to go over all of the source you use meticulously and collate all its license text.

That job *would* be made easier by LICENSE files.

It *would* be. If LICENSE files weren't useless.


## An object lesson: FFmpeg

Preface: I *love* FFmpeg. This is not a complaint about FFmpeg. And, FFmpeg is *trying* to do this right. But, FFmpeg's failure in this manner is a perfect demonstration of the problem.

Let's say you're writing a piece of software that's going to use FFmpeg's libraries. FFmpeg is under the LGPL, which, surprisingly, is one of the few licenses that doesn't quite require attribution. But, since it requires providing the complete source code, I suppose the authors assumed that sufficed. You are required to inform users that the source code for this component is available, and tell them where they can get it.

But, not every file in FFmpeg is under the LGPL. For example, `libavcodec/jfdctfst.c` is under a 3-clause BSD license. That license includes the requirement that «the accompanying documentation must state that "this software is based in part on the work of the Independent JPEG Group"». Luckily for us, the LICENSE.md file in FFmpeg makes that clear: it states unambiguously that the whole is under the LGPL, but some parts, including `libavcodec/jfdctfst.c` are under other licenses. So, great, problem solved, right?

Well, the thing *I'm* using in FFmpeg is its newish `arnndn` filter. So, I dutifully look in the LICENSE file, and everything seems fine, so I add no special attribution.

And, I've just violated the license of the half-dozen authors of that file. It's distributed under the MIT license, which includes the requirement that «Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution». Maybe, *maybe* you could argue that the source code is “other materials provided with the distribution”, since you have to provide the source code under LGPL anyway. But, that's a stretch; unless you actually attach the source code along with the binaries, it's not *provided with*, it's just *offered*.

So, what went wrong? What went wrong is that you assumed the authors of FFmpeg would do the work you yourself were avoiding doing: meticulously cataloging the particular attribution requirements of every single file you depend upon. But, they failed. The LICENSE.md file of FFmpeg is flatly incorrect, and misleading, because it's not maintained.

Whose job is it to maintain this file? Who would *want* that job? Of course it's not maintained!

This is a particularly egregious case because of the sheer complexity of FFmpeg, but rare is the LICENSE file that's well maintained. Even if it's maintained for the software itself (which usually only happens when that's trivial), it rarely suggests that dependencies may have their own licensing requirements. The best LICENSE file is one that simply explains that the actual text of the license is in licensed files, and you'll need to find it there.


## LICENSE Files Considered Harmful (redux)

The very presence of a LICENSE file implies greater clarity than most open source authors are actually willing to provide. If we attempt to follow the license but naïvely assume that the LICENSE file is correct, nine times out of ten we will be misled.

I wish I could offer a better solution than “dig through everything and try to find all the licenses”. I really do. But creating a file that hides the complexity behind a lie does *not* achieve that goal.


## How to make your licensing clear

Because some people are comforted by the presence of a LICENSE file, I *do* usually include one, but only to make clear that the text of the license isn't actually there. Instead, I create a LICENSE.md templated on the following:

```
<short name of license used by most of the source code>.

The license text of this code is in the licensed files, where it belongs. Putting license text in LICENSE files [does more harm than good](https://yahweasel.github.io/license-files-considered-harmful/).

This file is here because some people are comforted by its presence.
```

## What I do

How I manage licensing Hell depends on the programming language. Languages in which the distributed code is in a text form often make attribution (a bit) easier.

For C/C++, I do exactly what I suggested above: meticulously go through every source file looking for license information. `grep` is your friend for this.

For, e.g., TypeScript/JavaScript, I make sure to use `/*!` for all my own license comments, and make sure to use minifiers that keep common license patterns, which include `/*!` and `/* @license`.

The license-header situation is a real nightmare in TypeScript and JavaScript. It's common to see repositories that just declare that they're under “the MIT license” (or whatnot), but include the license text *nowhere*. That's quite a conundrum because part of the license is that you *must* include the license text. Of course, the license doesn't restrict the original author, but this means that *technically*, to use source like this at all, you ought to add the license text that the author was too lazy to yourself. Otherwise, you're violating the license. For all my pedantry, I am willing to take situations like this as a clear sign that the author in question does not intend to hold users to those terms of the relevant license, but it is generally a sign of poor license literacy.
