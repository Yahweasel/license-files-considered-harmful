# LICENSE Files Considered Harmful

I do not put license text in the LICENSE files of repositories, and indeed would prefer not to have LICENSE files at all. The reason is not that I'm particularly blaise about licenses; in fact, the reason is exactly the opposite. It is my opinion that putting license information in a LICENSE file, while obviously convenient, is doing much more harm than good.

My license text is stored in the licensed files, in standard, common license headers. For libraries I often use [SPDX license headers](https://spdx.org/licenses/), though I'll admit I'm not completely consistent with that.

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

Let's say you're writing a piece of software that's going to use FFmpeg's libraries. FFmpeg is under the LGPL, which, bizarrely, is one of the few licenses that doesn't quite require attribution. But, since it requires providing the complete source code, I suppose the authors assumed that sufficed. You are required to inform users that the source code for this component is available, and tell them where they can get it.

But, not every file in FFmpeg is under the LGPL. For example, `libavcodec/jfdctfst.c` is under a 3-clause BSD license. That license includes the requirement that «the accompanying documentation must state that "this software is based in part on the work of the Independent JPEG Group"». Luckily for us, the `LICENSE.md` file in FFmpeg makes that clear: it states unambiguously that the whole is under the LGPL, but some parts, including `libavcodec/jfdctfst.c` are under other licenses. So, great, problem solved, right?

Well, the thing *I'm* using in FFmpeg is its newish `arnndn` filter. So, I dutifully look in the LICENSE file, and everything seems fine, so I add no special attribution.

And, I've just violated the license of the half-dozen authors of that file. It's distributed under the MIT license, which includes the requirement that «Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution». Maybe, *maybe* you could argue that the source code is “other materials provided with the distribution”, since you have to provide the source code under LGPL anyway. But, that's a stretch; unless you actually attach the source code along with the binaries, it's not *included*, it's just *offered*.

So, what went wrong? What went wrong is that you assumed the authors of FFmpeg would do the work you yourself were avoiding doing: meticulously cataloging the particular attribution requirements of every single file you depend upon. But, they failed. The `LICENSE.md` file of FFmpeg is flatly incorrect, and misleading, because it's not maintained.

Whose job is it to maintain this file? Who would *want* that job? Of course it's not maintained!


# LICENSE Files Considered Harmful (redux)

The very presence of a `LICENSE` file implies greater confidence than most open source authors are actually willing to provide. If we attempt to follow the license but naïvely assume that the `LICENSE` file is correct, nine times out of ten we will be misled.

I wish I could offer a better solution than “dig through everything and try to find all the licenses”. I really do. But creating a file that hides the complexity behind a lie does *not* achieve that goal.
