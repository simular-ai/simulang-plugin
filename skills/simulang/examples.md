# Simulang Examples

Reference examples for common Simulang workflows. See [full documentation](https://docs.simular.ai/simular-pro/examples).

---

## Background Browser: Parallel Web Research

### Fetch and Summarize Multiple News Sources

```javascript
async function main() {
    try {
        const urls = [
            "https://news.google.com/",
            "https://www.bbc.com/news",
            "https://www.reuters.com/",
            "https://www.cnn.com/"
        ];

        // Open all pages in parallel
        const pages = await Promise.all(urls.map(async (u) => {
            const p = await browser.newtab(u);
            await p.wait({ waitTime: 2 });
            return p;
        }));

        // Fetch content from all pages
        const contents = await Promise.all(pages.map(async (p, i) => {
            try {
                await p.wait({ waitTime: 1 });
                const text = await p.content();
                return { url: urls[i], text };
            } catch (e) {
                console.log("content error:", urls[i], e);
                return { url: urls[i], text: "" };
            }
        }));

        // Combine and summarize
        const combinedText = contents
            .map((c, idx) => `### Source ${idx + 1}: ${c.url}\n${c.text || ""}`)
            .join("\n\n");

        const summary = await pages[0].ask({
            prompt: "Summarize the key headlines across all sources. Focus on consensus and important divergences.",
            context: { text: combinedText }
        });

        console.log(summary);
    } catch (err) {
        console.log("Error:", err);
    }
}
```

### Simple LinkedIn Feed Summary

```javascript
async function main() {
    var page = await browser.newtab('https://www.linkedin.com/feed/')
    await page.wait({ waitTime: 3 })
    var content = await page.content()
    var response = await page.ask({
        prompt: 'Summarize the top posts and discussions',
        context: { text: content }
    })
    console.log(response)
}
```

### Search and Extract with Google

```javascript
async function main() {
    var page = await browser.newtab('https://www.google.com/search?q=latest+AI+news')
    await page.wait({ waitTime: 2 })
    var content = await page.content()

    var summary = await page.ask({
        prompt: 'Extract the top 5 search results with titles and brief descriptions. Return as a numbered list.',
        context: { text: content }
    })

    console.log(summary)
}
```

### Interactive Form Automation

```javascript
async function main() {
    var page = await browser.newtab('https://example.com/signup')
    await page.wait({ waitTime: 2 })

    await page.click({ concept: 'email input field' })
    await page.type({ text: 'user@example.com' })

    await page.click({ concept: 'password input field' })
    await page.type({ text: 'securePassword123' })

    await page.click({ concept: 'submit button' })
    await page.wait({ waitTime: 3 })

    var content = await page.content()
    var result = await page.ask({
        prompt: 'Did the signup succeed? What message is shown?',
        context: { text: content }
    })

    console.log(result)
}
```

### Multi-Site Price Comparison

```javascript
async function main() {
    const product = "MacBook Pro 14"
    const sites = [
        `https://www.amazon.com/s?k=${encodeURIComponent(product)}`,
        `https://www.bestbuy.com/site/searchpage.jsp?st=${encodeURIComponent(product)}`,
        `https://www.walmart.com/search?q=${encodeURIComponent(product)}`
    ]

    const pages = await Promise.all(sites.map(url => browser.newtab(url)))
    await Promise.all(pages.map(p => p.wait({ waitTime: 3 })))

    const results = await Promise.all(pages.map(async (p, i) => {
        const content = await p.content()
        const prices = await p.ask({
            prompt: `Extract the top 3 product listings with names and prices for "${product}". Return as JSON array.`,
            context: { text: content }
        })
        return { site: sites[i], data: prices }
    }))

    // Final comparison
    const comparison = await pages[0].ask({
        prompt: 'Compare prices across all sites and recommend the best deal.',
        context: { text: JSON.stringify(results) }
    })

    console.log(comparison)
}
```

---

## Reading Structured Information

### Extract Discord Server Names

```javascript
function main() {
    open({app: "Discord"});

    var content = pageContent();
    var serverList = ask({
        prompt: "List the top 10 Discord server names visible in the left sidebar. Do not include the direct messages. Return as JSON array.",
        context: content
    });
    
    return JSON.parse(serverList);
}
```

### Get News Headlines from TechCrunch

```javascript
function main() {
    open({url: "https://techcrunch.com"});
    wait({waitTime: 3});
    
    const content = pageContent();
    const headlinesResponse = ask({
        prompt: "Extract all news headlines from this TechCrunch page. Return only a JSON array of headline strings.",
        context: content
    });
    
    const headlines = JSON.parse(headlinesResponse);
    
    for (let i = 0; i < headlines.length; i++) {
        console.log(`${i + 1}. ${headlines[i]}`);
    }
    
    return {success: true, headlinesCount: headlines.length};
}
```

### Extract Email Information from Gmail

```javascript
function main() {
    open({url: "https://gmail.com"});
    wait({waitTime: 3});
    
    // Click first unread email
    let content = pageContent();
    let firstUnreadEmail = ask({
        prompt: "Find the first unread email in the inbox and return its subject line or identifying text that can be clicked. Return only the clickable text.",
        context: content
    });
    
    click({concept: firstUnreadEmail});
    wait({waitTime: 2});
    
    // Extract email data
    content = pageContent();
    let emailData = ask({
        prompt: "From this email content, extract: 1) sender email address, 2) date of the email, 3) a brief summary. Return as JSON with keys 'senderEmail', 'date', 'summary'.",
        context: content
    });
    
    return JSON.parse(emailData);
}
```

### Get Real Estate Listings from Redfin

```javascript
function main() {
    open({app: "Google Chrome"});
    wait({waitTime: 2});
    
    press({key: "l", cmd: true});
    type({text: "redfin.com/city/14325/CA/Palo-Alto", withReturn: true});
    wait({waitTime: 3});
    
    let content = pageContent();
    let result = ask({
        prompt: "Extract all house listings from this Redfin page. For each house, get: address, bedrooms, bathrooms, squareFeet, price. Return as JSON array.",
        context: content
    });
    
    return JSON.parse(result);
}
```

---

## Iterating Over Multiple Items

### Process All Blog Posts

```javascript
function getBlogPostUrls() {
    open({url: "simular.ai/blog"});
    wait({waitTime: 3});
    
    var content = pageContent();
    var result = ask({
        prompt: "Find all blog post URLs on this page. Return as a JSON array of complete URLs only.",
        context: content
    });
    
    return JSON.parse(result);
}

