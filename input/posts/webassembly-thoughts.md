Title: Reflections on Webassembly - May/Nov 19
Published: 03/11/2019 17:00 
Tags: [Webassembly, Thoughts] 
---

# Initial post

I rewatched this talk from 2014, [The Birth and Death of JavaScript](https://www.destroyallsoftware.com/talks/the-birth-and-death-of-javascript), and am still amazed at how prescient it was and still is. 

The blurb for this is:
“This science fiction / comedy / absurdist / completely serious talk traces the history of JavaScript, and programming in general, from 1995 until 2035.”

ASM JS is still there, but what Gary Bernhardt calls Metal in the talk is WebAssembly. And WebAssembly (Wasm) is coming, and not just for the web. 

In the .Net world, we see the first glimpses of the power of Wasm with the arrival of [Blazor](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor), which allows the execution of code running on the .net framework (using the mono runtime) inside a browser.

Wasm is not stopping there. With the [standardising of WASI](https://hacks.mozilla.org/2019/03/standardizing-wasi-a-webassembly-system-interface/), a system interface to run Wasm outside the web, we are truly moving towards architecture independence. 

It might take until 2035, like in the video, for Wasm to take over the world, but I have the feeling we will see its impact in the near future. Looking at the progress on Wasm in the two last years, I would recommend any programmer to read up on it and understanding its implications on the way they work. 

For further reading on Wasm, I would recommend the mozilla [WebAssembly Archives](https://hacks.mozilla.org/category/webassembly/) and in particular, the [WebAssembly post mvp future by Lin Clark](https://hacks.mozilla.org/2018/10/webassemblys-post-mvp-future/). 

# Update in november 2019 

Over the last few months, I have seen Wasm cropping up in more and more places, for example as a way to deploy code to running inside databases, such as the case of [Wasm for Postgres](https://medium.com/wasmer/announcing-the-first-postgres-extension-to-run-webassembly-561af2cfcb1) which allows stored procedures to run webassembly code.

Another case of that is the reflection on using [WebAssembly as a glue between programming languages](https://youtu.be/3yVc5t-g-VU) by allowing type information and guarantees to flow between programming languages. This would allow future systems to take the best of breed choice of programming language for each component of the system while still allowing compile time reasoning on type safety by using a type preserving compiler. 

<?# YouTube 3yVc5t-g-VU /?>

As I noted on twitter, as I read, watch and learn about webassembly, I'm getting the feeling this is going to be a titanic shift in the way we reason about programs, languages, frameworks and software design in general.

"The avalanche has already started, it is too late for the pebbles to vote" - Ambassador Kosh, Babylon 5. 