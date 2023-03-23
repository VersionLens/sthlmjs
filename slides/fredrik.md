So what can we do with this "function"?

![File layout of a Version Repository](/images/VersionRepoToVersions.png "Version Repository to Versions") <!-- .element: style="max-width: 65%" -->



Why?
* Collaboration <!-- .element: class="fragment" -->
* Debugging <!-- .element: class="fragment" -->
* A/B testing <!-- .element: class="fragment" -->
Note:
- allow non-developers to run and test any version of the app
- save hours of senior dev time spent switching locally to other dev's branches, for coordination
- debug any combination of source commits in isolation, with specific data loaded



How Versions fit in your workflow:
- build & run the latest Version of any feature/release/PR branch <!-- .element: class="fragment" -->
- create a Version on seeing a magic word in a commit message <!-- .element: class="fragment" -->
- create a Version on receiving messages in Slack or similar <!-- .element: class="fragment" -->



But there's another *crazy* thing we can do...



Hello ChatGPT!

- ChatGPT can generate and edit code! <!-- .element: class="fragment" -->
- It's not perfect! But it can be useful, and it's improving very quickly. <!-- .element: class="fragment" -->
- If it's a junior frontend developer today, it could be a mid-level, full-stack developer by the end of the year. <!-- .element: class="fragment" -->



Would you let ChatGPT _eval_ directly in your terminal?

Personally, I would not (especially not Bing) <!-- .element: class="fragment" -->

But what if we could eval it's responses in a sandboxed environment... like a Version? <!-- .element: class="fragment" -->



Putting it all together...
<iframe style="border: 1px solid rgba(0, 0, 0, 0.1);" width="800" height="450" src="https://www.figma.com/embed?embed_host=share&url=https%3A%2F%2Fwww.figma.com%2Fproto%2FcEMki65Nf1NNhr3YMqPGoZ%2FVersion-Lens-Video-Prototype%3Fnode-id%3D295-1508%26viewport%3D370%252C146%252C0.03%26scaling%3Dscale-down%26starting-point-node-id%3D295%253A1508" allowfullscreen></iframe>



## Thank you
- This presentation: [sthlmjs.versionlens.com](https://sthlmjs.versionlens.com)
- Full-stack JS Version Repository: [github.com/VersionLens/version-store-version-repository](https://github.com/VersionLens/version-store-version-repository)
- [versionlens.com](https://versionlens.com)
- pascal@versionlens.com, fredrik@versionlens.com
