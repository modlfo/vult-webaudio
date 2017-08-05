# Minimal example using Vult with WebAudio

This example shows how to embed the Vult compiler in a Web page and generate a [ScriptProcessorNode](https://developer.mozilla.org/en-US/docs/Web/API/ScriptProcessorNode) to execute your Vult code with [WebAudio](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) in a browser.

The full code is in the `index.html`

You can check the results in this link http://modlfo.github.io/vult

## Embedding Vult

The Vult compiler is available as JavaScript code in three different flavors:

- [Command line application (requires Node.js)](https://www.npmjs.com/package/vult)
- [Node.js library](https://www.npmjs.com/package/vultlib)
- [Web Browser library](https://github.com/modlfo/vult/releases)

In this case we are gonna use the Web Browser library. To embed the latest release in your page you can use the following address:

```
<script src="https://modlfo.github.io/vult/javascripts/vultweb.js"></script>
```

You can download or link an specific version from the compiler from the [releases](https://github.com/modlfo/vult/releases) page. For example:

```
<script src="https://github.com/modlfo/vult/releases/download/v0.3.27/vultweb.js"></script>
```

When the script is loaded it provides the global `vult` which contains all the methods to call the compiler. The API is the same as in the [Node.js library](https://www.npmjs.com/package/vultlib), which provides all the functionality of the of the [command line application](https://github.com/modlfo/vult/wiki/Command-Line-Options).

## Generating ScriptProcessorNode

To generate code that is ready for WebAudio we can use the `webaudio` template available when generating JavaScrip. This is done by calling the function `vult.generateJs` and providing the appropriate options:

```
   var result = vult.generateJs([
      {
         file: 'test.vult',   // Name of the file
         code: #code#         // the code itself as a string
      }
      ],
      {
         output: "Test",      // output name (used to prefix the code)
         template: 'webaudio' // here we specify the template
      });
```

The result is an array with either errors or generated files. The errors are objects like this:
```
{
   msg  : String, // Error message
   file : String, // File name
   line : Number, // Line where it occurs
   col  : Number  // Column where it occurs
}
```

If the code generation is successful we will get objects like this (same kind as the input files):
```
{
   file : "foo.cpp",      // File name or extension
   code : ".. my code .." // Generated code
}
```
When generating Web Audio code we only get one file.

## Executing the code

The generated code is a string so we have to use `eval` in Javscript to execute it:

```
var code = result[0].code; // When generating Js there's only one file
var vultProcessor = eval(code);
```

The `vultProcessor` is a function that receives the `audioContext` and returns the `ScriptProcessorNode`.

```
var audioContext = new AudioContext();
var vultProcessorNode = vultProcessor(audioContext);
```

This node can be connected to your audio graph:

```
source = audioContext.createBufferSource();
source.connect(vultProcessorNode);
vultProcessorNode.connect(audioContext.destination);
```