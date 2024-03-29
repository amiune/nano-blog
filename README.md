# Nano Blog

Minimal javascript ajax blog for github pages using markdown for posts. See [live demo here](https://hernan.amiune.com/nano-blog/) 

The goals of nano blog are:
- Easy to deploy (no installation or builds)
- Work with Github pages (free to host)
- Create the posts using markdown
- Can edit the posts using github web or app or your fav markdown editor

It consists of one html page and the following js script:

```js

const BLOG_URL = "YOUR_BLOG_URL"; // example: "https://hernan.amiune.com/nano-blog/"
const REPO_ADDRESS = "YOUR_GITHUB_REPOSITORY_ADDRESS"; // example: "amiune/nano-blog";
const ORDER_NEW_POSTS_FIRST = false;

function slug_to_title(slug) {
    // The first part of the slug is used to save the date
    // this allows to order the posts
    slug = slug.substring(slug.indexOf("-")+1);
    let words = slug.split('-');
    for (let i = 0; i < words.length; i++) {
      let word = words[i];
      words[i] = word.charAt(0).toUpperCase() + word.slice(1);
    }
    return words.join(' ');
  }

function reload_page(){
    
    url = window.location.href;
    markdown_to_fetch = "";

    if(url.includes("#!")){
        // Show blog post
        slug = url.split("#!")[1];
        slug = slug.replace("?","");
        slug = slug.replace("#","");
        markdown_to_fetch = BLOG_URL + "posts/" + slug + ".md";

        fetch(markdown_to_fetch).then(async (response) => {
            if (response.ok) {
                text = await response.text();
                let md = window.markdownit();
                document.getElementById('content').innerHTML = md.render(text);
            }
            else{
                // Error 404
                // https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics
                const metaRobots = document.createElement('meta');
                metaRobots.name = 'robots';
                metaRobots.content = 'noindex';
                document.head.appendChild(metaRobots);
                let md = window.markdownit();
                document.getElementById('content').innerHTML = md.render("# 404: Page not found");
            }
        })

        // Save pageview in google analytics
        if (typeof gtag !== "undefined") {
            gtag('event', 'page_view', {
                page_title: slug_to_title(slug),
                page_location: window.location.href
            });
        }
    }
    else{
        // Show blog list of posts
        url = "https://api.github.com/repos/" + REPO_ADDRESS + "/git/trees/main?recursive=1";
        fetch(url)
            .then((response) => response.json())
            .then((json_response) => {
                files_list = json_response.tree;
                if(ORDER_NEW_POSTS_FIRST)files_list.reverse();
                let list = document.createElement("ul");
                // show posts from oldest to newest
                for (let i = 0; i < files_list.length; i++) {
                    const element = files_list[i];
                    if(element.path.includes("posts/")){
                        let li = document.createElement('li');
                        let file_name = element.path.split("posts/")[1];
                        let slug = file_name.replace(".md","");
                        let a = document.createElement('a');
                        a.innerText = slug_to_title(slug);
                        a.setAttribute("href", BLOG_URL + "#!" + slug);
                        li.appendChild(a);
                        list.appendChild(li);
                    }
                }
                let div = document.createElement('div');
                let h1 = document.createElement('h1');
                h1.innerText = document.title;
                div.appendChild(h1);
                div.appendChild(list);
                document.getElementById('content').innerHTML = div.innerHTML;
            });
    }
    
    // Highlight code if Prismjs is loaded
    if (typeof window.Prism !== "undefined") {
        setTimeout(window.Prism.highlightAll,1000);
    }
}

window.addEventListener('popstate', function (event) {
    reload_page();
});

reload_page();
```

And the following html page:

```html

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Nano Blog</title>
  </head>

<body>
    <main class="container">
        <section>
          <div id="content"></div>
        </section>
    </main>
    <script src="js/nano-blog.js"></script>
</body>
</html>

```

This example use Pico.css for the Theme
