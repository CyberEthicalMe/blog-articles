## Generate Table of Contents with a single click

After writing down and publishing my [longest write-up](/thm-yotjf) I thought that this is too hard to read, and I desperately need the table of contents for it. I did a quick research and found out that indeed it is possible, I scripted that out - and now I'm sharing that with you 🙌

> The basic idea comes from the @[Sai Laasya Vabilisetty](@Laasya_Setty) article from [Markdown Cheatsheet to Write a Stunning Article!](https://laasyasettyblog.hashnode.dev/markdown-cheatsheet-to-write-a-stunning-article)

# What does it look like?

For article code:

```
# What does it look like?

//content

# Regular, laborious way

//content

# Preferred version

//content

# Wrapping up

//content

# Bonus materials

//content


```

The final result looks like this:

***
# Contents

1. [What does it look like?](#what-does-it-look-like)
2. [Regular, laborious way](#regular-laborious-way)
3. [Preferred version](#preferred-version)
4. [Wrapping up](#wrapping-up)
5. [Bonus materials](#bonus-materials)
***

# Regular, laborious way

Just type it in manually.

```
***
# Contents

1. [What does it look like?](#what-does-it-look-like)
2. [Regular, laborious way](#regular-laborious-way)
3. [Preferred version](#preferred-version)
4. [Wrapping up](#wrapping-up)
5. [Bonus materials](#bonus-materials)
***
```

# Preferred version

Go into `Preview` mode (for already published articles `Edit` -> `Preview`) and paste the following code into the [Developer Tools](https://en.wikipedia.org/wiki/Web_development_tools) Console.

![Screenshot_5.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624555472781/Urae8P4bg.png)


* Numbered list

Minified, one liner:

```js
let ct="***\n# Contents\n";$("h1").each((i,e) => {ct+=`${i+1}. [${e.innerHTML}](#${e.id})\n`});console.log(ct+="***");
```

Pretty print
```js
let ct = "***\n# Contents\n";
$("h1").each((i, e) => {
    ct += `${i+1}. [${e.innerHTML}](#${e.id})\n`
});
console.log(ct += "***");
```


* Bullet list

Minified, one liner:

```js
let ct="***\n# Contents\n";$("h1").each((i,e) => {ct+=`* [${e.innerHTML}](#${e.id})\n`});console.log(ct+="***");
```

Pretty print
```js
let ct = "***\n# Contents\n";
$("h1").each((i, e) => {
    ct += `* [${e.innerHTML}](#${e.id})\n`
});
console.log(ct += "***");
```

The above JavaScript uses the fact that the only `h1` tags available on the `Preview` are the `#` headings existing in the article.

# Wrapping up

> 🔔 `CyberEthical.Me` is maintained purely from your donations - if you would like to boost the community, consider one-time sponsoring at the 🍻 [Buymeacoffee](https://www.buymeacoffee.com/asentinn) or use the [Sponsor](/sponsor) button.

Now copy the output of the script from the console and paste it, preferably, somewhere at the top of the article.

> Just don't use it as an article opener - when sharing your post on social, sometimes first lines of text land in the preview - this won't look good with all headers scrabled together.

# Bonus materials

## Go to top

> Kudos to @[Jordan Craig](@jc1812) for [this comment](https://amarachiazubuike.com/how-to-create-a-table-of-content-on-hashnode-ckem4cyih01sw99s12bgd4c37#ckeuxifk102mmv7s11yp163jq) in the article from @[Amarachi Emmanuela Azubuike](@amarachukwu)

Now, when you have the table of contents, it is good to have "Go to top" action links spread inside the article, so the reader can easily jump back and forth between headings and listing.

```
[Back to top](#contents) ⤴
```

It looks and acts (click it) like this:

[Back to top](#contents) ⤴

## Go fancy <a id="fancy"></a>

As you can see, the presented method requires you to update table of contents each time heading text changes - because the anchor link will be broken and will no longer link to the section.

There is a more tedious but universal workaround by using own anchors.
```
# Go fancy <a id="fancy"></a>
```

That way, you can jump to **any** location where the anchor is placed. Code for this will look like this:

```md
1. [Go fancy!](#fancy)
```

>Try it out:
1. [Go fancy!](#fancy)

Now, then - how to automate this?

Assuming you have already anchored your article - use the following snippets:

* Numbered list
```js
let ct="***\n# Contents\n"; $("a[id]").each((i,e) => {ct+=`${i+1}. [](#${e.id})\n`});console.log(ct+="***");
```

* Bullet list
```js
let ct="***\n# Contents\n"; $("a[id]").each((i,e) => {ct+=`* [](#${e.id})\n`});console.log(ct+="***");
```

Because this is universal method - first time you have to fill in the titles of anchors - but after that, you can modify text of headers or other elements to your liking.

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

## Bookmark Action

And now my favorite one.

> This may not work in all browsers - [compatibility table](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/writeText#browser_compatibility).

In your browser, add new page to bookmarks

![Screenshot_6.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624562418295/upEVR843r.png)

In the title, add text you want to be displayed on Bookmarks bar, and in the URL paste the following one liner:

```js
javascript:(function() {$("[id='contents-clip']").remove();let ct="***\n# Contents\n";$("h1").each((i,e) => {ct+=`* [${e.innerHTML}](#${e.id})\n`});navigator.clipboard.writeText(ct+="***").then(function() {}, function(err) {console.error(err);});}());
```

![Screenshot_7.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624562938716/uq7L79wvJ.png)

Pretty print:

```
javascript: (function() {
    $("[id='contents-clip']").remove();
    let ct = "***\n# Contents\n";
    $("h1").each((i, e) => {
        ct += `* [${e.innerHTML}](#${e.id})\n`
    });
    navigator.clipboard.writeText(ct += "***").then(function() {}, function(err) {
        console.error(err);
    });
}());
```
> Script will use the `navigator.clipboard` API to insert Table of Contents code into clipboard. Read about it [here](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/writeText).

Save the bookmark. Now you can go into `Preview` mode, click that bookmark, and you have the whole Table of Contents in your clipboard. All is left is go back to `Write` and paste!

### Did you find it useful? Please let me know in the comments 👇!