function extractPostInfo(url) {
    open({url: url});
    wait({waitTime: 3});
    
    var content = pageContent();
    var result = ask({
        prompt: "Extract the blog post title and create a short summary (2-3 sentences). Format as JSON with 'title' and 'summary'.",
        context: content
    });
    
    return JSON.parse(result);
}

function main() {
    var blogUrls = getBlogPostUrls();
    var allPosts = [];
    
    for (var i = 0; i < blogUrls.length; i++) {
        var url = blogUrls[i];
        var postInfo = extractPostInfo(url);
        postInfo.url = url;
        allPosts.push(postInfo);
        
        // Close tab and continue
        press({key: "w", cmd: true});
        wait({waitTime: 1});
        
        console.log(`Title: ${postInfo.title}`);
        console.log(`Summary: ${postInfo.summary}`);
    }
    
    return {success: true, posts: allPosts};
}
```

---

## Writing Structured Output

### Write to Google Sheets

```javascript
function main() {
    open({url: "https://sheets.google.com/create"});
    wait({waitTime: 2});
    
    // Add headers
    setGoogleSheetCellValue({cell: "A1", value: "Name"});
    setGoogleSheetCellValue({cell: "B1", value: "Summary"});
    setGoogleSheetCellValue({cell: "C1", value: "Date"});
    
    // Add data row
    setGoogleSheetCellValue({cell: "A2", value: "Simular"});
    setGoogleSheetCellValue({cell: "B2", value: "Simular pro can do anything!"});
    setGoogleSheetCellValue({cell: "C2", value: "08/30/2025"});
    
    return {success: true};
}
```

### Write to Excel

```javascript
function main() {
    open({app: "Microsoft Excel"});
    press({key: "n", cmd: true});
    wait({waitTime: 2});
    
    // Add headers
    click({concept: "cell A1"});
    type({text: "Name"});
    press({key: "tab"});
    type({text: "Summary"});
    press({key: "tab"});
    type({text: "Date"});
    
    // Add data row
    click({concept: "cell A2"});
    type({text: "Simular"});
    press({key: "tab"});
    type({text: "Simular pro can do anything!"});
    press({key: "tab"});
    type({text: "08/30/2025"});
    
    return {success: true};
}
```

---

## Advanced Click Patterns

### Click with Spatial Relations

When multiple elements share the same description, use spatial relations:

```javascript
// Click element relative to an anchor
click({
    at: "link Introduction", 
    spatialRelation: "above,closest", 
    anchorConcept: "heading Simular Pro"
});

// Click button contained in a dialog
click({
    at: "button close", 
    spatialRelation: "containedIn", 
    anchorConcept: "group dialog Find and replace"
});
```

**Spatial relation options:**
- Distance: `closest`, `furthest`
- Position: `left`, `right`, `above`, `below`
- Containment: `contains`, `containedIn`

### Click Using Vision Mode

For elements without text descriptions (images, icons):

```javascript
// Click an image/logo
click({at: "simular logo", mode: "vision"});

// Click empty space
click({at: "the space below the date", mode: "vision"});
```

### Click Using Text and Screenshot

Combines text description with visual context:

```javascript
click({
    at: "link Introduction in the Simular Browser section", 
    mode: "textAndScreenshot"
});
```

---

## Common Patterns

### Wait for Page Load

```javascript
open({url: "https://example.com"});
wait({waitTime: 3});  // Wait 3 seconds for page to load
```

### Get Page Content and Query

```javascript
var content = pageContent();
var result = ask({
    prompt: "Your question here. Return as JSON.",
    context: content
});
var parsed = JSON.parse(result);
```

### Keyboard Shortcuts

```javascript
press({key: "l", cmd: true});  // Cmd+L (focus URL bar)
press({key: "w", cmd: true});  // Cmd+W (close tab)
press({key: "n", cmd: true});  // Cmd+N (new window/document)
press({key: "tab"});           // Tab key
```
